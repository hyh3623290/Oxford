2019年8月5日做程序员。接触工作正式1年，加培训半年



**001**（2019.3.4）（安装部分没有看）

安装Linux方法有很多，可以装在虚拟机上，也可以安装双系统

vmware官网 - 先注册账号 - 左栏下载 - WorkStation Pro - 百度找个注册号（激活号?）- 下一步一直 - 之后再下载centos（官网）- get centos now - DVD ISO（带图形界面）- 找带aliyun的链接（比较快）



# 认识Linux

linux实际指的是操作系统内核，平时说的centos，ubuntu，redhat，fedora，debian是它的发行版

Linux和Mac都是类Unix系统。Linux最擅长的就是做服务器用



# 网络端口

​	网络的服务启动之后要向外提供服务，必须得有一个门，这个端口就是提供服务的那个门，不同的服务有不同的端口 ：http - 80	https - 443	mysql - 3306.

​	但是这些端口是有可能起冲突的，比如apache是一个web服务，ngnix是一个反向代理的服务，同事也可以做web服务器使用。如果同时启动两个服务的话就会冲突，都用的80端口，就需要改他们的配置文件

​	进程是一个动态的概念，也就是说一个程序，在硬盘上躺着的一个可执行文件，执行起来了，就产生一个动态的效果，这个动态的效果要占用cpu，还有占用内存，在系统里面时活动的，那么它一活动起来就是进程。当操作系统要分别这些进程就要给个编号，就是PID



# 服务

服务是一个特殊的进程，它的生命周期和普通的服务的生命周期时不一样的，它随着操作系统的启动而执行，随着操作系统的关机而结束，这种进程可以叫做服务，而且这个服务没有界面，就在后台跑着



 # 终端

终端是给你提供了一个人机交互的界面



**002**（安装软件部分没看）



**003**

# 操作系统概述



## 远程登录Linux

window系统下

- putty
- Xshell
- 在Cmder终端环境下使用ssh命令

Linux或mac

- ssh命令

最早的登录方式叫telnet，但是不是加密的，后来就出现了ssh

如果是mac电脑且是linux的，打开终端后输入ssh会出现一些帮助信息，现在假设有个服务器，

```js
ping 192.168.0.21 // 看是否开启

ssh root@192.168.0.21 // root是要登录的服务器的用户名

// 接下来需要输入密码，登录成功就可以看到前面的提示符变了

```

#表示是超级管理员，权限最大，$符号表示普通权限





```react
import {createBrowserHistory} from 'history'

const RouterContext = createContext

export class BrowserRouter extends Component {
  constructor() {
    this.history = createBrowserHistory()
  }
  render() {
      return (
          <RouterContext.Provider calue={{history: this.history}}>
              {this.props.history}
          </RouterContext.Provider>
      )
  }
}

export class Route extends Component {
  
}

export class Link extends Component {
  
}
```











































































