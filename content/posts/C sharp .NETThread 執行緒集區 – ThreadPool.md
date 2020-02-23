+++
title = "[C#.NET][Thread] 執行緒集區 – ThreadPool"
author = "Hzllaga"
date =  2018-09-06
description = "執行緒集區會維持一個最小閒置執行緒數目，當使用ThreadPool.QueueUserWorkItem方法時會喚醒正在休眠的執行緒，這樣一來就比我們不斷直接使用Thread類別重新建立執行緒還要來的節省資源一點，執行緒集區內的執行緒工作滿檔時，會再自動增加執行緒的數量。"
categories = ["转载"]
tags = [".NET"]
+++
看了安德魯的文章，可以讓人裡解ThreadPool背後在玩的把戲 ThreadPool [實作 #1. 基本概念](https://columns.chicken-house.net/2007/12/14/threadpool-%E5%AF%A6%E4%BD%9C-1-%E5%9F%BA%E6%9C%AC%E6%A6%82%E5%BF%B5/)，絕對會讓您收獲不少。<!--more-->

執行緒集區會維持一個最小閒置執行緒數目，當使用[ThreadPool.QueueUserWorkItem](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.threadpool.queueuserworkitem?redirectedfrom=MSDN&view=netframework-4.8#overloads)方法時會喚醒正在休眠的執行緒，這樣一來就比我們不斷直接使用Thread類別重新建立執行緒還要來的節省資源一點，執行緒集區內的執行緒工作滿檔時，會再自動增加執行緒的數量。

另外MSDN有提到，

如果應用程式受到突然增加的活動影響 (例如，大量佇列的執行緒集區工作)，請使用 [SetMinThreads](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.threadpool.setminthreads?redirectedfrom=MSDN&view=netframework-4.8#System_Threading_ThreadPool_SetMinThreads_System_Int32_System_Int32_) 方法增加最小閒置執行緒的數目。否則，建立新閒置執行緒時的內建延遲可能會造成瓶頸。

這表示若有需要大量執行緒數量要增加時，反而會因為這些延遲而拖垮系統，保哥也是曾經受到它的荼毒 ，[使用 ThreadPool 時應注意預設同時執行的 Thread 數量](https://blog.miniasp.com/post/2009/04/10/The-thread-pool-worker-threads-per-available-processor) 。

[ThreadPool.QueueUserWorkItem](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.threadpool.queueuserworkitem?redirectedfrom=MSDN&view=netframework-4.8#overloads)方法需要傳入一個[WaitCallback委派](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.waitcallback?redirectedfrom=MSDN&view=netframework-4.8)，這個委派的簽名是
{{< highlight csharp>}}
public delegate void WaitCallback (
    Object state
)
{{</  highlight  >}}
所以只要建立跟它有相同簽名的方法，就可以建立WaitCallback委派了。

ThreadPool.QueueUserWorkItem方法只有兩種多載，除了固定要傳WaitCallback委派，另一個參數就是object物件，所以表示我們可以傳很大的東西進去DoWorker(object number)方法裡
{{< highlight csharp >}}
static void Main(string[] args)
{
    ThreadPool.QueueUserWorkItem(new WaitCallback(DoWorker), i);
    ThreadPool.QueueUserWorkItem(new WaitCallback(DoWorker));
}
static void DoWorker(object number)
{
    //TODO..
}
{{</  highlight  >}}

## 先來看看簡單的使用範例

### 傳入一個參數
{{< highlight csharp >}}
static void Main(string[] args)
{
    for (int i = 0; i < 10; i  )
    {
        ThreadPool.QueueUserWorkItem(new WaitCallback(DoWorker), i);
    }
    Console.WriteLine("Main thread exit");
    Console.ReadKey();
}
static void DoWorker(object number)
{
    Thread t = Thread.CurrentThread;
    Console.WriteLine("No.{0}-Thread[{1}]:{2}", number, t.ManagedThreadId, t.ThreadState);
    Thread.Sleep(1000);
}
{{</  highlight  >}}
因為閒置的執行緒被喚醒，所以可以看到，有許多重複的執行緒會出現

![](https://cdn.wtfsec.org/img/20200223172435.png)

### 傳入多個參數

若需要傳入不只一個參數，我們可以利用類別(class)或結構(struct)來傳入，當然你也可以傳入集合…反正你想的到的資料型態什麼都能傳，只要再DoWorker方法裡記得轉型就可以了。
{{< highlight csharp >}}
static void Main(string[] args)
{
    Book book = new Book();
    book.author = "余小章";
    book.price = 1000;
    book.title = "MVP";
    ThreadPool.QueueUserWorkItem(new WaitCallback(DoWorker), book); 

    Console.WriteLine("Main thread exit");
    Console.ReadKey();
}
static void DoWorker(object book)
{
    Thread t = Thread.CurrentThread;
    Book b = (Book)book;
    Console.WriteLine("Thread[{0}]:{1}", t.ManagedThreadId, t.ThreadState);
    Console.WriteLine("author:{0},title{1},price:{2}", b.author, b.title, b.price); 

    Thread.Sleep(1000);
}
public struct Book
{
    public decimal price;
    public string title;
    public string author;
}
{{</  highlight  >}}
![](https://cdn.wtfsec.org/img/20200223172541.png)