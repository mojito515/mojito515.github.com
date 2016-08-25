---
layout: post
title: "如何使用gitlab的flow以及代码review"
date: 2016-03-09 11:50:02 +0800
comments: true
categories:
---
<!--more-->
###Git工作流

我们在工作中经常用到git来管理自己的代码，也会涉及到多人协作的场景，
被广泛使用的三种工作流如下：

* ####Git flow
* ####Github flow
* ####Gitlab flow

#####以下只简单总结三种flow的特点和弊端,具体的介绍和比较请移步阮一峰老师的文章[《Git工作流》](http://7xpd26.com1.z0.glb.clouddn.com/gitlab_create_project.png)

####Git flow
 >典型的长期维护master分支和develop分支，因为是FDD（功能驱动开发），所以会在协作开发中衍生出 功能分支（feature branch）、补丁分支（hotfix branch）、预发版分支（release branch），完成之后会合并到develop或者master分支，之后删除。优点是清晰可控，但这个模式是基于“版本发布”的，目标是一段时间产出一个新版本，不适合“持续发布”的网站开发。



####Github flow
 >只有一个master长期分支，需要协同的人可以fork代码（其实就是新建了一个自己的分支，并且pull到了master上的代码），当你的功能需求代码完成之后，或者需要讨论的时候，就向master发起一个pull request。通知到别人评审、讨论、review你的代码，方便的是，在request提交之后评审的过程中，你还可以提交代码。等到你的request被accept，分支会合并到master，重新部署后，你原来的那个分支就可以删除啦。缺点是有时你的产品发布的代码版本和你master最新的版本并不是一个（比如因为苹果审核需要时间，那么你的代码就需要另一个分支来保留线上版本）。

####Gitlab flow
>引入了“上游优先”（upsteam first）的原则。只存在一个主分支master，它是所有其他分支的"上游"。只有上游分支采纳的代码变化，才能应用到其他分支。版本发布"的项目，建议的做法是每一个稳定版本，都要从master分支拉出一个分支。使用gitlab建立group project，可以将成员全部添加进小组中，每个人的提交都以分支合并进master分支的方式进行，我们可以将master设置成protected branch，这样就做到了强制代码review的机制，利于提升代码的质量。Issue 用于 Bug追踪和需求管理。建议先新建 Issue，再新建对应的功能分支。



-----------------

###Gitlab如何使用

首先，在gitlab的console中创建工程，创建好后会有如下图的命令提示，告知你怎样在本地创建代码项目并push（使用sourcetree更简单）：

![create project](http://7xpd26.com1.z0.glb.clouddn.com/gitlab_create_project.png)

项目创建完成之后，给项目添加成员：

![new member](http://7xpd26.com1.z0.glb.clouddn.com/gitlab_new_member.png)

把master分支设置成受保护分支，这样成员在提交代码的时候，只能先提交merge request（强制做代码review）：

![proetcted branch](http://7xpd26.com1.z0.glb.clouddn.com/gitlab_protected_branch.png)

在本地，以developer的身份push代码，会显示不成功：

![error](http://7xpd26.com1.z0.glb.clouddn.com/gitlab_push_error.png)

正常流程中，是先本地从master上拉取新建分支：

![new branch](http://7xpd26.com1.z0.glb.clouddn.com/gitlab_new_feature.png)

当有代码需要提交push的时候，在gitlab的console中创建merge request 完成代码向master分支的提交：

![merge request](http://7xpd26.com1.z0.glb.clouddn.com/gitlab_merge_request.png)

负责review的小伙伴可以对代码进行评论，在accept之前，该分支中再次push的commit都归属于这次merge request。accept之后，分支自动合并到master分支中（可以勾选直接删除merge的功能分支）:

![review](http://7xpd26.com1.z0.glb.clouddn.com/gitlab_review.png)

至此，一次完整的代码提交过程就完成了。当然，在项目上线之后，会有“下游”的分支，例如 生产版本的分支、预生产版本的分支也会加入到protected branch的行列。
