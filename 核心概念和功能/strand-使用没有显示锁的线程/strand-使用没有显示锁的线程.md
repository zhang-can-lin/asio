## Strand: 使用没有显示锁定的线程
### strand被定义为处理程序的严格顺序调用(例如, 没有并发调用时), 使用strand可以在多线程程序中执行代码而无需显示锁定(例如使用互斥量)
### strand可以是隐式的或者显示的, 如以下代替方法所示
* 从一个线程里调用`io_context::run()`意味着所有的完成处理程序在一个隐式的strand被调用, 因为`io_context`保证处理程序只能从`run()`内部调用
* 如果存在与连接关联的异步操作的链（例如，在半双工协议实现（如HTTP）中），则不可能同时执行处理程序。
这是一个隐式strand
* 一个显示的strand是`strand<>`或者`io_context::strand`实例, 所有的事件完成处理函数对象需要使用`boost::asio::bind_executor()`被绑定到strand, 或者通过`strand`对象post或者dispatch
  
### 对于组合异步操作(例如使用`async_read()`, `async_read_until()`),如果所有的完成处理程序经过一个strand, 则所有中间处理程序也应该经过这个同样的strand,这是确保线程安全访问的必要条件,线程可以安全的访问在调用方和组合操作(例如`async_read()`)之间共享的所有对象
### 为了达到这个, 所有异步操作通过调用`get_associated_executor()`返回完成处理程序关联的executor
``` c++
boost::asio::associated_executor_t<Handler> a 
        = boost::asio::get_associated_executor(h);
```
### 被关联的`executor`必须满足executor的要求,异步操作将使用它提交中间和最终处理程序以供执行
``` c++
class my_handler
{
public:
  // Custom implementation of Executor type requirements.
  using  executor_type = my_executor;

  // Return a custom executor implementation.
  executor_type get_executor() const noexcept
  {
    return my_executor{};
  }

  void operator()() { ... }
};
```