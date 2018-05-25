
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
