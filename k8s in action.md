
## 第一章  Kubernetes介绍

*  一个进程不单单只属于某一个命名空间，而属于每个类型的一个命名空间
*  基于Docker容器的镜像和虚拟机镜像的一个很大的不同是容器镜像是由多层构成，它能在多个镜像之间共享和征用。
*  Docker是一个打包、分发和运行应用程序的平台。
*  容器中进程写入位于底层的一个文件时，此文件的一个拷贝在顶层被创建，进程写的是此拷贝。
* 一旦应用程序运行起来，Kubernetes就会不断地确认应用程序的部署状态始终与你提供的描述相匹配。

## Chapter2. First steps with Docker and Kubernetes

* ![img](https://learning.oreilly.com/library/view/kubernetes-in-action/9781617293726/Images/02fig02_alt.jpg)

* > The client and daemon don’t need to be on the same machine at all. If you’re using Docker on a non-Linux OS, the client is on your host OS, but the daemon runs inside a VM. Because all the files in the build directory are uploaded to the daemon, if it contains many large files and the daemon isn’t running locally, the upload may take longer.

* ```shell
  $ docker exec -it kubia-container bash
  ```

* > -i, which makes sure STDIN is kept open. You need this for entering commands into the shell.  
  >
  >   -t, which allocates a pseudo terminal (TTY).    
  >
  > You need both if you want the use the shell like you’re used to. (If you leave out the first one, you can’t type any commands, and if you leave out the second one, the command prompt won’t be displayed and some commands will complain about the TERM variable not being set.)

## Chapter 3. Pods: running containers in Kubernetes

> Kubernetes sends a **SIGTERM** signal to the process and waits a certain number of seconds **(30 by default)** for it to shut down gracefully. If it doesn’t shut down in time, the process is then killed through **SIGKILL**. To make sure your processes are always shut down gracefully, they need to handle the **SIGTERM** signal properly.

> Deleting everything with the all keyword doesn’t delete absolutely everything. Certain resources (like **Secrets**, which we’ll introduce in [chapter 7](https://learning.oreilly.com/library/view/kubernetes-in-action/9781617293726/Text/07.html#ch07)) are preserved and need to be deleted explicitly.