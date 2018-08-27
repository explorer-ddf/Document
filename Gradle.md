# Gradle

## 基本知识

### Gradle的三种主要对象

- Project对象：每个build.gradle会转换成一个Project对象。

- Gradle对象：构建初始化时创建，整个构建执行过程中只有这么一个对象，一般很少去修改这个默认配置脚本。

- Settings对象：每个settings.gradle会转换成一个Settings对象。

### 一个项目在构建时的流程
1. 为当前项目创建一个Settings类型的实例。

2. 如果当前项目存在settings.gradle文件，则通过该文件配置刚才创建的Settings实例。

3. 通过Settings实例的配置创建项目层级结构的Project对象实例。

4. 最后通过上面创建的项目层级结构Project对象实例去执行每个Project对应的build.gradle脚本。

### 生命周期
Gradle构建系统有自己的生命周期，初始化、配置和运行三个阶段。

1.  初始化阶段，会去读取根工程中setting.gradle中的include信息，决定有哪几个工程加入构建，创建project实例，比如下面有三个工程：

	```
	include ':app', ':lib1', ':lib2'
	```

2. 配置阶段，会去执行所有工程的build.gradle脚本，配置project对象，一个对象由多个任务组成，此阶段也会去创建、配置task及相关信息。

3. 运行阶段，根据gradle命令传递过来的task名称，执行相关依赖任务。


### 自定义任务task属性

```
task myTask {
    ext.myProperty = "myValue"
}

task printTaskProperties << {
    println myTask.myProperty
}
```

### Project对象API

对于构建脚本中每个project其实Gradle都创建了一个 Project类型的对象来关联，当构建脚本执行时它会去配置所关联的Project对象；构建脚本中每个被调用的方法和属性都委托给了当前Project对象。

Eg：下面两个println语句的输出是一样的

```
println name

println project.name
```

### Gradle变量声明

在Gradle脚本中有两种类型的变量可以声明，如下：

- 局部变量
- 扩展变量

局部变量使用关键字def声明，它只在声明的地方可见，如下：

```
def dest = "dest"

task copy(type: Copy) {
      form "source"
      into dest

}
```

在Gradle中所有被增强的对象可以拥有自定义属性（譬如projects、tasks、source sets等），使用ext扩展块可以一次添加多个属性。如下：

```
apply plugin: "java"

//向Project对象添加两个扩展属性，当这些扩展属性被添加后，它们就像预定义的属性一样可以被读写。
ext {
    springVersion = "3.1.0.RELEASE"
    emailNotification = "build@master.org"
}

sourceSets.all { ext.purpose = null }

sourceSets {
    main {
        purpose = "production"
    }
    test {
        purpose = "test"
        }
    plugin {
        purpose = "production"
    }
}

task printProperties << {
    println springVersion
    println emailNotification
    sourceSets.matching { it.purpose == "production" }.each { println it.name}
} 
```

### gradle中的properties文件

在Android Studio 创建一个项目的时候，rootProject下面会生成gradle.properties和local.properties文件，其中，gradle.properties中的内容不需要显示调用就可以直接在build.gradle中进行使用。

在同一个项目中，build.gradle可以直接获取其同级或者其父级(父级也要有build.gradle)的properties文件。

