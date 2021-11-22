docker中有四个对象：镜像image，容器container，网络network，数据卷volume。几乎所有docker以及周边生态的功能，都在围绕着他们所展开的。

#### 常用命令

1. ```text
   docker pull 镜像:tag                ##拉取远程仓库tag版本号的镜像
   docker images                       ##查询本地仓库的所有镜像
   docker rmi 镜像id                   ##删除镜像
   docker tag 镜像id 镜像:TAG          ##重命名镜像
   docker run -it --privileged -d --name developer --rm imageId  ##根据本地某个镜像创建一个容器
   docker ps                           ##查询本地容器
   docker ps -a                        ##查询本地所有运行的容器
   docker rm containerId				##删除容器
   docker attach containerId			##进入容器，需要先start容器或者docker run创建容器
   docker export 容器id > x:/xx/xx.tar ##导出容器快照
   docker import - x:/xx/xx.tar        ##导入容器快照
   docker save 镜像id -o x:/xx/xx.tar   ##导出镜像
   docker load -i x:/xx/xx.tar    ##导入镜像
   

   // 下载image
   docker pull ubuntu 
   
   //查看已下载image
   docker image ls 
   
   // 删除image
   docker image rm <imageName>  
   
   // 运行
   docker run ubuntu --name ubuntu 
   // 使用ubuntu image 创建一个container 并设置名字为 ubuntu
   // 运行时会检测系统中有没有该image如果没有会自动调用`docker pull unbutu`下载
   
   // 查看run参数
   docker run --help
   
   // 停止ubuntu
   docker stop ubuntu  
   
   // 进入容器
   docker attach ubuntu
   ```
   
1. 启动 Docker 服务 `$ sudo systemctl start docker`

2. 实现 Docker 服务开机自启动`$ sudo systemctl enable docker`

3. `docker version`

   在 Docker 服务启动之后，我们先来尝试一个最简单的查看 Docker 版本的命令：`docker version`。

   这个命令能够显示 Docker C/S 结构中的服务端 ( docker daemon ) 和客户端 ( docker CLI ) 相关的版本信息。

   win10下如果报错`Error response from daemon: open \\.\pipe\docker_engine_linux: The system cannot find the file speci`

   则运行命令

   `Net stop com.docker.service `

   `Net start com.docker.service`

   就好了

4. docker info 查看

5. `docker search name`命令搜索镜像

6. docker images 列出本地docker中得所有镜像

7. docker inspect redis:3.2 能够查看镜像的信息外，`docker inspect` 还能查看容器等之前我们所提到的 Docker 对象的信息，而传参的方式除了传递镜像或容器的名称外，还可以传入镜像 ID 或容器 ID

8. docker rmi 删除镜像 参数是镜像名称或ID；支持同时删除多个镜像，只需要通过空格传递多个镜像 ID 或镜像名即可`docker rmi redis:3.2 redis:4.0`

9. docker create --name nginx nginx:1.12 创建容器，通过 `--name` 这个选项来配置容器名

10. ocker start nginx  通过 `docker create` 创建的容器，是处于 Created 状态的，其内部的应用程序还没有启动，所以我们需要通过 `docker start` 命令来启动它

11. docker run --name nginx -d nginx:1.12 通过 `docker run` 这个命令将 `docker create` 和 `docker start` 这两步操作合成为一步，进一步提高工作效率

    需要注意的一点是，通常来说我们启动容器会期望它运行在“后台”，而 `docker run` 在启动容器时，会采用“前台”运行这种方式，这时候我们的控制台就会衔接到容器上，不能再进行其他操作了。我们可以通过 `-d` 或 `--detach` 这个选项告诉 Docker 在启动后将程序与控制台分离，使其进入“后台”运行。

12. docker ps -a  通过 `docker ps` 这个命令，我们可以罗列出 Docker 中的容器；要列出所有状态的容器，需要增加 `-a` 或 `--all` 选项。

13. docker stop nginx 将正在运行的容器停止，我们可以使用 `docker stop` 命令。容器停止后，其维持的文件系统沙盒环境还是存在的，内部被修改的内容也都会保留，我们可以通过 `docker start` 命令将这个容器再次启动。

