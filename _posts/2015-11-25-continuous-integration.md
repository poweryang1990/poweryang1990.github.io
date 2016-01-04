---
layout: post
title: 持续集成部署
tags: CI CD TeamCity Git
categories: .Net
---

###1、流程
更新编译代码、代码质量检查、更新数据库、执行单元测试、部署后台服务、执行接口测试、部署前台应用、执行UI测试、构建结果发布  
http://www.infoq.com/cn/articles/develop-continuous-integration-around-automation-test
###2、持续集成之“依赖管理”
http://www.infoq.com/cn/news/2011/05/ci-dependency-management
###3、eg:Continuous Integration & Delivery For GitHub With TeamCity
http://www.codeproject.com/Articles/703244/Continuous-Integration-Delivery-For-GitHub-With-Te  

### 4、Using TeamCity for ASP.NET development: Deploying

* [Installing/Configuring TeamCity for use with IIS](https://johanleino.wordpress.com/2013/03/19/using-teamcity-for-asp-net-development/) 
* [MSBuild requirements for web package publishing](https://johanleino.wordpress.com/2013/03/19/using-teamcity-for-asp-net-development-part-2/)
* [Deploying via Web Deploy](https://johanleino.wordpress.com/2013/04/02/using-teamcity-for-asp-net-development-part-3/)
* [Backup (pre-deploy)](https://johanleino.wordpress.com/2013/05/24/using-teamcity-for-asp-net-development-part-4/)

### 5、GitLabCI GitLab Runner  
* 安装Runner  
    https://gitlab.com/gitlab-org/gitlab-ci-multi-runner

* 配置.gitlab-ci.yml
    
<pre><code>
    variables:
    Solution: UOKO.SSO.sln
    before_script:
    - 'echo off'
    stages:
    - nugetrestore
    - build
    nugetrestore:
    stage: nugetrestore
    script:
        - 'cd .nuget'
        - 'echo restoring..'
        - 'nuget restore "../%Solution%"'
        - 'cd ..'
        - 'echo restored'
    only:
        - master
    tags:
        - zhipingrunner

    build:
    stage: build
    script:
        - 'echo building..'
        - '"%SystemRoot%\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe" "%Solution%" /t:Build /property:Configuration=Debug /p:VisualStudioVersion=12.0'
    only:
        - master
    tags:
        - zhipingrunner
 </code></pre>

### 6、MSbuild AppCmd MSDeploy
<pre><code>

    #打包 MSBuild="%SystemRoot%\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe"

    MSBuild "F:\UOKOWorkspace\Git\uoko.sso\UOKO.SSO.Server\UOKO.SSO.Server.csproj" /T:Package /property:Configuration=Release /P:VisualStudioVersion=12.0 
    /P:PackageLocation="F:\发布\pakagezip\uokossodeploy.zip"

    #创建站点 appcmd="%SystemRoot%\C:\Windows\System32\inetsrv\appcmd.exe"
    appcmd add site /name:"ids.uoko.ioc"  /bindings:http://ids.uoko.ioc:80 /physicalPath:"F:\发布\ids.uoko.ioc"
    appcmd add appplool /name:ids.uoko.ioc
    appcmd set app ids.uoko.ioc/ -applicationPool:ids.uoko.ioc

    #本地部署
    F:\发布\pakagezip\uokossodeploy.deploy.cmd "-setParam:name='IIS Web Application Name',value=ids.uoko.ioc" /y

    #远程部署
    F:\发布\pakagezip>uokossodeploy.deploy.cmd /Y /U:simple-deploy /P:deploy /A:Basic "/M:https://WIN-DEVTEST-002:8172/msdeploy.axd?site=ids.uoko.ioc" -allowUntrusted "-setParam:name='IIS Web Application Name',value='ids.uoko.ioc'"
 </code></pre>