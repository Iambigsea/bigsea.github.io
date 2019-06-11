## 这是一篇入门基础的文章，其实说笔记应该更加恰当吧
### 基本要素：函数和变量
#### Hello,world!
先来看一个hello world 程序
```Kotlin
    fun main(args:Array<String>){
        println("Hello,world!")
    }
```
从这里可以看到几点特性，

    函数通过fun关键字来声明
    参数名在前，参数类型在‘:’后面。变量的声明也是这样
    函数可以声明在文件外层，而不用在类里面
    可以省略没行代码结尾的分号
#### 函数
函数的声明以关键字fun开始，后面跟着函数名，再后面是参数列表，用()装起来，多个参数用','分隔，最后面的是返回值，用':'隔开。kotlin里面的if是一个表达式，不是语句
![函数](https://github.com/Iambigsea/bigsea.github.io/blob/master/%E5%87%BD%E6%95%B0.png) <br/>
可以让上面的函数变得更加简单
```Kotlin
    fun max(a: Int, b: Int) = if (a > b) a else b
```
这里的返回类型被省略了，是因为kotlin作为一门静态语言，可以通过if表达式推导出返回值类型是Int类型。还有包含函数体的'{}'被简化成‘=’。这里只有表达式函数体才能省略返回值类型，因为代码块函数体里面可能有多个return。表达式函数体是指函数直接返回一个表达式。代码块函数体是指在函数体写在'{}'号中的<br>
#### 变量
java中声明变量是已变量类型开始，但在kotlin中，因为类型是可以省略的，所以，在kotlin中，变量声明是已关键字开始，然后是变量名称，最后可以加上类型（不加也可以）。
```kotlin
    val age = 18
    //这里是省略了类型，完成的是这样的
    val age:Int = 18
```
如果不指定变量的类型，编译器会分析初始化器表达式的值，并把它作为变量的类型，在上面的例子中，变量的初始化器42的类型是Int，那么变量类型就是这个类型。
##### 可变变量和不可变变量
声明变量的关键字有两个

    val(来自于value)--不可变引用。使用val声明的变量不能再初始化之后再次赋值。它对应的是Java的final变量。
    var(来自variable)--可变变量。这种变量的值可以被改变。这种声明对应的是java的普通（非）final变量。
定义了val变量的代码块执行期间，val变量只能进行唯一一次初始化。但是如果编译器能够确保唯一一条初始化语句会被执行，可以根据条件，使用不同的值来初始化它，例如if else 语句中。尽管val引用自身是不可变的，但是它只想的对象可能是可变的，例如引用指向对象，但是改变对象的值是可以的。<br>
##### 更简单的字符串格式化：字符串模板
```kotlin
    fun main(args:Array<String>){
        val name = if(args.size>0) args[0] else "Kotlin"
        println("Hello $name!")
    }
```
kotlin可以在字符串字面值引用局部变量，只需要在变量名称前加上字符$,这样等价于Java中的字符串链接("Hello" +name + "!"),效率一样，但是更加紧凑。当然，这样的表达式会进行静态检查，如果你试着引用一个不存在的变量，代码根本不会编译。<br>
如果要使用'$'字符，则需要对它进行转义'\$x',这样会打印$x,并不会把x解释成变量的引用。<br>
用${}的方式可以应用更加复杂的表达式,只需要把表达式放到花括号中。
```kotlin
    fun main(args:Array<String){
        println(Hello,${if(args.size >0)args[0]else "some"})
    }
```
### 类和属性
先看一个简单的类
```Java
public class Person{
    private final String name;
    public Person(String name){
        this.name = name;
    }
    public String getName(){
        return name;
    }
}
```
这种类只有数据没有其他代码通常叫做值对象，kotlin表示
```kotlin
class Person(val name:String)
```
注意从Java到Kotlin转换的过程中public修饰符消失了。在kotlin中，public是默认的可见性，所以可以省略它
#### 属性
类的概念就是把数据和处理数据的代码封装成一个单一的实体。在Java中，数据存储在字段中。通常还是私有的。如果想让类的使用者访问到数据，得提供访问器方法：一个getter，还可能有一个setter。在Java中，字段和其访问器的组合常常被叫做属性。<br>
在kotlin中，属性是头等的语言特性，完全替代了字段和访问器方法。在勒种声明一个属性和声明一个变量一样：使用val和var关键字。声明成val的属性是只读的，而var属性是可变的。
```kotlin
class Person(
    val name:String,//只读属性：生成一个字段和一个简单的getter
    var isMarrried:Boolean//可写属性：一个字段、一个getter和一个setter
)
```
基本上，当声明属性的时候，就声明了对应的访问器（只读属性有一个getter，而可写属性基友getter也有setter）。

    这里有个和java互相调用的问题，这里定义的字段是私有的，如果Java调用的收用getter和setter方法即可，如果是is开头的属性，那么设置的时候就是setxx，get的时候是isXX
    如果在Java中设置了setXX和getXX，那么会当成名称为XX的属性访问方法。
#### 自定义访问器
先简单的介绍下概念，就是可以自定义getter和setter方法。有备用字段，还有一个叫啥来着
#### Kotlin源码布局：目录和包
Java把所有的类组织成包。kotlin也有类似的概念，每个kotlin文件都能以一条package开头，而文件中定义的所有类、函数、属性都会被放到这个包中。如果其他的文件中定义的声明也有相同的包，这个文件可以直接使用它们，如果包不同，则要导入它们。导入语句放在package语句下面。<br>
Kotlin不区分导入的是类还是函数，而且，可以使用import关键字导入任意种类的声明。例如顶层函数（即不包含在类里面的函数）<br>
也可以再包名称后面加上".\*"来导入包中定义的所有声明（包括类、函数、属性）。<br>
和java不同的是java需要把类放到包结构相匹配的文件与目录中，而Kotlin则不用，可以把多个类放到同一个文件,也没有对磁盘上源文件的布局加任何限制。

    但是大多数时候遵循java的目录布局并根据包结构把源文件放到目录中，依然是一个不错的选择，特别是Kotlin和java混用的项目。
    并且应该毫不犹豫的把多个类放到同一个文件中，特别是很小的类。kotlin中的类通常很小
### 表示和处理选择：枚举和”when“
#### 声明枚举类
```kotlin    
enum class Color{
    RED,ORANGE,YELLOW,GREEN,BLUE,INDIGO,VIOLET        
}
```
kotlin声明枚举比java多了class关键字，enum是一个软关键字，只有在class前面才有意义，其他时候可以当做名称来使用。<br>
和java一样，枚举并不是值得列表：可以给枚举声明属性和方法。
```kotlin
enum class Color(val r:Int,var g:Int,var b:Int){
    RED(255,0,0),BLUE(0,0,255),GREEN(0,255,0);
    fun rgb() = (r*255 + g)*255 +b
}
fun getValue(){
    println(Color.RED.rgb())
}
```
枚举常量用的 声明构造函数、方法和属性的语句和常规类一致。如果要在枚举类中定义方法，就要使用分号把枚举常量列表和方法定义分开，这里是kotlin语法中唯一必须使用分号的地方。
#### 使用"when"处理枚举类
```kotlin
public fun hehe(color: ColorSimple): String =
    when (color) {
        ColorSimple.RED -> "1----"
        ColorSimple.GREEN, ColorSimple.VIOLET -> "2----"
        ColorSimple.YELLOW, ColorSimple.BLUE -> "3-----"
        else ->"4----"
    }
```
和if一样，when也是一个有返回值的表达式，可以多个值合并到一个分子，用','隔开，else 和switch的default一致，不需要break语句<br>
可以再"when"结构中使用任意对象，也可以使用不带参数的"when"
```kotlin
private var colors = ColorSimple.ORANGE
public fun addValue(a: Int, b: Int) =
    when {
        a + b > 0 -> "大于0"
        colors == ColorSimple.ORANGE -> "表达式中使用其他对象"
        else -> "都不符合"
    }
```
#### 智能转换：合并类型检查和转换
is: 在kotlin中，要使用is检查来判断一个变量是否是某种类型，并且如果你检测过一个变量是某种类型，后面就不需要转换它了，可以就把它当做你检查过的类型来使用，事实上编译器为了执行了类型转换，这种行为就成为智能转换。
```kotlin
interface IIs{}

class Is1:IIs{
    fun is1(){
        println("is1")
    }
}
fun isChange(i :IIs){
    if(i is Is1){
        i.is1()
    }
}
```

    智能转换只在变量经过is检查且之后不再发生变化的情况下有效。当对一个类的属性进行智能转换的时候，属性必须是val，并且没有自定义访问器。
as：使用as关键字来标识到特定类型的显示转换
```kotlin
    var n = e as Num
```
#### 用"when"代替"if"
kotlin没有三元表达式，因为if表达式有返回值。如果if分支中只有一个表达式，花括号是可以省略的。如果if分支是一个代码块，代码块中的最后一个表达式会作为结果返回。
```kotlin
fun whenIfTest(i:IIs){
    when(i){
        is As1 -> i.as1()
        is As2 -> i.as2()
        else -> throw IllegalArgumentException("unknown expression")
    }
}
```
when表达式并不仅限于检查值是否相等，这里使用了另一种when分支的形式，检查when实参的值的类型，这里应用了一次智能转换。
#### 代码块作为"if"和"when"的分支
if和when都可以使用代码块作为分支主体。这种情况下，最后一个表达式就是结果。

    规则--"代码块中最后的表达式就是结果"，在所有使用代码块并期望得到一个结果的地方成立。同样的规则对try主体和catch子语句也有效
### 迭代事务：”while“循环和”for“循环
在本章讨论的所有特性中，kotlin的迭代应该是和java最接近的。when循环和java完全一样。for循环仅以唯一一种形式存在，和java的for-each循环一直。其写法for<item> in <elements>和c#一致。
#### "while"循环
kotlin有while循环和do-while循环，语法和java一直
```kotlin
while(condition){
    ......   当condition为true时执行循环体
}
do{
    ..... 循环体会第一次无条件执行，此后，当condition为true时才执行
}while(condition)
```
#### 迭代数字：区间和数列
kotlin中没有常规的java for循环。kotlin使用了区间的概念。
    
    区间本质上就是两个值之间的间隔，这两个值通常是2个数字：一个起始值，一个结束值。使用..运算符来表示。
    kotlin中的区间是包含的或者闭合的，以为着第二个值始终是区间的一部分。使用整数区间做的最基本的事就是迭代其中所有的值。
    如果能够迭代区间中所有的值，这种区间就叫数列
```kotlin
fun regionTest(){
        var a = 1..10
        for (i in a){
            println(i)
        }
        for(i in a step 3){
            println(i)
        }
        for(i in 10 downTo 1 step 2){
            println(i)
        }
        for(i in 1 until  10 step 4){
            println(i)
        }
    }
```
step是代表带步长的数列，它容许调过一些数字，步长可以是负数，这种情况是递减的<br>
10 downTo 1是步长-1的数列。然后把步长的绝对值变成了2，，但是方向保持不变（事实上，步长被设置成了-2）<br>
until函数创建的是不包含结束值的区间。
#### 迭代map
```kotlin
var map = mutableMapOf<Char,String>()
for (c in 'A'..'F'){
    var p = Integer.toBinaryString(c.toInt())
    map[c] = p
}
for((key,value) in map){
    println("key$key = value$value")
}
```
代码运用了一个实用小技巧，根据建来访问和更新map的简明语法 map\[c]= p,可以用map\[c]来读取值，而不再需要调用get和set
#### 使用"in"检查集合和区间的成员
使用in运算符来检查一个值是否在区间中，或者它的逆运算!in,来检查这个值是否不在区间中。区间不仅限于字符，只要有一个支持示例比较的任意类（实现了java的comparable接口），就能创建这种类型的对象区间。如果是这样的区间，并不能列举出这个区间的所有对象，但是仍然可以用in运算符检查一个对象的对象是否属于这个区间
```kotlin
println(30 in 1..100)
println(30 !in 1..100)
println('k' in 'j'..'z')
println("Kotlin" in "Java".."Scala")
```
### Kotlin中的异常
kotlin的异常处理方式和java以及其他许多语言的处理方式相似。一个函数可以正常结束，也可以在出现错误的情况下抛出异常。方法的调用者能够捕获到这个异常并处理它，如果没有被处理，异常会沿着调用栈再次抛出。<br>
kotlin中异常处理语句的基本形式和java类似，抛出异常的方式也不例外
```kotlin
var i = 1
var re = if(i >0){
    i
}else{
    throw IllegalArgumentException("re must be >0")
}
```
和其他所有类一样，不必使用new 关键字来创建异常示例。和Java不同的是，kotlin中的throw的结构是一个表达式，能做为另个表达式的一部分使用。
#### "try" "catch"和"finally"
和java一样，使用带有catch和finally子句的try结构来处理异常
```kotlin
    fun readNumber(reader:BufferedReader):Int?{//不必显示的指定这个函数可能抛出的异常
        try{
            val line = reader.readLine()
            return Integer.parseInt(line)
        }catch (e:NumberFormatException){
            return null
        }finally {
            reader.close()
        }
    }
```
#### "try"作为表达式
kotlin中的try关键字就像if和when一样，引入了一个表达式，可以把它赋值给一个变量，不同于if，你总是需要用花括号把语句主体括起来。和其他语句一样，如果其主题包含多个表达式，那么整个try表达式的值就是最后一个表达式的值。
```kotlin
fun tryTest(i :String){
        var b = try{
            Integer.parseInt(i)
        }catch (e:NumberFormatException){
            null
//            return//如果是return，那么就方法返回了
        }
    }
```
如果一个try代码块执行一切正常，代码块中最后一个表达式就是结果，如果捕获到了一个异常，响应的catch代码块中最后一个表达式就是结果。
## 小结
-fun关键字用来声明函数，var关键字和val关键字分别用来声明可变变量和可读变量
-字符串模板帮助你避免繁琐的字符串连接。在变量名称前加上$前缀或者用${}包围一个表达式，来吧值注入到字符串中。
-值对象在kotlin中以简洁的方式表示
-熟悉的if现在带返回值的表达式
-when表达式类似于java的switch但功能更加强大
-在检查过变量具有某种类型之后不必显示的转换它的类型：编译器使用智能转换自动帮你完成。
-for、while、和do-while与java类似，但是for循环现在更加方便，特别是当你要迭代map的时候，又或者是迭代集合需要下标的时候
-简洁的语法1..5会创建一个区间。区间和数列容许kotlin在for循环中使用统一的语法和同一套抽象机制，并且还可以使用in运算符和!in运算符来检查值是否属于某个区间。
-kotlin中异常的处理和java非常相似，除了kotlin不要你声明函数可以抛出的异常
