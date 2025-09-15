---
title: maven基础
tags:
  - maven
categories: maven
copyright: true
---

gradle：google发布的，安卓用的比较多，一些开源的项目比如Spring也使用的这个构建的，使用DSL语言构建管理，非常难搞。

### 配置

maven配置了基本的环境变量之后还要再配一下opts，因为maven也是用java写的，有的时候构建的项目很大也会导致OOM，新建一个用户变量MAVEN_OPTS值是-Xms128m -Xmx512m就够了应该.

一般配置完settings.xml之后不要把这个文件放在maven的目录下，因为以后maven可能升级，这样还要去搞一下这个文件，所以要把这个配置文件放在用户目录下，用户目录下有一个.m2文件夹，如果没有就运行mvn help:system命令，然后就有了，这样的话以后不管怎么弄maven他都会去.m2文件夹下找配置文件。

### 大概的体系结构

1.  首先会去看.m2文件夹下的配置文件
2.  解析工程的pom.xml文件
3.  然后去本地仓库找依赖
4.  没找到的就去远程仓库下载到本地仓库

查看中央仓库地址：在maven的安装文件夹的lib目录下有个maven-model-builder的jar包，jar包里有一个超级pom文件，里面写了默认的中央仓库地址：https://repo.maven.apache.org/maven2/。

简单的打包：

mvn clean package：先清理上次编译的文件，然后编译项目在打包。

包名：一般都是公司域名倒过来，如果公司没有官网，就用com.公司名称缩写

### 坐标

maven有五个维度来定位唯一的jar包：groupId + artifactId + version + packaging + classifier，正常情况下用前三个就可以定位到了，企业级的坐标命名：

*   groupId：公司或者组织的官网域名倒叙然后再加上项目名，比如com.net5008.oa；
*   artifactId：项目名-模块名，比如权限模块：oa-auth；
*   version：版本号；
*   packaging：打包的方式，war或者jar或者pom；
*   classifier：定义工程的附属项目，

### dependency

<groupId>

<artifactId>

<version>

<type>：引入的类型，一般不用写，如果依赖的工程的打包方式写上pom，这块也应该写pom；

<scope>：maven有三种classpath，编译、测试和打包，scope标注了引用的jar包在那个classpath下可用，有四个选项：

* compile：默认的，三个classpath都可以用；
* test：测试范围有效，一般就是测试的框架使用；
* runtime：测试和打包有效，写代码（编译）的时候没有；
* provided：测试和编译有效，但是打包的时候是没有的；

比如说我们引入的junit，写上<scope>test</scope>之后，不能在正常的代码中使用@Test注解，只能在test包下使用。

<optional>：可选依赖，会让依赖传递失效，比如A依赖B，B依赖C，但是B中对C声明了optional，那么A就不会依赖C

#### 传递性依赖

maven会下载你依赖中依赖的其他jar包，会把所有的依赖链下完，自动下载。传递性依赖就是我们依赖了A，A自己依赖了B，如果我们对A的scope是compile，A对B是runtime，那么我们对B也是runtime。

#### 依赖调解

依赖链条可能会很长，当简介引用了相同的项目，但是版本不一样，就会选用最近的依赖版本，如果长度相等，就使用先声明的。

#### 既然有依赖调解为什么还会有依赖冲突？

这是因为maven选择了错误的版本，如果发生了冲突一般就选择最新的版本，因为正常的开源项目高版本都会兼容低版本，如果不兼容赶紧换插件，太垃圾了。

运行mvn dependency:tree，然后他会分析版本，找到最新的是哪个依赖引用的，然后把其他引用冲突的依赖给排除。

```xml
<exclusions>
	<exclusion>
		<groupId></groupId>
		<artifactId></artifactId>
	</exclusion>
</exclusions>
```

### 仓库

*   本地仓库：本地的仓库
*   私服：有的公司由于安全性不会让你使用外网，或者自己开发的中间件需要在别的项目使用，就要用到私服，只有私服连外网或者私服上有自己开发的中间件；
*   其他仓库：比如Jboss仓库、Google仓库，私服上和中央仓库上都没有就可以去其他仓库找找；
*   镜像仓库：由于中央仓库在国外，所以下载非常慢，阿里云的镜像仓库差不多和中央仓库一样，由于在国内，下载jar包非常快，镜像仓库没有再去中央仓库上找；
*   中央仓库：maven的超父pom中定义的仓库地址；

