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

# 参考
[Gradle脚本基础全攻略](https://blog.csdn.net/yanbober/article/details/49314255)

[Groovy脚本基础全攻略](https://blog.csdn.net/yanbober/article/details/49047515)


