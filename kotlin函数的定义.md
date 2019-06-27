### 在kotlin中创建集合
```kotlin
var list = arrayListOf<Int>(1, 3, 4)
println(list.javaClass)//class java.util.ArrayList
var set = hashSetOf<Int>(2, 3, 4)
println(set.javaClass)//class java.util.HashSet
var map = hashMapOf<Int, Int>(1 to 3, 2 to 3, 4 to 5)
println(map.javaClass)//class java.util.HashMap
```
- kotlin中用的集合是标准的java集合类。但尽管和java的集合类完全一致，但kotlin的集合还封装了其他的功能，比如过去列表的最后一个元素 list.last(),获取集合的最大值，list.max()
- 注意，to并不是一个特殊的结构，而是一个普通的函数。
### 让函数更好调用
先来看一段函数
```kotlin
fun test() {
    var list = arrayListOf<Int>(1, 2, 3, 4, 5)
    val joinToString = joinToString(list, ";", "[", "]")
    println(joinToString)//[1;2;3;4;5]
}
fun <E> joinToString(collection: Collection<E>, separator: String, prefix: String, postfix: String): String {
    var result = StringBuffer(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
            result.append(element)
        }
     result.append(postfix)
     return result.toString()
}
```
很容易就看出来这个函数是为了自定义打印集合里的元素的方法
#### 命名参数
 我们要关注的第一个问题就是函数的可读性。举个例子，来看看joinToString方法的调用:
 ```kotlin
 joinToString(list, " ", " ", ".")
 ```
 我们很难看出那个String对应的是哪个参数，虽然可以通过ide，或者从调用代码来查看，但是这依然很隐晦。在java中可以通过写注释的方法来知名参数的名称<br>
 在kotlin中可以做的更优雅
 ```kotlin
joinToString(list, separator = ";",postfix =  "]",prefix =  "[")//prefix和postfix的位置可以调换
joinToString(list, separator = ";","]",prefix =  "[")//error，因为separator已经显示的标明了参数的名称
 ```
当调用kotlin定义的函数时，可以显示的标明一些参数的名称。<br>

    不幸的是，当调用java函数的时候，不能采用命名参数。
#### 默认参数值
java中一个普遍存在的问题是重载的函数实在太多了，这些重载，原本是为了向后兼容，方便这些api的使用，这样会导致很多的重复，而且调用的时候容易不知道调用哪个。<br>
在kotlin中，可在声明函数的时候，指定参数的默认值，这样就可以避免创建重载的函数。
```kotlin
fun <E> joinToString(
    collection: Collection<E>,
    separator: String = ",",
    prefix: String = "[",
    postfix: String = "]"
): String
```
现在在调用的时候就可以用所有的参数来调用函数或者省略部分函数
```kotlin
joinToString(list,",","|","|")//|1,2,3,4,5|
joinToString(list,",")//[1,2,3,4,5]
joinToString(list)//[1,2,3,4,5]
```
- 当使用常规的调用语法时，必须按照函数声明中定义的参数顺序来给定参数，能省略的只有排在某位的参数
- 如果使用命名参数，可以省略中间的一些参数，也可以按照你想要的任意顺序给定你需要的参数

        考虑到java中没有默认值的概念，当java调用kotlin函数的时候，必须显示的指定所有参数，如果需要能对java调用者更简便，
        可以使用@JvmOverloads注解他，这个指示编译器会生成java的重载函数，从最后一个开始省略每个参数
