1. AOF(Append Only File)，记录了每个操作日志
   1. AOF会记录查询命令吗？不会，因为查询日志对于数据恢复没有作用
2. 为什么使用AOF恢复？从数据库恢复数据有以下弊端：
   1. 数据库压力变大
   2. 速度慢
3. AOF是如何实现的？每个写命令执行完毕后将日志写入磁盘
4. AOF是写后日志，相比数据库的写前日志，比较两者的不同
   1. 写前日志可能会阻塞数据的写入
   2. 写后日志避免了语句的检查开销，因为能写入的都是运行成功的命令
   3. 如果数据写入成功后服务器宕机，该条命令会丢失
   4. 写后模式影响下一条数据的写入
5. AOF日志写入时机
   1. Always(同步写回，每条命令执行完毕就把日志写入磁盘），能做到基本不丢数据，但是会影响性能
   2. Everysec（每秒写回，每条命令执行完毕后放入缓冲区，每隔一秒把缓冲区的日志写入磁盘），相比同步写回提升了性能，但是宕机时会丢失一秒内的数据，属于是折中做法
   3. No（操作系统控制的写回。每个命令执行完毕后放入缓冲区，由操作系统决定何时写回磁盘），性能好但是其由操作系统控制写回时机
6. AOF文件变大会带来什么影响？
   1. 文件系统本身对文件大小有限制，无法保存过大文件
   2. 文件太大的话，后续追加命令效率低
   3. 宕机后过大的文件恢复起来缓慢
7. 如何解决AOF文件变大？AOF重写机制，即将多条命令合并成一条，如合并多条set命令
8. AOF重写会阻塞线程吗？不会。因为AOF重写时主线程会fork出bgrewriteaof子线程，重写操作在子线程中完成
9. AOF重写过程：一个拷贝，两个日志 
   1. 一个拷贝：主线程fork出bgrewriteaof子线程时会把包含最新数据的内存拷贝给子线程，子线程就可以在不影响主线程的情况下逐一拷贝 
   2. 两个日志：由于主线程未阻塞，redis可以处理新的命令。第一个日志是指正在使用的AOF日志，当收到新命令后redis会把它写入缓冲区。第二个日志是指正在的重写日志，收到新命令时也会写到重写日志的缓冲区