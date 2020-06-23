---
title: WebServer 源码阅读
date: 2019-07-30 20:33:37
tags:
---

- __thread 关键字（thread_local）
- [eventfd](http://man7.org/linux/man-pages/man2/eventfd.2.html)
- typedef std::function<void()> Functor
- O_CLOEXEC,EFD_NONBLOCK
- extern 关键字
- snprintf 函数
- syscall
- std::bind 函数
- priority_queue
- [struct sigaction](http://man7.org/linux/man-pages/man2/sigaction.2.html)
  - 参考：https://www.cnblogs.com/lit10050528/p/5116566.html
- fcntl 函数
- epoll_create1 
- [epoll_ctl](http://man7.org/linux/man-pages/man2/epoll_ctl.2.html) 
- EPOLLIN,EPOLLOUT,EPOLLET,EPOLLHUP,EPOLLEDHUP,EPOLLERR
- [epoll_wait](http://man7.org/linux/man-pages/man2/epoll_wait.2.html)
- [std::function<void()>](https://zh.cppreference.com/w/cpp/utility/functional/function)
- [std::weak_ptr<T>::lock](https://en.cppreference.com/w/cpp/memory/weak_ptr/lock)
- [accept](http://man7.org/linux/man-pages/man2/accept.2.html)
- [MMAP](http://man7.org/linux/man-pages/man2/mmap.2.html)
- nocopyable 类：将构造函数以及拷贝构造函数等声明为私有
- pthread_mutex_t
- pthread_mutex_init
- pthread_mutex_lock
- pthread_mutex_destroy
- pthread_mutex_unlock
- pthread_cond_t
- pthread_cond_init
- pthread_cond_wait
- pthread_cond_signal
- pthread_cond_broadcast
- pthread_cond_destroy
- pthread_cond_timedwait
- emplace_back
- [read](http://man7.org/linux/man-pages/man2/read.2.html)
- [write](http://man7.org/linux/man-pages/man2/write.2.html)
- [sharedfromthis()](https://en.cppreference.com/w/cpp/memory/enable_shared_from_this/shared_from_this)
- struct timeval
- [shared_ptr<T>::reset()](https://en.cppreference.com/w/cpp/memory/shared_ptr/reset)
- [weak_ptr<T>::lock()](https://en.cppreference.com/w/cpp/memory/weak_ptr/lock)
- [\#pragma once](https://zh.wikipedia.org/wiki/Pragma_once)

I/O 线程可能在 epoll_wait 出阻塞，通过 eventfd 修改counter，使 epoll_wait 返回 eventfd 对应的事件。

Server::handNewConn() 中从 eventLoopThreadPool 获取一个 eventLoop， 也就是说本次获得的是 vector<EventLoop> loops_ 中的一个 loop 对象，此时还是在 eventLoopThreadPool 线程中运行，则需要通过改变 loop 对象中的 eventfd 来唤醒该线程，因为该 loop 对象所属的线程仍然阻塞在 epoll_wait 处。同时对于连接的定时器是在 HttpData::newEvent 中设置的，调用 loop 对应的 epoll::epoll_add 函数，如果 timeout 参数大于 0 则 TimerManager::addTimer(std::shared_ptr<HttpData> SPHttpData, int timeout)，也就是说，针对每组 <HttpData, timeout> 建立一个节点，加入堆中。