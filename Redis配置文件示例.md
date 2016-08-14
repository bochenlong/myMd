```
# Redis 配置文件示例
#
# 为了加载配置，Redis必须以配置文件作为第一参数启动。
# ./redis-server /path/to/redis.conf

# 当你需要配置内存大小的时候，必须要带上单位，
# 通常的格式就是 1k 5gb 4M 等酱紫：
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# 单位不区分大小写，你写1GB 1Gb 1gB 是一样的的。

################################## INCLUDES ###################################

# 假如说你有一个可用于所有的 redis server 的标准配置模板，
# 但针对某些 server 又需要一些个性化的设置，
# 你可以使用 include 来包含一些其他的配置文件，这对你来说是非常有用的。
#
# 但是要注意哦，include 是不能被 config rewrite 命令改写的
# 由于 redis 总是以最后的加工线作为一个配置指令值，所以你最好是把 include 放在这个文件的最前面，
# 以避免在运行时覆盖配置的改变，相反，你就把它放在后面（其实就是你把标准模板放在前面，需要特殊配置的写在后面会覆盖标准配置）。
#
# include /path/to/local.conf
# include /path/to/other.conf

################################## NETWORK #####################################

# 默认情况下，redis 在 server 上所有有效的网络接口上监听客户端连接。
# 你如果只想让它在一个或者多个网络接口上监听，那你就绑定一个IP或者多个IP。
#
# 示例，多个IP用空格隔开:
#
# bind 192.168.1.100 10.0.0.1
# bind 127.0.0.1 ::1
#
bind 127.0.0.1

# 保护模式防止 redis 实例被网络操作恶意关闭。
# 
# 当保护模式开启时，而且：
#       1）redis 监听全部的网络接口
#       2）没有配置密码验证
#  
# 时，redis 服务器只接受本地（127.0.0.1 and ::1）或者是Unix实例的socket链接。
#
# 默认情况下，保护模式是开启的，除非你确定你接受其他的客户端链接，即使这些链接没有密码验证、指定网络接口时才关闭。
protected-mode yes
```
<!-- more -->
```
# 指定Redis监听端口，默认端口为6379
port 6379

# tcp连接数
# 在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。注意Linux内核默默地将这个值减小
# 到/proc/sys/net/core/somaxconn的值，所以需要确认增大somaxconn和tcp_max_syn_backlog
# 两个值来达到想要的效果。
tcp-backlog 511

# 如果redis不监听端口，还怎么与外界通信呢，其实redis还支持通过unix socket方式来接收请求。
# 可以通过unixsocket配置项来指定unix socket文件的路径，并通过unixsocketperm来指定文件的权限。
# 指定用来监听Unix套套接字的路径。没有默认值， 所以在没有指定的情况下Redis不会监听Unix套接字
# unixsocket /tmp/redis.sock
# unixsocketperm 700

# 当一个redis-client一直没有请求发向server端，那么server端有权主动关闭这个连接，可以通过timeout来设置“空闲超时时限”，
# 单位为秒。当客户端在这段时间内没有发出任何指令，那么关闭该连。
# 默认值：0代表禁用，永不关闭 
timeout 0

# TCP keepalive.
#
# 如果非零，则设置SO_KEEPALIVE选项来向空闲连接的客户端发送ACK，由于以下两个原因这是很有用的：
#
# 1）能够检测无响应的对端
# 2）让该连接中间的网络设备知道这个连接还存活
#
# 在Linux上，这个指定的值(单位：秒)就是发送ACK的时间间隔。
# 注意：要关闭这个连接需要两倍的这个时间值。
# 在其他内核上这个时间间隔由内核配置决定
#
# TCP连接保活策略，可以通过tcp-keepalive配置项来进行设置，单位为秒，
# 假如设置为60秒，则server端会每60秒向连接空闲的客户端发起一次ACK请求，以检查客户端是否已经挂掉，
# 对于无响应的客户端则会关闭其连接。所以关闭一个连接最长需要120秒的时间。如果设置为0，则不会进行保活检测
#
# 这个选项的一个合理值是60秒
tcp-keepalive 0

################################# GENERAL #####################################

# Redis默认不是以守护进程的方式运行，可以通过该配置项修改，使用yes启用守护进程方式。
# 注意配置成守护进程后Redis会将进程号写入文件/var/run/redis.pid
daemonize no

# 如果你在 upstart or systemd 中运行redis，redis 提供你supervision tree功能；
# 具体配置：
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised no

# 当Redis以守护进程方式运行时，Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
pidfile /var/run/redis.pid

# 指定日志记录级别 
# Redis总共支持四个级别：debug、verbose、notice、warning，默认为verbose 
# debug  记录很多信息，用于开发和测试
# varbose 很多精简的有用信息，不像debug会记录那么多
# notice 普通的verbose，常用于生产环境
# warning 只有非常重要或者严重的信息会记录到日志 
loglevel notice

# 指明日志文件名。也可以使用"stdout"来强制让Redis把日志信息写到标准输出上。
# 默认为标准输出，如果配置Redis为守护进程方式运行，而这里又配置为日志记录方式为标准输出，则日志将会发给/dev/null
logfile ""

# 要使用系统日志记录器，只要设置 "syslog-enabled" 为 "yes" 就可以了。
# 然后根据需要设置其他一些syslog参数就可以了。
# syslog-enabled no

# 指定linux系统日志syslog的标示符，若是"syslog-enabled=no"，则这个选项无效
# syslog-ident redis

# 指定linux系统日志syslog 设备(facility)， 必须是USER或者LOCAL0到LOCAL7之间
# syslog-facility local0

# 设置数据库的数目。
# 默认数据库是 DB 0，你可以在每个连接上使用 select <dbid> 命令选择一个不同的数据库，
# 但是 dbid 必须是一个介于 0 到 databasees - 1 之间的值
databases 16

################################ SNAPSHOTTING  ################################
#
# 存 DB 到磁盘：
#
#   格式：save <间隔时间（秒）> <写入次数>
#
#   根据给定的时间间隔和写入次数将数据保存到磁盘
#
#   下面的例子的意思是：
#   900 秒后如果至少有 1 个 key 的值变化，则保存
#   300 秒后如果至少有 10 个 key 的值变化，则保存
#   60 秒后如果至少有 10000 个 key 的值变化，则保存
#
#   注意：你可以注释掉所有的 save 行来停用保存功能。
#   也可以直接一个空字符串来实现停用：
#   save ""

save 900 1
save 300 10
save 60 10000

# 如果用户开启了RDB快照功能，那么在redis持久化数据到磁盘时如果出现失败，默认情况下，redis会停止接受所有的写请求。
# 这样做的好处在于可以让用户很明确的知道内存中的数据和磁盘上的数据已经存在不一致了。
# 如果redis不顾这种不一致，一意孤行的继续接收写请求，就可能会引起一些灾难性的后果。
# 如果下一次RDB持久化成功，redis会自动恢复接受写请求。
# 当然，如果你不在乎这种数据不一致或者有其他的手段发现和控制这种不一致的话，你完全可以关闭这个功能，以便在快照写入失败时，也能确保redis继续接受新的写请求。
stop-writes-on-bgsave-error yes

# 是否在 dump .rdb 数据库的时候使用 LZF 压缩字符串
# 默认都设为 yes
# 如果你希望保存子进程节省点 cpu ，你就设置它为 no ，
# 不过这个数据集可能就会比较大
rdbcompression yes

# 因为版本5的RDB有一个CRC64算法的校验和放在了文件的最后。这将使文件格式更加可靠但在
# 生产和加载RDB文件时，这有一个性能消耗(大约10%)，所以你可以关掉它来获取最好的性能。
#
# 生成的关闭校验的RDB文件有一个0的校验和，它将告诉加载代码跳过检查
rdbchecksum yes

# 数据库的文件名
dbfilename dump.rdb

# 数据库的文件路径
dir ./

################################# REPLICATION #################################

# 主从复制。使用 slaveof 来让一个 redis 实例成为另一个reids 实例的副本。
# 注意这个只需要在 slave 上配置。
# 当本机为从服务时，在Redis启动时，它会自动从主服务进行数据同步。
# slaveof <masterip> <masterport>

# 如果主服务master设置了密码(通过下面的 "requirepass" 选项来配置)，slave服务连接master的密码，那么slave在开始同步之前必须进行身份验证，否则它的同步请求会被拒绝。
# 当本机为从服务时，设置主服务的连接密码
# masterauth <master-password>

# 当一个slave失去和master的连接，或者同步正在进行中，slave的行为有两种可能：
# 1) 如果 slave-serve-stale-data 设置为 "yes" (默认值)，slave会继续响应客户端请求，可能是正常数据，也可能是还没获得值的空数据。
# 2) 如果 slave-serve-stale-data 设置为 "no"，slave会回复"正在从master同步(SYNC with master in progress)"来处理各种请求，除了 INFO 和 SLAVEOF 命令。
slave-serve-stale-data yes

# 你可以配置一个 slave 实体是否接受写入操作。
# 通过写入操作来存储一些短暂的数据对于一个 slave 实例来说可能是有用的，
# 因为相对从 master 重新同步数而言，据数据写入到 slave 会更容易被删除。
# 但是如果客户端因为一个错误的配置写入，也可能会导致一些问题。
#
# 从Redis2.6默认所有的slave为只读
#
# 注意：只读的 slaves 没有被设计成在 internet 上暴露给不受信任的客户端。
# 它仅仅是一个针对误用实例的一个保护层。
# 一个只读的slave支持所有的管理命令比如config,debug等。为了限制你可以用'rename-command'来
# 隐藏所有的管理和危险命令来增强只读slave的安全性
slave-read-only yes

# 新的slave和重新连接的slave是不能继续接收差异复制，需要做“完全同步”。 RDB文件是主到从的传送。发送可以以两种不同的方式发生：
#   1 磁盘备份（默认）：Redis的主服务器创建一个新的进程，在磁盘上写入文件RDB。然后由父进程以增量的方式传输到slave。
#   2 无盘：Redis的主创建一个新的进程，直接写入RDB文件到slave的socket。不经过硬盘和主进程。
# 基于硬盘的话，RDB文件创建后，一旦创建完毕，可以同时服务更多的slave。基于socket的话， 新slave来了后，得排队（如果超出了repl-diskless-sync-delay还没来），完事儿一个再进行下一个。
# 当用diskless的时候，master等待一个repl-diskless-sync-delay的秒数，如果没slave来的话，就直接传，后来的得排队等了。否则就可以一起传。
# 由于现在带宽已经很大了，对于缓慢的硬盘，无盘备份效果会更好。
repl-diskless-sync no

# 无盘同步延迟时间
repl-diskless-sync-delay 5

# slave根据指定的时间间隔向服务器发送ping请求。
# 时间间隔可以通过 repl_ping_slave_period 来设置。
# 默认10秒
# repl-ping-slave-period 10

# 以下选项设置同步的超时时间
#
# 1）slave在与master SYNC期间有大量数据传输，造成超时
# 2）在slave角度，master超时，包括数据、ping等
# 3）在master角度，slave超时，当master发送REPLCONF ACK pings
# 
# 确保这个值大于指定的repl-ping-slave-period，否则在主从间流量不高时每次都会检测到超时
# repl-timeout 60

# 是否在slave套接字发送SYNC之后禁用 TCP_NODELAY ？
#
# 如果你选择“yes”Redis将使用更少的TCP包和带宽来向slaves发送数据。
# 但是这将使数据传输到slave上有延迟，Linux内核的默认配置会达到40毫秒，
# 如果你选择了 "no" 数据传输到salve的延迟将会减少但要使用更多的带宽默认我们会为低延迟做优化，
# 但高流量情况或主从之间的跳数过多时，把这个选项设置为“yes”是个不错的选择。
repl-disable-tcp-nodelay no

# 设置数据备份的backlog大小。
# backlog是一个slave在一段时间内断开连接时记录salve数据的缓冲，
# 所以一个slave在重新连接时，不必要全量的同步，而是一个增量同步就足够了，将在断开连接的这段时间内slave丢失的部分数据传送给它。
# 同步的backlog越大，slave能够进行增量同步并且允许断开连接的时间就越长。
# backlog只分配一次并且至少需要一个slave连接
# repl-backlog-size 1mb

## 当master在一段时间内不再与任何slave连接，backlog将会释放。以下选项配置了从最后一个slave断开开始计时多少秒后，backlog缓冲将会释放。
# 0表示永不释放backlog
# repl-backlog-ttl 3600

# slave的优先级是一个整数展示在Redis的Info输出中。如果master不再正常工作了，
# Sentinel将用它来选择一个slave提升=升为master。优先级数字小的salve会优先考虑提升为master，
# 所以例如有三个slave优先级分别为10，100，25，
# Sentinel将挑选优先级最小数字为10的slave。
# 0作为一个特殊的优先级，标识这个slave不能作为master，所以一个优先级为0的slave永远不会被Sentinel挑选提升为master
# 默认优先级为100
slave-priority 100

# 如果master少于N个延时小于等于M秒的已连接slave，就可以停止接收写操作。
# N个slave需要是“oneline”状态
# 延时是以秒为单位，并且必须小于等于指定值，是从最后一个从slave接收到的ping（通常每秒发送）开始计数。
# 例如至少需要3个延时小于等于10秒的slave用下面的指令：
# 两者之一设置为0将禁用这个功能。
# 默认 min-slaves-to-write 值是0（该功能禁用）并且 min-slaves-max-lag 值是10。
# 
# For example to require at least 3 slaves with a lag <= 10 seconds use:
#
# min-slaves-to-write 3
# min-slaves-max-lag 10
#
# Setting one or the other to 0 disables the feature.
#
# By default min-slaves-to-write is set to 0 (feature disabled) and
# min-slaves-max-lag is set to 10.

################################## SECURITY ###################################

# 要求客户端在处理任何命令时都要验证身份和密码。
# 这个功能在有你不信任的其它客户端能够访问redis服务器的环境里非常有用。
# 为了向后兼容的话这段应该注释掉。而且大多数人不需要身份验证(例如:它们运行在自己的服务器上)
# 警告：因为Redis太快了，所以外面的人可以尝试每秒150k的密码来试图破解密码。这意味着你需要
# 一个高强度的密码，否则破解太容易了。
#
# requirepass foobared

# 命令重命名
# 在共享环境下，可以为危险命令改变名字。比如，你可以为 CONFIG 改个其他不太容易猜到的名字，
# 这样内部的工具仍然可以使用，而普通的客户端将不行。
# 例如：
#
# rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
# 也可以通过改名为空字符串来完全禁用一个命令：
#
# rename-command CONFIG ""
#
# 请注意：改变命令名字被记录到AOF文件或被传送到从服务器可能产生问题。

################################### LIMITS ####################################

# 设置最多同时连接的客户端数量。默认这个限制是10000个客户端，然而如果Redis服务器不能配置
# 处理文件的限制数来满足指定的值，那么最大的客户端连接数就被设置成当前文件限制数减32（因
# 为Redis服务器保留了一些文件描述符作为内部使用）
# 一旦达到这个限制，Redis会关闭所有新连接并发送错误'max number of clients reached'
# maxclients 10000

# 不要用比设置的上限更多的内存。一旦内存使用达到上限，Redis会根据选定的回收策略（参见：maxmemmory-policy）删除key
# 如果因为删除策略Redis无法删除key，或者策略设置为 "noeviction"，Redis会回复需要更
# 多内存的错误信息给命令。例如，SET,LPUSH等等，但是会继续响应像Get这样的只读命令。
# 在使用Redis作为LRU缓存，或者为实例设置了硬性内存限制的时候（使用 "noeviction" 策略）的时候，这个选项通常事很有用的。
# 警告：当有多个slave连上达到内存上限的实例时，master为同步slave的输出缓冲区所需
# 内存不计算在使用内存中。这样当驱逐key时，就不会因网络问题 / 重新同步事件触发驱逐key
# 的循环，反过来slaves的输出缓冲区充满了key被驱逐的DEL命令，这将触发删除更多的key，
# 直到这个数据库完全被清空为止
# 
# 总之...如果你需要附加多个slave，建议你设置一个稍小maxmemory限制，这样系统就会有空闲
# 的内存作为slave的输出缓存区(但是如果最大内存策略设置为"noeviction"的话就没必要了)
#
# maxmemory <bytes>

# 最大内存策略：如果达到内存限制了，Redis如何选择删除key。你可以在下面五个行为里选：
# 
# volatile-lru -> 根据LRU算法生成的过期时间来删除。
# allkeys-lru -> 根据LRU算法删除任何key。
# volatile-random -> 根据过期设置来随机删除key。 
# allkeys->random -> 无差别随机删。 
# volatile-ttl -> 根据最近过期时间来删除（辅以TTL） 
# noeviction -> 谁也不删，直接在写操作时返回错误。
# 
# 注意：对所有策略来说，如果Redis找不到合适的可以删除的key都会在写操作时返回一个错误。
#       目前为止涉及的命令：set setnx setex append
#       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
#       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
#       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
#       getset mset msetnx exec sort
#
# 默认值如下：
# maxmemory-policy noeviction

# LRU和最小TTL算法的实现都不是很精确，但是很接近（为了省内存），所以你可以用样本量做检测。
# 例如：默认Redis会检查5个key然后取最旧的那个，你可以通过下面的配置指令来设置样本的个数。
# maxmemory-samples 5

############################## APPEND ONLY MODE ###############################

# 默认情况下，Redis是异步的把数据导出到磁盘上。这种模式在很多应用里已经足够好，但Redis进程
# 出问题或断电时可能造成一段时间的写操作丢失(这取决于配置的save指令)。
#
# AOF是一种提供了更可靠的替代持久化模式，例如使用默认的数据写入文件策略（参见后面的配置）
# 在遇到像服务器断电或单写情况下Redis自身进程出问题但操作系统仍正常运行等突发事件时，Redis能只丢失1秒的写操作。
#
# AOF和RDB持久化能同时启动并且不会有问题。
# 如果AOF开启，那么在启动时Redis将加载AOF文件，它更能保证数据的可靠性。
# 请查看 http://redis.io/topics/persistence 来获取更多信息.

appendonly no

# 追加加文件名字（默认："appendonly.aof"）

appendfilename "appendonly.aof"

# fsync() 系统调用告诉操作系统把数据写到磁盘上，而不是等更多的数据进入输出缓冲区。
# 有些操作系统会真的把数据马上刷到磁盘上；有些则会尽快去尝试这么做。
#
# Redis支持三种不同的模式：
#
# no：不要立刻刷，只有在操作系统需要刷的时候再刷。比较快。
# always：每次写操作都立刻写入到aof文件。慢，但是最安全。
# everysec：每秒写一次。折中方案。 
#
# 默认的 "everysec" 通常来说能在速度和数据安全性之间取得比较好的平衡。根据你的理解来
# 决定，如果你能放宽该配置为"no" 来获取更好的性能(但如果你能忍受一些数据丢失，可以考虑使用
# 默认的快照持久化模式)，或者相反，用“always”会比较慢但比everysec要更安全。
#
# 请查看下面的文章来获取更多的细节
# http://antirez.com/post/redis-persistence-demystified.html 
# 
# 如果不能确定，就用 "everysec"

# appendfsync always
appendfsync everysec
# appendfsync no

# 如果AOF的同步策略设置成 "always" 或者 "everysec"，并且后台的存储进程（后台存储或写入AOF
# 日志）会产生很多磁盘I/O开销。某些Linux的配置下会使Redis因为 fsync()系统调用而阻塞很久。
# 注意，目前对这个情况还没有完美修正，甚至不同线程的 fsync() 会阻塞我们同步的write(2)调用。
#
# 为了缓解这个问题，可以用下面这个选项。它可以在 BGSAVE 或 BGREWRITEAOF 处理时阻止fsync()。
# 
# 这就意味着如果有子进程在进行保存操作，那么Redis就处于"不可同步"的状态。
# 这实际上是说，在最差的情况下可能会丢掉30秒钟的日志数据。（默认Linux设定）
# 
# 如果把这个设置成"yes"带来了延迟问题，就保持"no"，这是保存持久数据的最安全的方式。

no-appendfsync-on-rewrite no

# 自动重写AOF文件
# 如果AOF日志文件增大到指定百分比，Redis能够通过 BGREWRITEAOF 自动重写AOF日志文件。
# 
# 工作原理：Redis记住上次重写时AOF文件的大小（如果重启后还没有写操作，就直接用启动时的AOF大小）
# 
# 这个基准大小和当前大小做比较。如果当前大小超过指定比例，就会触发重写操作。你还需要指定被重写
# 日志的最小尺寸，这样避免了达到指定百分比但尺寸仍然很小的情况还要重写。
#
# 指定百分比为0会禁用AOF自动重写特性。

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

# AOF文件可能在尾部是不完整的（上次system关闭有问题，尤其是mount ext4文件系统时没有加上data=ordered选项。只会发生在os死时，redis自己死不会不完整）。
# 那redis重启时load进内存的时候就有问题了。发生的时候，可以选择redis启动报错，或者load尽量多正常的数据。
# 如果aof-load-truncated是yes，会自动发布一个log给客户端然后load（默认）。如果是no，用户必须手动redis-check-aof修复AOF文件才可以。
aof-load-truncated yes

################################ LUA SCRIPTING  ###############################

# 如果达到最大时间限制（毫秒），redis会记个log，然后返回error。
# 当一个脚本超过了最大时限。只有SCRIPT KILL和SHUTDOWN NOSAVE可以用。第一个可以杀没有调write命令的东西。要是已经调用了write，只能用第二个命令杀。
# 设置成0或者负值，时限就无限。
lua-time-limit 5000

################################ REDIS CLUSTER  ###############################
#
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# WARNING EXPERIMENTAL: Redis Cluster is considered to be stable code, however
# in order to mark it as "mature" we need to wait for a non trivial percentage
# of users to deploy it in production.
# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#
# 开启集群
#
# cluster-enabled yes

# 每一个集群节点都有一个集群配置文件
#
# cluster-config-file nodes-6379.conf

# 集群节点的超时时间，单位为毫秒
#
# cluster-node-timeout 15000

# 控制从节点FailOver相关的设置
# 设为0，从节点会一直尝试启动FailOver.
# 设为正数，失联大于一定时间（factor*节点TimeOut），不再进行FailOver
#
# cluster-slave-validity-factor 10

# 最小从节点连接数
#
# cluster-migration-barrier 1

# 默认为Yes,丢失一定比例Key后（可能Node无法连接或者挂掉），集群停止接受写操作
# 设置为No，集群丢失Key的情况下仍提供查询服务
#
# cluster-require-full-coverage yes

################################## SLOW LOG ###################################

# Redis慢查询日志可以记录超过指定时间的查询。运行时间不包括各种I/O时间，例如：连接客户端，
# 发送响应数据等，而只计算命令执行的实际时间（这只是线程阻塞而无法同时为其他请求服务的命令执行阶段）
# 
# 你可以为慢查询日志配置两个参数:一个指明Redis的超时时间(单位为微秒)来记录超过这个时间的命令
# 另一个是慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录从队列中移除。
#
# 下面的时间单位是微秒，所以1000000就是1秒。注意，负数时间会禁用慢查询日志，而0则会强制记录
# 所有命令。
slowlog-log-slower-than 10000

# 这个长度没有限制。只是要主要会消耗内存。你可以通过 SLOWLOG RESET 来回收内存。
slowlog-max-len 128

################################ LATENCY MONITOR ##############################

# 默认情况下禁用延迟监控,因为它基本上是不需要的，单位为毫秒
latency-monitor-threshold 0

############################# EVENT NOTIFICATION ##############################

# Redis 能通知 Pub/Sub 客户端关于键空间发生的事件
# 这个功能文档位于http://redis.io/topics/keyspace-events
#
# 例如：如果键空间事件通知被开启，并且客户端对 0 号数据库的键 foo 执行 DEL 命令时，将通过
# Pub/Sub发布两条消息：
# PUBLISH __keyspace@0__:foo del
# PUBLISH __keyevent@0__:del foo
#
# 可以在下表中选择Redis要通知的事件类型。事件类型由单个字符来标识：
#
# K    键空间通知，以__keyspace@<db>__为前缀
# E    键事件通知，以__keysevent@<db>__为前缀
# g    DEL , EXPIRE , RENAME 等类型无关的通用命令的通知, ...
# $    String命令
# l    List命令
# s    Set命令
# h    Hash命令
# z    有序集合命令
# x    过期事件（每次key过期时生成）
# e    驱逐事件（当key在内存满了被清除时生成）
# A    g$lshzxe的别名，因此”AKE”意味着所有的事件
#
# notify-keyspace-events 带一个由0到多个字符组成的字符串参数。空字符串意思是通知被禁用。
#
# 例子：启用List和通用事件通知：
# notify-keyspace-events Elg
#
# 例子2：为了获取过期key的通知订阅名字为 __keyevent@__:expired 的频道，用以下配置
# notify-keyspace-events Ex
#
# 默认所用的通知被禁用，因为用户通常不需要该特性，并且该特性会有性能损耗。
# 注意如果你不指定至少K或E之一，不会发送任何事件。
notify-keyspace-events ""

############################### ADVANCED CONFIG ###############################

# 当有大量数据时，适合用哈希编码(这会需要更多的内存)，元素数量上限不能超过给定限制。
# Redis Hash是value内部为一个HashMap，如果该Map的成员数比较少，则会采用类似一维线性的紧凑格式来存储该Map, 
# 即省去了大量指针的内存开销，如下2个条件任意一个条件超过设置值都会转换成真正的HashMap，
# 当value这个Map内部不超过多少个成员时会采用线性紧凑格式存储，默认是64,即value内部有64个以下的成员就是使用线性紧凑存储，超过该值自动转成真正的HashMap。
hash-max-ziplist-entries 512
# list数据类型节点值大小小于多少字节会采用紧凑存储格式
hash-max-ziplist-value 64

# 列表也以特殊的方式节省大量空间编码。
# 允许每个节点内部列表条目的数量可以指定
# 指定一个固定的最大大小或元素的最大数目。
# 使用-5到-1指定固定最大大小：
# -5: max size: 64 Kb  <-- 不推荐用于正常工作
# -4: max size: 32 Kb  <-- 不推荐
# -3: max size: 16 Kb  <-- 有可能不推荐
# -2: max size: 8 Kb   <-- good
# -1: max size: 4 Kb   <-- good
# 正数意味着精确的指定每个节点的数量，
# 性能最高的选项通常是2（8 kb大小）或1（4 kb大小），
# 但如果您使用的情况是独特的，调整设置必要的。
list-max-ziplist-size -2

# 列表页可以压缩 
# 为了快速push/pop，列表的头和尾总是不压缩
# 配置:
# 0: 全部不压缩
# 1: [head]->node->node->...->node->[tail]
#    [head], [tail] 不压缩; 头尾之间的node将压缩.
# 2: [head]->[next]->node->node->...->node->[prev]->[tail]
#     [head->next], [tail->prev] 不压缩; 之间的node将压缩
# 3: [head]->[next]->[next]->node->node->...->node->[prev]->[prev]->[tail]
# 同上.
list-compress-depth 0

# 还有这样一种特殊编码的情况：数据全是64位无符号整型数字构成的字符串。
# 下面这个配置项就是用来限制这种情况下使用这种编码的最大上限的。
set-max-intset-entries 512

# 与第一、第二种情况相似，有序序列也可以用一种特别的编码方式来处理，可节省大量空间。
# 这种编码只适合长度和元素都符合下面限制的有序序列：
zset-max-ziplist-entries 128
zset-max-ziplist-value 64

# 关于HyperLogLog的介绍：http://www.redis.io/topics/data-types-intro#hyperloglogs  
# HyperLogLog稀疏表示限制设置，如果其值大于16000，则仍然采用稠密表示，因为这时稠密表示更能有效使用内存  
# 建议值为3000  
hll-sparse-max-bytes 3000

# 哈希刷新，每100个CPU毫秒会拿出1个毫秒来刷新Redis的主哈希表(顶级键值映射表)。
# redis所用的哈希表实现(见dict.c)采用延迟哈希刷新机制：你对一个哈希表操作越多，哈希刷新操作就越频繁；
# 反之，如果服务器非常不活跃那么也就是用点内存保存哈希表而已。
# 默认是每秒钟进行10次哈希表刷新，用来刷新字典，然后尽快释放内存。
# 建议：
# 如果你对延迟比较在意的话就用 "activerehashing no"，每个请求延迟2毫秒不太好嘛。
# 如果你不太在意延迟而希望尽快释放内存的话就设置 "activerehashing yes"。
activerehashing yes

# 客户端的输出缓冲区的限制，可用于强制断开那些因为某种原因从服务器读取数据的速度不够快的客户端，
# （一个常见的原因是一个发布/订阅客户端消费消息的速度无法赶上生产它们的速度）
#
# 可以对三种不同的客户端设置不同的限制：
# normal -> 正常客户端
# slave -> slave和 MONITOR 客户端
# pubsub -> 至少订阅了一个pubsub channel或pattern的客户端
#
# 下面是每个client-output-buffer-limit语法:
# client-output-buffer-limit <class><hard limit> <soft limit> <soft seconds>
# 一旦达到硬限制客户端会立即被断开，或者达到软限制并持续达到指定的秒数（连续的）。
# 例如，如果硬限制为32兆字节和软限制为16兆字节/10秒，客户端将会立即断开
# 如果输出缓冲区的大小达到32兆字节，或客户端达到16兆字节并连续超过了限制10秒，就将断开连接。
#
# 默认normal客户端不做限制，因为他们在不主动请求时不接收数据（以推的方式），只有异步客户端
# 可能会出现请求数据的速度比它可以读取的速度快的场景。
#
# pubsub和slave客户端会有一个默认值，因为订阅者和slaves以推的方式来接收数据
#
# 把硬限制和软限制都设置为0来禁用该功能
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit slave 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# Redis调用内部函数来执行许多后台任务，如关闭客户端超时的连接，清除未被请求过的过期Key等等。
#
# 不是所有的任务都以相同的频率执行，但Redis依照指定的“hz”值来执行检查任务。
#
# 默认情况下，“hz”的被设定为10。提高该值将在Redis空闲时使用更多的CPU时，但同时当有多个key
# 同时到期会使Redis的反应更灵敏，以及超时可以更精确地处理。
#
# 范围是1到500之间，但是值超过100通常不是一个好主意。
# 大多数用户应该使用10这个默认值，只有在非常低的延迟要求时有必要提高到100。
hz 10

# 当一个子进程重写AOF文件时，如果启用下面的选项，则文件每生成32M数据会被同步。为了增量式的
# 写入硬盘并且避免大的延迟高峰这个指令是非常有用的
aof-rewrite-incremental-fsync yes
```
