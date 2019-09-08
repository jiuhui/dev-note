# gradle学习

## 1、gradle任务

1、定义任务输出hello world

```groovy
task hello {
   doLast {
      println 'hello world'
   }
}
```

```bash
$ gradle -q hello
hello world

```

```groovy
 task upper{
	doLast{
		String expString = "hello wrod"
		println "original = "+expString
		println "upper case is ="+expString.toUpperCase()
	}
	doLast{
		println "=========="
	}
 }
```

```
$ gradle -q upper
original = hello wrod
upper case is =HELLO WROD
==========
```

2、任务的相互依赖

```groovy
task hello doLast {
    println 'Hello world!'
}
task intro(dependsOn: hello) doLast {
    println "I'm Gradle"
}
```

```bash
$ gradle -q intro
Hello world!
I'm Gradle
```

```groovy
task taskY doLast {
   println 'taskY'
}
task taskX doLast {
   println 'taskX'
}
taskY.dependsOn taskX

```

```bash
$ gradle -q taskY
taskX
taskY
```

3、模糊匹配的依赖

```groovy
task taskX doLast {
   println 'taskX'
}

taskX.dependsOn {
   tasks.findAll { 
     task -> task.name.startsWith('lib') 
   }
}
task lib1 doLast {
   println 'lib1'
}
task lib2 doLast {
   println 'lib2'
}
task notALib doLast {
   println 'notALib'
}
```

```bash
$ gradle -q taskX
lib1
lib2
taskX
```

4、用description添加描述

```groovy
task hello doLast {
	description 'this is first hello world'
   println "hello world"
}
```

```bash
$ gradle -q hello
hello world
```

5、doFirst

```groovy
task compile doLast{
	println 'we are doing the compile'
}

compile.doFirst{
	println 'do first'
}
```

```bash
$ gradle -q compile
do first
we are doing the compile
```

6、通过抛出异常跳过任务

```groovy
// 这是一个没有跳过的依赖
task taskX doLast{
	println 'taskx'
}
task taskY (dependsOn:taskX) doLast{
	println 'taskY'
}
task taskZ doLast{
	println 'taskZ'
}
taskY.dependsOn taskZ
==============================
$ gradle -q taskY
taskx
taskZ
taskY

// 这是跳过taskX的
task taskX doLast{
	throw new StopExecutionException()
	println 'taskx'
}
task taskY (dependsOn:taskX) doLast{
	println 'taskY'
}
task taskZ doLast{
	println 'taskZ'
}
taskY.dependsOn taskZ
==================================================
$ gradle -q taskY
taskZ
taskY
```

7、gradle task执行会打印执行的汇总

```groovy
task task1 doLast {
   println 'compiling source #1'
}

task task2(dependsOn: task1) doLast {
   println 'compiling unit tests #2'
}

task task3(dependsOn: [task1, task2]) doLast {
   println 'running unit tests #3'
}

task task4(dependsOn: [task1, task3]) doLast {
   println 'building the distribution #4'
}

```

```bash
$ gradle task4

> Task :task1
compiling source #1

> Task :task2
compiling unit tests #2

> Task :task3
running unit tests #3

> Task :task4
building the distribution #4

BUILD SUCCESSFUL in 688ms
4 actionable tasks: 4 executed

```

8、用-x排除任务

gradle task4 -x task1

9、gradle -q projects 命令来列出所选项目及其子项目的项目层次结构

10、gradle -q tasks --all 列出属于多个项目的所有任务

## 2、使用gradle构建java项目

新建一个项目文件夹  执行 gradle init  --type java-application  默认会生成src等目录

https://guides.gradle.org/building-spring-boot-2-projects-with-gradle/官方文档地址  创建spring boot项目的

build.gradle文件中增加：

```groovy
apply plugin: 'java'
```

意思是应用一个java插件。

指定版本和java编译的版本

```groovy
version = 0.1.1
sourceCompatibility = 1.8

```

如果是可执行的文件指定java的main方法入口

```groovy
jar {
   manifest {
      attributes 'Main-Class': 'com.yiibai.main.Application'
   }
}

```

执行  gradle build

```bash
$ gradle build

BUILD SUCCESSFUL in 1s
1 actionable task: 1 executed
```

