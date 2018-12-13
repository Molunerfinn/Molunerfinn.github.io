title: CentOS7初接触Ⅰ
id: 210
categories:
  - Linux
  - 日志
date: 2015-06-09 23:57:33
tags:
---

## 这是篇记录，自觉得不能算是教程

本篇文章重点： [双系统引导](#jump1)， [安装NVIDIA显卡驱动](#jump2)
<!--more-->
<span id="jump1"></span>
#### 双系统引导

上篇文章介绍了如何通过硬盘安装[CentOS7.1](http://molunerfinn.com/centos7-1inwin8-1.html "http://molunerfinn.com/centos7-1inwin8-1.html")，感觉安装的时候花费的时间还是不少的。鸟哥的LINUX私房菜里也说过LINUX其实并不是想象中地那么好装的。

不过装完系统后才是真正折腾的时候。装完CentOS后我遇到的第一个问题便是双系统引导的问题。之前安装在虚拟机里其实并没有太多的问题，毕竟在WIN下开虚拟机比从原始开机状态引导两个系统来得简单得多。说一下我遇到的问题。我的问题是在BIOS里选择CENTOS为第一启动系统，开机后能看见识别到了CentOS和Windows的bootmanager。进入CentOS没问题一切正常，但是选择WIN BOOTMANAGER时，就开始报错，无法引导启动的EFI文件，也就意味着不能启动我原先在电脑里的WIN8。

于是我开始寻找解决办法。开始进行系统引导方面的搜寻。由于我CentOS能够识别出WIN的启动管理，那么我自然是想要采用CentOS为第一启动系统，然后WIN8为可选启动系统。百度CENTOS下引导WINDOWS，能出来十几页的教程。不过总结一下，大多数是这样的情况：

*   Centos从一开始装完后就无法识别WIN的启动程序，需要手动添加WIN的启动项的
*   Centos能够识别出启动程序但是就是启动不了的（跟我类似）

网络上大家绝大多数是第一种情况，并且这种情况下的解决办法也是五花八门。主要还是有两种：

*   通过修改Centos里启动项配置文件的
*   通过WINDOWS下EASYBCD这个软件管理启动项，找到CENTOS的启动项的。

所以这个就很头疼。第二种情况下给出的解决办法实在太少，我就开始尝试第一种解决办法，希望通过手动配置CENTOS的启动项配置文件来找到WIN8的启动引导程序。于是就出现了是无效路径的情况。我很奇怪，吸取了装CENTOS的时候我对系统分区不明确的教训，这次明明我的路径正确为什么还是不行，这点我是想不通。有个别的网友跟我的情况一样，不过很遗憾，他们提出的疑问没有人去解答。
于是我就开始尝试第二种办法。后来发现这是本质上的不行。通过EASYBCD创建的引导项是基于MBR引导的，而我的两个系统WIN8＆CENTOS都是基于EFI引导的，所以这种办法在我了解了原理之后直接PASS。

那像我这样的情况难道就不能解决了么？我后来是想出了一个折中的办法。WIN8下用EASYBCD将WIN BOOTMANAGER置为第一启动项。那么开机后便会出现WINBOOTMANAGER，虽然只有WIN8系统这个选项，但是我们按ESC取消WINBOOTMANAGER的引导直接进入BOOT MANAGER，这个时候就能选择我们要启动CENTOS或者是WIN8了。有人说这个办法跟开机按F12有什么区别？区别在于更稳一点，如果开机的那瞬间忘了按F12，那么就有可能没有办法进入自己想进入的系统了嘛。这个办法就是通过WINBOOTMANAGER做一个缓冲，然后通过BOOTMANAGER来实现系统的引导。虽然多了一步，但是在目前情况下我自己觉得是最佳的解决办法了＝。＝

&nbsp;

#### 安装NVIDIA显卡驱动
因为我的电脑是NVIDIA的显卡，在一开始装完CENTOS后浏览网页发现CENTOS的自带显卡驱动实在效果差劲。所以就考虑装一下NVIDIA的显卡驱动，与之前的情况不太一样，显卡驱动在一个小问题解决后就顺利安装成功，比我想象的速度快多了。
##### 显卡驱动安装
去英伟达官网下载相应的显卡驱动。官网界面还算比较友好，很快就能找到相应驱动的下载。这里需要强调的是，在`点击下载` 这个选项这里我们不直接左键下载，那样会开启一个新的网页从而无法下载。右键这个按钮，然后执行`另存为` ,在非中文目录下建立一个文件夹把需要下载的`*.run`文件放进去就OK。
##### 屏蔽nouveau
用root账户或者用`su root`切换到root用户下，打开`/lib/modprobe.d/dist-blacklist.conf`这个目录，然后找到`blacklist nvidiafb`,将其注释：`#blacklist nvidiafb`。
然后在这个禁用列表里找个地方添加如下语句：

`blacklist nouveau`

`options nouveau modeset=0`
##### 重建initramfs image的步骤
在终端输入如下命令：`mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak`以及
`dracut /boot/initramfs-$(uname -r).img $(uname -r)`
#####切换默认启动界面为命令行模式
在终端下输入如下命令：`systemctl set-default multi-user.target`
然后重启系统，并用root账户登录。
##### 检查nouveau是否已经禁用
命令行界面下输入：`ls mod | grep nouveau`
若显示找不到这个东西，那么就说明我们已经屏蔽了它。反之还必须重新做一下屏蔽nouveau的步骤。
##### 执行安装
进入你刚才创建的那个文件夹。例如输入：`cd /downloads/NVIDIA/`，然后再输入：`./NVIDIA-Linux-x86_64-346.72.run`,注意！这个文件名是你下载下来的驱动文件名，不一定和我的一样！安装过程中如果不出意外那么就一路选择accept，那么等到安装结束后会自动回到命令行界面。有意外的话我等会会说明。
##### 切换默认启动界面为图形化界面
输入：`systemctl set-default graphical.target`
然后重启。这个时候重启后就能看到一闪而过的NVIDIA的图案了。进入系统在`应用-其他`的选项里便能看到`NVIDIA X Server Settings`设置菜单了。说明驱动已经安装成功。
##### 安装时候我遇到的问题
![](http://img2.piegg.cn/NVIDIA驱动问题.jpg)
在执行安装这个步骤的时候，成功运行了`NVIDIA-Linux-x86_64-346.72.run`这个文件，但是在安装过程中，出现了一个选项，要求你是否提供kernel module，如果你选择提供，那么会要求你去找寻这个文件的path；如果选择不提供，那么直接安装失败。我们并没有kernel module啊，那怎么办？这个时候我看完安装说明里的要求，于是就懂了。当我们BIOS里的`Security boot`是被设置成`Enable`的话，那么就会影响到英伟达显卡驱动安装。所以我们就只需把`Security boot`设置成`disabled`就行了，问题就能解决。不过要注意的是，如果你的机器里的WINDOWS是原厂WIN8，并且系统并没有重装过，那么请一定要注意，把`Security boot`设置成`disabled`后，将不能启动WIN8。这个是微软的一个机制，如果不懂的同学可以百度一下。由于我的WIN8是重装过的，所以并没有影响。但是我还没有试过将`Security boot`设置成`disabled`，安装完驱动后，再改回去的话驱动还能不能运行。总之如果是WIN7系统或者非原厂WIN8这个问题就没有了。改成`disabled`即可解决。
