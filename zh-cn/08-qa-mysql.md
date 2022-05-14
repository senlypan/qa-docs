# MySQL数据库QA

> 作者: 潘深练
>
> 创建: 2022-03-31

## 〇、MySQL前言

### 0.1、MySQL起源和分支

MySQL 是最流行的关系型数据库软件之一，由于其体积小、速度快、开源免费、简单易用、维护成本低等，在集群架构中易于扩展、高可用，因此深受开发者和企业的欢迎。

![08-qa-mysql#history](../_media/images/08-qa-mysql/mysql-history-001.png)

MySQL发展历程如下：

![08-qa-mysql#history](../_media/images/08-qa-mysql/mysql-history-002.png)

MySQL主流分支如下图所示：

![08-qa-mysql#history](../_media/images/08-qa-mysql/mysql-history-003.png)

MySQL从最初的1.0、3.1到后来的8.0，发生了各种各样的变化。被Oracle收购后，MySQL的版本演化出了多个分支，除了需要付费的MySQL企业版本，还有很多MySQL社区版本。还有一条分支非常流行的开源分支版本叫Percona Server，它是MySQL的技术支持公司Percona推出的，也是在实际工作中经常碰到的。Percona Server在MySQL官方版本的基础上做了一些补丁和优化，同时推出了一些工具。另外一个非常不错的版本叫MariaDB，它是MySQL的公司被Oracle收购后，MySQL的创始人Monty先生，按原来的思路重新写的一套新数据库，同时也把 InnoDB 引擎作为主要存储引擎，也算 MySQL 的分支。

### 0.2、MySQL应用架构演变

网站在不同的并发访问量级和数据量级下，MySQL应用架构的演变过程。

用户请求 --> 应用层 --> 服务层 --> 存储层

#### 架构V1.0 - 单机单库

一个简单的小型网站或者应用背后的架构可以非常简单, 数据存储只需要一个MySQL Instance就能满足数据读取和写入需求（这里忽略掉了数据备份的实例），处于这个的阶段系统，一般会把所有的信息存到一个MySQL Instance里面。

![08-qa-mysql#history](../_media/images/08-qa-mysql/mysql-history-004.png)

#### V1.0 瓶颈

- 数据量太大，超出一台服务器承受
- 读写操作量太大，超出一台服务器承受
- 一台服务器挂了，应用也会挂掉（可用性差）

#### 架构V2.0 - 主从架构

V2.0架构主要解决架构V1.0下的 **高可用** 和 **读扩展** 问题，通过给Instance挂载从库解决读取的压力，主库宕机也可以通过主从切换保障高可用。在MySQL的场景下就是通过主从结构（双主结构也属于特殊的主从），主库抗写压力，通过从库来分担读压力，对于写少读多的应用，V2.0主从架构完全能够胜任。

![08-qa-mysql#history](../_media/images/08-qa-mysql/mysql-history-005.png)

#### V2.0瓶颈

- 数据量太大，超出一台服务器承受
- 写操作太大，超出一台M服务器承受

#### 架构V3.0 - 分库分表

对于V1.0和V2.0遇到 **写入瓶颈** 和 **存储瓶颈** 时，可以通过水平拆分来解决，水平拆分和垂直拆分有较大区别，垂直拆分拆完的结果，每一个实例都是拥有全部数据的，而水平拆分之后，任何实例都只有全量的1/n的数据。以下图所示，将Userinfo拆分为3个Sharding，每个Sharding持有总量的1/3数据，3个Sharding数据的总和等于一份完整数据。

![08-qa-mysql#history](../_media/images/08-qa-mysql/mysql-history-006.png)

#### V3.0挑战

- 数据如何路由成为一个关键问题， 一般可以采用范围拆分，List拆分、Hash拆分等。
- 如何保持数据的一致性也是个难题。


#### 架构V4.0 - 云数据库

云数据库（云计算）现在是各大IT公司内部作为 **节约成本** 的一个突破口，对于数据存储的MySQL来说，如何让其成为一个saas（Software as a Service）是关键点。MySQL作为一个saas服务，服务提供商负责解决 **可配置性**，**可扩展性**，**多用户存储结构设计** 等这些疑难问题。

![08-qa-mysql#history](../_media/images/08-qa-mysql/mysql-history-007.png)


## 一、MySQL架构原理

### 1.1、MYSQL体系架构

![08-qa-mysql#history](../_media/images/08-qa-mysql/mysql-arch-000.png)

MySQL Server架构自顶向下大致可以分网络连接层、服务层、存储引擎层和系统文件层。

#### 1）、网络连接层

客户端连接器（Client Connectors）：提供与MySQL服务器建立的支持。目前几乎支持所有主流的服务端编程技术，例如常见的 Java、C、Python、.NET等，它们通过各自API技术与MySQL建立连接。

#### 2）、服务层（MySQL Server）

服务层是MySQL Server的核心，主要包含 **系统管理和控制工具**、**连接池**、**SQL接口**、**解析器**、**查询优化器**和**缓存**六个部分。

- **连接池（Connection Pool）**：负责存储和管理客户端与数据库的连接，一个线程负责管理一个连接。

- **系统管理和控制工具（Management Services & Utilities）**：例如备份恢复、安全管理、集群管理等

- **SQL接口（SQL Interface）**：用于接受客户端发送的各种SQL命令，并且返回用户需要查询的结果。比如DML、DDL、存储过程、视图、触发器等。

- **解析器（Parser）**：负责将请求的SQL解析生成一个"解析树"。然后根据一些MySQL规则进一步检查解析树是否合法。

- **查询优化器（Optimizer）**：当“解析树”通过解析器语法检查后，将交由优化器将其转化成执行计划，然后与存储引擎交互。

> 
> select uid,name from user where gender=1;
> 
> 选取--》投影--》联接 策略
> 
> 1）select先根据where语句进行选取，并不是查询出全部数据再过滤
> 
> 2）select查询根据uid和name进行属性投影，并不是取出所有字段
> 
> 3）将前面选取和投影联接起来最终生成查询结果
> 

- **缓存（Cache&Buffer）**： 缓存机制是由一系列小缓存组成的。比如表缓存，记录缓存，权限缓存，引擎缓存等。如果查询缓存有命中的查询结果，查询语句就可以直接去查询缓存中取数据。

#### 3）、存储引擎层（Pluggable Storage Engines）

存储引擎负责MySQL中数据的存储与提取，与底层系统文件进行交互。MySQL存储引擎是插件式的，服务器中的查询执行引擎通过接口与存储引擎进行通信，接口屏蔽了不同存储引擎之间的差异 。现在有很多种存储引擎，各有各的特点，最常见的是 **MyISAM** 和 **InnoDB**。

#### 4）、系统文件层（File System）

该层负责将数据库的数据和日志存储在文件系统之上，并完成与存储引擎的交互，是文件的物理存储层。主要包含日志文件，数据文件，配置文件，pid 文件，socket 文件等。

- **日志文件**  
    - 错误日志（Error log）
        - 默认开启，show variables like '%log_error%'
    - 通用查询日志（General query log）
        - 记录一般查询语句，show variables like '%general%';
    - 二进制日志（binary log）
        - 记录了对MySQL数据库执行的更改操作，并且记录了语句的发生时间、执行时长；但是它不
        - 记录select、show等不修改数据库的SQL。主要用于数据库恢复和主从复制。
        - show variables like '%log_bin%'; //是否开启
        - show variables like '%binlog%'; //参数查看
        - show binary logs;//查看日志文件
    - 慢查询日志（Slow query log）
        - 记录所有执行时间超时的查询SQL，默认是10秒。
        - show variables like '%slow_query%'; //是否开启
        - show variables like '%long_query_time%'; //时长

- **配置文件**
    - 用于存放MySQL所有的配置信息文件，比如my.cnf、my.ini等。

- **数据文件**
    - db.opt 文件：记录这个库的默认使用的字符集和校验规则。
    - frm 文件：存储与表相关的元数据（meta）信息，包括表结构的定义信息等，每一张表都会有一个frm 文件。
    - MYD 文件：MyISAM 存储引擎专用，存放 MyISAM 表的数据（data)，每一张表都会有一个.MYD 文件。
    - MYI 文件：MyISAM 存储引擎专用，存放 MyISAM 表的索引相关信息，每一张 MyISAM 表对应一个 .MYI 文件。
    - ibd文件和 IBDATA 文件：存放 InnoDB 的数据文件（包括索引）。InnoDB 存储引擎有两种表空间方式：独享表空间和共享表空间。独享表空间使用 .ibd 文件来存放数据，且每一张InnoDB 表对应一个 .ibd 文件。共享表空间使用 .ibdata 文件，所有表共同使用一个（或多个，自行配置）.ibdata 文件。
    - ibdata1 文件：系统表空间数据文件，存储表元数据、Undo日志等 。
    - ib_logfile0、ib_logfile1 文件：Redo log 日志文件。

- **pid 文件**
    - pid 文件是 mysqld 应用程序在 Unix/Linux 环境下的一个进程文件，和许多其他 Unix/Linux 服务端程序一样，它存放着自己的进程 id。

- **socket 文件**
    - socket 文件也是在 Unix/Linux 环境下才有的，用户在 Unix/Linux 环境下客户端连接可以不通过 TCP/IP 网络而直接使用 Unix Socket 来连接 MySQL。

### 1.2、MySQL的执行逻辑

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-arch-001.jpg)

#### 1）、MySQL组件QA

- MySQL的框架有几个组件, 各是什么作用? 
    - 连接器：负责跟客户端建立连接、获取权限、维持和管理连接。
    - 查询缓存：查询请求先访问缓存(key 是查询的语句，value 是查询的结果)。命中直接返回。不推荐使用缓存，更新会把缓存清除(关闭缓存：参数 query_cache_type 设置成 DEMAND)。
    - 分析器：对 SQL 语句做解析，判断sql是否正确。
    - 优化器：决定使用哪个索引，多表关联（join）的时候，决定各个表的连接顺序。
    - 执行器：执行语句，先判断用户有无查询权限，使用表定义的存储引擎。

- Server层和存储引擎层各是什么作用?
    - Server 层主要是功能层面，包括连接器、查询缓存、分析器、优化器、执行器等，涵盖 MySQL 的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。
    - 存储引擎层主要是存储相关，负责数据的存储和提取。其架构模式是插件式的，支持 InnoDB、MyISAM、Memory 等多个存储引擎。现在最常用的存储引擎是 InnoDB，它从 MySQL 5.5.5 版本开始成为了默认存储引擎。

- you have an error in your SQL syntax 这个保存是在词法分析里还是在语法分析里报错?
    - 语法分析
    - 【延伸】分析器如果没有命中查询缓存，就会开始真正执行语句了。首先，MySQL 需要知道你要做什么，因此需要对 SQL 语句做解析。分析器先会做 `词法分析`。你输入的是由多个字符串和空格组成的一条 SQL 语句，MySQL 需要识别出里面的字符串分别是什么，代表什么。MySQL 从你输入的 `select` 这个关键字识别出来，这是一个查询语句。它也要把字符串 `T` 识别成 `表名 T`，把字符串 `ID` 识别成 `列 ID`。做完了这些识别以后，就要做 `语法分析`。根据词法分析的结果，语法分析器会根据语法规则，判断你输入的这个 SQL 语句是否满足 MySQL 语法。如果你的语句不对，就会收到 `You have an error in your SQL syntax` 的错误提醒。

- 对于表的操作权限验证在哪里进行?
    - 执行器和优化器。
    - 【详细】MySQL 通过分析器知道了你要做什么，通过优化器知道了该怎么做，于是就进入了执行器阶段，开始执行语句。开始执行的时候，要先判断一下你对这个表 T 有没有执行查询的权限，如果没有，就会返回没有权限的错误 (在工程实现上，如果命中查询缓存，会在查询缓存返回结果的时候，做权限验证。查询也会在优化器之前调用 `precheck` 验证权限)。如果有权限，就打开表继续执行。打开表的时候，执行器就会根据表的引擎定义，去使用这个引擎提供的接口，进行数据提取索要。

- 执行器的执行查询语句的流程是什么样的? 
    - 例如 SQL 语句 ： `select * from T where ID=10 ;` 
    - 1、调用 InnoDB 引擎接口取这个表的第一行，判断 ID 值是不是 10，如果不是则跳过，如果是则将这行存在结果集中；
    - 2、调用引擎接口取“下一行”，重复相同的判断逻辑，直到取到这个表的最后一行。
    - 3、执行器将上述遍历过程中所有满足条件的行组成的记录集作为结果集返回给客户端。

- MySQL的查询缓存是如何工作的？
    - MySQL 拿到一个查询请求后，会先到查询缓存看看，之前是不是执行过这条语句。之前执行过的语句及其结果可能会以 key-value 对的形式，被直接缓存在内存中。key 是查询的语句，value 是查询的结果。如果你的查询能够直接在这个缓存中找到 key，那么这个 value 就会被直接返回给客户端。
    - 如果语句不在查询缓存中，就会继续后面的执行阶段。执行完成后，执行结果会被存入查询缓存中。你可以看到，如果查询命中缓存，MySQL 不需要执行后面的复杂操作，就可以直接返回结果，这个效率会很高。

- 大多数情况下不建议使用查询缓存，为什么呢？
    - 因为查询缓存往往弊大于利。
    - 查询缓存的失效非常频繁，只要有对一个表的更新，这个表上所有的查询缓存都会被清空。因此很可能你费劲地把结果存起来，还没使用呢，就被一个更新全清空了。对于更新压力大的数据库来说，查询缓存的命中率会非常低。除非你的业务就是有一张静态表，很长时间才会更新一次。比如，一个系统配置表，那这张表上的查询才适合使用查询缓存。
    - 好在 MySQL 也提供了这种“按需使用”的方式。你可以将参数 query_cache_type 设置成 DEMAND，这样对于默认的 SQL 语句都不使用查询缓存。而对于你确定要使用查询缓存的语句，可以用 SQL_CACHE 显式指定，像下面这个语句一样：` select SQL_CACHE * from T where ID=10 ;`
    - 需要注意的是，MySQL 8.0 版本直接将查询缓存的整块功能删掉了，也就是说 8.0 开始彻底没有这个功能了。

- InnoDB 的 redo log 是什么？crash-safe 能力指什么？
    - `redo log` 是 `InnoDB` 引擎中的重做日志。`redo log` 是固定大小的，比如可以配置为一组 4 个文件，每个文件的大小是 1GB，那么总共就可以记录 4GB 的操作。从头开始写，写到末尾就又回到开头循环写，如下面这个图所示。 
    - ![08-qa-mysql#002](../_media/images/08-qa-mysql/qa-mysql-crash-safe.jpg)
    - `write pos` 是当前记录的位置，一边写一边后移，写到第 3 号文件末尾后就回到 0 号文件开头。`checkpoint` 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。
    - `write pos` 和 `checkpoint` 之间的是空闲部分，可以用来记录新的操作。如果 `write pos` 追上 `checkpoint`，表示满了，这时候不能再执行新的更新，得停下来先擦掉一些记录，把 `checkpoint` 推进一下。
    - 有了 `redo log`，`InnoDB` 就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失，这个能力称为 `crash-safe`。

- binlog 和 redo log 有什么关系？
    - 1、`redo log` 是 `InnoDB` 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。
    - 2、`redo log` 是物理日志，记录的是“在某个数据页上做了什么修改”；`binlog` 是逻辑日志，记录的是这个语句的原始逻辑，比如 `update T set c=c+1 where ID=2;` 中 “给 ID=2 这一行的 c 字段加 1 ”。
    - 3、`redo log` 是循环写的，空间固定会用完；`binlog` 是可以追加写入的。“追加写”是指 `binlog` 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。
    - 4、`redolog`和`binlog`具有关联行，在恢复数据时，`redolog`用于恢复主机故障时的未更新的物理数据，`binlog`用于备份操作。每个阶段的log操作都是记录在磁盘的，在恢复数据时，`redolog` 状态为 **commit** 则说明`binlog`也成功，直接恢复数据；如果`redolog`是 **prepare**，则需要查询对应的`binlog`事务是否成功，决定是回滚还是执行。

- 为什么会有两份日志呢？
    - 因为最开始 MySQL 里并没有 `InnoDB` 引擎。MySQL 自带的引擎是 `MyISAM`，但是 `MyISAM` 没有 `crash-safe` 的能力，`binlog` 日志只能用于归档。而 InnoDB 是另一个公司以插件形式引入 MySQL 的，既然只依靠 binlog 是没有 `crash-safe` 能力的，所以 `InnoDB` 使用另外一套日志系统——也就是 `redo log` 来实现 `crash-safe` 能力。


### 1.3、MySQL运行机制

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-arch-002.png)

#### 1）、建立连接（Connectors&Connection Pool）

通过客户端/服务器通信协议与MySQL建立连接。MySQL 客户端与服务端的通信方式是 “ **半双工** ”。对于每一个 MySQL 的连接，时刻都有一个线程状态来标识这个连接正在做什么。

**通讯机制：**

- 全双工：能同时发送和接收数据，例如平时打电话。
- 半双工：指的某一时刻，要么发送数据，要么接收数据，不能同时。例如早期对讲机
- 单工：只能发送数据或只能接收数据。例如单行道

**线程状态：**

- show processlist; //查看用户正在运行的线程信息，root用户能查看所有线程，其他用户只能看自己的
    - id：线程ID，可以使用kill xx；
    - user：启动这个线程的用户
    - Host：发送请求的客户端的IP和端口号
    - db：当前命令在哪个库执行
    - Command：该线程正在执行的操作命令
        - Create DB：正在创建库操作
        - Drop DB：正在删除库操作
        - Execute：正在执行一个PreparedStatement
        - Close Stmt：正在关闭一个PreparedStatement
        - Query：正在执行一个语句
        - Sleep：正在等待客户端发送语句
        - Quit：正在退出
        - Shutdown：正在关闭服务器
    - Time：表示该线程处于当前状态的时间，单位是秒
    - State：线程状态
        - Updating：正在搜索匹配记录，进行修改
        - Sleeping：正在等待客户端发送新请求
        - Starting：正在执行请求处理
        - Checking table：正在检查数据表
        - Closing table : 正在将表中数据刷新到磁盘中
        - Locked：被其他查询锁住了记录
        - Sending Data：正在处理Select查询，同时将结果发送给客户端
    - Info：一般记录线程执行的语句，默认显示前100个字符。想查看完整的使用show full processlist;

#### 2）、查询缓存（Cache&Buffer）

这是MySQL的一个可优化查询的地方，如果开启了查询缓存且在查询缓存过程中查询到完全相同的SQL语句，则将查询结果直接返回给客户端；如果没有开启查询缓存或者没有查询到完全相同的 SQL 语句则会由解析器进行语法语义解析，并生成“解析树”。

- 缓存Select查询的结果和SQL语句
- 执行Select查询时，先查询缓存，判断是否存在可用的记录集，要求是否完全相同（包括参数值），这样才会匹配缓存数据命中。
- 即使开启查询缓存，以下SQL也不能缓存
    - 查询语句使用SQL_NO_CACHE
    - 查询的结果大于query_cache_limit设置
    - 查询中有一些不确定的参数，比如now()
- show variables like '%query_cache%'; //查看查询缓存是否启用，空间大小，限制等
- show status like 'Qcache%'; //查看更详细的缓存参数，可用缓存空间，缓存块，缓存多少等

#### 3）、解析器（Parser）

将客户端发送的SQL进行语法解析，生成"解析树"。**预处理器** 根据一些MySQL规则进一步检查“解析树”是否合法，例如这里将检查数据表和数据列是否存在，还会解析名字和别名，看看它们是否有歧义，最后生成新的“解析树”。


#### 4）、查询优化器（Optimizer）

根据“解析树”生成最优的执行计划。MySQL使用很多优化策略生成最优的 **执行计划**，可以分为两类：**静态优化**（编译时优化）、**动态优化**（运行时优化）。

- 等价变换策略
    - 5=5 and a>5 改成 a > 5
    - a < b and a=5 改成b>5 and a=5
    - 基于联合索引，调整条件位置等
- 优化count、min、max等函数
    - InnoDB引擎min函数只需要找索引最左边
    - InnoDB引擎max函数只需要找索引最右边
    - MyISAM引擎count(*)，不需要计算，直接返回
- 提前终止查询
    - 使用了limit查询，获取limit所需的数据，就不在继续遍历后面数据
- in的优化
    - MySQL对in查询，会先进行排序，再采用二分法查找数据。比如where id in (2,1,3)，变成 in (1,2,3)

#### 5）、查询执行引擎

查询执行引擎负责执行 SQL 语句，此时查询执行引擎会根据 SQL 语句中表的存储引擎类型，以及对应的API接口与底层存储引擎缓存或者物理文件的交互，得到查询结果并返回给客户端。若开启用查询缓存，这时会将SQL 语句和结果完整地保存到查询缓存（Cache&Buffer）中，以后若有相同的 SQL 语句执行则直接返回结果。

- 如果开启了查询缓存，先将查询结果做缓存操作
- 返回结果过多，采用增量模式返回

### 1.4、MySQL存储引擎

存储引擎在MySQL的体系架构中位于第三层，负责MySQL中的数据的存储和提取，是与文件打交道的子系统，它是根据MySQL提供的文件访问层抽象接口定制的一种文件访问机制，这种机制就叫作存储引擎。

使用 **show engines** 命令，就可以查看当前数据库支持的引擎信息。

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-engines-001.png)

**在5.5版本之前默认采用MyISAM存储引擎，从5.5开始采用InnoDB存储引擎。**

- InnoDB：**支持事务**，具有**提交**，**回滚**和**崩溃恢复**能力，**事务安全**
- MyISAM：不支持事务和外键，**访问速度快**
- Memory：利用内存创建表，访问速度非常快，因为数据在内存，而且默认使用Hash索引，但是一旦关闭，数据就会丢失
- Archive：归档类型引擎，仅能支持insert和select语句
- csv：以CSV文件进行数据存储，由于文件限制，所有列必须强制指定not null，另外CSV引擎也不支持索引和分区，适合做数据交换的中间表
- BlackHole: 黑洞，只进不出，进来消失，所有插入数据都不会保存
- Federated：可以访问远端MySQL数据库中的表。一个本地表，不保存数据，访问远程表内容。
- MRG_MyISAM：一组MyISAM表的组合，这些MyISAM表必须结构相同，Merge表本身没有数据，对Merge操作可以对一组MyISAM表进行操作。

#### 1)、InnoDB和MyISAM对比

InnoDB和MyISAM是使用MySQL时最常用的两种引擎类型，我们重点来看下两者区别。

- 事务和外键

InnoDB支持事务和外键，具有安全性和完整性，适合大量insert或update操作

MyISAM不支持事务和外键，它提供高速存储和检索，适合大量的select查询操作

- 锁机制

InnoDB支持行级锁，锁定指定记录。基于索引来加锁实现。

MyISAM支持表级锁，锁定整张表。

- 索引结构

InnoDB使用聚集索引（聚簇索引），索引和记录在一起存储，既缓存索引，也缓存记录。

MyISAM使用非聚集索引（非聚簇索引），索引和记录分开。

- 并发处理能力

MyISAM使用表锁，会导致写操作并发率低，读之间并不阻塞，读写阻塞。

InnoDB读写阻塞可以与隔离级别有关，可以采用 [多版本并发控制](https://baike.baidu.com/item/MVCC/6298019)（MVCC）来支持高并发

- 存储文件

InnoDB表对应两个文件，一个.frm表结构文件，一个.ibd数据文件。InnoDB表最大支持64TB；

MyISAM表对应三个文件，一个.frm表结构文件，一个MYD表数据文件，一个.MYI索引文件。从MySQL5.0开始默认限制是256TB。

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-engines-002.png)

- 适用场景
    - MyISAM
        - 不需要事务支持（不支持）
        - 并发相对较低（锁定机制问题）
        - 数据修改相对较少，以读为主
        - 数据一致性要求不高
    - InnoDB
        - 需要事务支持（具有较好的事务特性）
        - 行级锁定对高并发有很好的适应能力
        - 数据更新较为频繁的场景
        - 数据一致性要求较高
        - 硬件设备内存较大，可以利用InnoDB较好的缓存能力来提高内存利用率，减少磁盘IO
- 总结
    - 两种引擎该如何选择？
        - 是否需要事务？有，InnoDB
        - 是否存在并发修改？有，InnoDB
        - 是否追求快速查询，且数据修改少？是，MyISAM
        - 在绝大多数情况下，推荐使用InnoDB

- 扩展资料：各个存储引擎特性对比

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-engines-003.png)

#### 2)、InnoDB存储结构

从MySQL 5.5版本开始默认使用InnoDB作为引擎，它擅长处理事务，具有自动崩溃恢复的特性，在日常开发中使用非常广泛。下面是官方的InnoDB引擎架构图，主要分为内存结构和磁盘结构两大部分。

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-innodb-buffer-001.png)

- **InnoDB内存结构**（内存结构主要包括Buffer Pool、Change Buffer、Adaptive Hash Index和Log Buffer四大组件）

    - Buffer Pool：缓冲池，简称BP。BP以Page页为单位，默认大小16K，BP的底层采用 **链表** 数据结构管理Page。在InnoDB访问表记录和索引时会在Page页中缓存，以后使用可以减少磁盘IO操作，提升效率。
        
        - Page管理机制

        Page根据状态可以分为三种类型：

            - free page ：空闲page，未被使用
            - clean page：被使用page，数据没有被修改过
            - dirty page：脏页，被使用page，数据被修改过，页中数据和磁盘的数据产生了不一致
        
        针对上述三种page类型，InnoDB通过三种链表结构来维护和管理
            
            - free list ：表示空闲缓冲区，管理free page
            - flush list：表示需要刷新到磁盘的缓冲区，管理dirty page，内部page按修改时间排序。脏页即存在于flush链表，也在LRU链表中，但是两种互不影响，LRU链表负责管理page的可用性和释放，而flush链表负责管理脏页的刷盘操作。
            - lru list：表示正在使用的缓冲区，管理clean page和dirty page，缓冲区以midpoint为基点，前面链表称为new列表区，存放经常访问的数据，占63%；后面的链表称为old列表区，存放使用较少数据，占37%。

        - 改进型LRU算法维护
            - 普通LRU：末尾淘汰法，新数据从链表头部加入，释放空间时从末尾淘汰
            - 改性LRU：链表分为new和old两个部分，加入元素时并不是从表头插入，而是从中间midpoint位置插入，如果数据很快被访问，那么page就会向new列表头部移动，如果数据没有被访问，会逐步向old尾部移动，等待淘汰。
            
            每当有新的数据需要使用buffer pool中的page时，InnoDb引擎会判断是否有空闲页，是否足够，如果有就将free page从free list列表删除，放入到LRU列表中。没有空闲页，就会根据LRU算法淘汰LRU链表默认的页，将内存空间释放分配给新的页。
        
        - Buffer Pool配置参数
            - show variables like '%innodb_page_size%'; //查看page页大小
            - show variables like '%innodb_old%'; //查看lru list中old列表参数
            - show variables like '%innodb_buffer%'; //查看buffer pool参数

            建议：将innodb_buffer_pool_size设置为总内存大小的60%-80%，innodb_buffer_pool_instances可以设置为多个，这样可以避免缓存争夺。

    - Change Buffer：写缓冲区，简称CB。在进行DML操作时，如果BP没有其相应的Page数据，并不会立刻将磁盘页加载到缓冲池，而是在CB记录缓冲变更，等未来数据被读取时，再将数据合并恢复到BP中
        - ChangeBuffer占用BufferPool空间，默认占25%，最大允许占50%，可以根据读写业务量来进行调整。参数innodb_change_buffer_max_size;
        - 当更新一条记录时，该记录在BufferPool存在，直接在BufferPool修改，一次内存操作。如果该记录在BufferPool不存在（没有命中），会直接在ChangeBuffer进行一次内存操作，不用再去磁盘查询数据，避免一次磁盘IO。当下次查询记录时，会先进性磁盘读取，然后再从ChangeBuffer中读取信息合并，最终载入BufferPool中。
        - 写缓冲区，仅适用于非唯一普通索引页，为什么？
            - 如果在索引设置唯一性，在进行修改时，InnoDB必须要做唯一性校验，因此必须查询磁盘，做一次IO操作。会直接将记录查询到BufferPool中，然后在缓冲池修改，不会在ChangeBuffer操作。
    
    - Adaptive Hash Index：自适应哈希索引，用于优化对BP数据的查询。InnoDB存储引擎会监控对表索引的查找，如果观察到建立哈希索引可以带来速度的提升，则建立哈希索引，所以称之为自适应。InnoDB存储引擎会自动根据访问的频率和模式来为某些页建立哈希索引。

    - Log Buffer：日志缓冲区，用来保存要写入磁盘上log文件（Redo/Undo）的数据，日志缓冲区的内容定期刷新到磁盘log文件中。日志缓冲区满时会自动将其刷新到磁盘，当遇到BLOB或多行更新的大事务操作时，增加日志缓冲区可以节省磁盘I/O。
        - LogBuffer主要是用于记录InnoDB引擎日志，在DML操作时会产生Redo和Undo日志。
        - LogBuffer空间满了，会自动写入磁盘。可以通过将innodb_log_buffer_size参数调大，减少磁盘IO频率
        - innodb_flush_log_at_trx_commit参数控制日志刷新行为，默认为1
            - 0 ： 每隔1秒写日志文件和刷盘操作（写日志文件LogBuffer--> **OS cache**，刷盘OScache-->磁盘文件），最多丢失1秒数据
            - 1 ： 事务提交，立刻写日志文件和刷盘，数据不丢失，但是会频繁IO操作
            - 2 ： 事务提交，立刻写日志文件，每隔1秒钟进行刷盘操作

- **InnoDB磁盘结构**（InnoDB磁盘主要包含Tablespaces，InnoDB Data Dictionary，Doublewrite Buffer、Redo Log
和Undo Logs。）

    - 表空间（Tablespaces）：用于存储表结构和数据。表空间又分为系统表空间、独立表空间、通用表空间、临时表空间、Undo表空间等多种类型；
        - 系统表空间（The System Tablespace）
        包含InnoDB数据字典，Doublewrite Buffer，Change Buffer，Undo Logs的存储区域。系统表空间也默认包含任何用户在系统表空间创建的表数据和索引数据。系统表空间是一个共享的表空间因为它是被多个表共享的。该空间的数据文件通过参数innodb_data_file_path控制，默认值是ibdata1:12M:autoextend(文件名为ibdata1、12MB、自动扩展)。
        - 独立表空间（File-Per-Table Tablespaces）
        默认开启，独立表空间是一个单表表空间，该表创建于自己的数据文件中，而非创建于系统表空间中。当innodb_file_per_table选项开启时，表将被创建于表空间中。否则，innodb将被创建于系统表空间中。每个表文件表空间由一个.ibd数据文件代表，该文件默认被创建于数据库目录中。表空间的表文件支持动态（dynamic）和压缩（commpressed）行格式。
        - 通用表空间（General Tablespaces）
        通用表空间为通过create tablespace语法创建的共享表空间。通用表空间可以创建于mysql数据目录外的其他表空间，其可以容纳多张表，且其支持所有的行格式。
        
        CREATE TABLESPACE ts1 ADD DATAFILE ts1.ibd Engine=InnoDB; //创建表空间ts1
        CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1; //将表添加到ts1表空间
        
        - 撤销表空间（Undo Tablespaces）
        撤销表空间由一个或多个包含Undo日志文件组成。在MySQL 5.7版本之前Undo占用的是System Tablespace共享区，从5.7开始将Undo从System Tablespace分离了出来。InnoDB使用的undo表空间由innodb_undo_tablespaces配置选项控制，默认为0。参数值为0表示使用系统表空间ibdata1;大于0表示使用undo表空间undo_001、undo_002等。
        - 临时表空间（Temporary Tablespaces）
        分为session temporary tablespaces 和global temporary tablespace两种。session temporary tablespaces 存储的是用户创建的临时表和磁盘内部的临时表。global temporary tablespace储存用户临时表的回滚段（rollback segments ）。mysql服务器正常关闭或异常终止时，临时表空间将被移除，每次启动时会被重新创建。
    - 数据字典（InnoDB Data Dictionary）
    InnoDB数据字典由内部系统表组成，这些表包含用于查找表、索引和表字段等对象的元数据。元数据物理上位于InnoDB系统表空间中。由于历史原因，数据字典元数据在一定程度上与InnoDB表元数据文件（.frm文件）中存储的信息重叠。
    - 双写缓冲区（Doublewrite Buffer）
    位于系统表空间，是一个存储区域。在BufferPage的page页刷新到磁盘真正的位置前，会先将数据存在Doublewrite 缓冲区。如果在page页写入过程中出现操作系统、存储子系统或mysqld进程崩溃，InnoDB可以在崩溃恢复期间从Doublewrite 缓冲区中找到页面的一个好备份。在大多数情况下，默认情况下启用双写缓冲区，要禁用Doublewrite 缓冲区，可以将innodb_doublewrite设置为0。使用Doublewrite 缓冲区时建议将innodb_flush_method设置为O_DIRECT。
        - MySQL的innodb_flush_method这个参数控制着innodb数据文件及redo log的打开、刷写模式。有三个值：fdatasync(默认)，O_DSYNC，O_DIRECT。设置O_DIRECT表示数据文件写入操作会通知操作系统不要缓存数据，也不要用预读，直接从InnodbBuffer写到磁盘文件。
        - 默认的fdatasync意思是先写入操作系统缓存，然后再调用fsync()函数去异步刷数据文件与redo log的缓存信息。
    - 重做日志（Redo Log）
    重做日志是一种基于磁盘的数据结构，用于在崩溃恢复期间更正不完整事务写入的数据。MySQL以循环方式写入重做日志文件，记录InnoDB中所有对Buffer Pool修改的日志。当出现实例故障（像断电），导致数据未能更新到数据文件，则数据库重启时须redo，重新把数据更新到数据文件。读写事务在执行的过程中，都会不断的产生redo log。默认情况下，重做日志在磁盘上由两个名为ib_logfile0和ib_logfile1的文件物理表示。
    - 撤销日志（Undo Logs）
    撤消日志是在事务开始之前保存的被修改数据的备份，用于例外情况时回滚事务。撤消日志属于逻辑日志，根据每行记录进行记录。撤消日志存在于系统表空间、撤消表空间和临时表空间中。

- **新版本结构演变**
    ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-innodb-buffer-002.png)
    - MySQL 5.7 版本
        - 将 Undo日志表空间从共享表空间 ibdata 文件中分离出来，可以在安装 MySQL 时由用户自行指定文件大小和数量。
        - 增加了 temporary 临时表空间，里面存储着临时表或临时查询结果集的数据。
        - Buffer Pool 大小可以动态修改，无需重启数据库实例。
    - MySQL 8.0 版本
        - 将InnoDB表的数据字典和Undo都从共享表空间ibdata中彻底分离出来了，以前需要ibdata中数据字典与独立表空间ibd文件中数据字典一致才行，8.0版本就不需要了。
        - temporary 临时表空间也可以配置多个物理文件，而且均为 InnoDB 存储引擎并能创建索引，这样加快了处理的速度。
        - 用户可以像 Oracle 数据库那样设置一些表空间，每个表空间对应多个物理文件，每个表空间可以给多个表使用，但一个表只能存储在一个表空间中。
        - 将Doublewrite Buffer从共享表空间ibdata中也分离出来了。

#### 3)、InnoDB线程模型
    
![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-innodb-thread-001.png)
- **IO Thread**
在InnoDB中使用了大量的AIO（Async IO）来做读写处理，这样可以极大提高数据库的性能。在InnoDB1.0版本之前共有4个IO Thread，分别是write，read，insert buffer和log thread，后来版本将read thread和write thread分别增大到了4个，一共有10个了。
    - read thread ： 负责读取操作，将数据从磁盘加载到缓存page页。4个
    - write thread：负责写操作，将缓存脏页刷新到磁盘。4个
    - log thread：负责将日志缓冲区内容刷新到磁盘。1个
    - insert buffer thread ：负责将写缓冲内容刷新到磁盘。1个

- **Purge Thread**
事务提交之后，其使用的undo日志将不再需要，因此需要Purge Thread回收已经分配的undo页。
show variables like '%innodb_purge_threads%';

- **Page Cleaner Thread**
作用是将脏数据刷新到磁盘，脏数据刷盘后相应的redo log也就可以覆盖，即可以同步数据，又能达到redo log循环使用的目的。会调用write thread线程处理。
show variables like '%innodb_page_cleaners%';

- **Master Thread**
Master thread是InnoDB的主线程，负责调度其他各线程，优先级最高。作用是将缓冲池中的数据异步刷新到磁盘 ，保证数据的一致性。包含：脏页的刷新（page cleaner thread）、undo页回收（purge thread）、redo日志刷新（log thread）、合并写缓冲等。内部有两个主处理，分别是每隔1秒和10秒处理。
    - 每1秒的操作：
        - 刷新日志缓冲区，刷到磁盘
        - 合并写缓冲区数据，根据IO读写压力来决定是否操作
        - 刷新脏页数据到磁盘，根据脏页比例达到75%才操作（innodb_max_dirty_pages_pct，innodb_io_capacity）

    - 每10秒的操作：
        - 刷新脏页数据到磁盘
        - 合并写缓冲区数据
        - 刷新日志缓冲区
        - 删除无用的undo页

#### 4)、InnoDB数据文件

- **InnoDB文件存储结构**
![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-tablespaces.png)
InnoDB数据文件存储结构：分为一个ibd数据文件-->Segment（段）-->Extent（区）-->Page（页）-->Row（行）
    - **Tablesapce**
    表空间，用于存储多个ibd数据文件，用于存储表的记录和索引。一个文件包含多个段。
    - **Segment**
    段，用于管理多个Extent，分为数据段（Leaf node segment）、索引段（Non-leaf nodesegment）、回滚段（Rollback segment）。一个表至少会有两个segment，一个管理数据，一个管理索引。每多创建一个索引，会多两个segment。
    - **Extent**
    区，一个区固定包含64个连续的页，大小为1M。当表空间不足，需要分配新的页资源，不会一页一页分，直接分配一个区。
    - **Page**
    页，用于存储多个Row行记录，大小为16K。包含很多种页类型，比如数据页，undo页，系统页，事务数据页，大的BLOB对象页。
    - **Row**
    行，包含了记录的字段值，事务ID（Trx id）、滚动指针（Roll pointer）、字段指针（Field pointers）等信息。

Page是文件最基本的单位，无论何种类型的page，都是由page header，page trailer和page body组成。如下图所示

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-tablespaces-page.png)

- **InnoDB文件存储格式**
    - 通过 SHOW TABLE STATUS 命令
    ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-file-format.png)

    一般情况下，如果row_format为REDUNDANT、COMPACT，文件格式为Antelope；如果row_format为DYNAMIC和COMPRESSED，文件格式为Barracuda。
        - 通过 information_schema 查看指定表的文件格式
        ```mysql
            select * from information_schema.innodb_sys_tables;
        ```