14. docker rm nginx 当需要完全删除容器时，可以通过 `docker rm` 命令将容器进行删除。正在运行中的容器默认情况下是不能被删除的，我们可以通过增加 `-f` 或 `--force` 选项来让 `docker rm` 强制停止并删除容器，不过这种做法并不妥当。

15. docker exec  进入到容器 docker exec nginx more /etc/hostname ，`docker exec` 命令能帮助我们在正在运行的容器中运行指定命令，这对于服务控制，运维监控等有着不错的应用场景。但是在开发过程中，我们更常使用它来作为我们进入容器的桥梁。

16. 在 Linux 中，大家熟悉的控制台软件应该是 Shell 和 Bash 了，它们分别由 sh 和 bash 这两个程序启动

    **`docker exec -it nginx bash` **

    在借助 `docker exec` 进入容器的时候，我们需要特别注意命令中的两个选项不可或缺，即 `-i` 和 `-t` ( 它们俩可以利用简写机制合并成 `-it` )。

    其中 `-i` ( `--interactive` ) 表示保持我们的输入流，只有使用它才能保证控制台程序能够正确识别我们的命令。而 `-t` ( `--tty` ) 表示启用一个伪终端，形成我们与 bash 的交互，如果没有它，我们无法看到 bash 内部的执行结果。

17. docker attach nginx 衔接到容器，用于将当前的输入输出流连接到指定的容器上。这个命令最直观的效果可以理解为我们将容器中的主程序转为了“前台”运行 ( 与 `docker run` 中的 `-d` 选项有相反的意思 )。

18. docker run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes mysql 

    docker run -d --name webapp --link mysql webapp:latest

    创建一个 MySQL 容器，将运行我们 Web 应用的容器连接到这个 MySQL 容器上，打通两个容器间的网络，实现它们之间的网络互通。

19. 暴露端口: 端口的暴露可以通过 Docker 镜像进行定义，也可以在容器创建时进行定义。在容器创建时进行定义的方法是借助 `--expose` 这个选项。

    docker run -d --name mysql -e MYSQL_RANDOM_ROOT_PASSWORD=yes --expose 13306 --expose 23306 mysql:5.7

20. 通过别名连接:

    docker run -d --name webapp --link mysql:database webapp:latest

    使用 `--link <name>:<alias>` 的形式，连接到 MySQL 容器，并设置它的别名为 database。当我们要在 Web 应用中使用 MySQL 连接时，我们就可以使用 database 来代替连接地址了。

    String url = "jdbc:mysql://database:3306/webapp";
    
21. 将宿主操作系统中的目录挂载到容器,通过传递 `-v` 或 `--volume` 选项来指定内外挂载的对应目录或文件

    docker run -d --name nginx -v /webapp/html:/usr/share/nginx/html nginx:1.12

    使用 `-v` 或 `--volume` 来挂载宿主操作系统目录的形式是 `-v <host-path>:<container-path>` 或 `--volume <host-path>:<container-path>`，其中 host-path 和 container-path 分别代表宿主操作系统中的目录和容器中的目录。这里需要注意的是，为了避免混淆，Docker 这里强制定义目录时必须使用绝对路径，不能使用相对路径。

    当挂载了目录的容器启动后，我们可以看到我们在宿主操作系统中的文件已经出现在容器中了。

    docker exec nginx ls /usr/share/nginx/html

    在 `docker inspect` 的结果里的 Mounts字段，我们可以看到有关容器数据挂载相关的信息。
    
22. 删除数据卷：docker volume rm appdata在 `docker rm` 删除容器的命令中，我们可以通过增加 `-v` 选项来删除容器关联的数据卷。docker rm -v webapp

    docker volume prune -f

    `docker volume prune` 这个命令，它可以删除那些没有被容器引用的数据卷。



### Dockerfile

