---
layout: post
title:  "pyftpdlib 基本功能简介"
categories: Python
tags:  Python  
author: DengYuting
---
* content
{:toc}

pyftpdlib是一个 Python 中用于提供 FTP 服务的库，可以创建带用户名密码或者是匿名访问的 FTP 服务，并且可以使用 TSL/SSL 加密提供安全保护，定制限制上传下载速度等。




# ftp 的工作模式
FTP 主要分为两种工作模式：主动模式和被动模式。工作原理如下
- 主动模式  
  1. 客户端以随机非特权端口 N 向服务器段发起链接请求（此处的21为 FTP 服务默认端口，可以被更改）
  2. 客户端开始侦听 N+1 端口
  3. 服务端主动<span style="color:red;">以20端口</span>连接到客户端的N+1端口
- 被动模式  
  1. 客户端以非特权端口连接到服务器的21端口
  2. 服务端开启一个非特权端口为被动端口，并返回给客户端
  3. 客户端以非特权端口N+1<span style="color:red;">主动链接到服务器的被动端口</span>
- 区别  
从上面的对比可以看出两者的区别有以下几点  
  1. 开放的端口：在被动模式下，服务端会多开启一个非特权端口提供给客户端连入，在安全性上比主动模式低。
  2. 存在的问题：如果客户端开启了防火墙，很有可能服务端的主动请求会被拒绝导致链接失败。


# pyftpdlib 安装
```
pip3 install pyftpdlib
```

# pyftpdlib 样例
首先引入一个官方的基础用例，可实现 FTP 的基础功能，在接下来的两章中对主要的库进行解释。  

```
import os

from pyftpdlib.authorizers import DummyAuthorizer
from pyftpdlib.handlers import FTPHandler
from pyftpdlib.servers import FTPServer

def main():
    # Instantiate a dummy authorizer for managing 'virtual' users
    authorizer = DummyAuthorizer()

    # Define a new user having full r/w permissions and a read-only
    # anonymous user
    authorizer.add_user('user', '12345', '.', perm='elradfmwMT')
    authorizer.add_anonymous(os.getcwd())

    # Instantiate FTP handler class
    handler = FTPHandler
    handler.authorizer = authorizer

    # Define a customized banner (string returned when client connects)
    handler.banner = "pyftpdlib based ftpd ready."

    # Specify a masquerade address and the range of ports to use for
    # passive connections.  Decomment in case you're behind a NAT.
    #handler.masquerade_address = '151.25.42.11'
    #handler.passive_ports = range(60000, 65535)

    # Instantiate FTP server class and listen on 0.0.0.0:2121
    address = ('', 2121)
    server = FTPServer(address, handler)

    # set a limit for connections
    server.max_cons = 256
    server.max_cons_per_ip = 5

    # start ftp server
    server.serve_forever()

if __name__ == '__main__':
    main()
```

# pyftplib 主要结构

