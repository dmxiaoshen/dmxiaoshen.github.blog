title: Maven简单配置
date: 2014-11-18 21:17:11
categories: note
tags: [maven, 配置]
---

## Maven简单配置

### 配置本地仓库  
打开**maven安装目录**，编辑**conf/settings.xml**文件，加入`<localRepository>F:\repository</localRepository>`  
其中**F:\repository**就是需要配置的本地仓库地址。  
**注意:**这个是全局的配置文件，如想为每个用户单独配置，请在每个仓库单独配置，简单来讲，按如上举例，  

1. 先在F盘建立repository文件夹。  
2. 把maven安装目录/conf/settings.xml文件复制一份到第一步建立的文件夹下。  
3. 修改F:/repository/settings.xml,加入`<localRepository>F:\repository</localRepository>`。

### 远程仓库  
maven的默认中央仓库：  
```xml
 <repositories>  
  <repository>  
    <id> central</id>  
    <name> Maven Repository Switchboard</name>  
    <layout> default</layout>  
    <url>http://repo1.maven.org/maven2</url>  
    <snapshots>  
      <enabled> false</enabled>  
    </snapshots>  
  </repository>  
 </repositories>
```

如果觉得默认中央仓库很慢，想配置其他远程仓库，比如[开源中国的maven库][1]。  
修改**setting.xml**文件:

```xml
 <mirrors>
  <!-- mirror | Specifies a repository mirror site to use instead of a given
  repository. The repository that | this mirror serves has an ID that matches
  the mirrorOf element of this mirror. IDs are used | for inheritance and direct
  lookup purposes, and must be unique across the set of mirrors. | -->
  <mirror>
   <id>nexus-osc</id>
   <mirrorOf>*</mirrorOf>
   <name>Nexus osc</name>
   <url>http://maven.oschina.net/content/groups/public/</url>
 </mirror>
</mirrors>
```


**补充：**如果还需要osc的thirdparty仓库或多个仓库，需要如下修改:

```xml
 <mirrors>
  <!-- mirror | Specifies a repository mirror site to use instead of a given
   repository. The repository that | this mirror serves has an ID that matches
   the mirrorOf element of this mirror. IDs are used | for inheritance and direct
   lookup purposes, and must be unique across the set of mirrors. | -->
   <mirror>
    <id>nexus-osc</id>
    <mirrorOf>central</mirrorOf>
    <name>Nexus osc</name>
    <url>http://maven.oschina.net/content/groups/public/</url>
   </mirror>
   <mirror>
    <id>nexus-osc-thirdparty</id>
    <mirrorOf>thirdparty</mirrorOf>
    <name>Nexus osc thirdparty</name>
    <url>http://maven.oschina.net/content/repositories/thirdparty/</url>
  </mirror>
 </mirrors>
```

这里是配置 Maven 的 mirror 地址指向OSChina 的 Maven 镜像地址。 在执行 Maven 命令的时候， Maven 还需要安装一些插件包，这些插件包的下载地址也让其指向 OSChina 的 Maven 地址。修改如下内容。 

```xml
<profile>
    <id>jdk-1.4</id>
 
    <activation>
	<jdk>1.4</jdk>
    </activation>
 
    <repositories>
	<repository>
	    <id>nexus</id>
	    <name>local private nexus</name>
	    <url>http://maven.oschina.net/content/groups/public/</url>
	    <releases>
		<enabled>true</enabled>
	    </releases>
	    <snapshots>
		<enabled>false</enabled>
	    </snapshots>
	</repository>
    </repositories>
    <pluginRepositories>
	<pluginRepository>
	    <id>nexus</id>
	    <name>local private nexus</name>
	    <url>http://maven.oschina.net/content/groups/public/</url>
	    <releases>
		<enabled>true</enabled>
	    </releases>
	    <snapshots>
		<enabled>false</enabled>
	    </snapshots>
	</pluginRepository>
    </pluginRepositories>
</profile>
```

当然也可以给每个项目单独配置仓库，需在项目的pom.xml文件中配置，如：  
```xml
<project>
  ...
  <repositories>
    <repository>
      <id>jboss</id>
      <name>JBoss Repository</name>
      <url>http://repository.jboss.com/maven2/</url>
      <releases>
        <enabled>true</enabled>
      </releases>   
     <snapshots>
      <enabled>false</enabled>
     </snapshots>
     <layout>default</layout>
    </repository>
  </repositories>
  ...
</project>
```

### 私服Nexus  
> Nexus是一个Maven仓库管理器，用来搭建私有仓库服务器。建立公司/组织的私有仓库的的好处是便于管理，节省公网带宽，利用内网下载依赖项速度快，还有一个非常有用的功能就是能有效管理内部项目的SNAPSHOT版本，实现各个模块间的共享. 

还没有玩过，以后补充。

参考网址:  

* <http://maven.oschina.net/help.html>
* <http://my.oschina.net/aiguozhe/blog/101537>


[1]: http://maven.oschina.net/ "开源中国maven帮助"





