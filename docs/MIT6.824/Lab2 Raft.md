## **Debug建议**

善用https://raft.github.io/上的可视化。server，AppendEntries RPC和RequestVote RPC(传递的小球)等都是可以点击和设置的，不确定某一个条件具体该怎么设置或者懒得从头到尾再翻一遍论文的时候很有帮助

Lab2将作为之后实验的基础，保证没有bug很重要。如果只跑个四五次并不能确定已经没有错误了，可以使用一个批量并发测试脚本 https://gist.github.com/jonhoo/f686cacb4b9fe716d5aa

例如2A

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211213131742-653bb1a3aeeae85a97e8c5b97823038e-Raft2Adebug.png" style="zoom: 33%;" />



<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211213131742-c46fe187d37662a6706134924e4e9cee-Raft2AdebugLog8err.png" style="zoom: 25%;" />



## 实验思路

### 2A

**实验结果**

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211213131742-c0a6bcdcfff42ce53b6b85223f87676f-Raft2Apass.png" style="zoom: 33%;" />

<img src="https://gitee.com/systemX1/image-hosting-service/raw/main/img/6824/20211213131742-f99c33d2077f36bb09286cd0ed785a85-Raft2Apass2.png" style="zoom:50%;" />



遇到更大的term 无论是AppendEntries RPC还是RequestVote RPC都要转变为Follower



```go
for {
    select {
        case <-heartbeat:
        rf.mu.Lock()
        if rf.stat != Leader {
            rf.mu.Unlock()
            break
        }
        rf.mu.Unlock()
        rf.appendEntries(nil)
        case t1 := <-rf.electTimer.C:
        rf.mu.Lock()
        if rf.stat == Leader {
            rf.mu.Unlock()
            break
        }
        rf.mu.Unlock()
        rf.startElection()
    }
}
```