##### 私仓

使用nexus就可以搭建自己的私仓，挺简单的，用的时候百度一下，主要有三种仓库。

*   group：包含了hosted和proxy，主要是为了用方便，在配置文件中配置组的地址，他就会自己选择去宿主和代理仓库中查找jar包了。
*   hosted：宿主仓库，自己写的jar放到这里。
*   proxy：代理仓库，放其他仓库和中央仓库，比如Jboss仓库，中央仓库放阿里云的镜像仓库。

包的版本：

*   release：就是测试完了可以再生产环境使用的；
*   snapshot：正在开发的版本；
*   3rd part：不能在中央仓库获取的第三方jar，比如有的支付的SDK，这个SDK并不是开源的，需要自己放到私仓中；

配置私服

在settings.xml中修改：

```xml
<profiles>
    <profile>
        <id>nexus</id>
      	<repositories>
            <repository>
                <!-- 下面两个id都配置成central，因为超pom中给的就是central，我们也叫central，这样就会覆盖超级pom中的配置 -->
                <id>central</id>
                <name>Nexus </name>
                <!--正常来讲要写仓库组的地址，但是下面配置了拦截所有请求都转到仓库组，所以这个url没有用了-->
      	  		<url>http://central</url>
                <!--releases和snapshots版本的jar都可以下载-->
                <releases><enabled>true</enabled></releases>
                <snapshots><enabled>true</enabled></snapshots>
            </repository>
        </repositories>
      	<pluginRepositories>
            <pluginRepository>
                <id>central</id>
                <name>Nexus Plugin Repository</name>
                <!--仓库组的地址-->
      			<url>http://central</url>
          		<releases><enabled>true</enabled></releases>
          		<snapshots><enabled>true</enabled></snapshots>
        	</pluginRepository>
      	</pluginRepositories>
	</profile>
</profiles>

<activeProfiles>
	<activeProfile>nexus</activeProfile>
</activeProfiles>

<mirrors>
    <!--强制让下载jar都走nexus，就是配置的私仓-->
	<mirror>
		<id>nexus</id>
		<mirrorOf>*</mirrorOf>
        <!--仓库组的地址-->
		<url>http://localhost:8081/nexus/content/groups/public</url>
	</mirror>
</mirros>
```

nexus权限：

默认的admin账号密码是admin123，拥有所有角色的权限，还有一个游客账号anonymous，只有查找和下载的权限，我们可以自己创建权限、角色、用户。一般还要创建一个角色deployment，专门上传jar包的。

项目打包到私服：

在项目的pom中添加：

```xml
<distributionManagement>
    <!-- 发布版打包地址 -->
	<repository>
		<id>nexus-releases</id>
		<name>Nexus Release Repository</name>
        <!--maven-releases宿主仓库地址-->
		<url>http://localhost:8081/nexus/content/repositories/releases/</url>
	</repository>
    <!-- 开发版打包地址 -->
	<snapshotRepository>
		<id>nexus-snapshots</id>
		<name>Nexus Snapshot Repository</name>
        <!--maven-snapshots宿主仓库地址-->
		<url>http://localhost:8081/nexus/content/repositories/snapshots/</url>
	</snapshotRepository>
</distributionManagement>
```

在settings.xml中添加用户名和密码：

```xml
<servers>
    <!-- server中的id要和项目中的snapshotRepository和repository中的id一致，如果不一样就找不到用户名和密码，nexus就会使用默认的游客账号部署，如果游客账号没有权限就不能打包到私服 -->
	<server>
		<id>nexus-releases</id>
		<username>deployment</username>
		<password>deployment123</password>
	</server>
	<server>
		<id>nexus-snapshots</id>
		<username>deployment</username>
		<password>deployment123</password>
	</server>
    <!-- 项目中是没有配置nexus-3rd-party的，需要的话就加上 -->
    <server>
		<id>nexus-3rd-party</id>
		<username>deployment</username>
		<password>deployment123</password>
	</server>
</servers>
```

