# 这是一篇kotlin入门的文章，记录基础语法
## kotlin是啥

### kotlin的主要特性
Kotlin是一种针对Java平台的新编程语言。Kotlin简洁、安全、务实，并且专注于与Java代码的互操作性。它几乎可以用在现在Java使用的任何地方：服务端开发、Android引用等。Kotlin可以很好的和现在的Java库和框架一起工作，而且性能水平和Java旗鼓相当。

### 基本示例
``` kotlin
//数据类
data class Person(val name: String, val age: Int? = 19)//可空类型，实参的默认值
//顶层函数
fun main(args:Array<String>){
    var persons = listOf(Person("少年帅海"), Person("青年水啊哈",28))
    val oldest = persons.maxBy { it.age ?: 0 }//lambda表示式，Elvis运算符
    //字符串模板
    println("the oldest is $oldest")
}

//输出 the oldest is Person(name=青年水啊哈, age=28) 自动生成的toString方法
```
### 主要特性
#### 目标平台：服务端、Android及任何Java运行的地方。
Java互操作性。无论是哪个库提供的API，都可以在Kotlin中使用，可以调用java的方法，继承Java类的实现和Java接口，在Kotlin上应用Java的注解，等等。Kotlin也是被编程成class文件来运行的。代码可以互相转换

    * Java转kotlin:把代码复制到Kotlin文件中，触发"Convert Java File to Kotlin File"。或者在androidSutio中选中Code->Convert Java File to Kotlin File
    * Kotlin转java:比java转kotlin麻烦一些。选中androidStudio->tools->Kotlin->Show Kotlin ByteCode,然后在右边的Kotlin ByteCode界面中选择Decompile即可。如图
![kotlin转java](https://github.com/Iambigsea/bigsea.github.io/blob/master/Kotlin%E8%BD%ACJava.png)
    
#### 静态类型
和Java一样是一种静态类型语言，并且支持类型推导
#### 函数式和面向对象
面向对象和java是一致的，函数式编程的核心概念是：

    * 头等函数，把函数当做值使用，可以用变量保存，可以当做参数传递，或者作为其他函数的返回值
    * 不可变性，使用不可变对象，这保证了它们的状态在其创建之后不能再变化
    * 无副作用，使用的是纯函数。此类函数在输入相同时，产生相同的结果

函数式编程风格的好处是：简洁、多线程安全（这个我也不太理解），还有更加容易测试（咋理解）
#### 免费并且开源
### 疑惑