- **File文件格式（File-Format）**
在早期的InnoDB版本中，文件格式只有一种，随着InnoDB引擎的发展，出现了新文件格式，用于支持新的功能。目前InnoDB只支持两种文件格式：Antelope 和 Barracuda。
    - Antelope: 先前未命名的，最原始的InnoDB文件格式，它支持两种行格式：COMPACT和REDUNDANT，MySQL 5.6及其以前版本默认格式为Antelope。
    - Barracuda: 新的文件格式。它支持InnoDB的所有行格式，包括新的行格式：COMPRESSED和 DYNAMIC。
通过innodb_file_format 配置参数可以设置InnoDB文件格式，之前默认值为Antelope，5.7版本开始改为Barracuda。

- **Row行格式（Row_format）**
表的行格式决定了它的行是如何物理存储的，这反过来又会影响查询和DML操作的性能。如果在单个page页中容纳更多行，查询和索引查找可以更快地工作，缓冲池中所需的内存更少，写入更新时所需的I/O更少。

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-row-format.png)

- InnoDB存储引擎支持四种行格式：REDUNDANT、COMPACT、DYNAMIC和COMPRESSED。
DYNAMIC和COMPRESSED新格式引入的功能有：**数据压缩**、**增强型长列数据的页外存储**和**大索引前缀**。每个表的数据分成若干页来存储，每个页中采用B树结构存储；如果某些字段信息过长，无法存储在B树节点中，这时候会被单独分配空间，此时被称为溢出页，该字段被称为页外列。
    - REDUNDANT 行格式
    使用REDUNDANT行格式，表会将变长列值的前768字节存储在B树节点的索引记录中，其余的存储在溢出页上。对于大于等于786字节的固定长度字段InnoDB会转换为变长字段，以便能够在页外存储。
    - COMPACT 行格式
    与REDUNDANT行格式相比，COMPACT行格式减少了约20%的行存储空间，但代价是增加了某些操作的CPU使用量。如果系统负载是受缓存命中率和磁盘速度限制，那么COMPACT格式可能更快。如果系统负载受到CPU速度的限制，那么COMPACT格式可能会慢一些。
    - DYNAMIC 行格式
    使用DYNAMIC行格式，InnoDB会将表中长可变长度的列值完全存储在页外，而索引记录只包含指向溢出页的20字节指针。大于或等于768字节的固定长度字段编码为可变长度字段。DYNAMIC行格式支持大索引前缀，最多可以为3072字节，可通过innodb_large_prefix参数控制。
    - COMPRESSED 行格式
    COMPRESSED行格式提供与DYNAMIC行格式相同的存储特性和功能，但增加了对表和索引数据压缩的支持。

