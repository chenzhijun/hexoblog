Ubuntu 环境配置


1: 点一下放大,再点一下缩小
gsettings set org.compiz.unityshell:/org/compiz/profiles/unity/plugins/unityshell/ launcher-minimize-window true

获取锁
	```
		sudo rm /var/cache/apt/archives/lock
		sudo rm /var/lib/dpkg/lock
		sudo rm /var/lib/apt/lists/lock
	```
2: 翻墙软件
    a)安装shadowsocks-qt5;图形界面	
	```
		sudo add-apt-repository ppa:hzwhuang/ss-qt5
		sudo apt-get update
		sudo apt-get install shadowsocks-qt5
	```
    b)安装chrome--可以在软件中心安装chromium
    c)浏览器插件SwitchyOmega
    d)shadowsocks-qt 图形化ss
    e)安装zsh,oh-my-zsh,auto-suggestion
	查看安装了那些shell:`cat /etc/shells`
	zsh: `chsh -s /bin/zsh`
	oh my zsh:`sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`
	auto-suggestion:`git clone git://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions`, 在plugin中开启，plugins=(git zsh-autosuggestions)
    f) 安装linuxbrew和proxychains-ng
安装linuxbrew:
```
    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install)"

    echo 'export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"' >>~/.bash_profile
    echo 'export MANPATH="/home/linuxbrew/.linuxbrew/share/man:$MANPATH"' >>~/.bash_profile
    echo 'export INFOPATH="/home/linuxbrew/.linuxbrew/share/info:$INFOPATH"' >>~/.bash_profile

    PATH="/home/linuxbrew/.linuxbrew/bin:$PATH" //直接在命令行里面执行

    brew install gcc
    brew update --force
```
安装并配置proxychains 

	```
		http://echohn.github.io/2016/05/29/to-build-the-fullstack-tools-for-over-the-wall/
		
		brew install proxychains-ng

		sudo vi /etc/proxychains.conf

		将socks4 127.0.0.1 9095改为
		socks5  127.0.0.1 1080  //1080改为你自己的端口
	```
测试是否安装好proxychains-ng:`proxychains4 curl www.google.com`


    g)修改terminal大小和每次打开的窗口位置和大小.
	```
	安装compize config 控制窗口显示位置	
	```

    h) 安装autojump apt-get install autojump . 之后记得在zshrc里面设置plugin

    i) 安装Java
```
export JAVA_HOME=/home/zhijun/DevEnv/jdk1.8.0_131
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

    j) 安装unity tweak tool
       ctrl alt 上下左右,切换工作区
       ctrl alt shift 上下左右,将当前的软件移到工作区 

	```
		vim的补全时可能会报错_arguments:450: _vim_files: function definition file not found，此时只需要删掉 ~/.zcompdump，关闭shell窗口，重新进入即可
	```

    k) 安装MySQL, `apt-get install mysql-server mysql-client` ,之后进行配置`sudo mysql_secure_installation`
	`grant all on snailblog.* to 'man_user' identified by 'test1234';`

    l) 在目录/home/zhijun/.local/share/applications 中可以设置dash中可以找到的软件




ps：安装abuntu16之后要做的事情:
1) 删除不要的软件
sudo apt-get remove unity-webapps-common thunderbird totem rhythmbox empathy brasero simple-scan gnome-mahjongg aisleriot gnome-mines cheese transmission-common gnome-orca webbrowser-app gnome-sudoku  landscape-client-ui-install onboard deja-dup
2) 安装vim
sudo apt-get install vim
3) 安装chromiunm
sudo apt-get install libappindicator1 libindicator7
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt-get -f install
4) 安装 wps
sudo apt-get install wps-office
5) 安装sublime
sudo add-apt-repository ppa:webupd8team/sublime-text-3
sudo apt-get update
sudo apt-get install sublime-text 

6) 安装其他软件
sudo add-apt-repository ppa:diesch/testing
sudo apt-get update
sudo apt-get install classicmenu-indicator
sudo apt-get install vpnc git
sudo apt-get install openssh-server
sudo apt-get install unrar/ unrar x test.rar
sudo apt-get install lnav

7) 安装monaco字体:https://github.com/cstrap/monaco-font
找个readme的链接，下载字体后双击就可以了




======================================
export JAVA_HOME=/home/chen/DevEnv/jdk1.8.0_131
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH

export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"
export MANPATH="/home/linuxbrew/.linuxbrew/share/man:$MANPATH"
export INFOPATH="/home/linuxbrew/.linuxbrew/share/info:$INFOPATH"
export PATH="/home/linuxbrew/.linuxbrew/bin:$PATH"


alias apt="sudo apt"
alias dpkg="sudo dpkg"
alias docker="sudo docker"
alias open="xdg-open"
alias p="proxychains4"
=================================================



$ sudo add-apt-repository ppa:noobslab/themes
$ sudo apt-get update
$ sudo apt-get install flatabulous-theme


=========================================


下载nodejs 的源码包：
解压后进入目录：
./configure
sudo make install
=========================



:s/old/new/
vim编辑器会跳到 old 第一次出现的地方,并用 new 来替换。可以对替换命令作一些修改来替
换多处文本。
 :s/old/new/g :一行命令替换所有 old 。
 :n,ms/old/new/g :替换行号 n 和 m 之间所有 old 。
 :%s/old/new/g :替换整个文件中的所有 old 。
 :%s/old/new/gc :替换整个文件中的所有 old ,但在每次出现时提示
