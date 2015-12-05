## 分布式锁

分布式锁的作用和场景，balabalabala...

首先看看线程锁的常见操作：
	
	tryLock()  --尝试加锁，失败则立即返回，成功即获取锁
	lock() -- 加锁，如果当前锁没其它进程持有则等待直到获取锁
	unLock() -- 释放自己持有的锁

常见的代码逻辑：

>     lock();
>     try{
> 	    //xxxxxx业务逻辑
>     }catch(ex){
>       print ex;
>     }finally{
>       unlock();
>     }

所有我们实现的锁也需要具备这三个基本的接口，我们以一个实际的案例来讲述分布式锁的使用场景和方法

> 如何保证addUser这个操作不被并发执行？

常见的分布式锁（互斥锁，读写锁除外）实现方式：
> 1. 数据库的主键唯一性
> 2. Redis的SETNX
> 3. zookeeper的createPath
 
### 1. 数据库的Insert

通过数据库的主键互斥来实现锁的互斥，这里只是简单讲解基础的原理，真正的生产环境的逻辑往往比这复杂的多，需要考虑公平、非公平锁，超时等问题

**OP_INSERT**：`INSERT INTO TB_Lock (lock_id, acquredTime) VALUES('Lock_AddUser', now);`

（1）tryLock()

`SELECT * FROM TB_Lock WHERE id =  'Lock_AddUser'`

如果记录不存在则表示可以获取锁，然后尝试Lock，执行**OP_INSERT**操作

当然如果**OP_INSERT**插入报唯一键冲突，则表明此时有另外一个进程优先获得锁

（2）lock()

略.

（3）unLock()

`DELETE FROM TB_Lock WHERE id = 'Lock_AddUser'`

删除对应的记录即可

**注意**：unLock()极有可能由于进程异常退出而未被执行，导致锁一直无法释放，所以我们需要提供超时检测的机制，当然也有简单的方式，直接把unLock()、lock()和业务操作放在一个事务里面，保证锁能够很好的释放。

### 2. Redis的SETNX

略，Redis实现分布式锁的一个最大的问题在于**怎么才能保证锁的释放？**这是一个比较麻烦的事情，即使提供超时机制也有可能出现脑裂问题

### 3. 使用zookeeper实现分布式锁

略.