在创建表和索引时，文件格式都被用于每个InnoDB表数据文件（其名称与*.ibd匹配）。修改文件格式的方法是重新创建表及其索引，最简单方法是对要修改的每个表使用以下命令：
    - ALTER TABLE 表名 ROW_FORMAT=格式类型; 

#### 5)、Undo Log

- **Undo Log介绍**
    - Undo：意为撤销或取消，以撤销操作为目的，返回指定某个状态的操作。
    - Undo Log：数据库事务开始之前，会将要修改的记录存放到 Undo 日志里，当事务回滚时或者数据库崩溃时，可以利用 Undo 日志，撤销未提交事务对数据库产生的影响。
    - Undo Log产生和销毁：Undo Log在事务开始前产生；事务在提交时，并不会立刻删除undolog，innodb会将该事务对应的undo log放入到删除列表中，后面会通过后台线程purge thread进行回收处理。Undo Log属于逻辑日志，记录一个变化过程。例如执行一个delete，undolog会记录一个insert；执行一个update，undolog会记录一个相反的update。
    - Undo Log存储：undo log采用段的方式管理和记录。在innodb数据文件中包含一种rollback segment回滚段，内部包含1024个undo log segment。可以通过下面一组参数来控制Undo log存储。
    ```mysql
        show variables like '%innodb_undo%';
    ```