如果私服在本地，使用mvn clean install就可以打包到本地私服了，如果私服不在本地，需要使用mvn clean deploy命令。如果上传的是snapshot版本会带一个时间戳，因为开发版本可能会频繁的上传，但是版本号没改，我觉得是这样，据说依赖是不需要使用时间戳的那个version，写项目中定义的就可以，但是我们的阿里云私仓是不好使的，需要使用时间戳的版本号。如果需要把第三方的jar推到3rd party仓库中，要使用命令：

```shell
# 这里有jar在本地的路径，需要上传的仓库路径，还有DrepositoryId，这个适合上面server中nexus-3rd-party是一样的
mvn deploy:deploy-file -DgroupId=com.csource -DartifactId=fastdfs-client-java -Dversion=1.24 -Dpackaging=jar -Dfile=F:\DevelopmentKit\fastdfs_client_v1.24.jar -Durl=http://localhost:8081/repository/3rd-party/ -DrepositoryId=nexus-3rd-party
```

##### 生命周期

maven有三种声明周期，clean、default和site，每个周期都是独立的，每个周期还有一些phase，每个phase有很多plugin的goal，一些phase是绑定了plugin的goal的，执行phase就会执行绑定的goal。

###### clean的phase

*   pre clean
*   clean
*   post clean

###### default的phase

*   validate：校验这个项目的一些配置信息是否正确；
*   initialize：初始化构建状态，比如设置一些属性，或者创建一些目录；
*   generate-sources：自动生成一些源代码，然后包含在项目代码中一起编译；
*   process-sources：处理源代码，比如做一些占位符的替换；
*   generate-resources：生成资源文件，主要是去处理各种xml、properties那种配置文件，去做一些配置文件里面占位符的替换；
*   process-resources：将资源文件拷贝到目标目录中，方便后面打包；
*   compile：编译项目的源代码；
*   process-classes：处理编译后的代码文件，比如对java class进行字节码增强；
*   generate-test-sources：自动化生成测试代码；
*   process-test-sources：处理测试代码，比如过滤一些占位符；
*   generate-test-resources：生成测试用的资源文件；
*   process-test-resources：拷贝测试用的资源文件到目标目录中；
*   test-compile：编译测试代码；
*   process-test-classes：对编译后的测试代码进行处理，比如进行字节码增强；
*   test：使用单元测试框架运行测试；
*   prepare-package：在打包之前进行准备工作，比如处理package的版本号；
*   package：将代码进行打包，比如jar包；
*   pre-integration-test：在集成测试之前进行准备工作，比如建立好需要的环境；
*   integration-test：将package部署到一个环境中以运行集成测试；
*   post-integration-test：在集成测试之后执行一些操作，比如清理测试环境；
*   verify：对package进行一些检查来确保质量过关；
*   install：将package安装到本地仓库中，这样开发人员自己在本地就可以使用了；
*   deploy：将package上传到远程仓库中，这样公司内其他开发人员也可以使用了；

###### site生命周期的phase：

*   pre-site
*   site
*   post-site
*   site-deploy

##### phase默认绑定的plugin goal：

process-resources -> resources:resources

compile -> compiler:compile

process-test-resources -> resources:testResources

test-compile -> compiler:testCompile

test -> surefire:test

package -> jar:jar或者war:war

install -> install:install

deploy -> deploy:deploy

site生命周期的默认绑定是：

site -> site:site

site-deploy -> site:deploy

clean生命周期的默认

clean -> clean:clean

##### 命令和phase的关系：

mvn后面可以跟任何生命周期的任何phase，比如mvn clean install，这时候就会先执行clean phase前面所有phase和clean phase，然后在执行install phase，因为pre clean没有绑定任何plugin，所以啥都不执行，clean phase绑定了clean:clean插件，所以会清除上次编译的东西，install phase也是一样的，这些都可以在执行命令时的过程的日志看到；

还有一种命令比如mvn dependency:tree，这个命令的意思就是不执行phase，直接执行plugin的goal。

##### plugin

默认情况下没有配置plugin是只有默认的phase绑定的plugin的，默认的plugin的在我们打包的时候是不会将依赖放在打出来的jar包中，要实现这种特殊的功能就要在pom中配置这些plugin，在配置plugin绑定到那个phase上去。举个例子：

