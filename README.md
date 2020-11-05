#Docker OpenStack Swift Object, Container, Account Server

This is a docker file that creates an OpenStack swift object, container, and account server image. 
At this point you must have already started a proxy server using following:
https://github.com/cy0713/docker-swift-proxy


You can specify the object server workers and storage device at run time. Furthermore, you can
specify the ip address, path, and password of the machine from where you can scp
the ring files (you specified this machine when you ran conatiner for proxy server).


## startmain.sh

This Dockerfile uses supervisord to manage the processes.
Dockerfile we will be starting multiple services in the container, such as
object server, account server, container server, etc..


## Usage

首先需要在storage node上面为swift-disk分配一块地方，目前采用跟saio的步骤一样

1. 为回环设备创建文件
```bash
$ sudo mkdir -p /srv
$ sudo truncate -s 1GB /srv/swift-disk
$ sudo mkfs.xfs /srv/swift-disk
```

2. 编辑/etc/fstab并添加

```bash
$ /srv/swift-disk /mnt/sdb1 xfs loop,noatime 0 0
```

3. 创建swift数据挂载点

```bash
$ sudo mkdir /mnt/sdb1
$ sudo mount -a
```

4. 用```df -h```查看创建的回环设备，假设为 /dev/loop2，那么STORAGE DEVICE 就是 loop2


```bash
chloe@host:# docker run -d -e SWIFT_OWORKERS=8 -e SWIFT_DEVICE=loop2 -e SWIFT_SCP_COPY=root@192.168.3.68:~/docker-swift-proxy/files:654321 -p 8010:6010 -p 8011:6011 -p 8012:6012 --privileged -t swift-object
```

Over here, we mapped ports on host. In case you are launching multiple containers you need to choose different ports. Please note that you have to use the same ports that you mentioned when you launch proxy server as
ring files already contain this information.


Similarly, storage device that container
will use for storing the data is specified. The device will be mounted inside the container as we are launching
container in priviliged mode. Please, be sure you specify
the correct device which has enough disk space. Using incorrect device can be catastrophic.


SWIFT_OWORKERS is used to set the object workers dynamically.

The ring files created at the proxy server needs to be copied to the object servers as well. SWIFT_SCP_COPY
contains the remote location path from where ring files can be copied. root@192.168.3.68:~/docker-swift-proxy/files is the remote path, whereas 654321 is the `scp password`.

At this point OpenStack Swift object server is running.


```bash
hulk0@host1:~$ docker ps
CONTAINER ID        IMAGE                            COMMAND                CREATED             STATUS              PORTS                              NAMES
54810e6f88e8        swift-object                     "/bin/sh -c /usr/loc   13 minutes ago      Up 13 minutes       0.0.0.0:6010-6012->6010-6012/tcp   jovial_tesla
```

We need to launch as many containers as we specified at the time of launching proxy server. Once you are done with
that you can go back to the host running proxy server and start using object store to store and retrieve files/objects.


That's it!