- **Undo Log作用**
    - 实现事务的原子性
        - Undo Log 是为了实现事务的原子性而出现的产物。事务处理过程中，如果出现了错误或者用户执行了 ROLLBACK 语句，MySQL 可以利用 Undo Log 中的备份将数据恢复到事务开始之前的状态。
    - 实现多版本并发控制（MVCC）
        - Undo Log 在 MySQL InnoDB 存储引擎中用来实现多版本并发控制。事务未提交之前，Undo Log 保存了未提交之前的版本数据，Undo Log 中的数据可作为数据旧版本快照供其他并发事务进行快照读。
        ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-undo-log.png)

    事务A手动开启事务，执行更新操作，首先会把更新命中的数据备份到 Undo Buffer 中。
    事务B手动开启事务，执行查询操作，会读取 Undo 日志数据返回，进行快照读

#### 6)、Redo Log和Binlog

Redo Log和Binlog是MySQL日志系统中非常重要的两种机制，也有很多相似之处，下面介绍下两者细节和区别。

- **Redo Log日志**
    - Redo Log介绍
        - Redo：顾名思义就是重做。以恢复操作为目的，在数据库发生意外时重现操作。
        - Redo Log：指事务中修改的任何数据，将最新的数据备份存储的位置（Redo Log），被称为重做日志。
        - Redo Log 的生成和释放：随着事务操作的执行，就会生成Redo Log，在事务提交时会将产生Redo Log写入Log Buffer，并不是随着事务的提交就立刻写入磁盘文件。等事务操作的脏页写入到磁盘之后，Redo Log 的使命也就完成了，Redo Log占用的空间就可以重用（被覆盖写入）。
    - Redo Log工作原理
    Redo Log 是为了实现事务的持久性而出现的产物。防止在发生故障的时间点，尚有脏页未写入表的 IBD 文件中，在重启 MySQL 服务的时候，根据 Redo Log 进行重做，从而达到事务的未入磁盘数据进行持久化这一特性。
    ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-redo-log.png)
    - Redo Log写入机制
    Redo Log 文件内容是以顺序循环的方式写入文件，写满时则回溯到第一个文件，进行覆盖写。

    ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-redo-log-write.png)

    如图所示：
        - write pos 是当前记录的位置，一边写一边后移，写到最后一个文件末尾后就回到 0 号文件开头；
        - checkpoint 是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件；

        write pos 和 checkpoint 之间还空着的部分，可以用来记录新的操作。如果 write pos 追上checkpoint，表示写满，这时候不能再执行新的更新，得停下来先擦掉一些记录，把 checkpoint推进一下。
    
    以上机制为 crash-safe ，Innodb 出现故障时的自动恢复机制。
    - Redo Log相关配置参数

    每个InnoDB存储引擎至少有1个重做日志文件组（group），每个文件组至少有2个重做日志文件，默认为ib_logfile0和ib_logfile1。可以通过下面一组参数控制Redo Log存储：
    
        - show variables like '%innodb_log%';
    Redo Buffer 持久化到 Redo Log 的策略，可通过 Innodb_flush_log_at_trx_commit 设置：
        - 0：每秒提交 Redo buffer ->OS cache -> flush cache to disk，可能丢失一秒内的事务数据。由后台Master线程每隔 1秒执行一次操作。
        - 1（默认值）：每次事务提交执行 Redo Buffer -> OS cache -> flush cache to disk，最安全，性能最差的方式。
        - 2：每次事务提交执行 Redo Buffer -> OS cache，然后由后台Master线程再每隔1秒执行OS cache -> flush cache to disk 的操作。
    一般建议选择取值2，因为 MySQL 挂了数据没有损失，整个服务器挂了才会损失1秒的事务提交数据。
    ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-redo-type.png)

