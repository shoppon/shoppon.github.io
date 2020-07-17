# 并发

## 线程安全

### 原子操作

原子操作是不可分割的操作，**CPU指令**是多线程不可分割的**最小单元**。

赋值语句`int foo = 1`/加减操作等都不是原子操作。

### 场景

- `stl`容器是非线程安全的，并发访问可能发生`Segment Fault`错误。

- 基本类型如`uint32_t`是线程安全吗？

## 锁

### Posix

`posix`锁定义`pthread_mutex_t`。

使用RAII技术实现自动加锁和解锁：

```c++
class MutexLock {
 public:
  // pthread_mutex_lock(mu)
  explicit MutexLock(Mutex* mu) : mu_(mu->get()) { gpr_mu_lock(mu_); }
  // pthread_mutex_unlock(mu)
  explicit MutexLock(gpr_mu* mu) : mu_(mu) { gpr_mu_lock(mu_); }
  ~MutexLock() { gpr_mu_unlock(mu_); }

  MutexLock(const MutexLock&) = delete;
  MutexLock& operator=(const MutexLock&) = delete;

 private:
  gpr_mu* const mu_;
};
```

### 锁类型

**自旋锁：**

**互斥锁：**

### C++11中的锁

锁类型：`std::mutex m_lock`

需要加锁时：`std::lock_guard<std::mutex> lock(m_lock)`，在`lock`的作用域结束时会自动释放锁。

## 线程

### 设置线程名称

使用`prctl`设置线程名称。

```c++
#include <sys/prctl.h>

prctl(PR_SET_NAME, "ThreadName");  
```

使用`pthread_setname_np`设置线程名。

### 线程池

## 参考

- [C++11原子操作与无锁编程](https://zhuanlan.zhihu.com/p/24983412)

- [《C++ Concurrency in Action》中文版](https://wiki.jikexueyuan.com/project/cplusplus-concurrency-action/content/about_this_book/about_this_book-chinese.html)

- [Are C++ Reads and Writes of an int Atomic?](https://stackoverflow.com/questions/54188/are-c-reads-and-writes-of-an-int-atomic)
