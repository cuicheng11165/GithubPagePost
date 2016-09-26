---
layout: post
title:  ".NET线程池的介绍"
date:   2016-02-04
author: 独上高楼
category: 编程
tags: [CSharp]
---

## 线程池

当你创建一个新的线程的时候，大概需要几百微秒的时间来构建一个新的私有变量堆栈，1M的用户模式堆栈，加载一些必要的DLL。无论如何创建一个线程都是一个浪费资源的行为，线程池就是为了通过线程的重用来避免线程重建的开销。

当CLR初始化的时候，线程池里没有线程。线程池内部会维护一个请求队列，当你的程序想执行异步操作的时候，你调用的方法会请求线程池的资源进入请求队列。如果线程池没有线程，线程池会创建新的线程，当任务结束的时候，线程不会马上被销毁，而是变为闲置状态，一段时间后，会自动结束。这样避免了频繁创建销毁线程带来的开销。

有以下几种方式来进行线程池的操作:
  
* 通过Task Parallel Library（TPL），在Framework 4.0开始支持
* 通过调用ThreadPool.QueueUserWorkItem
* 通过异步委托
* 通过 BackgroundWorker

### 通过TPL来使用线程池
.NET Frameworkd 4.0 中引入了**Task**来访问线程池，可以说是ThreadPool.QueueUserWorkItem的升级版，泛型的Task&lt;TResult&gt;是匿名委托方式的升级版。新的Task可以更方便灵活的使用线程池。
你可以使用如下方式来使用Task： 
 
    static void Main()// The Task class is in System.Threading.Tasks
    {
      Task.Factory.StartNew (Go);
    }
     
    static void Go()
    {
      Console.WriteLine ("Hello from the thread pool!");
    }


Task&lt;TResult&gt; 是一个泛型版的Task： 

    static void Main()
    {
      // Start the task executing:
      Task<string> task = Task.Factory.StartNew<string>
    ( () => DownloadString ("http://www.linqpad.net") );
     
      // We can do other work here and it will execute in parallel:
      RunSomeOtherMethod();
     
      // When we need the task's return value, we query its Result property:
      // If it's still executing, the current thread will now block (wait)
      // until the task finishes:
      string result = task.Result;
    }
     
    static string DownloadString (string uri)
    {
      using (var wc = new System.Net.WebClient())
    return wc.DownloadString (uri);
    }

### 使用ThreadPool来访问线程池

使用QueueUserWorkItem可以简单的通过线程池来执行一个委托：
    static void Main()
    {
      ThreadPool.QueueUserWorkItem (Go);
      ThreadPool.QueueUserWorkItem (Go, 123);
      Console.ReadLine();
    }
     
    static void Go (object data)   // data will be null with the first call.
    {
      Console.WriteLine ("Hello from the thread pool! " + data);
    }

### 异步委托来访问线程池
通过异步委托来执行线程需要以下几步：

1. 初始化一个委托
2. 调用BeginInvoke，获得一个IAsyncResult返回值
3. 当你需要异步委托的返回值时，调用EndInvoke，传入上一步返回的IAsyncResult


示例代码：

    static void Main()
    {
      Func<string, int> method = Work;
      IAsyncResult cookie = method.BeginInvoke ("test", null, null);
      //
      // ... here's where we can do other work in parallel...
      //
      int result = method.EndInvoke (cookie);
      Console.WriteLine ("String length is: " + result);
    }
     
    static int Work (string s) { return s.Length; }

上面的例子，当执行委托结束时，也可以指定一个回调方法来通知调用者：

    static void Main()
    {
      Func<string, int> method = Work;
      method.BeginInvoke ("test", Done, method);
      // ...
      //
    }
     
    static int Work (string s) { return s.Length; }
     
    static void Done (IAsyncResult cookie)
    {
      var target = (Func<string, int>) cookie.AsyncState;
      int result = target.EndInvoke (cookie);
      Console.WriteLine ("String length is: " + result);
    }
    