- **Binlog日志**
    - Binlog记录模式
    Redo Log 是属于InnoDB引擎所特有的日志，而MySQL Server也有自己的日志，即 Binarylog（二进制日志），简称Binlog。Binlog是记录所有数据库表结构变更以及表数据修改的二进制日志，不会记录SELECT和SHOW这类操作。Binlog日志是以事件形式记录，还包含语句所执行的消耗时间。开启Binlog日志有以下两个最重要的使用场景。

        - 主从复制：在主库中开启Binlog功能，这样主库就可以把Binlog传递给从库，从库拿到Binlog后实现数据恢复达到主从数据一致性。
        - 数据恢复：通过mysqlbinlog工具来恢复数据。

        Binlog文件名默认为“主机名_binlog-序列号”格式，例如oak_binlog-000001，也可以在配置文件中指定名称。文件记录模式有STATEMENT、ROW和MIXED三种，具体含义如下。
        
        - ROW（row-based replication, RBR）：日志中会记录每一行数据被修改的情况，然后在slave端对相同的数据进行修改。
            - 优点：能清楚记录每一个行数据的修改细节，能完全实现主从数据同步和数据的恢复。
            - 缺点：批量操作，会产生大量的日志，尤其是alter table会让日志暴涨。
      
        - STATMENT（statement-based replication, SBR）：每一条被修改数据的SQL都会记录到master的Binlog中，slave在复制的时候SQL进程会解析成和原来master端执行过的相同的SQL再次执行。简称SQL语句复制。
            - 优点：日志量小，减少磁盘IO，提升存储和恢复速度
            - 缺点：在某些情况下会导致主从数据不一致，比如last_insert_id()、now()等函数。
      
        - MIXED（mixed-based replication, MBR）：以上两种模式的混合使用，一般会使用STATEMENT模式保存binlog，对于STATEMENT模式无法复制的操作使用ROW模式保存binlog，MySQL会根据执行的SQL语句选择写入模式。
    
    - Binlog文件结构
    MySQL的binlog文件中记录的是对数据库的各种修改操作，用来表示修改操作的数据结构是Logevent。不同的修改操作对应的不同的log event。比较常用的log event有：Query event、Row event、Xid event等。binlog文件的内容就是各种Log event的集合。
    Binlog文件中Log event结构如下图所示：
        ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-binlog.png)
    - Binlog写入机制
        - 根据记录模式和操作触发event事件生成log event（事件触发执行机制）
        - 将事务执行过程中产生log event写入缓冲区，每个事务线程都有一个缓冲区
        Log Event保存在一个binlog_cache_mngr数据结构中，在该结构中有两个缓冲区，一个是stmt_cache，用于存放不支持事务的信息；另一个是trx_cache，用于存放支持事务的信息。
        - 事务在提交阶段会将产生的log event写入到外部binlog文件中。
        不同事务以串行方式将log event写入binlog文件中，所以一个事务包含的log event信息在binlog文件中是连续的，中间不会插入其他事务的log event。
    - Binlog文件操作
        - Binlog状态查看
            ```mysql
                show variables like 'log_bin';
            ```
        - 开启Binlog功能
            ```mysql
                mysql> set global log_bin=mysqllogbin;
                ERROR 1238 (HY000): Variable 'log_bin' is a read only variable
            ``` 
            需要修改my.cnf或my.ini配置文件，在[mysqld]下面增加log_bin=mysql_bin_log，重启MySQL服务。
            ```properties
                #log-bin=ON
                #log-bin-basename=mysqlbinlog
                binlog-format=ROW
                log-bin=mysqlbinlog
            ```
        - 使用show binlog events命令
            ```mysql
                show binary logs; //等价于show master logs;
                show master status;
                show binlog events;
                show binlog events in 'mysqlbinlog.000001';
            ```
        - 使用mysqlbinlog 命令
            ```mysql
                mysqlbinlog "文件名"
                mysqlbinlog "文件名" > "test.sql"
            ```
        - 使用 binlog 恢复数据
            ```mysql
                //按指定时间恢复
                mysqlbinlog --start-datetime="2020-04-25 18:00:00" --stop-
                datetime="2020-04-26 00:00:00" mysqlbinlog.000002 | mysql -uroot -p1234
                //按事件位置号恢复
                mysqlbinlog --start-position=154 --stop-position=957 mysqlbinlog.000002
                | mysql -uroot -p1234
            ```
            mysqldump：定期全部备份数据库数据。mysqlbinlog可以做增量备份和恢复操作。
        - 删除Binlog文件
        ```mysql
            purge binary logs to 'mysqlbinlog.000001'; //删除指定文件
            purge binary logs before '2020-04-28 00:00:00'; //删除指定时间之前的文件
            reset master; //清除所有文件
        ```
        可以通过设置expire_logs_days参数来启动自动清理功能。默认值为0表示没启用。设置为1表示超出1天binlog文件会自动删除掉。
    - Redo Log和Binlog区别
        - Redo Log是属于InnoDB引擎功能，Binlog是属于MySQL Server自带功能，并且是以二进制文件记录。
        - Redo Log属于物理日志，记录该数据页更新状态内容，Binlog是逻辑日志，记录更新过程。
        - Redo Log日志是循环写，日志空间大小是固定，Binlog是追加写入，写完一个写下一个，不会覆盖使用。
        - Redo Log作为服务器异常宕机后事务数据自动恢复使用，Binlog可以作为主从复制和数据恢复使用。Binlog没有自动crash-safe能力。

