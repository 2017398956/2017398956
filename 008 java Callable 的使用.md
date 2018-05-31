# java Callable 实现原理分析 #

## 前言 ##

我们常用的创建线程方式一般有下面 2 种：

1. 继承Thread，重写run方法
1. 实现Runnable接口，重新run方法

其实在 Executor 框架中还有一种方法可以实现异步，那就是实现 Callable 接口并重写call方法。虽然是实现 Callable ，但是在 Executor 实际运行时，会将 Runnable 的实例或 Callable 的实例转化为 RunnableFuture 的实例，而 RunnableFuture 继承了 Runnable 和 Future 接口，这点将在下文详细解释。了解到这些 ，那么它和 Runnable 有什么不同呢？ Callable 与 Runnable 相比有以下 2 点不同：

1. Callable 可以在任务结束的时候提供一个返回值，Runnable 无法提供这个功能
1. Callable 的 call 方法分可以抛出异常，而 Runnable 的 run 方法不能抛出异常。

## 实现原理 ##

在介绍 Callable 的实现原理前，我们先看看它是怎么使用的：

		public class ThreadTest {

		    public static void main(String[] args) {
				System.out.println("main start");
		        ExecutorService threadPool = Executors.newSingleThreadExecutor();
		        // Future<?> future = threadPool.submit(new MyRunnable()) ;
		        Future<String> future = threadPool.submit(new MyCallable());
		        try {
					// 这里会发生阻塞
		            System.out.println(future.get());
		        } catch (Exception e) {
		
		        } finally {
		            threadPool.shutdown();
		        }
				System.out.println("main end");
		    }
		}


		public class MyCallable implements Callable<String> {

		    @Override
		    public String call() throws Exception {
		        // 模拟耗时任务
		        Thread.sleep(3000);
		        System.out.println("MyCallable 线程：" + Thread.currentThread().getName());
		        return "MyCallable" ;
		    }
		}

		public class MyRunnable implements Runnable {

		    @Override
		    public void run() {
		        // 模拟耗时任务
		        try {
		            Thread.sleep(3000);
		        } catch (InterruptedException e) {
		            e.printStackTrace();
		        }
		        System.out.println("MyRunnable");
		    }
		}

运行上面的代码你将得到如下结果：

	main start
	// 这里会阻塞一段时间，才会打印下面的内容
	MyCallable 线程：pool-1-thread-1
	MyCallable
	main end

通过上面的代码我们验证了 Callable 和 Runnable 的不同点，那么 Callable 是怎么实现的呢？首先，我们将 Callable 的实现类 MyCallable 传递给了 **ExecutorService.submit()** 而 ExecutorService 是一个接口，那么其必将在它的实现类中覆写 submit() 方法；跟进 ：Executors.newSingleThreadExecutor() 

![](/images/008_01.png)

**（图一）**

我们发现 ExecutorService 的实现类是 FinalizableDelegatedExecutorService ，再跟进：

![](/images/008_02.png)

**（图二）**

我们发现 FinalizableDelegatedExecutorService 中没有 submit() 方法，那么其必在父类中实现了该方法，在跟进父类：

![](/images/008_03.png)

**（图三）**

从 DelegatedExecutorService 中我们可以看出其 submit() 方法的实现调用了 **ExecutorService.submit()**，是不是感觉又回到原点了，循环调用？当然不是这样的，注意这里的 e.submit 是 DelegatedExecutorService 的一个局部变量 ExecutorService 的方法，而 e 又是通过构造方法赋值的。现在让我们回到 DelegatedExecutorService 的子类 FinalizableDelegatedExecutorService 看看构造方法中传递的是什么，从图二和图一我们可以看到，该构造方法中传递的 ExecutorService 实现类是 ThreadPoolExecutor 。终于找到正主了，让我们跟进 ThreadPoolExecutor 是如何实现 submit() 的，跟进后你会发现 ThreadPoolExecutor 中没有 submit() 方法，那我们只好再到父类中找了：