1. FROM 通过 FROM 指令指定一个基础镜像,再从这个镜像上进行构建操作
   1. FROM <image> [AS <name>] 
   2. FROM <image>[:<tag>] [AS <name>] 
   3. FROM <image>[@<digest>] [AS <name>]
   4. 既然选择一个基础镜像是构建新镜像的根本，那么 Dockerfile 中的第一条指令必须是 FROM 指令，因为没有了基础镜像，一切构建过程都无法开展
   5. 一个 Dockerfile 要以 FROM 指令作为开始并不意味着 FROM 只能是 Dockerfile 中的第一条指令。在 Dockerfile 中可以多次出现 FROM 指令，当 FROM 第二次或者之后出现时，表示在此刻构建时，要将当前指出镜像的内容合并到此刻构建镜像的内容里
2. RUN 指令就是用于向控制台发送命令的指令,在 RUN 指令之后，我们直接拼接上需要执行的命令，在构建时，Docker 就会执行这些命令，并将它们对文件系统的修改记录下来，形成镜像的变化
   1. RUN <command> 
   2. RUN ["executable", "param1", "param2"]
   3. RUN 指令是支持 \ 换行的，如果单行的长度过长，建议对内容进行切割，方便阅读。而事实上，我们会经常看到 \ 分割的命令
3. ENTRYPOINT 和 CMD 
   1. ENTRYPOINT ["executable", "param1", "param2"] 
   2. ENTRYPOINT command param1 param2 
   3. CMD ["executable","param1","param2"] 
   4. CMD ["param1","param2"] 
   5. CMD command param1 param2
   6. ENTRYPOINT 指令和 CMD 指令的用法近似，都是给出需要执行的命令，并且它们都可以为空，或者说是不在 Dockerfile 里指出。
   7. 当 ENTRYPOINT 与 CMD 同时给出时，CMD 中的内容会作为 ENTRYPOINT 定义命令的参数，最终执行容器启动的还是 ENTRYPOINT 中给出的命令
4. EXPOSE 通过 EXPOSE 指令就可以为镜像指定要暴露的端口
   1. EXPOSE <port> [<port>/<protocol>...]
   2. 当我们通过 EXPOSE 指令配置了镜像的端口暴露定义，那么基于这个镜像所创建的容器，在被其他容器通过 `--link` 选项连接时，就能够直接允许来自其他容器对这些端口的访问了。
5. VOLUME 提供了 VOLUME 指令来定义基于此镜像的容器所自动建立的数据卷
   1. VOLUME ["/data"]
   2. 在 VOLUME 指令中定义的目录，在基于新镜像创建容器时，会自动建立为数据卷，不需要我们再单独使用 `-v` 选项来配置了。
6. COPY 和 ADD 当需要将一些软件配置、程序代码、执行脚本等直接导入到镜像内的文件系统里，使用 COPY 或 ADD 指令能够帮助我们直接从宿主机的文件系统里拷贝内容到镜像里的文件系统中
   1. COPY [--chown=<user>:<group>] <src>... <dest> 
   2. ADD [--chown=<user>:<group>] <src>... <dest> 
   3. COPY [--chown=<user>:<group>] ["<src>",... "<dest>"] 
   4. ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
   5. COPY 与 ADD 指令的定义方式完全一样，需要注意的仅是当我们的目录中存在空格时，可以使用后两种格式避免空格产生歧义。
   6. 对比 COPY 与 ADD，两者的区别主要在于 ADD 能够支持使用网络端的 URL 地址作为 src 源，并且在源文件被识别为压缩包时，自动进行解压，而 COPY 没有这两个能力
   7. 虽然看上去 COPY 能力稍弱，但对于那些不希望源文件被解压或没有网络请求的场景，COPY 指令是个不错的选择。



### 构建镜像

