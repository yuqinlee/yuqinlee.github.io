# Conda 简记

<!-- # Anaconda 记录 -->

## 1. Anaconda 与 Miniconda 与 Conda

<center>
<img src="https://conda.io/projects/conda/en/latest/_static/conda_logo_full.svg" alt="condalogo" style="zoom:50%;" />
</center>

**Conda**

**[维基：conda](https://zh.wikipedia.org/wiki/Conda)**

**[conda官网](https://docs.conda.io/en/latest/)**

>*任何语言的包、依赖项和环境管理 — Python、R、Ruby、Lua、Scala、Java、JavaScript、C/C++、FORTRAN 等。*
>
>Conda 是一个开源包管理系统和环境管理系统，在 Windows、macOS 和 Linux 上运行。Conda 快速安装、运行和更新包及其依赖项。**Conda 可在本地计算机上轻松创建、保存、加载和切换环境**。它是为 Python 程序创建的，但它可以打包和分发任何语言的软件。
>
>Conda 作为包管理器可帮助您查找和安装包。**如果需要不同版本的 Python 的包，则不需要切换到其他环境管理器，因为 conda 也是环境管理器。只需几个命令，就可以设置一个完全独立的环境来运行不同版本的 Python，同时在正常环境中继续运行通常版本的 Python**。
>
>conda 包和环境管理器包含在 Anaconda 和 Miniconda 的所有版本中。

以上为conda官网对于codna的说明，简单说就是包管理工具，并且提供设置不同的环境的功能，因此用户能在不同版本的python之间灵活切换，不同python环境各自独立。

**Anaconda**

**[维基：Anaconda](https://zh.wikipedia.org/wiki/Anaconda_(Python%E5%8F%91%E8%A1%8C%E7%89%88))**

**[Anaconda官网](https://www.anaconda.com/)**

>Anaconda是一个免费开源的Python和R语言的发行版本，用于计算科学（数据科学、机器学习、大数据处理和预测分析），Anaconda致力于简化软件包管理系统和部署。Anaconda的包使用软件包管理系统Conda进行管理

以上为维基Anaconda词条，简单说来就是python的一个发行版本，包括python、conda、许多科学计算需要的包，以及一些科学计算用到的工具等

**Minicona**

**[Miniconda官网](https://docs.conda.io/en/latest/miniconda.html)**

>Miniconda 是conda的免费最小安装程序。这是一个小的，引导版本的Anaconda，只包括conda，Python，他们依赖的包，和少数其他有用的包，包括pip，zlib和其他一些。

以上为Miniconda官网的介绍

## 2. 下载与安装

### 下载

> 下载Anaconda还是Miniconda？
>
> 如果只是需要使用到不同的Python的环境，**没有机器学习、科学计算等需求**，下载Miniconda其实就行，需要一些包后续使用conda或pip安装即可。但是有些包因为版本问题等不是十分容易安装，而Anaconda自带很多常用、不常用的包，在这方面会比较方便。因此安装Anaconda肯定是没问题的，Anaconda安装包更大，并且自带Spyder、Jupyter等（安装后可删除），如果怕麻烦，电脑空间也够，那就安装Anaconda
>
> 如果有强迫症，讨厌捆绑很多东西(比如本人)，那自然……

**Miniconda下载**

1. [官网](https://docs.conda.io/en/latest/miniconda.html)
2. [清华镜像](https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/)

**Anaconda下载**

1. [官网](https://www.anaconda.com/)
2. [清华镜像](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/)

官网下载速度可能比较慢，从清华镜像下载会快很多

### 安装

Miniconda 与 Anaconda 安装界面类似

<img src="Anaconda%E8%AE%B0%E5%BD%95.assets/image-20210214135615761.png" alt="image-20210214135615761" style="zoom: 80%;" />

点击 next，点击I Agree，如下

<img src="Anaconda%E8%AE%B0%E5%BD%95.assets/image-20210214135724640.png" alt="image-20210214135724640" style="zoom:80%;" />

Just Me安装在当前用户；All Users给所用用户安装。因为一台计算机中用户可能有多个(尽管Windows用户很多时候只有一个……)，给所有用户安装那只要是这台计算机中的用户都能使用Anaconda/Miniconda，Just Me就是只给当前所使用的用户安装~~（也不是说别的用户就完全不能用）~~，按个人喜好勾选，Windows个人还是建议All Users，毕竟多数时候是只是用一个用户的emmm

点击Next，之后选择安装路径，

<img src="Anaconda%E8%AE%B0%E5%BD%95.assets/image-20210214140931475.png" alt="image-20210214140931475" style="zoom:80%;" />

点击Next

<img src="Anaconda%E8%AE%B0%E5%BD%95.assets/image-20210214141133596.png" alt="image-20210214141133596" style="zoom:80%;" />

两个选择框选择环境变量的添加，第一个是添加环境变量，第二个是将Anaconda/Miniconda自带的Python作为系统的python

第一个个人建议勾选(**不勾选之后再添加也可，网上有博客说勾选出问题了的，不过我是没出过问题哈，自己添加可参考下面安装完成后的环境变量，Anaconda与Miniconda类似**)，第二个如果计算机还有其他Python根据情况选择，如果和其他Python产生冲突，之后调整环境变量顺序即可。

Install等待安装即可，都勾选安装完成后涉及Anaconda的环境变量如下

![image-20210214141745350](Anaconda%E8%AE%B0%E5%BD%95.assets/image-20210214141745350.png)

可以打开cmd(**环境变量已经配好了**)再验证下是否安装完成了

查看当前python版本（安装了其他版本python可在环境变量里看看顺序，在前面的优先）

```bash
python -V
```

查看conda版本

```bash
conda -V
```

查看pip版本

```bash
pip -V
```

启用base环境（Windows上，建议使用cmd，如果是使用cmd，提示符最前面会有base标记，Power shell的话没有）

```bash
activate base
```

## 3. 换源

使用conda更新、下载包时都比较慢，更换为国内源后会快很多

**参看：[国内可用Anaconda 源的镜像站及换国内源方法](https://www.cnblogs.com/dereen/p/anaconda_tencent_mirrors.html)**

**添加清华源**
命令行中直接使用以下命令（ubuntu命令行也可以）

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
# 设置搜索时显示通道地址
conda config --set show_channel_urls yes
```

**添加中科大源**

```bash
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/
conda config --set show_channel_urls yes
```

## 4. conda常用命令

- **查看conda版本，验证是否安装**

  ```bash
  conda --version 
  ```

- **更新至最新版本，也会更新其它相关包**

  ```bash
  conda update conda
  ```

- **更新所有包**

  ```bash
  conda update --all 
  ```

- **更新指定的包**

  ```bash
  conda update package_name
  ```

- <font color='red'>**显示所有的虚拟环境**</font>

  ```bash
  conda env list
  ```

  或者

  ```bash
  conda info -e
  ```

- <font color='red'>**切换至env_name环境**</font>

  ```bash
  conda activate env_name
  ```

- **退出环境**

  ```bash
  codna deactivate
  ```

- **<font color='red'>创建名为env_name的新环境</font>，并在该环境下安装名为package_name 的包，可以指定新环境的版本号，例如：`conda create -n python2 python=python2.7 numpy pandas`，创建了python2环境，python版本为2.7，同时还安装了numpy pandas包**

  ```bash
  conda create --name env_name   package_name python=3.7
  ```

  > `--name`可缩写为`-n`

- **查看所有已经安装的包**

  ```bash
  conda list
  ```

- **在当前环境中安装包**

  ```bash
  conda install package_name
  ```

- **在指定环境中安装包**

  ```bash
  conda install --name env_name package_name
  ```

- **删除指定环境中的包**

  ```bash
  conda remove --name env_name package 
  ```

- **删除当前环境中的包**

  ```bash
  conda remove package
  ```

- **复制old_env_name为new_env_name**

  ```bash
  conda create --name new_env_name --clone old_env_name
  ```

- **删除环境**

  ```bash
  conda remove --name env_name --all
  ```

- **清理**

  ```bash
  conda clean -p      //删除没有用的包
  conda clean -t      //删除tar包
  conda clean -y --all //删除所有的安装包及cache
  ```

  >conda clean就可以轻松搞定！第一步：通过conda clean -p来删除一些没用的包，这个命令会检查哪些包没有在包缓存中被硬依赖到其他地方，并删除它们。第二步：通过conda clean -t可以将删除conda保存下来的tar包。
  >
  ><https://blog.csdn.net/zhayushui/article/details/80433768>

- **复制/重命名/删除env环境**

  >Conda是没有重命名环境的功能的, 要实现这个基本需求, 只能通过愚蠢的克隆-删除的过程。
  >切记不要直接mv移动环境的文件夹来重命名, 会导致一系列无法想象的错误的发生!
  >
  ><font color='red'>注意：必须在base环境下进行以上操作，否则会出现各种莫名的问题。</font>
  >
  ><https://blog.csdn.net/zhayushui/article/details/80433768>

  ```bash
  //克隆oldname环境为newname环境
  conda create --name newname --clone oldname 
  //彻底删除旧环境
  conda remove --name oldname --all
  ```

## 5. MindSpore相关

[MindSpore官网](https://www.mindspore.cn/install)

[MindSpore安装](https://zhuanlan.zhihu.com/p/136172122)

## 6.jupyter notebook使用

[Jupyter Notebook 添加代码自动补全功能](https://www.jianshu.com/p/0ab80f63af8a)

## 其他参考

【1】[conda常用命令](https://zhuanlan.zhihu.com/p/67745160)

【2】[conda常用命令:安装，更新，创建，激活，关闭，查看，卸载，删除，清理，重命名，换源，问题](https://blog.csdn.net/zhayushui/article/details/80433768)

