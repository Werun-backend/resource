**（网上的教程都有点老，以下是踩了七个小时坑之后的呕心沥血之作）**

**前置需求**
本地安装了 Git, Maven, JDK-17( Jenkins 不支持 JDK-21 , 2022 年后弃用 JDK-8 之前的版本)，Github 上有可以正常运行，正常打包的 Java 程序，有一台正常运行且装有 JDK 的服务器

**STEP 1 下载安装 Jenkins**
[下载 Jenkins 的 war 包(官方地址)](https://www.jenkins.io/zh/download/)
在安装包根路径下，运行命令（linux环境、Windows环境都一样），窗口别关，挂着就行
`java -jar jenkins.war --httpPort=8080` 
在本地 8080 端口挂载 Jenkins （其他端口也行）
然后在浏览器访问  Jenkins ([localhost:8080](http://localhost:8080/))
没出问题大概会看到这个界面
![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613145434041-1588394623.png)
秘钥可以在上面提到的文件夹（记住这个文件夹，后面要考）找到，也可以在刚才挂着的窗口找到

填完之后大概会看到这个界面
这个是要你安装一些运行必要的插件，一会再点左边，先不着急
![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613145943627-1312827662.png)
有一点很坑！有一个插件 [cloudbees-folder](https://updates.jenkins-ci.org/download/plugins/cloudbees-folder/) 你得自己下载下来，然后加到上面那个文件夹的同级文件夹 plugins 里（大概是C:\Users\admin\.jenkins\plugins）
要不然，你会看到这个报错

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613151620425-1680233051.png)

现在可以安装插件了，但是有一些插件还是死活下载不下来，先不管他，一会儿按照[这篇博客](http://testingpai.com/article/1604671047314)设置一个镜像源就行
然后注册登录，就可以看到 Jenkins 的主界面了
之后去 Manage Jenkins -> Plugins 检查一下这两个插件安没安（ Publish over SSH 和 Deploy to container ），没装就装上，然后 Jenkins 就下载安装好了

**Step 2 Jenkins 配置**
下面是一些宏观配置，都在 Manage Jenkins 里
(1) Tools

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613155319589-1108357895.png)

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613155334869-1640815705.png)

(2) System

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613155634176-1408695313.png)

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613162954882-589953223.png)

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613155725284-1322980395.png)

(3) Security

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613155803000-1500954550.png)

好了，全局配置完了，接下来是新建一个 Jenkins 任务然后配置它
![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613161143387-1701842076.png)

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613161206173-1457969486.png)

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613161226023-337554310.png)

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613161351424-1726670307.png)

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613163021909-438226288.png)

<details>
<summary>Exec command</summary>

```
cd /java/app
sed -i 's/\r$//' stop.sh
sed -i 's/\r$//' start.sh
./stop.sh
./start.sh
```
</details>

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613163028451-648786299.png)

应用就可以了，这部分撒花

**Step 3 服务器配置**
<details>
<summary>start.sh</summary>

```
#!/bin/bash
export JAVA_HOME=/java/jdk/jdk1.8.0_221
echo ${JAVA_HOME}
echo 'Start the program : JenkinsTest-0.0.1-SNAPSHOT.jar' 
chmod 777 /home/ldp/app/JenkinsTest-0.0.1-SNAPSHOT.jar
echo '-------Starting-------' 
cd /java/jar
nohup ${JAVA_HOME}/bin/java -jar JenkinsTest-0.0.1-SNAPSHOT.jar &
echo 'start success'
```
</details>
<details>
<summary>stop.sh</summary>

```
#!/bin/bash
echo "Stop Procedure : JenkinsTest-0.0.1-SNAPSHOT.jar"
pid=`ps -ef |grep java|grep JenkinsTest-0.0.1-SNAPSHOT.jar|awk '{print $2}'`
echo 'old Procedure pid:'$pid
if [ -n "$pid" ]
then
kill -9 $pid
fi
```
</details>

将以上两个脚本的信息（ jdk 位置， jar 包位置）修改一下上传到服务器相应位置(在 Exec command 里cd 到的文件夹)就好了（别忘了给脚本 **执行** 权限）

**Step 4 Github配置**

![](https://img2024.cnblogs.com/blog/2996516/202406/2996516-20240613164425568-1747414512.png)

参考文献：
[(一）jenkins + GitHub 实现项目自动化部署](https://blog.csdn.net/w6990548/article/details/106242009)

[Jenkins自动化部署入门详细教程](https://www.cnblogs.com/wfd360/p/11314697.html)