1. docker build ./webapp 
2. 在编写好 Dockerfile 之后，我们就可以构建我们所定义的镜像了
3. `docker build` 可以接收一个参数，需要特别注意的是，这个参数为一个目录路径 ( 本地路径或 URL 路径 )，而并非 Dockerfile 文件的路径。在 `docker build` 里，这个我们给出的目录会作为构建的环境目录，我们很多的操作都是基于这个目录进行的。
4. 例如，在我们使用 COPY 或是 ADD 拷贝文件到构建的新镜像时，会以这个目录作为基础目录。
5. 在默认情况下，`docker build` 也会从这个目录下寻找名为 Dockerfile 的文件，将它作为 Dockerfile 内容的来源。如果我们的 Dockerfile 文件路径不在这个目录下，或者有另外的文件名，我们可以通过 `-f` 选项单独给出 Dockerfile 文件的路径。
6. docker build -t webapp:latest -f ./webapp/a.Dockerfile ./webapp
7. 在构建时我们最好总是携带上 `-t` 选项，用它来指定新生成镜像的名称。
8. docker build -t webapp:latest ./webapp



### Docker Compose 管理容器





```
docker run -t -i --privileged -p 137-139:137-139/tcp -p 445:445/tcp -p 3000:3000/tcp -p 3123:3123/tcp -p 8000:8000/tcp -p 8080:8080/tcp -d --name dev -v /web IMAGEID /bin/bash
```







#### docker镜像的导入和导出

## 启动命令

```bash
docker run -d -p 3000:80 twang2218/gitlab-ce-zh:9.0.3

docker run -d -p 8080:80 gitlab/gitlab-ce:latest
```

## 将容器修改提交到镜像

```bash

# 进入容器内部
[root@#localhost docker]# docker run -ti  ubuntu:14.04 /bin/bash
root@812a997f614a:/# id 
uid=0(root) gid=0(root) groups=0(root)

#做了一些修改
root@812a997f614a:/# echo update>update.txt
root@812a997f614a:/# exit
exit

[root@#localhost docker]# docker ps -a
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS                           PORTS                                   NAMES
812a997f614a        ubuntu:14.04                   "/bin/bash"              7 minutes ago       Exited (0) 22 seconds ago                                                zealous_euler
69304dea46c7        gitlab/gitlab-ce:latest        "/assets/wrapper"        About an hour ago   Exited (127) 44 minutes ago                                              competent_minsky
67ba866e21b0        gitlab/gitlab-ce:latest        "/assets/wrapper"        2 hours ago         Exited (137) About an hour ago                                           hungry_hoover
2a3d08a0a2ff        twang2218/gitlab-ce-zh:9.0.3   "/assets/wrapper"        2 hours ago         Exited (137) About an hour ago                                           nervous_wozniak
6db49540be99        twang2218/gitlab-ce-zh:9.0.3   "/assets/wrapper"        2 hours ago         Exited (255) 2 hours ago         22/tcp, 443/tcp, 0.0.0.0:3000->80/tcp   romantic_elion
b08a6d6ed716        gitlab/gitlab-ce:latest        "/assets/wrapper"        2 hours ago         Exited (255) 2 hours ago         22/tcp, 443/tcp, 0.0.0.0:8080->80/tcp   competent_brahmagupta
33fd0b1ebd27        gitlab/gitlab-ce:latest        "/assets/wrapper"        2 hours ago         Exited (127) 2 hours ago                                                 loving_brattain
6f53620a930c        twang2218/gitlab-ce-zh:9.0.3   "/assets/wrapper"        3 hours ago         Exited (127) 2 hours ago                                                 brave_galileo
88df78f77c4e        ubuntu:14.04                   "sleep 360"              4 days ago          Exited (137) 4 days ago                                                  testcopy
81a879a36bd3        wordpress                      "docker-entrypoint..."   4 days ago          Exited (0) 4 days ago                                                    wordpress
a57a3cc492b7        mysql                          "docker-entrypoint..."   4 days ago          Exited (0) 4 days ago                                                    mysqlwp

# 将修改多的镜像保存成一个新的
[root@#localhost docker]# docker commit 812a997f614a ubuntu:update
sha256:317f102584605694da424bc96764559a1ccfda13943353f4cbdfd89c96515e6b

[root@#localhost docker]# docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
ubuntu                      update              317f10258460        5 seconds ago       188 MB
twang2218/gitlab-ce-zh      9.0.3               36172b5fefab        44 hours ago        1.19 GB
gitlab/gitlab-ce            latest              5eff2e44957c        2 days ago          1.11 GB
mysql                       latest              9546ca122d3a        8 days ago          407 MB
wordpress                   latest              4ad41adc2794        2 weeks ago         401 MB
ubuntu                      14.04               7c09e61e9035        5 weeks ago         188 MB
daocloud.io/library/nginx   1.7.1               e3e043d3ed2f        2 years ago         499 MB

# 查看修改多的镜像和原来镜像之间的差异
[root@#localhost docker]# docker diff 812a997f614a
C /var
C /var/cache
C /var/cache/apt
D /var/cache/apt/srcpkgcache.bin
D /var/cache/apt/pkgcache.bin
C /var/lib
C /var/lib/apt
C /var/lib/apt/lists
A /var/lib/apt/lists/lock
A /var/lib/apt/lists/partial
A /var/lib/apt/lists/partial/archive.ubuntu.com_ubuntu_dists_trusty-updates_InRelease
A /update.txt
C /root
A /root/.bash_history
[root@#localhost docker]# 



```

