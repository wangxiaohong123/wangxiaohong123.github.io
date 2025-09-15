---
title: maven统一版本号修改
tags:
  - maven
categories: maven
copyright: true
---

引入 **versions-maven-plugin** 插件

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>versions-maven-plugin</artifactId>
    <version>2.7</version>
    <configuration>
        <generateBackupPoms>false</generateBackupPoms>
    </configuration>
</plugin>
```

设置父模块的版本号

```shell
mvn versions:set -DnewVersion=1.0.1
```

子模块同步父模块的版本号

```shell
mvn -N versions:update-child-modules
```

提交更新

```shell
mvn versions:commit
```