## 二、MySQL索引原理

### 2.1、索引类型

索引可以提升查询速度，会影响where查询，以及order by排序。MySQL索引类型如下：

- 从索引存储结构划分：B Tree索引、Hash索引、FULLTEXT全文索引、R Tree索引
- 从应用层次划分：普通索引、唯一索引、主键索引、复合索引
- 从索引键值类型划分：主键索引、辅助索引（二级索引）
- 从数据存储和索引键值逻辑关系划分：聚集索引（聚簇索引）、非聚集索引（非聚簇索引）

#### 1）、普通索引

这是最基本的索引类型，基于普通字段建立的索引，没有任何限制。

创建普通索引的方法如下：

- CREATE INDEX <索引的名字> ON tablename (字段名);
- ALTER TABLE tablename ADD INDEX [索引的名字] (字段名);
- CREATE TABLE tablename ( [...], INDEX [索引的名字] (字段名) );

#### 2）、唯一索引

与"普通索引"类似，不同的就是：索引字段的值必须唯一，但允许有空值 。在创建或修改表时追加唯一约束，就会自动创建对应的唯一索引。

创建唯一索引的方法如下：

- CREATE UNIQUE INDEX <索引的名字> ON tablename (字段名);
- ALTER TABLE tablename ADD UNIQUE INDEX [索引的名字] (字段名);
- CREATE TABLE tablename ( [...], UNIQUE [索引的名字] (字段名) ;

#### 3）、主键索引

它是一种特殊的唯一索引，不允许有空值。在创建或修改表时追加主键约束即可，每个表只能有一个主键。

创建主键索引的方法如下：

- CREATE TABLE tablename ( [...], PRIMARY KEY (字段名) );
- ALTER TABLE tablename ADD PRIMARY KEY (字段名);

#### 4）、复合索引

单一索引是指索引列为一列的情况，即新建索引的语句只实施在一列上；用户可以在多个列上建立索引，这种索引叫做组复合索引（组合索引）。复合索引可以代替多个单一索引，相比多个单一索引复合索引所需的开销更小。

索引同时有两个概念叫做窄索引和宽索引，窄索引是指索引列为1-2列的索引，宽索引也就是索引列超过2列的索引，设计索引的一个重要原则就是能用窄索引不用宽索引，因为窄索引往往比组合索引更有效。

创建组合索引的方法如下：

- CREATE INDEX <索引的名字> ON tablename (字段名1，字段名2...);
- ALTER TABLE tablename ADD INDEX [索引的名字] (字段名1，字段名2...);
- CREATE TABLE tablename ( [...], INDEX [索引的名字] (字段名1，字段名2...) );

复合索引使用注意事项：

- 何时使用复合索引，要根据where条件建索引，注意不要过多使用索引，过多使用会对更新操作效率有很大影响。
- 如果表已经建立了(col1，col2)，就没有必要再单独建立（col1）；如果现在有(col1)索引，如果查询需要col1和col2条件，可以建立(col1,col2)复合索引，对于查询有一定提高。


#### 5）、全文索引

查询操作在数据量比较少时，可以使用like模糊查询，但是对于大量的文本数据检索，效率很低。如果使用全文索引，查询速度会比like快很多倍。在MySQL 5.6 以前的版本，只有MyISAM存储引擎支持全文索引，从MySQL 5.6开始MyISAM和InnoDB存储引擎均支持。

创建全文索引的方法如下：

- CREATE FULLTEXT INDEX <索引的名字> ON tablename (字段名);
- ALTER TABLE tablename ADD FULLTEXT [索引的名字] (字段名);
- CREATE TABLE tablename ( [...], FULLTEXT KEY [索引的名字] (字段名) ;

和常用的like模糊查询不同，全文索引有自己的语法格式，使用 match 和 against 关键字，比如

```mysql
    select * from user where match(name) against('aaa');
```

全文索引使用注意事项：

- 全文索引必须在字符串、文本字段上建立。
- 全文索引字段值必须在最小字符和最大字符之间的才会有效。（innodb：3-84；myisam：4-84）
- 全文索引字段值要进行切词处理，按syntax字符进行切割，例如b+aaa，切分成b和aaa
- 全文索引匹配查询，默认使用的是等值匹配，例如a匹配a，不会匹配ab,ac。如果想匹配可以在布尔模式下搜索a*

```mysql
    select * from user where match(name) against('a*' in boolean mode);
```

### 2.2、索引原理

MySQL官方对索引定义：是存储引擎用于快速查找记录的一种数据结构。需要额外开辟空间和数据维护工作。

- 索引是物理数据页存储，在数据文件中（InnoDB，ibd文件），利用数据页(page)存储。
- 索引可以加快检索速度，但是同时也会降低增删改操作速度，索引维护需要代价。

索引涉及的理论知识：**二分查找法**、**Hash**和**B+Tree**。

#### 1）、二分查找法

二分查找法也叫作折半查找法，它是在有序数组中查找指定数据的搜索算法。它的优点是等值查询、范围查询性能优秀，缺点是更新数据、新增数据、删除数据维护成本高。

- 首先定位left和right两个指针
- 计算(left+right)/2
- 判断除2后索引位置值与目标值的大小比对
- 索引位置值大于目标值就-1，right移动；如果小于目标值就+1，left移动

举个例子，下面的有序数组有17 个值，查找的目标值是7，过程如下：

- 第一次查找

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-half-query-001.png)

- 第二次查找

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-half-query-002.png)

