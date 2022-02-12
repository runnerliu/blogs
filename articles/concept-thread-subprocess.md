## 子进程和线程的区别

### 相同点

- 二者都具有 ID ，一组寄存器，状态，优先级以及所要遵循的调度策略；
- 每个进程都有一个进程控制块，线程也拥有一个线程控制块；
- 线程和子进程共享父进程中的资源；线程和子进程独立于它们的父进程，竞争使用处理器资源；线程和子进程的创建者可以在线程和子进程上实行某些控制，比如，创建者可以取消、挂起、继续和修改线程和子进程的优先级；线程和子进程可以改变其属性并创建新的资源。

### 不同点

- 线程是进程的一部分, 一个没有线程的进程是可以被看作单线程的，如果一个进程内拥有多个进程，进程的执行过程不是一条线（线程）的，而是多条线（线程）共同完成的；
- 启动一个线程所花费的空间远远小于启动一个进程所花费的空间，而且，线程间彼此切换所需的时间也远远小于进程间切换所需要的时间；
- 系统在运行的时候会为每个进程分配不同的内存区域，但是不会为线程分配内存（线程所使用的资源是它所属的进程的资源），线程组只能共享资源。对不同进程来说，它们具有独立的数据空间，要进行数据的传递只能通过通信的方式进行，这种方式不仅费时，而且很不方便。而一个线程的数据可以直接为其他线程所用，这不仅快捷，而且方便；
- 与进程的控制表 PCB 相似，线程也有自己的控制表 TCB，但是 TCB 中所保存的线程状态比 PCB 表中少多了；
- 进程是系统所有资源分配时候的一个基本单位，拥有一个完整的虚拟空间地址，并不依赖线程而独立存在。

### 例子

进程和线程的区别在于粒度不同，进程之间的变量(或者说是内存)是不能直接互相访问的，而线程可以。线程一定会依附在某一个进程上执行.我举个例子，你在 Windows 下开一个 IE 浏览器，这个 IE 浏览器是一个进程，你用浏览器去打开一个pdf，IE 就去调用 Acrobat 去打开，这时 Acrobat 是一个独立的进程，就是 IE 的子进程。而 IE 自己本身同时用同一个进程开了 2 个网页, 并且同时在跑两个网页上的脚本，这两个网页的执行就是 IE 自己通过两个线程实现的。值得注意的是，线程仍然是 IE 的内容，而子进程 Acrobat 严格来说就不属于 IE ，是另外一个程序。之所以是 IE 的子进程，只是受 IE 调用而启动的而已。
子进程与父进程之间可以通过动态数据交换、OLE、管道、邮件槽等进行通信，使用内存映射文件是最便利的方法之一。



Read More:

> [子进程和线程的区别](http://blog.csdn.net/wangkehuai/article/details/7089323)