## 镜像的导入和导出

### export 和improt

```bash

[root@#localhost docker]# docker run -ti  ubuntu:update /bin/bash
root@cbe3cb7799ed:/# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  update.txt  usr  var

[root@#localhost docker]# 
[root@#localhost docker]# docker ps -a
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS                        PORTS                                   NAMES
cbe3cb7799ed        ubuntu:update                  "/bin/bash"              47 seconds ago      Exited (1) 6 seconds ago                                              adoring_kare
812a997f614a        ubuntu:14.04                   "/bin/bash"              16 minutes ago      Exited (0) 8 minutes ago                                              zealous_euler
69304dea46c7        gitlab/gitlab-ce:latest        "/assets/wrapper"        2 hours ago         Exited (127) 53 minutes ago                                           competent_minsky
67ba866e21b0        gitlab/gitlab-ce:latest        "/assets/wrapper"        2 hours ago         Exited (137) 2 hours ago                                              hungry_hoover
2a3d08a0a2ff        twang2218/gitlab-ce-zh:9.0.3   "/assets/wrapper"        2 hours ago         Exited (137) 2 hours ago                                              nervous_wozniak
6db49540be99        twang2218/gitlab-ce-zh:9.0.3   "/assets/wrapper"        2 hours ago         Exited (255) 2 hours ago      22/tcp, 443/tcp, 0.0.0.0:3000->80/tcp   romantic_elion
b08a6d6ed716        gitlab/gitlab-ce:latest        "/assets/wrapper"        2 hours ago         Exited (255) 2 hours ago      22/tcp, 443/tcp, 0.0.0.0:8080->80/tcp   competent_brahmagupta
33fd0b1ebd27        gitlab/gitlab-ce:latest        "/assets/wrapper"        3 hours ago         Exited (127) 2 hours ago                                              loving_brattain
6f53620a930c        twang2218/gitlab-ce-zh:9.0.3   "/assets/wrapper"        3 hours ago         Exited (127) 2 hours ago                                              brave_galileo
88df78f77c4e        ubuntu:14.04                   "sleep 360"              4 days ago          Exited (137) 4 days ago                                               testcopy
81a879a36bd3        wordpress                      "docker-entrypoint..."   4 days ago          Exited (0) 4 days ago                                                 wordpress
a57a3cc492b7        mysql                          "docker-entrypoint..."   4 days ago          Exited (0) 4 days ago                                                 mysqlwp

# 将镜像导出到文件
[root@#localhost docker]# docker export cbe3cb7799ed > update.tar


# 创建一个新静像从基于导出的文件
[root@#localhost /]# docker import - update < update.tar 
sha256:fd00d520a43eb5dc6cca8717fe0ca04cfdc53b02cad2fb5b50d877b8e6d6c3bc
[root@#localhost /]# docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
update                      latest              fd00d520a43e        13 seconds ago      165 MB
ubuntu                      update              317f10258460        11 minutes ago      188 MB
twang2218/gitlab-ce-zh      9.0.3               36172b5fefab        44 hours ago        1.19 GB
gitlab/gitlab-ce            latest              5eff2e44957c        2 days ago          1.11 GB
mysql                       latest              9546ca122d3a        8 days ago          407 MB
wordpress                   latest              4ad41adc2794        2 weeks ago         401 MB
ubuntu                      14.04               7c09e61e9035        5 weeks ago         188 MB
daocloud.io/library/nginx   1.7.1               e3e043d3ed2f        2 years ago         499 MB
[root@#localhost /]# 

```



