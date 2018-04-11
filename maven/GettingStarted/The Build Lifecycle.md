# Build生命周期简介
## 内容目录
* Build生命周期基础
* 用Build生命周期配置你的项目
	* 打包
	* 插件
* 生命周期参考
* 绑定的内建生命周期
## Build生命周期基础
maven基于build生命周期这个核心概念。它意味着创建和分配特定组件（项目）的过程是明确定义的。对于构建项目的人来说，这意味着只需要学习构建maven项目的命令的最小集合，而POM文件会确保他们得到他们希望的结果。
有三个内建的build生命周期：default、clean和site。default生命周期处理项目的部署，clean生命周期处理项目清理，而site生命周期处理生成项目文档。
### 一个build生命周期有多个阶段组成
以上每个build生命周期都由一组build阶段定义，一个build阶段代表着生命周期的一个步骤。
举个栗子，default生命周期包括下面这些阶段（完整的阶段列表，查看[LifeCycle Reference](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)）

* validate - 验证项目的正确性和必需信息是否可用
* compile - 编译项目源代码
* test-用对应的单元测试框架测试编译的代码，这些测试不应该要求代码被打包或部署
* package-提取编译后的代码并打包成可分发的格式，比如**JAR**
* verify-运行任何对集成测试结果的检测以确保质量标准得到满足
* install-安装软件包到本地仓库，作为其他项目的本地依赖
* deploy-在构建环境中执行，将最终的软件包复制到远程仓库，从而和其它开发者、项目共享

这些生命周期阶段（加上其它未展示的其它阶段）将被顺序执行来完成default生命周期。上面给出的生命周期阶段，这意味着当default生命周期被使用时，maven将首先验证项目，然后尝试编译源码，针对测试运行这些编译代，打包成二进制文件（即jar），针对jar包运行集成测试，验证集成测试，安装验证后的软件包到本地仓库，然后部署安装到远程仓库。
## 常用命令调用
在开发环境中。使用下面的调用构建和安装组件到本地仓库

	mvn install

这个命令将按顺序执行default生命周期的每一个阶段（validate，compile，package，等等）在执行install之前，你只需要调用执行最后一个build阶段，在这种情况下，install：
在构建环境中，使用下面的调用清除构建内容，然后部署组件到共享仓库

	mvn clean deploy


同样的命令可以在多模块脚本中使用（即一个项目有多个子项目）。maven遍历每个子项目执行clean，然后执行deploy（包括所有build阶段）

### 一个Build阶段由多个plugin goal组成

然而，即使每个build阶段负责build生命周期的一个特定步骤，但他们履行这些职责的方式可能是多变的。这是通过声明插件目标绑定到build阶段完成的

一个插件目标表示一个特定的任务（比build阶段更细），这个任务是有助于项目的构建和管理的。一个插件目标可以能被绑定到0个或多个build阶段。一个插件目标不是通过直接调用的方式绑定到任意可以在build生命周期外被执行的build阶段。执行顺序取决于插件目标和build阶段被执行的顺序。例如，考虑下面一个命令。clean和package参数是build阶段，而dependency:copy-denpendecied是插件目标。
	
	mvn clean dependency:copy-dependencies package

如果这个命令被执行，clean阶段将被最先执行（意味着他将允许所有clean生命周期之前的阶段，加上clean阶段自身）。而后是dependency:copy-dependencies这个插件目标，最后执行package阶段（和所有package之前的build阶段）
此外，如果一个插件目标绑定到一个或多个build阶段，这个目标将在所有这些阶段被调用。
更进一步，一个build阶段也可以被0个或多个目标绑定。如果一个build阶段没有绑定目标，那这个build阶段将不会被执行。单如果他被一个或多个目标绑定，它将执行所有这些目标。
（注意：maven2.0.5以上，一个阶段绑定的多个目标将按照POM文件中定义的顺序执行，然后多个同样的插件实例是不支持的。maven2.0.11以上多个同样的插件实例将被组合执行）

### 一些阶段通常不是通过命令行调用

以复合词命名的阶段（pre-*，post-*，process-*）通常不是直接通过命令行调用的。这些阶段对build进行排序，产生不为build外部使用的中间结果。在调用integration-test这个例子中，环境可能处于悬挂状态。

