# python包的发布

### setup安装

同我们写好一个包后，如果使用的话，需要把它放到相应目录下，而如果想要在python中使用，就需要用setup.py进行安装，之后就可以直接调包了。下面介绍下setup.py的写法：

setup.py

```python
#!/usr/bin/env python
# coding=utf-8

from setuptools import setup

'''
把redis服务打包成C:\Python27\Scripts下的exe文件
'''

setup(
    name="RedisRun",  #pypi中的名称，pip或者easy_install安装时使用的名称
    version="1.0",
    author="Andreas Schroeder",
    author_email="andreas@drqueue.org",
    description=("This is a service of redis subscripe"),
    license="GPLv3",
    keywords="redis subscripe",
    url="https://ssl.xxx.org/redmine/projects/RedisRun",
    packages=['DrQueue'],  # 需要打包的目录列表

    # 需要安装的依赖
    install_requires=[
        'redis>=2.10.5',
    ],

    # 添加这个选项，在windows下Python目录的scripts下生成exe文件
    # 注意：模块与函数之间是冒号:
    entry_points={'console_scripts': [
        'redis_run = DrQueue.RedisRun.redis_run:main',
    ]},

    # long_description=read('README.md'),
    classifiers=[  # 程序的所属分类列表
        "Development Status :: 3 - Alpha",
        "Topic :: Utilities",
        "License :: OSI Approved :: GNU General Public License (GPL)",
    ],
    # 此项需要，否则卸载时报windows error
    zip_safe=False
)
```

setup.py参数介绍

--name 包名称
--version (-V) 包版本
--author 程序的作者
--author_email 程序的作者的邮箱地址
--maintainer 维护者
--maintainer_email 维护者的邮箱地址
--url 程序的官网地址
--license 程序的授权信息
--description 程序的简单描述
--long_description 程序的详细描述
--platforms 程序适用的软件平台列表
--classifiers 程序的所属分类列表
--keywords 程序的关键字列表
--packages 需要处理的包目录（包含__init__.py的文件夹） 
--py_modules 需要打包的python文件列表
--download_url 程序的下载地址
--cmdclass 
--data_files 打包时需要打包的数据文件，如图片，配置文件等
--scripts 安装时需要执行的脚步列表
--package_dir 告诉setuptools哪些目录下的文件被映射到哪个源码包。一个例子：package_dir = {'': 'lib'}，表示“root package”中的模块都在lib 目录中。
--requires 定义依赖哪些模块 
--provides定义可以为哪些模块提供依赖 
--find_packages() 对于简单工程来说，手动增加packages参数很容易，刚刚我们用到了这个函数，它默认在和setup.py同一目录下搜索各个含有 __init__.py的包。

​                          其实我们可以将包统一放在一个src目录中，另外，这个包内可能还有aaa.txt文件和data数据文件夹。另外，也可以排除一些特定的包

​                          find_packages(exclude=["*.tests", "*.tests.*", "tests.*", "tests"])

--install_requires = ["requests"] 需要安装的依赖包
--entry_points 动态发现服务和插件，下面详细讲

### pypi发布