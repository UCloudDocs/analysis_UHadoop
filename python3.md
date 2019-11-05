#Python3安装办法（新建集群默认会安装Python3）：

验证 Python 3 环境的方法
在命令行输入 python3 或  python36 或 python3.6 能看到如下信息：
Python 3.6.8 (default, Nov  5 2019, 14:53:46) 
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 


在命令行输入 pip3 -V  能看下如下信息：
pip 18.1 from /usr/local/Python-3.6.8/lib/python3.6/site-packages/pip (python 3.6)


如果集群节点中没有Python3可以按以下方法 手工安装：

wget --limit-rate=10M http://mirrors.ucloud.cn/ucloud/udata/software/Python-3.6.8_version_1.tar.gz -O Python-3.6.8.tar.gz 
tar zxvf Python-3.6.8.tar.gz
cd Python-3.6.8
./configure --prefix=/usr/local/Python-3.6.8
make && make install 
ln -s /usr/local/Python-3.6.8/bin/python3.6 /bin/python3   
ln -s /usr/local/Python-3.6.8/bin/python3.6 /bin/python3.6   
ln -s /usr/local/Python-3.6.8/bin/python3.6 /bin/python36  
ln -s /usr/local/Python-3.6.8/bin/pip3 /bin/pip3 

