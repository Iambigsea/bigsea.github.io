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
#### 消除静态工具类:顶层函数和属性



## 总结
 
