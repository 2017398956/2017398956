# 通过一堂化学课来彻底理解 ThreadLocal

关于 ThreadLocal 相信很多读者都在网上看到了这样的介绍：ThreadLocal 为解决多线程程序的并发问题提供了一种新的思路；ThreadLocal的目的是为了解决多线程访问资源时的共享问题。如果你也这样认为的，请允许我在这里向你提出这样一个问题：多线程共享资源时，资源往往是唯一的（如写入文件），而 ThreadLocal 会为每个线程提供一个相同的数据副本，明显和资源共享不一样，那么为什么说 ThreadLocal 是为解决多线程程序的并发问题提供了一种新的思路呢？到这里相信你也发现有问题了吧，所以上面关于 ThreadLocal 的认知是不恰当的，下面我将以一堂化学实验课为例来轻松形象地理解 ThreadLocal 。

在化学实验课开始前，我们需要对 ThreadLocal 做个大概的了解：

1. ThreadLocal 的作用是什么？

		This class provides thread-local variables.  These variables differ from
		their normal counterparts in that each thread that accesses one (via its
		{@code get} or {@code set} method) has its own, independently initialized
		copy of the variable.  {@code ThreadLocal} instances are typically private
		static fields in classes that wish to associate state with a thread (e.g.,
		a user ID or Transaction ID).

	上面就是 JDK 中的注释，大概意思就是：ThreadLocal 类用来提供线程内部的局部变量。这种变量在多线程环境下访问(通过 get 或 set 方法访问)时能保证各个线程里的变量相对独立于其他线程内的变量。ThreadLocal 实例通常来说都是 private static 类型的，用于关联线程和线程的上下文。总之一句话： ThreadLocal 为每个线程提供独立的局部变量，即不受其它线程影响的局部变量。

1. initialValue 来为 ThreadLocal 设置初始值

	该函数是 protected 类型的，很显然是建议在子类重载该函数的，所以通常该函数都会以匿名内部类的形式被重载，以指定初始值。另外，该函数在调用 get 函数的时候会第一次调用，但是如果一开始就调用了 set 函数，则该函数不会被调用。通常该函数只会被调用一次，除非手动调用了 remove 函数之后又调用 get 函数，这种情况下，get 函数中还是会调用 initialValue 函数。

看到这里大部分读者应该还是不甚明白（因为根本就没说 ThreadLocal 的原理及用途，手动滑稽），没关系，下面我通过一堂化学实验课的实例来讲解清楚：

现在让我们把时间拨回到高中时代，此时你正在实验室做一个化学实验，而这个实验需要 2 种溶液：一种为 20% 的酒精 ， 另一种为 70% 的硫酸。另外，由于硫酸比较危险，学校让两名化学老师分别管理这两种溶液：M 老师管理各种浓度的酒精，N 老师管理各种浓度的硫酸。是不是突然觉得化学老师的岗位就相当于 ThreadLocal 。了解到这一点，让我们先看下 ThreadLocal 的定义ThreadLocal&lt;T&gt; 。ThreadLocal 是有泛型的，就像化学老师可以管理不同的试剂，对于酒精溶液有ThreadLocal&lt;M&gt;, 硫酸溶液 ThreadLocal&lt;N&gt; 。但是，化学实验需要特定浓度的试剂，这就是 initialValue 的作用了，请看下面的伪代码：

	ThreadLocal<M> threadLocal = new ThreadLocal<M>(){
            @Override
            protected M initialValue() {
                return 20% 的酒精 ;
            }
        } ;

每个学生都需要要独自完成各自的实验，和每个线程都去完成一定的业务是一样的；到这里，我们已经可以理解为什么 ThreadLocal 要为每个线程提供一个独立的副本：假设这些数据不是独立的，就像 老师 M 提供的 20% 酒精是一次性提供给所有学生的，根据常识，同学们做实验的效率是不一样的，如果在其他同学使用 20%酒精 前,已经有同学率先使用并改变了酒精的浓度，那么后面同学的实验就有可能失败。那么应该怎么做呢？从我们上实验课的经验我们可以知道老师都会为每位学生提供一份相同的试剂，而不同的学生可以类比为不同的线程，是不是和 ThreadLocal 的处理方式一模一样，是不是？！