## save 和load

（像当与镜像的备份和恢复）

```bash
# update是一个已经存在的镜像
[root@#localhost /]# docker save -o update1.tar update
[root@#localhost /]# ls
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  update1.tar  update.tar  usr  var
[root@#localhost /]# du -sh update1.tar 
166M    update1.tar

# 删除update镜像
[root@#localhost /]# docker rmi update
Untagged: update:latest
Deleted: sha256:fd00d520a43eb5dc6cca8717fe0ca04cfdc53b02cad2fb5b50d877b8e6d6c3bc
Deleted: sha256:14cc8cd7b783152682835346e5fe90860a9feeb684866688692285319d4e97ad

[root@#localhost /]# docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
ubuntu                      update              317f10258460        16 minutes ago      188 MB
twang2218/gitlab-ce-zh      9.0.3               36172b5fefab        44 hours ago        1.19 GB
gitlab/gitlab-ce            latest              5eff2e44957c        2 days ago          1.11 GB
mysql                       latest              9546ca122d3a        8 days ago          407 MB
wordpress                   latest              4ad41adc2794        2 weeks ago         401 MB
ubuntu                      14.04               7c09e61e9035        5 weeks ago         188 MB
daocloud.io/library/nginx   1.7.1               e3e043d3ed2f        2 years ago         499 MB

#导入镜像
[root@#localhost /]# docker load < update1.tar 
14cc8cd7b783: Loading layer [==================================================>] 173.8 MB/173.8 MB
Loaded image: update:latest
[root@#localhost /]# docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
update                      latest              fd00d520a43e        6 minutes ago       165 MB
ubuntu                      update              317f10258460        17 minutes ago      188 MB
twang2218/gitlab-ce-zh      9.0.3               36172b5fefab        44 hours ago        1.19 GB
gitlab/gitlab-ce            latest              5eff2e44957c        2 days ago          1.11 GB
mysql                       latest              9546ca122d3a        8 days ago          407 MB
wordpress                   latest              4ad41adc2794        2 weeks ago         401 MB
ubuntu                      14.04               7c09e61e9035        5 weeks ago         188 MB
daocloud.io/library/nginx   1.7.1               e3e043d3ed2f        2 years ago         499 MB
[root@#localhost /]# 

```

## Dockerfile

```bash

[root@#localhost ~]# mkdir docker_file
[root@#localhost ~]# cd docker_file/

[root@#localhost docker_file]# vi Dockerfile 

[root@#localhost docker_file]# cat Dockerfile 
FROM ubuntu:14.04

ENTRYPOINT ["/bin/echo"]

[root@#localhost docker_file]# docker build .
Sending build context to Docker daemon 2.048 kB
Step 1/2 : FROM ubuntu:14.04
 ---> 7c09e61e9035
Step 2/2 : ENTRYPOINT /bin/echo
 ---> Running in d53f31b93355
 ---> 26dd06d2e5a5
Removing intermediate container d53f31b93355
Successfully built 26dd06d2e5a5

#运行镜像

[root@#localhost docker_file]# docker run 26dd06d2e5a5

#加入一个参数

[root@#localhost docker_file]# docker run 26dd06d2e5a5 hello world
hello world
[root@#localhost docker_file]# vi Dockerfile 
[root@#localhost docker_file]# docker run 26dd06d2e5a5 hello world
hello world


#
[root@#localhost docker_file]# cat Dockerfile 
FROM ubuntu:14.04

#ENTRYPOINT ["/bin/echo","Hi world!"]
CMD ["/bin/echo","Hi world!"]

[root@#localhost docker_file]# docker build .
[root@#localhost docker_file]# docker run 12458a717ced
Hi world!


[root@#localhost docker_file]# docker run 12458a717ced /bin/date 
Sat Apr  8 12:08:14 UTC 2017



```

