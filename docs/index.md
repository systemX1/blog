# **Welcome to systemx1's blog**

For GitHub repository visit [systemX1/blog](https://github.com/systemX1/blog).

## **Contents**

├── [  42]  about.md

├── [4.0K]  Computer Networking A Top-Down Approach

├── [4.0K]  CS144

├── [4.0K]  CSAPP

├── [4.1K]  Golang

│&nbsp;&nbsp;&nbsp;&nbsp;├── [ 133]  basic_grammar.md

│&nbsp;&nbsp;&nbsp;&nbsp;└── [   0]  GMP_model.md

├── [ 491]  index.md

├── [ 21K]  MIT6.824

│&nbsp;&nbsp;&nbsp;&nbsp;├── [   0]  Fault-Tolerant VM Paper Notes.md

│&nbsp;&nbsp;&nbsp;&nbsp;├── [9.0K]  GFS Paper Notes.md

│&nbsp;&nbsp;&nbsp;&nbsp;├── [4.3K]  Lab1 MapReduce.md

│&nbsp;&nbsp;&nbsp;&nbsp;├── [1.7K]  Lab2 Raft.md

│&nbsp;&nbsp;&nbsp;&nbsp;└── [2.3K]  Raft Paper Notes.md

└── [4.3K]  Tools

&nbsp;&nbsp;&nbsp;&nbsp;└── [ 331]  Leetcode.md

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
vim /etc/nginx/nginx.conf
# 在http {}里添加以下内容
server {
    listen 			10000;
    server_name		localhost;
    
    location / {
    root <repo_dir>/site;
    index index.html;
    }
}
# restart nginx
sudo systemctl restart nginx
```
或者使用docker

```bash
docker container run \
  -d \
  --rm \
  --name nginx-blog \
  --mount type=bind,source="$PWD/site",target=/usr/share/nginx/html \
  --mount type=bind,source="$PWD/conf",target=/etc/nginx \
  -p 127.0.0.1:10000:80 \
  nginx
```