- 第三次查找

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-half-query-003.png)

- 第四次查找

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-half-query-004.png)

#### 2）、Hash结构

Hash底层实现是由Hash表来实现的，是根据键值 <key,value> 存储数据的结构。非常适合根据key查找value值，也就是单个key查询，或者说等值查询。其结构如下所示：

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-hash-query-001.png)

从上面结构可以看出，Hash索引可以方便的提供等值查询，但是对于范围查询就需要全表扫描了。

Hash索引在MySQL中Hash结构主要应用在Memory原生的Hash索引 、InnoDB自适应哈希索引。

InnoDB提供的自适应哈希索引功能强大，接下来重点描述下InnoDB自适应哈希索引。

InnoDB自适应哈希索引是为了提升查询效率，InnoDB存储引擎会监控表上各个索引页的查询，当InnoDB注意到某些索引值访问非常频繁时，会在内存中基于B+Tree索引再创建一个哈希索引，使得内存中的 B+Tree 索引具备哈希索引的功能，即能够快速定值访问频繁访问的索引页。

InnoDB自适应哈希索引：在使用Hash索引访问时，一次性查找就能定位数据，等值查询效率要优于B+Tree。

自适应哈希索引的建立使得InnoDB存储引擎能自动根据索引页访问的频率和模式自动地为某些热点页建立哈希索引来加速访问。另外InnoDB自适应哈希索引的功能，用户只能选择开启或关闭功能，无法进行人工干涉。

