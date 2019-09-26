1. 编译之前(运行`./configure --prefix=/usr/local/python36`),首先安装必要的依赖项：
	```shell
	sudo apt-get install build-essential zlib1g-dev libbz2-dev liblzma-dev libncurses5-dev libreadline6-dev libsqlite3-dev libssl-dev libgdbm-dev liblzma-dev tk8.5-dev lzma lzma-dev libgdbm-dev
	# for python3.8 an additional library is needed:
	sudo apt-get install libffi-dev
	```
上述依赖如果不解决，那么编译出来的python缺胳膊少腿，很痛苦。
问题是，该如何知道待编译的python需要哪些依赖库呢？

2. 安装Ipython交互式环境
	```shell
	sudo pip3 install IPython qtconsole PyQt5 PySide PySide2
	```
3. 源码编译一个程序，与所在系统环境变量有着密不可分的联系。
	经常会出现由于系统的环境不正确设置或者缺少某些依赖库而导致编译出现非预期情况的情形，这也是容器技术体现的优势之一。但如果由于某些原因不得不在非容器环境中编译软件，那么首要的问题是首先安装这个软件所需要的所有依赖库。不然的话，在编译过程中或者编译结束后，会出现各种各样缺失功能或者非预期的情况。那么如么知道Python或者任何一个软件所依赖那里东西呢？到Python官网，点击进入`Developer's Guide`(在页面最下方最不起眼的地带，我操你妈的！),里面会有详细的介绍`build`的过程，以下是官网介绍的安装python的依赖库的方法：
	```shell
	For UNIX based systems, we try to use system libraries whenever available. This means optional components will only build if the relevant system headers are available. The best way to obtain the appropriate headers will vary by distribution, but the appropriate commands for some popular distributions are below.

	On Fedora, Red Hat Enterprise Linux and other yum based systems:

	$ sudo yum install yum-utils
	$ sudo yum-builddep python3

	On Fedora and other DNF based systems:

	$ sudo dnf install dnf-plugins-core  # install this to use 'dnf builddep'
	$ sudo dnf builddep python3

	On Debian, Ubuntu, and other apt based systems, try to get the dependencies for the Python you’re working on by using the apt command.

	First, make sure you have enabled the source packages in the sources list. You can do this by adding the location of the source packages, including URL, distribution name and component name, to /etc/apt/sources.list. Take Ubuntu Bionic for example:

	deb-src http://archive.ubuntu.com/ubuntu/ bionic main

	For other distributions, like Debian, change the URL and names to correspond with the specific distribution.

	Then you should update the packages index:

	$ sudo apt-get update

	Now you can install the build dependencies via apt:

	$ sudo apt-get build-dep python3.6

	If that package is not available for your system, try reducing the minor version until you find a package that is available.
	```
4. 同其他`make`命令类似，编译`python`的时候，也可以多核心: `make -j`
5. 安装opencv库: `sudo pip3 install opencv-python`
6. 关于`Python`的库安装目录位置的觉醒：
	有两个名称：site-package和dist-package
	如果，你并没有从源码编译`Python`，那么你的Python是不会有`site-packages`目录的，只有`dist-packages`，这个目录是使用Debian软件管理器安装的Python库，包括`apt-get install python3-numpy`这种形式和`pip install numpy`这种形式。`dist-package`本身又分为两个位置的目录：
		* /usr/lib/python3/dist-packages/
		* /usr/local/lib/python3/dist-packages/
	第一个目录是系统要用到的一些Python库，比如`/usr/bin/gnome-terminal`或者`yum`这个命令，这些命令实际上都是一些Python脚本，只不过是使用了一些Python的图形化界面的库，来构建并显示终端，或者执行安装软件的操作等等，这些命令所依赖的库就都是使用`apt-get`这种Debian管理器来实现安装的，这些库就被安装在第一个目录中，以同用户自己安装的库比如`tensorflow`、`keras`、`numpy`等区分开来。第二个目录里面的库或者叫Python包其实也可以通过apt-get来安装，只不过`somehow`，它就被放在了第二个目录中...好吧，我承认自己还没弄懂。
	对于自己从源码编译的Python，它的所有库的存放位置在：
		* /usr/local/python3/lib/python3.6/site-packages/
	里面。至此，这几个视觉和精神懵懂区域就此明了！
	源码编译完`Python`以后，非常关键的一个文件就是`/usr/bin/python3`这个符号连接文件，因为自己编译的Python在一个自己指定的目录里面，它内部的所有东西跟系统的`Python`是完全隔离的，因此，并不会影响系统的Python，那么，该怎么去影响系统的Python呢？那就是通过改变`/usr/bin/python3`这个软链接所指向的Python解释器来实现。一旦将它指向了自己编译的Python，那么一切使用到python3的命令都会被影响到，自然包括终端启动器(`/usr/bin/gnome-ternimal`), 由于这些系统相关的命令往往会使用到Python的系统级别的库，而这些东西往往在自己编译的Python中并不存在，所以他娘地将软链接指向自己编译的Python以后往往很多系统其他的命令都不会正常使用了，比如终端就启动不起来了，完全他娘的没有任何反应！这个时候，有一个技巧来解决这些随着指向自己编译的Python解释器而来的问题：
		自己编译的Python并没有污染整个系统，原来就的Python解释器仍然可以正常工作，只不过不能够再通过python3来直接调用了，而是显式指明其名称，如python3.5, 它里面的库目录是完整且能够为系统所正常调用的。那么在终端，在调整符号链接指向自己编译的Python解释器的前提下，执行：
		```shell
		# /usr/bin/gnome-ternimal
		...
		Error: No mudule named 'gi'
		```
	提示说找不到`gi`这个库，而我们知道gnome-terminal是一个Python脚本而已，而在符号链接指向python3.5的时候是正常的，那么我们可以在终端中，先进入python3.5解释器，然后在python3.5环境中执行：
		```python
		>>import gi
		>>gi
			/usr/lib/python3/dist-packages/gi/__init__.py
		```
	这样就可以将python3.5中的gi这个狗币库模块找到了。然后直接将这库的文件夹拷贝到自己编译的Python库目录中：
		```shell
		cp -r /usr/lib/python3/dist-packages/gi /usr/local/python36/lib/python3.6/site-packages/
		```
	然后，修改一下.so文件的名称：从`35m`->`36m`,因为这个`Python`库应该是以.so格式存在的，而不是普通的`python`文件。
	另外一个叫`CommandNotFound`的库也是同样的套路，只不过它不是`.so`文件，而是普通的`.py`文件，直接拷贝过去即可。
	因为解决这个`gnome-ternimal`问题，使我重新认识了键盘快捷键的设置，然后顺便将自己钟爱的`ipython`解释器以快捷键的方式启动，而不用像以前那样首先启动`fish`，然后进入`bash`，然后输入自定义的命令`mypython`，直接在自定义快捷键那里将启动`ipython`的命令输入，然后即可。自由，真是妙不可言！
	
	
	
	
	
	
	
	
	
	

