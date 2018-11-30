---
layout:     post
title:      分布式文件系统GFS（google File Syste）
subtitle:   Google三大论文
date:       2018-11-30
author:     zhanghaichao
header-img: img/post-bg-e2e-ux.jpg
catalog: true
tags:
    - 分布式
    - 论文
---

论文地址：https://blog.csdn.net/together_cz/article/details/66969003


>GFS是一个大规模分布式文件系统，具有容错的特性（机器崩溃后的处理），并且具有较高性能，能够响应众多的客户端。

#### 1. 设计背景
- 因为机器众多，难免会有机器崩溃
- 存储的文件比较大
- 在文件后追加操作较多
- 主要包括两种读取 （read）操作：一种是大的顺序读取（单个文件读取几百 KB 甚至是几 MB）；另一种是小的随机读取（在随机位置读取几 KB）
- 支持并发（例如，多个客户端同时进行 append 操作）

#### 2. GFS 所需提供操作
create（文件创建）、delete（文件删除）、open（打开文件）、close（关闭文件）、read（读取文件）、write（写入文件）、record append（追加文件）、snapshot（快照）。

#### 3. 架构
GFS 的架构由一个 master 节点和许多台文件服务器（chunkserver）构成，并且有若干客户端（client）与之交互。

- 文件块（chunks），每块有一个全球唯一等64位标识符（chunk handle），它是在 chunk 被创建时由 master 分配的，每一个 chunk 会默认有3个备份，分别在不同的机器上。
- Master 存储所有的 metadata，包括命名空间（namespace）、访问控制信息（access control）、文件与 chunk 的映射关系（mapping）以及 chunk 的存储位置
- Master 管理 chunk 租约（lease）、孤儿Chunk的回收，chunkserver 之间的通信（心跳检测）
- Client 会和 master 以及 chunkserver 进行交互，client向 master 请求 metadata，然后向 chunkserver 进行读写操作
- client 与 chunkserver 都不会缓存文件数据，为的是防止数据出现不一致的状况。但是 client 会缓存 元数据 的信息。Chunk以本地文件的方式保存，Linux操作系统的文件系统缓存会把经常访问的数据缓存在内存中。

#### 4. 单一 Master 架构
读写操作，它只会告诉 client，它所请求操作的文件在哪个 chunkserver 上，然后 client 会根据 master 提供的信息，与对应的 chunkserver 进行通信。
Master 职责
a. 执行所有有关于 namespace 的操作
b.管理整个系统的 chunk replicas： 
c.做出 chunk replicas 的放置决定
d.创建 chunk/replicas
e.协调各种操作，保证 chunk 被完全复制
f. 负载均衡
g. 回收闲置空间

#### 5. chunk 
位置：master定期通过heartbeat 人更新chunk等位置信息，不需要保持同步通信，减少开销
大小：GFS 中将 chunk 的大小定为 64MB，它比一般的文件系统的块大小要大。
- 优点：
减少 client 与 master 的交互
client 可以在一个块上执行更多的操作，通过 TCP 长连接减少网络压力
减小 metadata 的大小
- 缺点：
一个 chunk 可以存更多的小文件了，这样的话如果有一个块存储了许多小文件，client 和它进行操作的几率大大提高，这个 chunk 的压力会很大  

#### 6. metadata

GFS 的 metadata 存储着 3 种类型的信息：
- 文件名以及 chunk 的名称
- 文件与 chunk 的映射关系
- 各个备份（replicas）的位置

Metadata 通常存储于内存中，前两种信息有时会存于磁盘中（operation log），优点是查看 metadata 信息时很方便，速度快，有利于 chunk 的垃圾回收、再备份、以及 chunk 迁移（为的是负载均衡）
Master服务器只需要不到64个字节的metadata就能够管理一个64MB的Chunk，所以不会受限与内存大小

#### 7.  一致性
在分布式文件系统中，很重要的一部分就是数据的复制（replica），为了保证分布式文件系统的高可用性，我们常常会把文件在不同的机器上存储多份，一致性的要求就是保证这些不同机器上的复制品（replicas）能够保持一致。

- GFS 通过在所有的备份（replicas）上应用顺序相同的操作来保证一个文件区域的 defined（如果一个文件区域在经过一系列操作之后依旧是一致的，并且客户端完全知晓对它所做的所有操作）
- GFS 会使用 chunk version（版本号）来检测 replicas 是否过期，过期的 replicas 既不会被读取也不会被写入
- GFS 通过握手（handshakes）来检测已经宕机的 chunkserver
- GFS 会通过校验和（checksuming）来检测文件的完整性

#### 8. 租约（lease）和修改的顺序（mutation order）
GFS 使用租约机制（lease）来保障 mutation 的一致性：多个备份中的一个持有 lease，这个备份被称为 primary replica（其余的备份为 secondary replicas），GFS 会把所有的 mutation 都序列化（串行化），让 primary 直行，secondary 也按相同顺序执行，primary 是由 master 选出来的。一个 lease 通常60秒会超时。

写操作的数据流程：
- client 向 master 请求持有 lease 的 chunk（primary replica）位置和其他 replicas 的位置（如果没有 chunk 持有 lease，那么 master 会授予其中一个 replica 一个 lease）
- master 返回 primary 的信息和其他 replicas 的位置，然后 client 将这些信息缓存起来
- client 会将数据发送到所有的 replicas，每个 chunkserver 会把数据存在 LRU 缓存中
- 在所有的 replicas 都收到了数据之后，client 会向 primary 发送写请求。primary 会给它所收到的所有 mutation 分配序列号，它会在自己的机器上按序列号进行操作
- primary 给 secondaries 发送写请求，secondaries 会按相同的序列执行操作
- secondaries 告知 primary 操作执行完毕
- primary 向 client 应答，期间的错误也会发送给 client，client 错误处理程序（error handler）会重试失败的 mutation

#### 9. Snapshot 快照
当 master 收到 snapshot 操作请求后：
- 废除所有的 lease，准备 snapshot（相当于暂停了所有写操作）
- master 记录所有操作，并且将记录写入磁盘
- master 将源文件和目录树的 metadata 进行复制，这样之前的记录就和当前的内存中所保存的状态对应起来了，新建的 snapshot 和源文件指向的会是同一个 chunk

#### 10. namespace 锁
 在 Master 对文件或者目录进行操作之前它首先需要获取一个锁，比如要对 /d1/d2/…/dn/leaf 进行操作，需要获得 /d1, /d1/d2, /d1/d2/…/dn的读锁，需要 /d1/d2/…/dn/leaf 的读锁或者写锁（根据不同的操作，锁也不同）

#### 11. 垃圾回收
文件 delete 之后，GFS 并不会立即对空间进行回收，而是等待垃圾回收机制会空间进行释放。

当文件被删除之后，Master 会想其他操作一样，把删除操作记录下来，但是不进行空间的回收，而是将这块空间命名为 hidden（并且包含被删除时的时间戳），Master 会定期进行扫描，把隐藏了一定时间的文件空间进行回收（这个时间是可以进行配置的），在此期间可以对这块空间的文件进行恢复（直接通过重命名回原来的名称就可以）。

除此之外，垃圾回收机制还会扫描孤儿 chunk（所有的文件都没有用到的非空 chunk），然后对这块 chunk 的 metadata 进行清除。