```mysql
    show engine innodb status \G;
    show variables like '%innodb_adaptive%';
```

#### 3）、B+Tree结构

MySQL数据库索引采用的是B+Tree结构，在B-Tree结构上做了优化改造。

- **B-Tree结构**
    - 索引值和data数据分布在整棵树结构中
    - 每个节点可以存放多个索引值及对应的data数据
    - 树节点中的多个索引值从左到右升序排列
    ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-b-tree-001.png)

    B树的搜索：从根节点开始，对节点内的索引值序列采用二分法查找，如果命中就结束查找。没有命中会进入子节点重复查找过程，直到所对应的的节点指针为空，或已经是叶子节点了才结束。

- **B+Tree结构**
    - 非叶子节点不存储data数据，只存储索引值，这样便于存储更多的索引值
    - 叶子节点包含了所有的索引值和data数据
    - 叶子节点用指针连接，提高区间的访问性能
    ![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-b+tree-001.png)

    相比B树，B+树进行范围查找时，只需要查找定位两个节点的索引值，然后利用叶子节点的指针进行遍历即可。而B树需要遍历范围内所有的节点和数据，显然B+Tree效率高。

#### 4）、聚簇索引和辅助索引 

**聚簇索引和非聚簇索引**：B+Tree的叶子节点存放主键索引值和行记录就属于聚簇索引；如果索引值和行记录分开存放就属于非聚簇索引。

**主键索引和辅助索引**：B+Tree的叶子节点存放的是主键字段值就属于主键索引；如果存放的是非主键值就属于辅助索引（二级索引）。

在InnoDB引擎中，主键索引采用的就是聚簇索引结构存储。

- 聚簇索引（聚集索引）

聚簇索引是一种数据存储方式，InnoDB的聚簇索引就是按照主键顺序构建 B+Tree结构。B+Tree的叶子节点就是行记录，行记录和主键值紧凑地存储在一起。 这也意味着 InnoDB 的主键索引就是数据表本身，它按主键顺序存放了整张表的数据，占用的空间就是整个表数据量的大小。通常说的**主键索引**就是聚集索引。

InnoDB的表要求必须要有聚簇索引：

1. 如果表定义了主键，则主键索引就是聚簇索引
2. 如果表没有定义主键，则第一个非空unique列作为聚簇索引
3. 否则InnoDB会从建一个隐藏的row-id作为聚簇索引

- 辅助索引

InnoDB辅助索引，也叫作二级索引，是根据索引列构建 B+Tree结构。但在 B+Tree 的叶子节点中只存了索引列和主键的信息。二级索引占用的空间会比聚簇索引小很多， 通常创建辅助索引就是为了提升查询效率。一个表InnoDB只能创建一个聚簇索引，但可以创建多个辅助索引。

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-index-001.png)

- 非聚簇索引

与InnoDB表存储不同，MyISAM数据表的索引文件和数据文件是分开的，被称为非聚簇索引结构。

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-index-002.png)


### 2.3、索引分析与优化

#### 1）、EXPLAIN

MySQL 提供了一个 EXPLAIN 命令，它可以对 SELECT 语句进行分析，并输出 SELECT 执行的详细信息，供开发人员有针对性的优化。例如：

```mysql
    EXPLAIN SELECT * from user WHERE id < 3;
```
EXPLAIN 命令的输出内容大致如下：

![08-qa-mysql#001](../_media/images/08-qa-mysql/qa-mysql-explain-001.png)

- select_type
    - 表示查询的类型。常用的值如下：
        - SIMPLE ： 表示查询语句不包含子查询或union
        - PRIMARY：表示此查询是最外层的查询
        - UNION：表示此查询是UNION的第二个或后续的查询 
        - DEPENDENT UNION：UNION中的第二个或后续的查询语句，使用了外面查询结果
        - UNION RESULT：UNION的结果
        - SUBQUERY：SELECT子查询语句
        - DEPENDENT SUBQUERY：SELECT子查询语句依赖外层查询的结果。

最常见的查询类型是SIMPLE，表示我们的查询没有子查询也没用到UNION查询。

- type 
    - 表示存储引擎查询数据时采用的方式。比较重要的一个属性，通过它可以判断出查询是全表扫描还是基于索引的部分扫描。常用属性值如下，从上至下效率依次增强。
        - ALL：表示全表扫描，性能最差。
        - index：表示基于索引的全表扫描，先扫描索引再扫描全表数据。
        - range：表示使用索引范围查询。使用>、>=、<、<=、in等等。
        - ref：表示使用非唯一索引进行单值查询。
        - eq_ref：一般情况下出现在多表join查询，表示前面表的每一个记录，都只能匹配后面表的一行结果。
        - const：表示使用主键或唯一索引做等值查询，常量查询。
        - NULL：表示不用访问表，速度最快。

- possible_keys
    - 表示查询时能够使用到的索引。注意并不一定会真正使用，显示的是索引名称。

- key
    - 表示查询时真正使用到的索引，显示的是索引名称。

- rows
    - MySQL查询优化器会根据统计信息，估算SQL要查询到结果需要扫描多少行记录。原则上rows是越少效率越高，可以直观的了解到SQL效率高低。

- key_len
    - 表示查询使用了索引的字节数量。可以判断是否全部使用了组合索引。
    key_len的计算规则如下：
        - 字符串类型
            - 字符串长度跟字符集有关：latin1=1、gbk=2、utf8=3、utf8mb4=4
            - char(n)：n*字符集长度
            - varchar(n)：n * 字符集长度 + 2字节
        - 数值类型
            - TINYINT：1个字节
            - SMALLINT：2个字节
            - MEDIUMINT：3个字节
            - INT、FLOAT：4个字节
            - BIGINT、DOUBLE：8个字节
        - 时间类型
            - DATE：3个字节
            - TIMESTAMP：4个字节
            - DATETIME：8个字节
        - 字段属性
            - NULL属性占用1个字节，如果一个字段设置了NOT NULL，则没有此项。

- Extra
    - Extra表示很多额外的信息，各种操作会在Extra提示相关信息，常见几种如下：
        - Using where
            - 表示查询需要通过索引回表查询数据。
        - Using index
            - 表示查询需要通过索引，索引就可以满足所需数据。
        - Using filesort
            - 表示查询出来的结果需要额外排序，数据量小在内存，大的话在磁盘，因此有Using filesort 建议优化。
        - Using temprorary
            - 查询使用到了临时表，一般出现于去重、分组等操作。

#### 2）、回表查询
#### 3）、覆盖索引
#### 4）、最左前缀原则
#### 5）、LIKE查询
#### 6）、NULL查询
#### 7）、索引与排序
#### 8）、LIKE查询


### 2.4、查询优化







## 三、MySQL事务和锁

### 3.1、ACID特性

### 3.2、事务控制的演进

### 3.3、事务隔离级别

### 3.4、锁机制与实战




## 四、MySQL集群架构

### 4.1、集群架构设计

### 4.2、主从模式

### 4.3、双主模式

### 4.4、分库分表






## 五、分库分表实战及中间件

### 5.1、分库分表背景

### 5.2、ShardingSphere

### 5.3、MyCat




## 六、运维和第三方工具

### 6.1、Yearning 

### 6.2、canal

### 6.3、DataX

### 6.4、percona-toolkit

### 6.5、MySQLMTOP

### 6.6、ELK

### 6.7、Prometheus 






## 七、推荐阅读

- [《MySQL 实战 45 讲》](https://time.geekbang.org/column/intro/100020801?tab=catalog) 林晓斌 网名丁奇，腾讯云数据库负责人
- [《MySQL 是怎样运行的：从根儿上理解 MySQL》](https://juejin.cn/book/6844733769996304392/) 小孩子4919 公众号 『我们都是小青蛙』