## **Download**

https://raft.github.io/raft.pdf

https://web.stanford.edu/~ouster/cgi-bin/papers/OngaroPhD.pdf

## **BASIC**

整个算法可分为3个子问题: Leader election(5.2), Log replication(5.3) and Safety(5.2, 5.4)

### **Log replication**

注意"apply"和"commit"的含义. "log"由"entry"组成，一个"entry"(方格)包括了"term"和"command"("x<-3", "y<-1", "y<-9")

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215130149-7070bbafac967b52c2bf6611a106e37f-RaftFigure2-1.png" style="zoom: 25%;" />

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215130149-7f75580590cb7472fb98f055c9d78050-RaftFigure3-5.png" style="zoom:25%;" />



当client向leader server发送了一个command后, command会先被封装进entry, 然后replicate到各个follower上(下图1), 当leader确认entry在超过半数server上replicated(下图2)后就会将其commit然后在本server的state machine上apply(下图3, 方格由虚线变实线), 然后告诉client command已经被执行(尽管此时还没有在followers上commit), 接着再告知followers, follower会在本server的state machine上apply(下图4). 如果follower crashed, leader会不定期向follower发送RPC.

The leader decides when it is safe to apply a log entry to the state machines; such an entry is called committed. Raft guarantees that committed entries are durable and will eventually be executed by all of the available state machines.

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215132340-4abe9aeee40e8cb07967545850e82d24-RaftVisualization1-1.png" style="zoom:35%;" />

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215131628-9045565611bffb132bb8cc3b2efb43c2-RaftVisualization1-2.png" style="zoom:35%;" />

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215132349-97df66d4851e914f00d4823614eed5d1-RaftVisualization1-3.png" style="zoom:35%;" />



<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215132356-bf1e3f26d00ce1c12e19ac70e19f6ac4-RaftVisualization1-4.png" style="zoom:35%;" />

The leader maintains a *nextIndex* for each follower, which is the index of the next log entry the leader will send to that follower. When a leader first comes to power, it initializes all *nextIndex* values to the index just after the last one in its log.

*nextIndex*(箭头)相当于leader对下一个entry发送的一种乐观估计, 理想情况下（初始化）为leader log的下一个. 而*matchIndex*(圆点)则比较保守.



如图所示 S5是leader, S1,S2宕机, S2有一个index为1的entry但还没commit.

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212558-c85d9685b04a4c5dcffb0af11d1015a3-RaftVisualization2-1.png" style="zoom:33%;" />

现在让S5宕机后重启, S4成为leader. 可以看到S4的nextIndex[]都为3, matchIndex[]都为0.

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212543-0d9658ce43e682d5ab56b1afc15225aa-RaftVisualization2-2.png" style="zoom:35%;" /> <img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212700-09992ebf5dce1b55fa657fdc8c413d51-RaftVisualization2-3.png" style="zoom:33%;" />

重启S2 S4向S2不定时发的AppendEntries RPC.

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212708-3d5e14aa5ae60eed8c26926de1ce3529-RaftVisualization2-4.png" style="zoom:33%;" />

因为S2不包含在prevLogTerm的prevLogIndex, S2 reply false, matchIndex也返回0(S2的commitIndex?).

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212726-0c97c0e218346bdec005fb0a64e65f4c-RaftVisualization2-5.png" style="zoom:33%;" />

S4收到后nextIndex[2]--.

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212742-b4245c2f0ccfd7861b0634aaa7464606-RaftVisualization2-6.png" style="zoom:28%;" />

S2 commitIndex设为1, 因为S2在index1是包含满足条件的entry的(上图2-2)reply succ且matchIndex返回1.

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212742-6d47230c865f108d2fc1e9358c442a82-RaftVisualization2-7.png" style="zoom:33%;" />

S4 matchIndex[2]++, "the matchIndex immediately precedes the nextIndex", 发送一个entry.

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212742-c94b47ff2277ecd07bb784512c857c04-RaftVisualization2-8.png" style="zoom:33%;" /> <img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212812-75e65059c565c62f9285788f5d50c35e-RaftVisualization2-9.png" style="zoom:33%;" />



现在来看S1. 重启S1, 第一次reply false; S4 nextIndex[1]--

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212858-25e0d5435f0f84ea857abf13af2c7233-RaftVisualization2-10.png" style="zoom:33%;" />

同样地, S4一直减到matchIndex[1]=nextIndex[1]-1时发送entry

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212906-152bea5d5d00e02180a292c2e35163c4-RaftVisualization2-11.png" style="zoom:33%;" /> <img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212915-37cb54487ec4189806753cb45454d87c-RaftVisualization2-12.png" style="zoom:33%;" />

之后每次发送一个entry, S1就可以逐渐同步了.

### **Safety**

**State Machine Safety Property**

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212345-f5f577f45846f5d6e367c4995a46ec99-RaftFigure3-2.png" style="zoom: 33%;" />

开始举了一个例子, a follower might be unavailable while the leader commits several log entries, then it could be elected leader and overwrite these entries with new ones; as a result, different state machines might execute different command sequences. 因此要成为leader显然应该满足一定的条件.

#### **Election restriction**

条件就是candidate’s log is at least as up-to-date as any other log in that majority(两个logs中长度最长的, 长度相同则最后一个entry的term最大的更加"up-to-date"), then it will hold all the committed entries.

####  **Committing entries from previous terms**

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215212335-039bd32b12c1d9e7bea6a2dc2cb8a0ff-RaftFigure3-7.png" style="zoom: 35%;" />















新的Leader必然知道旧Leader使用的term number，因为新term的过半服务器必然与旧term的过半服务器有重叠，而旧Leader的过半服务器中的每一个必然都知道旧Leader的任期号.

## **Questions**



## **Reference**

[The Raft Consensus Algorithm](https://raft.github.io/)

[6.824 Schedule: Spring 2021](https://pdos.csail.mit.edu/6.824/index.html) 