```xml
<plugins>
	<plugin>
        <!-- 定位plugin -->
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-source-plugin</artifactId>
        <version>3.0.1</version>
        <!-- 什么时候执行，可以有多个execution -->
        <executions>
            <execution>
                <id>attach-source</id>
                <!-- 绑定到那个phase -->
                <phase>verify</phase>
                <goals>
                    <!-- 执行那个goal，可以有多个 -->
                	<goal>jar-no-fork</goal>
                </goals>
            </execution>
        </executions>
    </plugin>
</plugins>
```

### 父子模块

正常的开源框架的源码都是一个大父模块然后里面各种子模块，只要在父模块里使用<modules>标签把其他模块加进来就变成了父子模块，这时候只是能扫描到子模块，依赖什么的还是要自己定义，而且这样的时候打包必须也要把父模块推到私仓上去，要不然别人从私仓下载很容易出现找不到parent的问题。只要在子模块中添加parent标签就可以把父模块中的依赖全部加载过来：

```xml
<parent>
	<groupId>com.zhss.oa</groupId>
	<artifactId>oa-parent</artifactId>
	<version>1.0.0-SNAPSHOT</version>
</parent>
```

加完这个命令之后就可以删掉子模块中的定义自己的groupId和version标签，这是最low的做法，因为子工程会全部强制继承父工程的所有依赖和所有插件，这样有很多依赖是多余的，这样会使打包变慢，工程包变胖，正确的做法是父工程使用<dependencyManagemnent>和<pluginManagement>来声明依赖和插件，子模块中只用groupId和artifactId来表示使用这个依赖，如果加上版本号了，就不使用父模块的依赖，这样虽然pom内容会变多，但是按需加载是很规范的。

还有一点需要优化就是版本号的同一管理，比如SpringBoot的相关依赖的version都是2.1.2.RELEASE，没必要每个依赖都写一遍，而且以后升级还可能少改，可以在<properties>标签中自定义标签，然后以后的依赖使用占位符来使用标签内容：

```xml
<properties>
    <springboot.release.version>2.1.2.RELEASE</springBoot.release.version>
</properties>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>${springboot.release.version}</version>
</dependency>
```

#### 让别人应用你得工程的时候强制使用你工程中的依赖的版本

比如说你有一个工程A，里面依赖了activiti7.0的版本，如果你不想让别人引用你得工程的时候还要再去引用activiti6.0的版本，这时候可以新建一个空工程叫A-bom，在这个里面添加：

```xml
<!-- 这是一个空工程，里面只有依赖 -->
<packaging>pom</packaging>

<dependencyManagemnent>
    <dependencies>
        <dependency>
            <groupId>com.****.A</groupId>
            <artifactId>A</artifactId>
    		<version>1.0.0.SNAOSHOT</version>
        </dependency>
        <dependency>
            <groupId>org.activiti</groupId>
            <artifactId>activiti.engine</artifactId>
    		<version>7.0.0</version>
        </dependency>
    </dependencies>
</dependencyManagemnent>
```

别人引用你得工程的时候需要使用<dependencyManagemnent>把bom工程依赖进去，并且type需要设置成pom，scope设置成import，这样就相当于把bom的<dependencyManagemnent>放到这个pom.xml中去，然后要用啥依赖需要单独引用，引用是不需要写version，如果写上version并且和bom中的不一致，就会报错。

### 常用插件

1.  默认继承surefire插件，在test的phase的时候会去执行test目录下test开头、test结尾或者testCase结尾的文件，如果不想执行test，就在命令后加上-DskipTests。

2.  测试代码覆盖率插件：

    ```xml
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>cobertura-maven-plugin</artifactId>
        <!-- 版本号会有默认的，包括configuration什么的都有默认的，可以不写，默认的就够用 -->
        <version>2.5.1</version>
    </plugin>
    ```

打完包之后在target下的site文件夹下有个index，打开就看到了覆盖率.

3.  web工程的冒烟测试插件，好像也用不到，现在是curage测试：

    ```xml
    <plugin>
    	<groupId>org.mortbay.jetty</groupId>
    	<artifactId>jetty-maven-plugin</artifactId>
    	<version>7.1.6.v20100715</version>
    	<configuration>
    		<scanIntervalSeconds>10</scanIntervalSeconds>
    		<webAppConfig>
    			<contextPath>/test</contextPath>
    		</webAppConfig>
    	</configuration>
    </plugin>
    ```

