---
layout: post
title: "我们日常生活中的 iBeacon"
date: 2018-06-06 11:25:56 
description: "kotlin"
tag: Android
---


### Tip 1,覆盖冲突
&nbsp;&nbsp;在kotlin中，实现继承通常遵循如下规则：如果一个类从它的直接父类继承了同一个函数的多个实现，那么它必须重写这个函数并且提供自己的实现。为表示使用父类中提供的方法，我们使用` super ` 关键字。

```
interface ProjectService {


    fun print(){
        println("I am a project")
    }

}
```

```
interface MoudleService {


    fun print(){
        println("I am a moudle")
    }
}

```

```
class ProjectServiceImp() : ProjectService, MoudleService {

    //由于实现的两个接口都有print方法，所以当前类必须复写这个方法
    //并且，不能直接使用super.print()调用，因为编译器不知道要调用哪个接口里面的print方法，会报错
    override fun print() {
        super<ProjectService>.print()
        super<MoudleService>.print()
    }

}
```

### Tip 2，接口中的属性
&nbsp;&nbsp;在接口中声明的属性可以是抽象的，或者是提供了访问器的实现；
```
interface ProjectService {


    val owner: String //抽象属性，类必须要实现

    val own: String
        get() = "Bruce"  //提供了访问器的实现



    fun print(){
        println("I am a project")
    }

}
```

### Tip 3，kotlin中object对象
&nbsp;&nbsp;kotlin中没有“静态属性和方法”,但也提供了类似的实现方式，使用` object `关键字；

```
//静态类
object StaticObject {

    //静态变量
    val userName: String = "Bruce"

    //静态方法
    fun action(){
        println("I am Bruce")
    }
}

```

```
fun main(args: Array<String>) {
    println(StaticObject.userName)
    StaticObject.action()
}
```

### Tip 4, 伴生对象
&nbsp;&nbsp;Kotlin中还提供了“伴生对象（companion object）”,并且一个类只能有一个伴生对象。伴生对象的初始化时在相应的类被加载解析时，与Java静态初始化器的语义相匹配。伴生对象的成员看起来像其他语言的静态成员，在运行时任然是真实对象的实例成员。而且还可以实现接口；

```
class ProjectServiceImp()  {

  //伴生对象定义
    companion object {
        val a = "Bruce"

        fun action(){
            println("我是伴生对象")
        }
    }

}
```
```
fun main(args: Array<String>) {

    //调用伴生对象属性和方法
    ProjectServiceImp.a
    ProjectServiceImp.action()
}
```



### Tip 5, 密封类
&nbsp;&nbsp;密封类的定义，就是在 ` class `关键字前面加上 ` sealed `修饰符。密封类的目的，就是为了限制类的继承结构，将一个值限制在有限集中的类型中，而不能有任何其他的类型。密封类的所有子类都必须与密封类在同一个文件中声明。
```
sealed class SealedClass

class Sub1SealedClass : SealedClass()

class Sub2SealedClass : SealedClass()
```

### Tip 6, 数据类
&nbsp;&nbsp;数据类的声明，在 ` class ` 前面加上 ` data `操作符。
```
data class DataClass(val name: String)
```
&nbsp;&nbsp;构造函数上面的`val/var`是必须要加上的，因为编译器要把主构造函数中声明的所有属性，自动生成一下函数：
```
equals()/hashCode()
toString()
copy()
```
&nbsp;&nbsp;如果我们自定义了这些函数，或者继承父类重写了这些函数，编译器就不会再去生成。
&nbsp;&nbsp;数据类有以下限制 ：
* **主构造函数需要至少有一个参数**
* **主构造函数的所有参数需要标记为val 或 var**
* **数据类不能是抽象，开放，密封或者内部的**


# 类委托待添加 ...

### Tip 7, 可观察属性委托
&nbsp;&nbsp;我们把属性委托给 ` Delegates.observable `函数，当属性值被重新赋值的时候，触发其中的回调函数 ` onChange `
```
class DelegatePropertiesDemo {

    var content: String by Delegates.observable(
            "",onChange = {
        property, oldValue, newValue ->

        println("newValue=${newValue}, oldValue=${oldValue}")
    })

}
```

```
fun main(args: Array<String>) {
    val demo = DelegatePropertiesDemo()
    demo.content = "Bruce"
    demo.content = "Jack"

}
```
输出结果：
```
newValue=Bruce, oldValue=
newValue=Jack, oldValue=Bruce
```

### Tip 8, 可否决属性委托
&nbsp;&nbsp;当我们把属性委托给这个函数是，可以通过 `onChange`函数的返回值是否为true，来选择属性的值是否需要改变。
```
class DelegatePropertiesDemo {

    var name: String by Delegates.vetoable("默认值", onChange = {
        property, oldValue, newValue ->  false
    })

}
```
```

fun main(args: Array<String>) {
    val demo = DelegatePropertiesDemo()

    demo.name = "老于"
    println(demo.name)

}
```
结果：  默认值

&nbsp;&nbsp;可以看出，当onChange函数返回值是false的时候，对属性name的赋值都没有生效，依然是原来的默认值。


### Tip 9, 非空属性委托

```
var age: String by Delegates.notNull()
```
跟 ` ? `可选符号有点类似；

### Tip 10，高阶函数引用
&nbsp;&nbsp;高阶函数是将函数用作参数或返回值的函数。
```
fun isOdd(x: Int): Boolean {
    return x % 2 == 1
}
```
```
fun main(args: Array<String>) {

  val list = listOf(3,5,2,6)

  //通过 :: 来引用一个函数
  val result = list.filter(::isOdd)

  println("过滤完的数组结果：${result}")
}
```
结果：
过滤完的数组结果：[3, 5]

### Tip 11，协程小例子
```
fun test2(){

    launch {
        doSomething()

        //下面的打印会挂起
        println("1，主协程挂起，这句话最后执行")
    }

    //防止主线程退出
    while (true){}
}


fun test1() = runBlocking{

    val job = launch(CommonPool) {

        doSomething()

        //下面的打印会挂起
        println("1，主协程挂起，这句话最后执行")
    }

    //持续等待，直到子协程执行完成
    job.join()

}

suspend fun doSomething(){

    //await，让异步协程等待执行结束
     val a = async {
         //模拟耗时操作
         println("2，协程开始执行")
         delay(5000L)
         getNum()
     }.await()

    println("3，dosomething执行成功 ${a}")

}

fun getNum():Int = 100
```


```
fun main(args: Array<String>) = runBlocking {

    test1()
}

执行结果：
2，协程开始执行
3，dosomething执行成功 100
1，主协程挂起，这句话最后执行

```
```
fun main(args: Array<String>) = runBlocking {

    test2()
}

执行结果：
2，协程开始执行
3，dosomething执行成功 100
1，主协程挂起，这句话最后执行
```
