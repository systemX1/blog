## **Download**

https://raft.github.io/raft.pdf

https://web.stanford.edu/~ouster/cgi-bin/papers/OngaroPhD.pdf

## **Overview**

整个算法可分为3个子问题: Leader election(5.2), Log replication(5.3) and Safety(5.2, 5.4)

注意"apply"和"commit"的含义. "log"由"entry"组成，一个"entry"(方格)包括了"term"和"command"("x<-3", "y<-1", "y<-9")

![](https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215130149-7070bbafac967b52c2bf6611a106e37f-RaftFigure2-1.png)

![](https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215130149-7f75580590cb7472fb98f055c9d78050-RaftFigure3-5.png)



当client向leader server发送了一个command后, command会先被封装进entry, 然后replicate到各个follower上(下图1), 当leader确认entry在超过半数server上replicated(下图2)后就会将其commit然后在本server的state machine上apply(下图3, 方格由虚线变实线), 然后告诉client command已经被执行(尽管此时还没有在followers上commit), 接着再告知followers, follower会在本server的state machine上apply(下图4). 如果follower crashed, leader会不定期向follower发送RPC.

The leader decides when it is safe to apply a log entry to the state machines; such an entry is called committed. Raft guarantees that committed entries are durable and will eventually be executed by all of the available state machines.

![](https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215132340-4abe9aeee40e8cb07967545850e82d24-RaftVisualization1-1.png)

![](https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215131628-9045565611bffb132bb8cc3b2efb43c2-RaftVisualization1-2.png)

![](https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215132349-97df66d4851e914f00d4823614eed5d1-RaftVisualization1-3.png)



![](https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211215132356-bf1e3f26d00ce1c12e19ac70e19f6ac4-RaftVisualization1-4.png)









新的Leader必然知道旧Leader使用的term number，因为新term的过半服务器必然与旧term的过半服务器有重叠，而旧Leader的过半服务器中的每一个必然都知道旧Leader的任期号.











## **Questions**



## **Reference**

[The Raft Consensus Algorithm](https://raft.github.io/)

[6.824 Schedule: Spring 2021](https://pdos.csail.mit.edu/6.824/index.html) 



