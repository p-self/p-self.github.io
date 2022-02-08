---
layout: post
title:  "ionic使用与开发流程记录"
date:   2017-08-22 14:40:19 +0800
categories: dev ionic
---

## 官网平台的编译流程

`ionic`的命令行工具只支持低版本的node版本（不支持最新的v8.3.0），所以在安装之前需要将node的版本切换为低版本：

```bash
 ~> set PATH /home/abel/soft/node-v6.9.5-linux-x64/bin $PATH                                                                                                                     
 ~> yarn global upgrade ionic --ignore-engines     
```

`ionic`的官方网站是:http://ionicframework.com/,在使用之前需要在平台注册用户，并创建一个测试工程.

![](/images/DeepinScreenshot_select-area_20170822143420.png)

ionic提供了源代码的托管（支持git仓库），通过一个简单的命令可以让本地的代码和远端的应用联系起来，以方便后续的应用的打包发布能事项；源代码托管是一个非常有用的过程，可以帮助开发者完成很多事情！

![](/images/DeepinScreenshot_select-area_20170822143601.png)

将本地的源代码和远程的app关联起来的操作需要校验账户，所以在命令执行的过程中需要输入邮箱账户和密码！

```bash
 /tmp> ionic start --pro-id 904c3e55
 
 Log into your Ionic account
 If you don't have one yet, create yours by running: ionic signup
 
 ? Email: xxx@xx.com
 
 ? Password: [hidden]
 [INFO] Using test for name.
```

本地源代码需要先纳入git的管理：

```bash
/tmp/test> git add .
/tmp/test> git commit -a -m "frist commit"
/tmp/test> git push ionic master
```

ionic的后台在接收到代码的push操作之后会自动触发git的hook，主动开始app的编译流程，git提交的日志中也可以看到对应操作的流程：

```bash
remote: Running: ionic build --prod            
remote: [ANNOUNCE] Hi! Welcome to CLI 3.9.     
remote:                                        
remote:            We decided to merge core plugins back into the main ionic CLI package. The @ionic/cli-plugin-ionic-angular, @ionic/cli-plugin-ionic1, @ionic/cli-plugin-cordova, and @ionic
/cli-plugin-gulp plugins have all been deprecated and won't be loaded by the CLI anymore. We listened to devs and determined they added unnecessary complexity. You can uninstall them from yo
ur project(s).                                 
remote:                                        
remote:            No functionality was removed and all commands will continue working normally. You may wish to review the CHANGELOG: https://github.com/ionic-team/ionic-cli/blob/master/CHA
NGELOG.md#changelog                            
remote:                                        
remote:            Thanks,                     
remote:            The Ionic Team              
remote: Build succeeded
remote: Uploading build...
remote: 
remote: Build upload successful.
remote: Running after script...
remote: $ clean-up
remote: Cleaning up files...
remote: Successful clean up
remote: Job succeeded
remote: 
To git.ionicjs.com:wanyi/aaa.git
 * [new branch]      master -> master
```

在前端管理界面，就可以进行对应的打包操作了：

![](/images/DeepinScreenshot_select-area_20170822144755.png)

编译完成后就可以下载安装：

![](/images/DeepinScreenshot_select-area_20170822145240.png)

## 使用ionic-view预览app

ionic提供了专门用来预览app效果的app，在AppStore和GooglePlay都可以下载，安装完后打开是如下图的效果，一样需要先登录账户：

![](/images/lALPACOG83H8-YvNB4DNBDg_1080_1920.png)

但是使用ionic-view只能预览不是属于自己的app，也就是别人分享给自己的才可以预览，这个听不方便，但是道理上讲得通（如果是自己的直接自己编译一个好了嘛）！