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

    * 函数通过fun关键字来声明
    * 参数名在前，参数类型在‘:’后面。变量的声明也是这样
    * 函数可以声明在文件外层，而不用在类里面
    * 可以省略没行代码结尾的分号
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

    * val(来自于value)--不可变引用。使用val声明的变量不能再初始化之后再次赋值。它对应的是Java的final变量。
    * var(来自variable)--可变变量。这种变量的值可以被改变。这种声明对应的是java的普通（非）final变量。
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
### 表示和处理选择：枚举和”when“

### 迭代事务：”while“循环和”for“循环

### Kotlin中的异常
