# 部署远程节点样例

基于github workflow，结合官方的action和华为云的 obs,ssh-remote和scp-remote等action，完成如下工作
1、完成java代码的打包
2、将打包好的jar/war包上传到obs归档
3、停止远程服务器上的服务，并完成备份
4、部署jar/war包和配置文件到远程服务，并启动服务
5、检查服务是否正常启动

## **前置工作**
(1).获取远端linux服务器的IP,账号,密码
(2).需要开通华为云的OBS服务，并建好桶，OBS主页:https://www.huaweicloud.com/product/obs.html,OBS文档:https://support.huaweicloud.com/obs/
(3).需要在项目的setting--Secret--Actions下添加 USERNAME,PASSWORD,和华为云OBS服务的ACCESSKEY,SECRETACCESSKEY两个参数,获取ak/sk方式:https://support.huaweicloud.com/api-obs/obs_04_0116.html

## **使用样例**
完成springcloud项目部署:
(1).项目打包
```yaml
    # 完成java项目打包
    - name: build maven project
      run: mvn clean -U package -Dmaven.test.skip
```

(2).obs-action的使用，通过华为云账号的AK,SK，将打包好的target/demoapp.jar包上传到华为云OBS桶bucket-test下的workflow/springboot-web/v1.0.0.1/目录
```yaml
    - name: Upload To Huawei OBS
      uses: huaweicloud/obs-helper@v1.0.0
      id: upload_file_to_obs
      with:
        access_key: ${{ secrets.ACCESSKEY }}
        secret_key: ${{ secrets.SECRETACCESSKEY }}
        region: region
        bucket_name: bucket-test
        operation_type: upload
        local_file_path: target/demoapp.jar
        obs_file_path: workflow/springboot-web/v1.0.0.1/
```
(3).ssh-remote-action的使用
使用该action批量发起远程命令，完成各类操作,将需要执行的命令写到commands参数后，一个命令一行
```yaml
    - name: install jdk,stop service
      uses: huaweicloud/ssh-remote-action@v1.2
      with:
        ipaddr: 192.168.158.132
        username: ${{ secrets.USERNAME }}
        password: ${{ secrets.PASSWORD }}
        commands: |
          yum install -y java-1.8.0-openjdk  java-1.8.0-openjdk-devel
          java -version
```

(4).scp-remote-action的使用
使用该action将本地的文件或者目录批量上传到远端服务器，或者将远端服务器上的文件或目录下载到本地
```yaml
    - name: deploy service
      uses: huaweicloud/scp-remote-action@v1.1
      with:
        ipaddr: 192.168.158.132
        username: ${{ secrets.USERNAME }}
        password: ${{ secrets.PASSWORD }}
        operation_type: upload
        operation_list: |
          file target/demoapp.jar /usr/local/
          file bin/demoapp.service /etc/systemd/system/
          file bin/start-demoapp.sh /usr/local/
          file bin/stop-demoapp.sh /usr/local/
```
完整样例请阅读 .github/workflows/deploy-jar-to-ecs-by-action.yml
另外提供全原生方案，不通过action，通过纯脚本部署，请参考.github/workflows/deploy-jar-to-ecs-by-command.yml