当然，做实验我们不可能只用一种试剂，如本实验我们需要酒精和硫酸，我们知道这两种试剂都是无色透明的，为了不把实验搞砸，我们需要区分开这两种试剂。通常的方式是为各个试剂贴上标签，这就类似于 ThreaLocalMap 的 key value 键值对。一般标签的方式有下面 2 种：

1. key 用来标记学生的名称，value 用来标记什么试剂，让老师管理试剂的分发。
2. key 用来标记从哪位老师那拿的试剂，value 用来标记什么试剂，让学生自己管理试剂，只要根据标签找到对应的老师要试剂就行了。

这两种方式现在看，没有什么不同，不过我们考虑到实验结束后要进行废液的处理（类比 java 中的 GC ）。如果采用方式1，老师需要记住每个做实验的学生并在实验结束后为其处理废液；如果采用方式2，老师发放试剂后就不需要管理，让学生自己处理废液（各个线程自己进行垃圾回收）即可。由于学生的数量远远多于老师，所以采用方式 2 效率更高，也和我们的生活经验相符。这就解释了为什么 ThreadLocalMap 是属于各个 Thread 的，而不是属于 ThreadLocal 的，有源码为证：

![](https://i.imgur.com/rPH7fbE.png)

看过 ThreadLocalMap 源码的读者可以发现 ThreadLocalMap 是使用 ThreadLocal 的弱引用作为 Key 的，

	static class ThreadLocalMap {
		static class Entry extends WeakReference<ThreadLocal<?>>{
		...

那么 ThreadLocal 就有可能引起内存泄露，理由如下： 
ThreadLocalMap 使用 ThreadLocal 的弱引用作为 key，如果一个 ThreadLocal 没有外部强引用引用他，那么系统 gc 的时候，这个 ThreadLocal 势必会被回收，这样一来，ThreadLocalMap 中就会出现 key 为 null 的 value，就没有办法访问这些 key 为 null 的 value，如果当前线程再迟迟不结束的话，这些 key 为 null 的 value 就会一直存在一条强引用链。所以，内存泄露的原因是 ThreadLocal 先于线程被回收，这也就解释了为什么 JDK 建议将 ThreadLocal 变量定义成 **private static** 的，这样的话 ThreadLocal 的生命周期就更长（类似于化学老师在同学们做完实验后最后一个走），由于一直存在 ThreadLocal 的强引用，所以 ThreadLocal 也就不会被回收，也就能保证任何时候都能根据 ThreadLocal 的弱引用访问到 Entry 的 value 值，然后 remove 它，防止内存泄露。

从上面可以看出正常情况下是不会发生内存泄露的，但是，某次实验课时，小明不小心将试剂瓶上的标签污染了，导致字迹模糊（说了多少遍标签朝手心  (。・`ω´・) ），那么 ThreadLocalMap 引用的 key 就为 null 了，怎么办？不用担心，ThreadLocalMap 已经考虑了这种情况，我们来看看 ThreadLocalMap 的 getEntry 方法：

       private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1);
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e);
        }

        private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key)
                    return e;
                if (k == null)
                    expungeStaleEntry(i);
                else
                    i = nextIndex(i, len);
                e = tab[i];
            }
            return null;
        }

        private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // expunge entry at staleSlot
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // Rehash until we encounter null
            Entry e;
            int i;
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) {
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {
                        tab[i] = null;

                        // Unlike Knuth 6.4 Algorithm R, we must scan until
                        // null because multiple entries could have been stale.
                        while (tab[h] != null)
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }

通过上面的代码我们可以看出 ThreadLocal 先直接通过索引位置ThreadLocal.threadLocalHashCode & (len-1) 得到 Entry ，如果 Entry 不为 null 并且 key 相同则返回 Entry ；如果 Entry 为 null 或者 key 不一致则向下一个位置查询，如果下一个位置的 key 和当前需要查询的 key 相等，则返回对应的 Entry ，否则，如果 key 值为 null ，则擦除该位置的 Entry，否则继续向下一个位置查询 。这样 Entry 内的 value 也就没有强引用链，自然会被回收，不会发生内存泄露，同理 set 也是这样处理的。但是，这一切都发生在调用 getEntry 或 set 时，如果我们没有调用这两个方法中的任一个，那么需要我们主动调用 remove 方法清空 value 的值，以防止内存泄露，就像实验结束后清理桌面重新归置好仪器一样。

好了，本次实验课到此结束，下课 ^_^









