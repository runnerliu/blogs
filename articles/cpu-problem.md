## CPU占用问题

### cpu占用很高的3大类型，9大场景

<img src="/Users/william/personal/blogs/images/2024-08-07T220400.png" alt="2024-08-07T220400" style="zoom:50%;" />

#### 业务类问题

##### 死循环

死循环是指程序在特定条件下进入了一个无限循环，无法跳出，导致CPU资源被完全占用。

例如：我们有一段代码用来检查文件的更新状态，但由于逻辑错误，条件永远无法满足，结果程序进入了死循环。

```
while (true) {
    if (file.isUpdated()) {
        break;
    }
}
```

##### 死锁

死锁是指两个或多个线程互相等待对方释放资源，导致所有线程都无法继续执行，CPU资源被消耗殆尽。

<img src="/Users/william/personal/blogs/images/2024-08-07T220700.png" alt="2024-08-07T220700" style="zoom:50%;" />

发生死锁后，就会存在忙等待或自旋锁等编程问题，从而导致 繁忙等待问题，从而导致 **CPU** 100%

#####  不必要的代码块

一些冗余、不必要的代码块在运行时占用了大量的CPU资源。

> 例如，不需要的地方使用synchronized块。

```
public synchronized void unnecessarySync() {
    // 执行一些不需要同步的操作
}
```

在不需要的地方使用synchronized块，会导致线程竞争和上下文切换

#### 并发类问题

##### 大量计算密集型的任务

大量计算密集型任务在同一时间运行，会导致CPU资源被完全占用。

> 例如：在数据分析或科学计算中，多个计算密集型任务同时运行

<img src="/Users/william/personal/blogs/images/2024-08-07T220900.png" alt="2024-08-07T220900" style="zoom:50%;" />

##### 大量并发线程

统中存在大量并发线程，线程切换频繁，导致CPU资源被大量消耗在上下文切换上

> 例如：Web服务器同时处理大量请求，每个请求都创建一个新线程

<img src="/Users/william/personal/blogs/images/2024-08-07T221000.png" alt="2024-08-07T221000" style="zoom:50%;" />

**解决方案**：使用线程池来限制并发线程数量

##### 大量的上下文切换

当系统中存在大量线程时，CPU在不同线程间频繁切换，导致性能下降

> 例如：一个程序中开启了数百个线程，每个线程都在不断进行I/O操作

```
for (int i = 0; i < 1000; i++) {
    new Thread(new IOHandler()).start();
}
```

线程是很宝贵的资源，开启线程一定要合理的控制线程数量

#### 内存类问题

##### 内存不足

当系统内存不足时，就会将磁盘存储作为虚拟内存使用，而虚拟内存的运行速度要慢得多。

> 例如：直接一次性加载一个非常大的文件到内存中，导致内存不足

```
byte[] largeData = Files.readAllBytes(Paths.get("largeFile.txt"));
```

这种过度的分页和交换会导致 CPU 占用率居高不下，因为处理器需要花费更多时间来管理内存访问，而不是高效地执行进程。

**解决方案**：优化内存使用，采用流式处理避免一次性加载大文件

```
try (BufferedReader reader = Files.newBufferedReader(Paths.get("largeFile.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        // 处理每一行数据
    }
}
```

#### 频繁GC

频繁的垃圾回收（GC）操作会占用大量CPU资源，导致性能下降。

> 例如：程序中频繁创建和销毁对象，导致GC频繁触发

```
for (int i = 0; i < 1000000; i++) {
    String temp = new String("temp" + i);
}
```

**解决方案**：优化对象创建和销毁，减少临时对象的生成。

#### 内存泄漏

内存泄漏导致可用内存逐渐减少，最终触发频繁的GC操作，占用大量CPU资源

> 例如：某个数据结构中不断添加对象，却从未删除，导致内存泄漏

```
List<Object> list = new ArrayList<>();
while (true) {
    list.add(new Object());
}
```

**解决方案**：定期清理不再使用的对象，使用合适的数据结构