## 构建的时候打个标签

```bash

[root@#localhost docker_file]# docker build -t yang:01 .
Sending build context to Docker daemon 3.584 kB
Step 1/2 : FROM ubuntu:14.04
 ---> 7c09e61e9035
Step 2/2 : CMD /bin/echo Hi world!
 ---> Running in 94e510f085d7
 ---> 6b33c8a6a32f
Removing intermediate container 94e510f085d7
Successfully built 6b33c8a6a32f
[root@#localhost docker_file]# docker images
REPOSITORY                  TAG                 IMAGE ID            CREATED             SIZE
yang                        01                  6b33c8a6a32f        5 seconds ago       188 MB
update                      latest              fd00d520a43e        29 minutes ago      165 MB
ubuntu                      update              317f10258460        41 minutes ago      188 MB
twang2218/gitlab-ce-zh      9.0.3               36172b5fefab        44 hours ago        1.19 GB
gitlab/gitlab-ce            latest              5eff2e44957c        2 days ago          1.11 GB
mysql                       latest              9546ca122d3a        8 days ago          407 MB
wordpress                   latest              4ad41adc2794        2 weeks ago         401 MB
ubuntu                      14.04               7c09e61e9035        5 weeks ago         188 MB
daocloud.io/library/nginx   1.7.1               e3e043d3ed2f        2 years ago         499 MB
[root@#localhost docker_file]# 


```

## 构建实例

将flask应用 打包的镜像中

### 编写python程序 hellp.py

```python
#!/usr/bin/env python

from flask import Flask
app = Flask(__name__)

@app.route('/hi')
def hello_world():
    return 'Hello World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)


```

### 编写Dockerfile

```yaml
FROM ubuntu:14.04

RUN apt-get update
RUN apt-get install -y python
RUN apt-get install -y python-pip
RUN apt-get clean all

RUN pip install flask

ADD hello.py /tmp/hello.py

EXPOSE 5000

CMD ["python","/tmp/hello.py"]

```





### docker 上传镜像

## 一、获取 Docker ID

想要上传镜像到 Docker Hub 上，首先，我们需要注册 [Docker Hub](https://links.jianshu.com/go?to=https%3A%2F%2Fhub.docker.com) 账号。打开 Docker Hub 网址 [https://hub.docker.com](https://links.jianshu.com/go?to=https%3A%2F%2Fhub.docker.com)，开始注册：

填写 Docker ID (也就是账号)，以及密码，Email, 点击继续。

接下来，Docker Hub 会发送验证邮件，到填写的邮箱当中：

点击验证即可，接下来，再次返回 Docker Hub 官网，用您刚刚注册的 Docker ID 和密码来登录账号！



## 二、创建镜像仓库

登录成功后，点击create a repository 创建一个镜像仓库

选择创建一个镜像仓库：`front-develop`

填写**仓库名称**、**描述信息**、**是否公开后**，点击创建。

仓库已经创建成功了，但是里面还没有任何镜像，接下来开始上传镜像，到此新创建的仓库中。

## 三、上传镜像

进入命令行，**用我们刚刚获取的 Docker ID 以及密码登录**，执行命令：

```bash
docker login
```

登录成功后，我们开始准备上传本地的 `update` 镜像：

首先，我们对其打一个新的标签，**前缀与我们新创建的 Docker ID 、仓库名保持一致**:

```
docker tag update wangqiangdocker/front-develop:1.0.0
```

查看本地信息，可以看到，标签打成功了。接下开，开始上传！执行命令：

```
docker push wangqiangdocker/front-develop:1.0.0
```

上传成功！去 Docker Hub 官网，新创建的仓库的信息页面验证一下，是否真的成功了

大工告成！！！













































































