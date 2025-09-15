---
title: 私仓相关
tags:
  - maven
categories: maven
copyright: true
---

仓库有三种：

*   group（仓库组）：将各种宿主仓库、代理仓库组成虚拟的仓库组，项目只要配置依赖一个仓库自，相当于自动连接仓库组内的各种仓库。

*   proxy（代理仓库）：代理了公司外部的各种仓库，比如Jboss、中央仓库，可以修改为阿里的。

*   hosted（宿主仓库）：公司内部的发布包。

平时用的仓库就五个

*   maven-central：maven中央仓库的代理仓库；
*   maven-public：仓库组，我们平时就连这个；
*   maven-release：公司内部经过完善的测试，可以在生产环境使用的发布包；
*   maven-snapshots：正在开发过程中的其他项目也需要依赖的的发布包；
*   3rd-party：这个是放中央仓库下载不了，我们自己下载然后传到这个仓库，3.9没有这个仓库，需要我们自己创建；

剩下三个是.net使用的。

账号有三种：

*   deployment：可以搜索构建，普通的开发账号，3.x之后需要自己创建；
*   admin：管理员，密码是admin123；
*   anonymous：匿名账号，可以下载和查看依赖；