![](/images/008_04.png)

**（图四）**

----------

![](/images/008_05.png)

**（图五）**

从 AbstractExecutorService 代码我们可以看出 Runnable 接口 和 Callable 接口的处理方式一样都是将其转换为 RunnableFuture 。而 RunnableFuture 是一个接口，那么其必有实现类来完成这个转换过程，让我们分别跟进这两个 newTaskFor 看看 Runnable 和 Callable 的实例都是怎么被转换的：

![](/images/008_06.png)

**（图六）**

从代码中我们可以看到 newTaskForjia() 方法的返回值是 FutureTask 类型的，再次跟进：

![](/images/008_07.png)

**（图七）**

FutureTask 的这两个构造方法的作用是为 callable 和 state 赋值，到此 Callable 和 Runnable 实例的处理方式一样了；不同的是 Runnable 的实例要传递给 Executors.callable 用于生成 Callable 的实例。下面我们看看这个转化的过程：

![](/images/008_08.png)

**（图八）**

RunnableAdapter 是 Callable 接口的一个实现类：

![](/images/008_09.png)

**（图九）**

RunnableAdapter 覆写了 Callable 的 call() 方法，并在执行 call() 时执行了 Runnable 的 run() 方法；这样就做得到了 Runnable 和 Callable 的统一。

既然 Runnable 和 Callable 统一了，那么我们再回头看看线程的执行方法 图五 的 **execute(Runnable runnable)** 是怎么实现的。 什么情况？ execute 传递的是 Runnable 实例，而我们把 Runnable 和 Callable 统一成 Callable 了。是不是感觉很奇怪？注意 图五 中 execute 传递的是 RunnableFuture 的实现类 FutureTask 的实例。而 RunnableFuture 实现了 Runnable 接口，并覆写了 Runnable 的 run() 方法：

![](/images/008_10.png)

**（图十）**

那么，Runnable 的实现类 FutureTask 是怎么实现 run() 方法的呢？

![](/images/008_11.png)

**（图十一）**

run() 方法中调用了 Callable.call() ， 而 run() 又是覆写的 Runnable 的 run() 方法，到此就理解为什么 图五 的 execute() 方法可以传 RunnableFuture 实例了： 先将 Runnable 或 Callable 的实例统一为 Callable 类型，再在执行 run() 方法时调用 Callable 的 call() 方法。

细心的读者可能发现了文章开头的 demo 中有句注释（// 这里会发生阻塞），那是什么原因导致阻塞呢？让我们跟进 Future 的实现类 FutureTask (由 图五 可以得到) 看看其 get() 方法是怎么实现的：

![](/images/008_12.png)

**（图十二）**

继续跟进：

![](/images/008_13.png)

**（图十三）**

awaitDone 是一个死循环，只有当待执行的 Callable 或 Runnable 结束时，才会跳出，这样就不难理解 图十二 的逻辑了，当 Callable 或 Runnable 已经结束时，直接给出返回值，否则就阻塞在 awaitDone() 方法，所以才有注释中那句注释。

## 结语 ##

本文到此已经大致把 Callable 实现异步的原理讲解清楚了，为了更方便的理解，在这里补一张使用 Callable 的流程图。

![](/images/008_14.png)

## 附录 ##

关于 Executor 是属于线程池的内容，不是本文重点，只需要知道：线程池在使用中要接收 Runnable 实例即可。

## 再附 ##

Callable 在日常开发中的使用场景很少，理解起来没那么形象。这里举一例：在网络并发的访问中，我们有时需要多个访问请求的结果一起作为下一个方法的参数，那么这时就可以使用 Callable 了；如果是在 Android 中要注意为这些请求包裹一个非 UI 线程（这里记作线程 A）以免调用时阻塞 UI 线程，当线程 A 中的所有网络访问都结束后（也包括其它耗时操作），就可以发送消息给主线程刷新 UI 了，当然你也可以使用 Rxjava 来实现同样的效果。