[https://blog.csdn.net/mr_tony/article/details/79122936](https://blog.csdn.net/mr_tony/article/details/79122936)

### buildscript方法

Android项目中，根工程默认的build.gradle应该是这样的：

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```
buildscript方法的作用是配置脚本的依赖，而我们平常用的compile是配置project的依赖。

### 在utils.gradle中定义了一些函数，在其他build.gradle中调用这些函数

 ```
//utils.gradle中定义了一个获取AndroidManifests.xmlversionName的函数  
def  getVersionNameAdvanced(){  
  下面这行代码中的project是谁？  
   defxmlFile = project.file("AndroidManifest.xml")  
   defrootManifest = new XmlSlurper().parse(xmlFile)  
   returnrootManifest['@android:versionName']    
}  
//现在，想把这个API输出到各个Project。由于这个utils.gradle会被每一个Project Apply，所以  
//我可以把getVersionNameAdvanced定义成一个closure，然后赋值到一个外部属性  
  下面的ext是谁的ext？  
ext{ //此段花括号中代码是闭包  
    //除了ext.xxx=value这种定义方法外，还可以使用ext{}这种书写方法。  
    //ext{}不是ext(Closure)对应的函数调用。但是ext{}中的{}确实是闭包。  
    getVersionNameAdvanced = this.&getVersionNameAdvanced  
 }  
 ```

上面代码中有两个问题：

- project是谁？
-  ext是谁的ext？

这两个问题归结到一起，其实就是：加载utils.gradle的Project对象和utils.gradle本身所代表的Script对象到底有什么关系？

Project和utils.gradle对于的Script的对象的关系是：

- 当一个Project apply一个gradle文件的时候，这个gradle文件会转换成一个Script对象。这个，相信大家都已经知道了。
- Script中有一个delegate对象，这个delegate默认是加载（即调用apply）它的Project对象。但是，在apply函数中，有一个from参数，还有一个to参数（参考图31）。通过to参数，你可以把delegate对象指定为别的东西。
- delegate对象是什么意思？当你在Script中操作一些不是Script自己定义的变量，或者函数时候，gradle会到Script的delegate对象去找，看看有没有定义这些变量或函数。


现在你知道问题1,2和答案了：

问题1：project就是加载utils.gradle的project。由于posdevice有5个project，所以utils.gradle会分别加载到5个project中。所以，getVersionNameAdvanced才不用区分到底是哪个project。反正一个project有一个utils.gradle对应的Script。

问题2：ext：自然就是Project对应的ext了。此处为Project添加了一些closure。那么，在Project中就可以调用getVersionNameAdvanced函数了

## Groovy

### 基本概念

Groovy是一种动态语言，它和Java类似，都在Java虚拟机中运行。当运行Groovy脚本时它会先被编译成Java类字节码，然后通过JVM虚拟机执行这个Java字节码类。

### 脚本与类（脚步的本质）

demo.groovy内容如下：

```
println 'Groovy world!'
```

被编译成Java类字节码在反编译后java文件：

```
import org.codehaus.groovy.runtime.InvokerHelper
class Main extends Script {                     
    def run() {                                 
        println 'Groovy world!'                 
    }
    static void main(String[] args) {           
        InvokerHelper.runScript(Main, args)     
    }
}
```

可以看见，上面我们写的groovy文件编译后的class其实是Java类，该类从Script类派生而来（查阅API）；可以发现，每个脚本都会生成一个static main方法，我们执行groovy脚本的实质其实是执行的这个Java类的main方法，脚本源码里所有代码都被放到了run方法中，脚本中定义的方法（该例暂无）都会被定义在Main类中。

### 闭包

#### 定义一个闭包：
{ [closureParameters -> ] statements }

#### 闭包的调用

- 闭包对象.call(参数)

- 闭包对象(参数)


#### 当闭包作为闭包或方法的最后一个参数时我们可以将闭包从参数圆括号中提取出来接在最后，如果闭包是唯一的一个参数，则闭包或方法参数所在的圆括号也可以省略 ：

```
def myFun(int num, String str, Closure closure){  
      closure()  
}  

myFun(1, "groovy", {  
   println"hello groovy!"  
})

//等价于上面的调用
myFun 1, "groovy",  {  
   println"hello groovy!"  
}

```

# 参考
[构建神器Gradle](http://jiajixin.cn/2015/08/07/gradle-android/)

[Gradle脚本基础全攻略](https://blog.csdn.net/yanbober/article/details/49314255)

[Groovy脚本基础全攻略](https://blog.csdn.net/yanbober/article/details/49047515)

[Gradle依赖管理](https://blog.csdn.net/pkaq_/article/details/53906668)

[美团Android自动化之旅—适配渠道包](https://tech.meituan.com/mt-apk-adaptation.html)

[使用Gradle生成一个App的不同版本，且可以同时安装在一个手机上](http://hugozhu.myalert.info/2014/08/03/50-use-gradle-to-customize-apk-build.html)

[深入理解Android之Gradle
](https://blog.csdn.net/innost/article/details/48228651#)