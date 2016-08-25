---
layout: post
title: "构建持续集成平台"
date: 2015-12-21 17:40:20 +0800
comments: true
categories: 敏捷开发
---
<!--more-->

* ###背景  
  **scrum**的敏捷开发当中有很多促进项目推进的办法和方案,
可以最大限度的利用开发人员有限的时间,但是只停留在管理
方式、交流沟通方式等****组织实践****上面。一方面要**快速迭代**、快速容错，
另一方面又要因为持续需要有产品构建，开发人员不得不每天
多次重复的构建代码、测试代码、部署代码。

  这一方面我们借鉴XP([什么是XP](http://baike.baidu.com/link?url=U89TWRLUuaWfOisUmkgvsVkunDl34lWEv3z3NNBUeY41iPnZOG7oWWGaCSdVcoE7PLJVTB7fGZkTL60aS3Q-3K))中
的**编程实践**手段，利用**持续集成技术**（[**什么是持续集成**](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)），**测试驱动开发**（TDD），**结对编程**
来支撑敏捷开发的项目推进。


* ###索引
- ####[java篇](#java)
- ####[ios篇](#ios)
- ####[android篇](#android)

* ###选择构建工具
我简单总结了一下几个开源持续集成工具（详细的比较可以移步[点我](http://cloud.51cto.com/art/201508/487605.htm)）：  

  * **Jenkins**  使用java语言编写 良好的插件环境，支持扩展
  * **Buildbot**   python语言开发的项目   已经为Mozilla、Webkit、Chromium所支持
  * **Travis**     适合新手   提供Saas  接入github账户
  * **Strider**    由node.js+javascript编写    需要安装mongodb和node.js   需要编写脚本  上手困难
  * **Drone**     开源的，支持各种语言的CI工具     但是你的项目必须是开源的 https://drone.io

  综上，公司项目中涉及到ios、android、java三种环境，并且代码并不开
源，所以在持续构建工具中使用**Jenkins**再合适不过。此外Jenkins
有很好地扩展能力，有很健全的插件支持各种环境的代码构建。
Jenkins本身是java实现，在tomcat中就能很好地运行，
这一点比较适合java出身的开发同仁。
- ###<span id="java">java项目的自动化构建</span>
**在linux环境下安装Jenkins**有两种方式：  

  1.`sudo java -jar jenkins.war –httpPort=18080 –ajp13Port=18009`  

  2.`yum install jenkins`  

  第一种方式下，如果你是SSH连接到linux主机，当你关闭连接的时候，这个命令也会被中断；  

  第二种方式,是在CentOS系统中使用yum命令安装（熟悉CentOS系统的人应该不陌生），安装完成之后,
Jenkins会成为系统中的一个service，只要在命令行执行`service jenkins start|stop|restart`
就可以完成服务的启动停止和重启。  

  **配置文件路径**为：`/etc/sysconfig/jenkins`，可能需要root权限。配置文件中主要需要修改的是启动
端口`JENKINS_PORT` (默认是8080)，使用jenkins的用户`JENKINS_USER`(默认生成一个jenkins)。
我这里的用户是cms,端口是8818,
  > JENKINS_USER="cms"  
  > JENKINS_AJP_PORT="8819"

  修改jenkins涉及到的目录和文件的权限所属：
>sudo chown -R cms /usr/lib/jenkins  
sudo chgrp -R cms /usr/lib/jenkins  
sudo chown -R cms  /var/log/jenkins  
sudo chgrp -R cms  /var/log/jenkins  
sudo chown -R cms  /var/lib/jenkins  
sudo chgrp -R cms  /var/lib/jenkins  
sudo chown -R cms  /var/cache/jenkins  
sudo chgrp -R cms  /var/cache/jenkins  

  之后就可以启动Jenkins了，执行命令`sudo service jenkins start`  

  ![show](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-show-jenkins.png)

  jenkins安装好之后，首先需要确认你本地是否有:

   **Maven**  负责编译java代码  
   **Git**    负责从代码管理服务器中拉取最新提交的代码  
   **JDK**   这个就不用说了，你没有jenkins你也安装不了  

  好了，这里就不赘述以上工具的环境变量的配置了，一定要有。

  打开Jenkins主界面，首先下载构建所需的插件，依次进入`系统管理`-`管理插件`，下载:  

   **GIT plugin**  
   **Maven Integration plugin**  
   **Email Extension Plugin**  （对jenkins自带邮件通知的扩展，可以自定义邮件模板）  

  ####配置Jenkins `系统管理`-`系统设置`  

  这里主要是对maven、Git、JDK的路径做一些配置  

  ![system-config](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-system-config.png)

  如果你下载了**Email Extension Plugin**插件，在系统配置中可以设置你想要的邮件通知属性  

  ![email-config](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-email-config.png)

  系统配置之后，回到主界面，选择`新建`，填写Item名称，选择构建一个maven项目  

  ![create-job](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-create-job.png)  

  下一步，源码管理中填入你拉取代码的git地址，可以是HTTP协议，也可以是SSH。当然，如果是**HTTP**，要有
对应的用户名和密码；如果是**SSH**，用户名和密码是git版本管理所在的服务器主机的用户名和密码，
并且需要在git版本管理器所在服务器上添加ssh登录的auto文件中添加公钥
private key是jenkins所在主机的用户私钥。  

  ![config-git](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-config-git.png)  

  构建触发器中勾选`Build whenever a SNAPSHOT dependency is built`和`Poll SCM`
这样构建的触发器，会每三分钟（H/3 * * * *）轮询一遍你的代码库，只要你往git的develop
分支中commit代码，Jenkins就会构建一次

  ![config-trigger](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-config-trigger.png)  

  编译使用的是maven，所以要确定到maven的pom文件和执行命令，如图  

  ![config-build](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-config-build.png)  

  添加构建后的动作，比如你希望执行什么shell、邮件通知到developer
或者管理员等  

  ![config-email](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-config-email.png)  

  这里打包好的文件在你的Jenkins主目录`/var/lib/jenkins`下地workspace里面，并且会保留
你每一次的构建代码包，有关构建结束的部署工作，你可以自己写**shell**脚本执行，也可以在pom文件中
写有关tomcat的插件属性，**利用maven**直接将代码部署到tomcat的部署目录下（其中也涉及到访问权限
的问题，这里就不展开描述了）。

  回到主目录，你的构建job会在job list中显示，其中`S`的颜色用来区分你构建失败还是成功，
蓝色是成功，红色是失败；`W`表示你最近几次构建的情况

  ![show-jobs](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-show-jobs.png)  

  除了自动触发构建任务，你也可以手动计划一次构建  

  ![build-list](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-build-list.png)  

  下图为每次测试结果的一个统计图：  

  ![show-test](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-show-test.png)  

  在变更记录中，能看到每次构建中，提交了哪些代码，commit的comments，谁提交的等等  

  ![show-summary](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-show-summary.png)  

  更具体的构建信息可以查看**Console Output** ，如下图：  

  ![show-console](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-show-console.png)  

  让我很舒服的是，当Jenkins自动构建完成之后，我会收到一个邮件提醒  

  ![show-email](http://7xpd26.com1.z0.glb.clouddn.com/jenkins-show-email.png)  

  最后，别让你的系统谁都可以访问，用户权限在`首页`-> `系统管理`->
`Configure Global Security`里配置。

- ###<span id="ios">ios项目的自动化构建</span>
**在Mac上安装Jenkins**比较方便，只要从官网上下载dmg安装包就可以。

  注意：如果你Mac上得JDK不是1.7+，安装会失败。  

  卸载Jenkins执行：` /Library/Application Support/Jenkins/Uninstall.command`  

  安装之后，Mac上会多出一个用户：Jenkins，如果你想更改成为你的用户来执行Jenkins，先停掉jenkins，
更改配置文件，再重启jenkins服务（launchctl有点像linux的Service），如下执行：

  `sudo launchctl unload /Library/LaunchDaemons/org.jenkins-ci.plist`  

  `sudo vim +1 +/daemon +'s/daemon/staff/' +/daemon +'s/daemon/bixiaopeng' +wq org.jenkins-ci.plist`  

  `sudo launchctl load /Library/LaunchDaemons/org.jenkins-ci.plist`  


  别忘了修改日志目录的用户权限和主目录的用户权限。

  ios的app构建需要有xcode环境，所以你的部署一定是台mac电脑，但我们做的也是自动构建，所以必然不能使用
xcode可视化工具进行构建，所以我尝试了**xcodebuild**,大概步骤是：build->archive->IPA  

  `xcodebuild -alltargets clean`  

  `xcodebuild -target HelloJenkins PROVISIONING_PROFILE="00000000-0000-0000-0000-000000000000" CONFIGURATION_BUILD_DIR=JenkinsBuild`  

  `xcodebuild -scheme HelloJenkins archive PROVISIONING_PROFILE="00000000-0000-0000-0000-000000000000" CODE_SIGN_IDENTITY="iPhone Developer: Justin Hyland (XXXXXXXXXX)" -archivePath ./JenkinsArchive/HelloJenkins.xcarchive`  

  `xcodebuild -exportArchive -exportFormat IPA -exportProvisioningProfile iOS\ Team\ Provisioning\ Profile:\ com.yourAPP.HelloJenkins -archivePath ./JenkinsArchive/HelloJenkins.xcarchive -exportPath ./JenkinsIPAExport/HelloJenkins.ipa`  

  详见：http://www.cnblogs.com/rosepotato/p/3884851.html

  但是构建并没有想象当中的顺利，如果你使用了第三方的jar包，总是会报错：  

  > no provisioning profile matches ‘xxx’

  通过调研，发现xcodebuild可以通过workspace文件构建，可以避开这些问题，
前提是你有scheme文件，跟ios的同时沟通完之后，他每次提交代码都会share scheme
文件上来。构建偶尔成功，但是还是频频报错。  

  `xcodebuild -workspace MyProject.xcworkspace -scheme MyScheme SYMROOT=$(PWD)/build`  

  最后的构建成功是借鉴了刘先宁在InfoQ上的一篇文章（[构建iOS持续集成平台（一）——自动化构建和依赖管理](http://www.infoq.com/cn/articles/build-ios-continuous-integration-platform-part1)）

  实际上是使用到了FaceBook给出的替代xcodebuild的解决方案**xctool**
顿时感觉，我之前就好像一直在用javac编译java代码，而不知道还有maven这个东西。
通过xctool和cocoapod，代码构建成功。  

  `xctool -workspace SDJG.xcworkspace -scheme SDJG clean`  

  `xctool -workspace SDJG.xcworkspace -scheme SDJG build SYMROOT=$(PWD)/JenkinsBuild`  

  `xcrun -sdk iphoneos PackageApplication -v bbbuild/Debug-iphoneos/SDJG.app -o /Users/fangrichird/git/shangde1216/JenkinsIPAExport/SDJG.ipa`  

  ios项目也就只停留在IPA这里了，因为TestFlight自从被苹果收购之后，
再不能通过jenkins的插件完成上传。解决办法停留在手动将IPA发向各种云
服务平台，或者直接用iTunes安装到测试机当中。

  至于jenkins用到的插件和其他环境部署，请参考 [java篇](#java)

- ###<span id="android">Android项目的自动化构建</span>

  **Android的集成部署**相对来说比较简单，跟我们的android工程师沟通，他们习惯使用
Android studio这种集成的ide，如果部署，有集成在IDE中的Gradle。  

  所以想要在linux上构建Android代码，只需要两件事情：

  1.安装Android SDK；  
  2.配置Gradle环境；(JDK环境就不用说了吧)

  other和[java篇](#java)的配置相同，只是再需要安装一个Gradle插件，如果
你需要向不同的应用市场打包，在Gradle的配置文件中配置就好了。