4.  自动部署插件，现在用不到了，都是SpringBoot的项目

    ```xml
    <plugin>
    	<groupId>org.codehaus.cargo</groupId>
    	<artifactId>cargo-maven2-plugin</artifactId>
    	<version>1.0</version>
    	<configuration>
    		<container>
    			<containerId>tomcat8x</containerId>
    			<type>remote</type>
    		</container>
    		<configuration>
    			<type>runtime</type>
    			<properties>
    				<cargo.remote.username>admin</cargo.remote.username>
    				<cargo.remote.password>admin</cargo.remote.password>
    				<cargo.tomcat.manager.url>
                        http://localhost:8080/manager
                    </cargo.tomcat.manager.url>
    				<cargo.servlet.port>8080</cargo.servlet.port>
    			</properties>
    		</configuration>
    	</configuration>
    </plugin>
    ```

5.  使用profile适配各个环境

    一般有五个环境：

    *   local环境，就是自己的开发环境，dev环境；
    *   beta环境，就是集成测试的环境，和其他开发把项目集成简单测试，保证主流程跑通；
    *   test环境，重新部署到测试服务器，进行QA测试；
    *   staging环境，让生产环境的一部分流量打到这个环境；
    *   prod环境，生产环境；

    ```xml
    <profiles>
    	<profile>
            <!-- 默认激活环境 -->
            <activation>
            	<activeByDefault>true</activeByDefault>
            </activation>
            <id>dev</id>
            <properties>
                <database.jdbc.driverClass>com.mysql.jdbc.Driver</database.jdbc.driverClass>
    			<database.jdbc.connectionURL>
                	jdbc:mysql://192.168.31.110:3306/oa_local
                </database.jdbc.connectionURL>
    			<database.jdbc.username>local</database.jdbc.username>
    			<database.jdbc.password>local</database.jdbc.password>
    		</properties>
    	</profile>
    	<profile>
            <id>test</id>
            <properties>
                <database.jdbc.driverClass>com.mysql.jdbc.Driver</database.jdbc.driverClass>
                <database.jdbc.connectionURL>
                    jdbc:mysql://192.168.31.110:3306/oa_dev
                </database.jdbc.connectionURL>
                <database.jdbc.username>dev</database.jdbc.username>
                <database.jdbc.password>dev</database.jdbc.password>
            </properties>
        </profile>
    </profiles>
    
    <!-- 添加过滤，配置路径下的文件会被检查有没有占位符，有就替换 -->
    <resources>
    	<resource>
            <directory>${project.basedir}/src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <testResources>
        <testResource>
            <directory>${project.basedir}/src/test/resources</directory>
            <filtering>true</filtering>
        </testResource>
    </testResources>
    ```

    打包时在命令后面加``-P[profile中的id]``就可以实现不同环境，但是都写到一个一个文件里就很乱，可以在main下新建个profiles文件夹，里面每个环境再创建一个文件夹，然后每个文件夹下有一套配置，在打包时根据不同的参数加载相应的配置，profile就变成了下面这样：

    ```xml
    <profile>
        <id>dev</id>
        <build>
            <resources>
                <resource>
                    <directory>src/main/profiles/beta</directory>
                    <includes>
                        <include>**/*.xml</include>
                        <include>**/*.properties</include>
                    </includes>
                    <filtering>true</filtering>
                </resource>
            </resources>
        </build>
    </profile>
    ```

### 版本管理和版本控制

版本管理说的是maven中的version，版本控制说的是远程代码仓库中的版本，比如maven在开发测试中的版本始终是1.0.0.SNAPSHOT，但是每次提交git中的代码版本都会生成一个新的，在确定上线的时候可能合并到master分支，这时候改的就是master分支的版本了。

1.0.0版本的1说得是大版本，大改动，第一个0是次要改动，比如添加个辅助模块，第三个0是增量版本，修改bug之类的

### 自定义archetype

创建一个模板，然后推到私服上，使用命令创建maven工程，感觉不常用，用的时候再来写

### 整合git

### 持续集成