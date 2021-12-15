# **Welcome to systemx1's blog**

For GitHub repository visit [systemX1/blog](https://github.com/systemX1/blog).

## **Contents**

├── [  42]  about.md

├── [4.0K]  Computer Networking A Top-Down Approach

├── [4.0K]  CS144

├── [4.0K]  CSAPP

├── [4.1K]  Golang

│   ├── [ 133]  basic_grammar.md

│   └── [   0]  GMP_model.md

├── [ 491]  index.md

├── [ 21K]  MIT6.824

│   ├── [   0]  Fault-Tolerant VM Paper Notes.md

│   ├── [9.0K]  GFS Paper Notes.md

│   ├── [4.3K]  Lab1 MapReduce.md

│   ├── [1.7K]  Lab2 Raft.md

│   └── [2.3K]  Raft Paper Notes.md

└── [4.3K]  Tools

​    └── [ 331]  Leetcode.md

## **Deployment**

直接用mkdocs serve

```bash
nohup mkdocs serve &
# 结束
lsof -i:10000
kill -9 <pid>
```

但更推荐mkdocs build生成静态内容然后用nginx部署

```bash
mkdocs build

```