### 消除静态工具类:顶层函数和属性
Kotlin中不需要静态的util类，直接把函数放到代码文件的顶层，不用从属任何类。这些类仍然是包内的成员，如果要在包外访问他，则需要import。
```kotlin
Join.kt
fun joinString(s:String){
    println(s)
}
```
在java调用的时候，会编译成下面这样
```java
public final class JoinKt {
   public static final void joinString(@NotNull String s) {
      System.out.println(s);
   }
}
```
因此，在java中调用时，导入JoinK他，然后调用JoinKt.joinString()即可<br>
如果不用自动生成的函数名的话，可以用 \@file:Jvm("xxx")的方式，这样就会生成自己明明的函数名"
#### 顶层属性
和函一样，属性也可以放在文件顶层。默认情况下，顶层属性和其他任意的属性一样，是通过访问器暴露给Java使用的(如果是val就只有一个getter方法，如果是var则有setter和getter方法)。如果想要把常量以public static final的方式暴露给java使用，则需要const来修饰。但是const只能修饰基本数据类型还有String类型的
#### 给别人的类添加方法：扩展函数和扩展属性
kotlin有一大特色，就是可以平滑的和现有代码集成。<br>
扩展函数非常简单，它就说类的一个成员函数，不过定义在类的外面。
```kotlin
//String 就是接收者类型         this是接收者对象
fun String.lastChar(): Char = this.get(this.length - 1)

fun Test() {
    //"extendFun"是接受者对象，String 是接收者类型
    var s = "extendFun"
    println(s.lastChar())
}
```
- 你所有要做的，就是把你要扩展的类或者接口名称，放到即将添加的函数前面。这个类的名称被称为接收者类型，用来调用这个扩展函数的对象，叫做接收者对象。<br>
- 调用这个方法也很简单，在导入包了之后，直接像调用类的成员函数那样即可。<br>
- 从某种意义上说，你已经为String类添加了自己的方法。即使字符串不是代码的一部分，也没有类的源码，你仍然可以再自己的项目中根据需要去扩展方法。<br>
- 不管类是用Java，kotlin或者是Groovy的其他JVM语言编写的，只要是它会编译为Java类，就可以为这个类添加自己的扩展。<br>
- 在扩展函数中，可以像其他成员函数一样用this，而且也可以像普通的成员函数一样，省略它。
- 和在类内部定义的方法不同的是，扩展函数不能访问私有的或者是受保护的成员。
#### 导入和扩展函数
定义的扩展函数需要导入之后才能使用。
```kotlin
import com.xx.xx.lastChar
import com.xx.xx.*
```
- 可以用导入类一样的语法来导入单个的函数，也可以用\*来导入
- 可以使用关键字as来修改导入的类或者函数名称，房子多个扩展函数重名。对于一般的函数可以用全名的方式类指出这个类或者函数，但是对于扩展函数，只能用as的方法
```kotlin
import com.xx.xx.lastChar as lalal
```
#### 从Java中调用扩展函数
- 事实上扩展函数是静态函数，它把调用对象作为了第一个参数。调用扩展函数，不会创建适配对象或者任何运行时的额外开销。
- 这使得Java调用kotlin的扩展函数变得非常简单，调用这个静态函数，把就守着对象作为第一个参数穿进去即可。和顶层函数一样，包含这个函数的Java类名，是有这个函数声明文件决定的
```kotlin
lastChar函数在ExtendFunDemo文件中
ExtendFunDemoKt.lastChar("adfads");
```
#### 作为扩展函数的工具函数
扩展函数无非就是静态函数的一个高效的语法糖，扩展函数的静态性质决定了扩展函数不能被子类重写。
#### 不可重载的扩展函数
```kotlin
open public class View(){
    open fun onClick(){
        println("view onClick")
    }
}

class Button():View(){
    override fun onClick(){
        println("botton onClick")
    }
}

fun View.show(){
    println("view show")
}

fun Button.show(){
    println("button show")
}

fun test(){
    var v:View = Button()
    v.onClick()//botton onClick
    v.show()//view show
}
```
当调用一个类型为View的变量的时候show方法的时候，对应的扩展函数会被调用，尽管实际上这个变量是Button对象，因为扩展函数将会在java中被编译为静态函数，同事接受者会作为第一个参数。
```java
View v = new Button();
//OverRideExtendMethod是文件名
OverRideExtendMethodKt.show(v)
```
- 如果一个类的成员函数和扩展函数有相同的签名，成员函数往往会被优先使用
#### 扩展属性
扩展属性提供了一种语法，用来扩展类的API，可以用来访问属性，用的是属性语法，而不是函数的语法。尽管它被称为属性，但是他们可以没有任何状态，因为没有合适的地方存储它，不可以给现有的java对象示例添加额外的字段。这块比较迷，先不管它
### 处理集合：可变参数、中缀调用和库的支持
这一节将会展示Kotlin标准库中用来处理集合的一些方法。另外也会涉及几个相关的语言特性：
- 可变参数的关键字vararg，可以用来声明一个函数函数，将可能有任意数量的参数
- 一个中缀表示法，当你在调用一些只有一个参数的函数时，使用它会让代码更加简练
- 结构声明，用来吧一个单独的组合值展开到多个变量中
#### 扩展Java集合的API

#### 可变参数：让函数支持任意数量的参数

#### 键值对的处理：中缀调用和结构声明

#### 三重引号
在三重引号的字符串中，不需要对任何字符进行转义，包括折行，反斜杠。
#### 让你的代码更加整洁：局部函数和扩展
kotlin可以再函数中嵌套函数，这种被称为嵌套函数。例如下面

扩展函数也可以被声明为局部函数，局部函数可以多层嵌套，但是这边容易让人费解，一般不建议多层嵌套

## 总结
 - kotlin没有定义自己的集合类，而是在java集合类的基础上提供了更丰富的API
 - Kotlin可以给函数参数定义默认值，这样大大降低了重载函数的必要性，而且明明参数让多参数的调用更加易读
 - kotlin允许更灵活的代码结构：函数和属性都可以直接在文件中声明，而不仅仅是在类中作为成员。
 - kotlin可以用扩展函数和属性来扩展任何类的API，包括在外部库中定义的类，而不需要修改其源码，也没有运行时开销。
 - 中缀调用提供了处理单个参数的，类似调用运算符的简明语法。
 - 三重引号的字符串提供了一种简洁的方式，解决了原生在Java中需要进行大量啰嗦大转移和字符串连接的问题。
 - 局部函数帮助你保存代码整洁的同时，避免重复。
 
 
 
 
 
 