![pyftpdlib 主要结构](https://user-gold-cdn.xitu.io/2019/9/30/16d7feaeb3805659?w=511&h=404&f=png&s=19097)
# pyftpdlib 主要类介绍
本部分只介绍用到频次可能较高的方法，类里面的方法并未全部列举在本文中，由于没有全部尝试，更多方法请参考官方的文档：https://pyftpdlib.readthedocs.io/en/latest/api.html
## Authorizer（class pyftpdlib.authorizers）
### DummyAuthorizer 类
> Authorizer 是用于处理用户鉴权的一个基础类，主要包含用户的管理功能（例如添加用户、删除用户、允许匿名登录等），它使用 FTPHandler 类来确认用户的密码、基础目录等鉴权信息。作为一个基础类，通常情况下，这个实例需要在程序的一开始被创建出来。

主要函数：
- add_user(username, password, homedir, perm="elr", msg_login="Login successful.", msg_quit="Goodbye.")  
添加用户，指定用户名、密码、主目录、权限、登录和登出的提示语，perm 的含义如下：    

```
Read permissions:

"e" = change directory (CWD, CDUP commands)
"l" = list files (LIST, NLST, STAT, MLSD, MLST, SIZE commands)
"r" = retrieve file from the server (RETR command)

Write permissions:
"a" = append data to an existing file (APPE command)
"d" = delete file or directory (DELE, RMD commands)
"f" = rename file or directory (RNFR, RNTO commands)
"m" = create directory (MKD command)
"w" = store a file to the server (STOR, STOU commands)
"M" = change file mode / permission (SITE CHMOD command) New in 0.7.0
"T" = change file modification time (SITE MFMT command) New in 1.5.3
```

- add_anonymous(homedir, **kwargs)  
是否允许用户匿名登录（无需用户名和密码），参数为匿名登录后的文件目录。
- remove_user(username)  
移除用户
- 等等

## Control connection（class pyftpdlib.handlers）
### FTPHandler 类
> FTP 服务端协议的实现，用户处理从客户端传过来的各种指令，并且调用对应的处理方法，需要将此类的实例与 Authorizer 类实例关联起来以实现登录认证。

主要方法：
- timeout  
指定连接的超时时间
- passive_ports  
指定使用哪些端口用作被动传输，需要传入类似于 rang(6000,6100) 的整数列表，pyftpdlib将不再使用内核分配的随机端口（默认为None）。
- max_login_attempts  
最大的登录尝试次数
- on_connect(self, username)  
成功链接时调用的方法，默认此方法为空内容，需要重写此函数以实现相关功能。    

```
class login_class(FTPHandler):
    def on_login(self, username, password):
        if username == "Johnny":
            print("Hello JohnnyGo!")
```

- on_disconnect()  
同上，还有类似的函数可以进行重写实现
- 等等
  
## Data Connection（class pyftpdlib.handlers）
用于定制化传输的各种参数限制，包含读取、写入速率等
  
### DTPHandler(sock_obj, cmd_channel)
> 传入的参数中，sock_obj是新建立的连接的基础套接字对象实例，cmd_channel是pyftpdlib.handlers.FTPHandler类实例。  

主要方法/成员变量：  
- timeout  
允许没有传输速度的最大时间，默认300秒
- ac_in_buffer_size
- ac_out_buffer_size
  
### ThrottledDTPHandler(sock_obj, cmd_channel)
> 用于替代 DTPHandler 的一个子类，用于限制最大的上传写入速度

主要参数：
- read_limit
每秒最大读取速度限制
- write_limit
每秒最大写入速度限制
  
### TLS_FTPHandler(conn, server)
> 使用FTPS (FTP over SSL/TLS)认证方式，PyOpenSSL模块需要优先被安装
- certfile  
用于鉴权的文件路径
- 等等
  
## Server（class pyftpdlib.servers）
  
### FTPServer(address_or_socket, handler, ioloop=None, backlog=100)
> 该类创建或者监听一个存在的套接字，发送请求给 handler（上一节中的FTPHander 等），同时开启异步的 IO 轮询，参数中，backlog 是传递给 socket.listen()的连接队列的最大长度，当新请求到来但队列已满时会抛出错误ECONNRESET。

主要参数：
- max_cons  
能够同时接收的最大连接数（默认512）
- max_cons_per_ip  
单 IP 允许的最大连接数（默认0 - 不限制）
- serve_forever(timeout=None, blocking=True, handle_exit=True)  
开启异步的 IO 轮询
- close()  
停止接受新的请求，但已有的请求不会被关闭
- close_all()  
断开已有的全部请求，同时通知 server_forever 停止轮询

### ThreadedFTPServer(address_or_socket, handler, ioloop=None, backlog=5)
> 从类 pyftpdlib.servers.FTPServer 扩展出来的一个新的类，当接到新的请求时会生成一个新的线程来处理，与 FTPHander 不同的是，这个类可以自由阻塞而不会挂起整个 IO 轮询。

### MultiprocessFTPServer(address_or_socket, handler, ioloop=None, backlog=5)
> 从类 pyftpdlib.servers.FTPServer 扩展出来的类，当接到新的请求时会生成一个新的进程来处理，与 FTPHander 不同的是，这个类可以自由阻塞而不会挂起整个 IO 轮询。
  
## Filesystems (class pyftpdlib.filesystems)
  
### AbstractedFS(root, cmd_channel)
> 提供了一套跨平台的接口来与 Windows 和 Unix 类操作系统的文件系统进行交互，它能隔离系统中真实路径与虚拟的 FTP 路径，使得用户不能逃脱主目录（应该是为了安全起见），并且提供了一些能够调用 os.* 指令的方法，其两个参数：root 指 FTP 能够访问的主目录，cmd_channel 为 FTPHandler 类

主要方法
-  root  
用户访问的主目录（真实路径）
-  cwd  
用户的工作目录（FTP 虚拟路径）
-  ftpnorm(ftppath)  
（Normalize规范化）根据提供的工作目录虚拟化出 FTP 的虚拟路径，例如工作路径为"/foo"，则"bar"显示的路径为"/foo/bar"
-  ftp2fs(ftppath)  
虚拟 -> 真实  
将虚拟的工作路径转化为真实路径（例如/home/user 为真实目录情况下，/foo 会被转为/home/user/foo）
- fs2ftp(fspath)  
真实 -> 虚拟   
将真实的路径转化为FTP绝对路径（例如/home/user 为真实目录情况下，/home/user/foo 会被转为/foo）
- validpath(path)  
确认路径是否有效，参数为真实的系统路径
-  mkdir(path)
-  isfile(path)
-  等等 可以传递一系列的系统指令
  
### FilesystemError
> 从AbstractedFS类方法中抛出的一类异常
  
### UnixFilesystem(root, cmd_channel)
> 用户实现一套"Unix"FTP 服务器，不同于上述的 AbstractedFS 类，UnixFileSystem 使用真实的系统路径，并且用户可以逃脱指定的FTP 根目录进入真实的文件系统。 
 
# 参考站点
- Python 实现 FTP 服务器：https://www.cnblogs.com/huangxm/p/6274645.html
- Pyftpdlib 官方文档：https://pyftpdlib.readthedocs.io/en/latest/tutorial.html