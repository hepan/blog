---
layout: post
title: Maven 相关
category: [Java,Maven]
comments: false
---

* content
{:toc}

具体参照：

[http://maven.apache.org/install.html](http://maven.apache.org/install.html)

[http://maven.apache.org/settings.html](http://maven.apache.org/settings.html)

### maven多模块设置

modules里定义是相对路径，跑测试的时候如：mvn test -pl ../test-web -DskipTests=false -Dtest=*  (或者多个test类以逗号分隔)

dependencyManagement定义依赖，子工程里引用不需要再加版本号，可以在父里统一版本

pluginManagement 定义插件，子工程可以直接引用

properties 里面可以定义maven系统属性如project.build.sourceEncoding，

建议不要定义maven.test.skip而采用skipTests配合maven-surefire-plugin替代，否则不能在工程里运行测试类

#### maven多模块编译某个工程并自动编译该工程所依赖的模块

```
进入父pom.xml目录

-pl 指定模块（相对路径）；-am指定自动编译依赖模块； -o离线模式(可选)；详细参考mvn -h

mvn clean compile -pl ../test-core -am -o
```

#### 父pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.yourgroup</groupId>
	<artifactId>yourgroup-springboot-center</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>
	<name>yourgroup-springboot-center</name>

	<!-- spring-boot -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.3.8.RELEASE</version>
		<relativePath />
	</parent>

	<modules>
		<module>../test-core</module>
		<module>../test-web</module>
	</modules>

	<properties>
		<!-- sys properties -->
		<skipTests>true</skipTests>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.8</java.version>
		<java_source_version>1.8</java_source_version>
		<java_target_version>1.8</java_target_version>
		<file_encoding>UTF-8</file_encoding>
		<yourgroup.version>1.0.0-SNAPSHOT</yourgroup.version>

		<!-- jar version properties -->
		<commons-logging.version>1.2</commons-logging.version>
		<commons-io.version>2.4</commons-io.version>
		<commons-lang.version>2.6</commons-lang.version>
		<commons-collections.version>3.2.1</commons-collections.version>
		<commons-beanutils.version>1.8.3</commons-beanutils.version>
		<commons-fileupload.version>1.3.1</commons-fileupload.version>
	</properties>

	<dependencyManagement> 
		<dependencies>
			<dependency>
				<groupId>${project.groupId}</groupId>
				<artifactId>test-core</artifactId>
				<version>${yourgroup.version}</version>
			</dependency>
			<dependency>
				<groupId>${project.groupId}</groupId>
				<artifactId>test-web</artifactId>
				<version>${yourgroup.version}</version>
			</dependency>
			<!-- commons jar -->
			<dependency>
				<groupId>commons-logging</groupId>
				<artifactId>commons-logging</artifactId>
				<version>${commons-logging.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-io</groupId>
				<artifactId>commons-io</artifactId>
				<version>${commons-io.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-lang</groupId>
				<artifactId>commons-lang</artifactId>
				<version>${commons-lang.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-collections</groupId>
				<artifactId>commons-collections</artifactId>
				<version>${commons-collections.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-beanutils</groupId>
				<artifactId>commons-beanutils</artifactId>
				<version>${commons-beanutils.version}</version>
			</dependency>
			<dependency>
				<groupId>commons-fileupload</groupId>
				<artifactId>commons-fileupload</artifactId>
				<version>${commons-fileupload.version}</version>
			</dependency>
			<!-- jetty -->
			<dependency>
				<groupId>org.eclipse.jetty.orbit</groupId>
				<artifactId>org.eclipse.jdt.core</artifactId>
				<version>3.8.2.v20130121</version>
				<scope>provided</scope>
			</dependency>
			<dependency>
				<groupId>org.eclipse.jetty.aggregate</groupId>
				<artifactId>jetty-all</artifactId>
				<version>9.2.3.v20140905</version>
				<scope>provided</scope>
			</dependency>
	</dependencyManagement>

	<build>
		<pluginManagement>
			<plugins>
				<plugin>
					<groupId>org.eclipse.jetty</groupId>
					<artifactId>jetty-maven-plugin</artifactId>
					<version>9.3.13.v20161014</version>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-resources-plugin</artifactId>
					<version>3.0.1</version>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-dependency-plugin</artifactId>
					<version>2.10</version>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-war-plugin</artifactId>
					<version>3.0.0</version>
				</plugin>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>3.5.1</version>
				</plugin>
				<plugin>
			        <groupId>org.apache.maven.plugins</groupId>
			        <artifactId>maven-surefire-plugin</artifactId>
			        <version>2.19.1</version>
			      </plugin>
			</plugins>
		</pluginManagement>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>${java.version}</source>
					<target>${java.version}</target>
					<encoding>${project.build.sourceEncoding}</encoding>
				</configuration>
				<executions>
	                <execution>
	                    <id>default-testCompile</id>
	                    <phase>test-compile</phase>
	                    <goals>
	                        <goal>testCompile</goal>
	                    </goals>
	                    <configuration>
	                        <skip>${skipTests}</skip>
	                    </configuration>
	                </execution>
	            </executions>
			</plugin>
			<plugin>
		        <groupId>org.apache.maven.plugins</groupId>
		        <artifactId>maven-surefire-plugin</artifactId>
		        <configuration>
		          	<skipTests>${skipTests}</skipTests>
		        </configuration>
		      </plugin>
		</plugins>
	</build>
</project>
```

#### 子pom.xml

mvn jetty:run 即可运行web工程

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.yourgroup</groupId>
		<artifactId>yourgroup-springboot-center</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<relativePath>../yourgroup-springboot-center</relativePath>
	</parent>
	<artifactId>test-web</artifactId>
	<packaging>war</packaging>
	<version>${yourgroup.version}</version>
	
	<properties>
	</properties>

	<dependencies>
			<groupId>${project.groupId}</groupId>
			<artifactId>test-core</artifactId>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<configuration>
					<warSourceDirectory>webapp</warSourceDirectory>
					<webappDirectory>target/${project.artifactId}</webappDirectory>
					<packagingExcludes>WEB-INF/lib/maven-*.jar,WEB-INF/lib/jetty-*.jar,WEB-INF/lib/servlet-api-*.jar,WEB-INF/lib/*servlet-api-*.jar,WEB-INF/lib/junit-*.jar</packagingExcludes>
				</configuration>
			</plugin>
			<plugin>
				  <groupId>org.eclipse.jetty</groupId>
				  <artifactId>jetty-maven-plugin</artifactId>
				  <configuration>
				   <webAppSourceDirectory>./src/main/webapp</webAppSourceDirectory>
				  	<httpConnector>
				  		<port>8080</port>
				  	</httpConnector>
				  	 <scanIntervalSeconds>2</scanIntervalSeconds>
				  	 <webApp>
					  	 <defaultsDescriptor>./src/test/resources/webdefault.xml</defaultsDescriptor>
					  	 <descriptor>./src/test/resources/web.xml</descriptor>
				  	 </webApp>
				  </configuration>
			</plugin>
		</plugins>
	</build>
</project>	
```

### settings配置文件

#### 通用仓库 settings.xml，（找jar包的顺序activeProfile->pom.xml的repositories-->mirrorOf_central）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository -->
  <localRepository>/yourname/program/apache/maven/repository</localRepository>

  <!-- interactiveMode
   | This will determine whether maven prompts you when it needs input. If set to false,
   | maven will use a sensible default value, perhaps based on some other setting, for
   | the parameter in question.
   |
   | Default: true
  <interactiveMode>true</interactiveMode>
  -->

  <!-- offline
   | Determines whether maven should attempt to connect to the network when executing a build.
   | This will have an effect on artifact downloads, artifact deployment, and others.
   |
   | Default: false
  <offline>false</offline>
  -->

  <!-- pluginGroups
   | This is a list of additional group identifiers that will be searched when resolving plugins by their prefix, i.e.
   | when invoking a command line like "mvn prefix:goal". Maven will automatically add the group identifiers
   | "org.apache.maven.plugins" and "org.codehaus.mojo" if these are not already contained in the list.
   |-->
  <pluginGroups>
    <!-- pluginGroup
     | Specifies a further group identifier to use for plugin lookup.
    <pluginGroup>com.your.plugins</pluginGroup>
    -->
  </pluginGroups>

  <!-- proxies
   | This is a list of proxies which can be used on this machine to connect to the network.
   | Unless otherwise specified (by system property or command-line switch), the first proxy
   | specification in this list marked as active will be used.
   |-->
  <proxies>
    <!-- proxy
     | Specification for one proxy, to be used in connecting to the network.
     |
    <proxy>
      <id>optional</id>
      <active>true</active>
      <protocol>http</protocol>
      <username>proxyuser</username>
      <password>proxypass</password>
      <host>proxy.host.net</host>
      <port>80</port>
      <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
    </proxy>
    -->
  </proxies>

  <!-- servers
   | This is a list of authentication profiles, keyed by the server-id used within the system.
   | Authentication profiles can be used whenever maven must make a connection to a remote server.
   |-->
  <servers>
    <!-- server
     | Specifies the authentication information to use when connecting to a particular server, identified by
     | a unique name within the system (referred to by the 'id' attribute below).
     |
     | NOTE: You should either specify username/password OR privateKey/passphrase, since these pairings are
     |       used together.
     |
    <server>
      <id>deploymentRepo</id>
      <username>repouser</username>
      <password>repopwd</password>
    </server>
    -->

    <!-- Another sample, using keys to authenticate.
    <server>
      <id>siteServer</id>
      <privateKey>/path/to/private/key</privateKey>
      <passphrase>optional; leave empty if not used.</passphrase>
    </server>
    -->
  </servers>

  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
     <mirror>
		<id>nexus-ibiblio</id>
		<mirrorOf>central</mirrorOf>
		<name>nexus-ibiblio</name>
		<url>http://mirrors.ibiblio.org/pub/mirrors/maven2/</url>
	</mirror>
  </mirrors>
  
  <profiles>
	<profile>
		<id>profile-nexus-aliyun</id>
		<activation>
			<activeByDefault>true</activeByDefault>
			<jdk>1.8</jdk>
		</activation>
		<properties>
			<maven.compiler.source>1.8</maven.compiler.source>
			<maven.compiler.target>1.8</maven.compiler.target>
			<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
		</properties>
		<repositories>
			<repository>
				<id>profile-nexus-aliyun</id>
				<name>nexus-aliyun</name>
				<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
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
				<id>profile-nexus-aliyun</id>
				<name>nexus-aliyun</name>
				<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
				<releases>
					<enabled>true</enabled>
				</releases>
				<snapshots>
					<enabled>false</enabled>
				</snapshots>
			</pluginRepository>
		</pluginRepositories>
	</profile>
  </profiles>

  <!-- activeProfiles   -->
  <activeProfiles>
    <activeProfile>profile-nexus-aliyun</activeProfile>
  </activeProfiles>
</settings>
```


#### 私有仓库 settings.xml

上传本地jar包到私有仓库：

mvn deploy:deploy-file -DgroupId=com.google -DartifactId=gson -Dversion=1.2 -Dpackaging=jar -Dfile=gson-1.2.jar -Durl=http://127.0.0.1:8081/repository/maven-3rd-party/ -DrepositoryId=yourServerId

（-DrepositoryId对应setting.xml里server的ID）

```xml
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <pluginGroups>
  </pluginGroups>
  <localRepository>/media/apache/maven/repository</localRepository>
  <proxies>
  </proxies>

  <servers>
    <server>
      <id>yourServerId</id>
      <username>yourName</username>
      <password>yourPassword</password>
    </server>
  </servers>

  <mirrors>
    <mirror>
      <id>yourMirrorId</id>
      <mirrorOf>*</mirrorOf>
      <name>yourMirrorName</name>
      <url>http://127.0.0.1:8081/repository/maven-public/</url>
    </mirror>
  </mirrors>

  <profiles>
        <profile>
            <id>yourMirrorId</id>
            <repositories>
                <repository>
                    <id>central</id>
                    <url>http://central</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>central</id>
                    <url>http://central</url>
                    <releases><enabled>true</enabled></releases>
                    <snapshots><enabled>true</enabled></snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>

    <activeProfiles>
        <activeProfile>yourMirrorId</activeProfile>
    </activeProfiles>
</settings>
```