代码覆盖工具比如Jacoco、执行容器插件如Tomcat，Cargo和Docker绑定目标到pre-integration-test阶段来准备集成测试容器环境。这些插件也绑定目标到post-integration-test阶段来收集覆盖率统计或注销集成测试容器。

故障安全（FailSage）和代码覆盖（code coverage）插件绑定目标到integration-test和verify阶段，The net result is test and coverage reports are available after the verify phase，如果integration-test被命令行调用，讲没有报告生成。更糟的是集成测试环境遗留在悬挂状态；Tomcat服务器或Docker实例处在运行状态，而maven可能不会自己中断。

## 使用Build生命周期配置你的项目
build生命周期简单易用，但当你用来为一个项目构建Maven Build，你如何将任务分配给各个build阶段呢？

### 打包
首先，最普遍的方式是通过POM元素&lt;packaging>给你的项目设置打包方式。一些有效的packaging值有jar、war、ear和pom。如果没有指定packaging值，它将默认为jar。

每个打包方式包含了一系列绑定到特定阶段的目标。例如，jar方式将绑定以下目标到default生命周期的各个build阶段。

|阶段|目标|
|:---|:---:|
|process-resources|resources:resources|
|compile|compiler:compile|
|process-test-resources|resources:testResources|
|test-compile|compiler:testCompile|
|test|surefire:test|
|package|jar:jar|
|install|install:install|
|deploy|deploy:deploy|

这是一个几乎标准的绑定集合，然而，一些packaging处理这些的方式有些不同。例如，一个项目纯粹是元数据（packaging值是pom）值绑定目标到install和deploy阶段（要查看一些packaging类型的goal-to-build-phase binding的完整列表，请参阅[Lifecycle Reference](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)）

### 插件

第二种添加目标到阶段的方式是在你的项目中配置插件。插件是提供目标给Maven的生成物。此外，一个插件可能有一个或多个目标，其中每个目标表示那个插件的一种功能。例如，Compiler插件有两个目标：compile和testCompile。前者编译你的主要程序的源代码，而后者编译你的测试代码的源代码。

你可以在后面的章节中看到，插件可以包含表征绑定目标到哪一个生命周期阶段的信息。注意，只添加插件本身是没有足够信息的，你必须指定你想作为你的build的一部分运行的目标。

配置的目标将被添加到已选定的packaging中已经绑定到生命周期的目标上。如果超过一个目标被绑定到一个特定的阶段上。使用顺序是来自packaging执行时的目标先，其后是POM中配置的目标。注意你可以用&lt;executions>元素来获取更多特定目标的执行顺序控制权限。

例如，Modello插件默认将它的modello:java目标绑定到generate-sources阶段（注意：modello:java目标生成java源代码），所以为了使用Modello插件，让它从模型中生成源代码并纳入build，你应该添加下面的 内容到你的POM文件的&lt;build>节点的&lt;plugins>元素中
	
	 <plugin>
	   <groupId>org.codehaus.modello</groupId>
	   <artifactId>modello-maven-plugin</artifactId>
	   <version>1.8.1</version>
	   <executions>
	     <execution>
	       <configuration>
	         <models>
	           <model>src/main/mdo/maven.mdo</model>
	         </models>
	         <version>4.0.0</version>
	       </configuration>
	       <goals>
	         <goal>java</goal>
	       </goals>
	     </execution>
	   </executions>
	 </plugin>
	
你可能想知道为什么&lt;executions>元素在那，这就是你为什么可以在需要时运行同样的目标多次而配置文件不同。单独的执行也可以被赋予一个id，以便在继承或应用配置文件时，您可以控制是否将目标配置合并或转换为额外的执行。

当给定多个与特定阶段相匹配的执行时，它们将按照在pom中指定的顺序执行，而继承的执行将首先运行。

现在，在modello:java情况下，它只在generate-sources阶段有意义，单一些目标可以在一个以上阶段使用，而且他们可能不会合理的默认值，在那些情况下，你可以自行制定阶段。例如，将设你有你个目标dispaly:time可以在命令行打印当前年时间，你希望他在process-test-resources阶段运行来表示何时开始测试，这将会配置如下：
	 <plugin>
	   <groupId>com.mycompany.example</groupId>
	   <artifactId>display-maven-plugin</artifactId>
	   <version>1.0</version>
	   <executions>
	     <execution>
	       <phase>process-test-resources</phase>
	       <goals>
	         <goal>time</goal>
	       </goals>
	     </execution>
	   </executions>
	 </plugin>