### 定义类继承结构
Kotlin的接口与Java8中的相似：它们可以好汉抽象方法的定义以及非抽象方法的实现（与Java8的默认方法相似），但是它们不能包含任何状态
#### Kotlin中的接口
```kotlin
interface Clickable {
    fun showOff()
}

class Button : Clickable {
    override fun showOff() {
        println("Button : showOff")
    }
}
```
使用interface关键字来声明一个接口。Kotlin类在类名后面使用冒号来替代了java中的extends和implements关键字。和Java一样一个类可以实现任意接口，但是只能继承一个类。<br>
Kotlin中使用override修饰符是强制要求的
```kotlin 
interface Clickable {
    fun showOff(){
        println("Clickable : showOff")
    }
}

interface Focusable {
    fun setFocusable(b: Boolean)
    fun showOff() {
        println("Focusable : showOff")
    }
}

class Button :Clickable,Focusable{
    override fun setFocusable(b: Boolean) {
    }

    override fun showOff() {
        super<Focusable>.showOff()
        super<Clickable>.showOff()
    }
}
```
接口的方法可以有一个默认实现，与Java8不同的是，Java8中需要在是线上标注defalt关键字。但是kotlin中不需要。<br>
在一个类要实现的多个接口有同一个实现了的方法，如果要调用父类的方法，那么用supper<xx>.xx() <br>
如果是Java类要实现改接口，那么接口的所有方法都要实现
    
#### open、final和abstract修饰符：默认为final
Java的类和方法都是默认open的，而Kotlin中的默认都是final的，因为对基类的修改会导致子类不正确的行为，就是所谓的脆弱的基类问题。<br>
如果想容许创建一个类的子类，需要使用open修饰符来标示这个类，此外，需要给每一个可以被重写的属性或方法添加open修饰符
如果重写了一个基类或者接口的成员，重写的成员默认是open的，如果需要阻止子类重写你的实现，可以显示的将重写的成员标注为final
```kotlin
open class RichButton : Clickable {
    fun disable() {}
    open fun animate() {}
    open fun click() {}
}
open class RichButton1 : Clickable {
    final override fun showOff() {}
}
```
#### 可见性修饰符：默认为public

#### 内部类和嵌套类：默认是嵌套类
#### 密封类：定义受限的类继承结构
### 声明一个带非默认构造方法或属性的类
#### 初始化类：主构造方法和初始化语句块
#### 构造方法：用不同的方式来初始化父类
#### 实现在接口中声明的属性
#### 通过getter或setter访问支持字段
#### 修改访问器的可见性
### 编译器生成的方法：数据类和类委托
#### 通用对象方法
#### 数据类：自动生成通用方法
#### 类委托：使用by关键字
### “object”关键字：将声明一个类与创建一个实例结合起来
#### 对象声明：创建单例易如反掌
#### 伴生对象：工厂方法和静态成员的地盘
#### 作为普通对象使用的伴生对象
#### 对象表达式：改变写法的匿名内部类

- Kotlin的接口与Java的相似，但是可以包含默认实现（Java从第八版才开始支持）和属性
- 所有的声明默认都是final和public的
- 要想使声明不是final的，将其标记为open
- internal声明在同一模块中可见
- 嵌套类默认不是内部类。使用inner关键字来存储外部类的引用。
- sealed类的子类只能嵌套在自身的声明中（kotlin1.1容许将子类放置在同一个文件的任意地方）
- 初始化语句块和从构造方法为初始化类实例提供了灵活性
- 使用field标识符在访问器方法体中引用属性的支持字段
- 数据类提供了编译器生成的equals、hashcode、toString、copy和其他方法
- 类委托帮助避免在代码中出现许多相似的委托方法
- 对象声明是Kotlin中定义单例的方法的地方
- 伴生对象（与包级别函数和属性一起）替代了Java静态方法和字段定义
- 伴生对象和其他对象一样，可以实现接口，也可以拥有有扩展函数和属性。
- 对象表达式是Kotlin中针对Java匿名内部类的替代品，并增加了诸如实现多个接口的能力和修改在创建对象的作用域中定义的变量的能力等功能。
