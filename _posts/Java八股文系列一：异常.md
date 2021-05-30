# 一、异常的继承关系

![image](https://gitee.com/ellen-eager/mypic/raw/master/image/exception.png)

Throwable是所有错误和异常的父类，分为Error和Exception，Exception又可以分为RutimeException运行时异常和受检查异常。

## 1.1 Error

Error是指Java程序在启动时或运行时，出现了系统内部错误和资源耗尽错误，如果出现了Error，那么系统只能记录错误并安全退出。常见的有OutOfMemoryError，大多数是由于堆内存不够了；StackOverflowError，大多数是由于栈内存不够了。

## 1.2 Exception

Exception是可以被Java异常处理机制处理的异常，是开发中异常处理的核心，其中包括运行期异常和编译期异常(受检查异常)

### 1.2.1 RuntimeException

RuntimeException是运行期异常，可以被捕获和处理。常见的RuntimeException有：

- **NullPointerException**：空指针异常
- **ArrayIndexOutOfBoundsException**：数组下标越界异常
- **ArithmeticException** ：算数异常，比如除数为0
- **ClassCastException**：类型转换异常

### 1.2.2 编译期异常

这种异常是除RuntimeException以外的异常，类型上属于Exception类及其子类，在编译阶段Java编译器会检查此类异常并强制程序捕获和处理异常，即强制要求程序在可能出现此类异常的地方进行捕获和处理。常见的有：

- **IOException**：输入流、输出流操作出现的异常
- **InterruptedException**：Thread.sleep()出现的异常
```
    try {
        Thread.sleep(100);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
```
- **ClassNotFoundException**：找不到类型异常
```
    try {
        Class.forName("images");
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
```
- **SQLException**：操作数据库异常

# 二、异常的处理方式

Java中异常处理的方式有两种，分别是抛出异常不处理和用try/catch块捕获异常并处理。

## 2.1 抛出异常不处理

遇到异常时不进行具体的处理，而是将异常抛给调用者，由调用者根据情况处理，类似于甩锅。抛出异常不处理有两种方式：throws、throw。
- throws作用于方法声明后面，后面跟的是类，用于说明这个方法可能会抛出异常。
- throw作用于方法内部，后面跟的是一个具体的异常对象，表示明确抛出一个异常。

 **throw**
 ```
    public void service() {
        String a = "hello world";
        int index = 20;
        if (index >= a.length()) {
            throw new StringIndexOutOfBoundsException();
        }
    }
 ```
 
 **throws**
 ```
 int getId() throws Exception;
 ```
 
## 2.2 用try/catch块捕获异常并处理
```
    public void controller() {
        try {
            service();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            System.out.println("nice"); 
        }
    }

    public void service() {
        String a = "hello world";
        int index = 20;
        if (index >= a.length()) {
            throw new StringIndexOutOfBoundsException();
        }
    }
```

# 三、异常的相关问题

## 3.1 子类方法声明抛出的异常类应比父类方法抛出的异常类更小或相等
1. 如果子类方法声明抛出的异常类比父类方法抛出的异常类更大，编译不通过。
2. 如果子类方法声明抛出的异常类比父类方法抛出的异常类更大，无法实现多态。父类型的指针指向一个子类的实例，调用实例的被重写的带有抛出异常声明的方法时将catch父类抛出的异常，无法捕获子类抛出的异常。
```
class Father
{
    public void start() throws IOException
    {
        throw new IOException();
    }
}

class Son extends Father
{
    public void start() throws Exception
    {
        throw new SQLException();
    }
}
/**********************假设上面的代码是允许的（实质上是编译不通过的）***********************/
class Test
{
    public static void main(String[] args)
    {
        Father[] objs = new Father[2];
        objs[0] = new Father();
        objs[1] = new Son();

        for(Father obj:objs)
        {
        //因为Son类抛出的实质是SQLException，而IOException无法处理它。
        //那么这里的try。。catch就不能处理Son中的异常。
        //多态就不能实现了。
            try {
                 obj.start();
            }catch(IOException)
            {
                 //处理IOException
            }
         }
   }
}
```

## 3.2 再次抛出异常
如果开发了一个供给其他程序员用的子系统，执行servlet的代码可能不想知道错误的具体细节，但是又要明确地知道servlet是否有问题，那么下面这种包装技术就既可以让用户抛出子系统中的高级异常，而又不会丢失原始异常的细节。
```
        try {
            //access the database
        } catch (SQLException e) {
            Throwable se = new ServletException("database error");
            se.initCause(e);
            throw e;
        }
```
当捕获到异常时，可以用下面的语句来重新得到原始异常。
```
        Throwable e = se.getCause();
```

## 3.3 当在try块或catch块中有返回值时，finally语句块也会被执行，并且finally中的return值会覆盖在try块或catch块中的return值。

## 3.4 实现自定义的受检查异常可以继承Exception类，自定义的运行时异常可以继承RuntimeException类。
