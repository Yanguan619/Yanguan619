---
title: pip与conda手册
comments: true
cover: 'https://anaconda.org/static/img/anaconda-symbol.svg'
abbrlink: 3798
date: 2022-01-22 00:00:00
updated: 2022-01-22 00:00:00
aliases:
tags:
categories:
---
# pip 与 conda 配置

up:: [[Python]]

## conda

> `conda`是`Anaconda`带的命令，有关`pip`的命令换成`conda`一般都适用。
> *不过我一般在conda环境下下载pip然后使用pip。*

## conda下载

下载包管理器 Anaconda：[Anaconda](https://www.anaconda.com/download) 或 [Miniconda](https://mirrors.bfsu.edu.cn/anaconda/miniconda/?C=M&O=D)

## pip 换源

```sh
pip config set global.index-url https://pypi.mirrors.ustc.edu.cn/simple/

pip config set global.extra-index-url "https://pypi.mirrors.ustc.edu.cn/simple/ https://pypi.douban.com/simple/ https://pypi.huaweicloud.com/simple/"
```

- 常用源：
  - 阿里云 http://mirrors.aliyun.com/pypi/simple/
  - 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/
  - 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/
  - 豆瓣 https://pypi.douban.com/simple/

*查看当前源*

`pip config list`

临时使用镜像源

````sh
pip install -i https://pypi.doubanio.com/simple/ --trusted-host pypi.doubanio.com pillow
````

更新 pip

`pip install --upgrade pip`

重新安装 pip

`python -m ensurepip`

<details close>
<summary> pip其他命令</summary>

1. 查看 pip 版本和路径 `pip -V`
2. 卸载已安装的库 `pip uninstall pillow`
3. 列出已经安装的库 `pip list`
4. 查看已安装的库的版本 `pip show matplotlib`
5. 查看指定库的所有版本 `pip install matplotlib==`
6. 更新库 `pip install --upgrade matplotlib`
7. 指定版本安装 (使用 ==,  >=,  <=,  >,  < 指定一个版本号 ) `pip install matplotlib>=3.5.1`
8. 将已经安装的库列表保存到文本文件中 `pip freeze > requirements.txt`
9. 根据依赖文件批量安装库，使用上面的txt文件，批量安装第三方库。 `pip install -r requirements.txt`
10. 离线安装库 `pip install pillow-4.2xxxxxxx.whl`
11. 查看当前源 `pip config list`
12. 临时使用镜像源 `pip install -i https://pypi.doubanio.com/simple/ --trusted-host pypi.doubanio.com pillow`

</details>

### 换源

{% tabs %}

<!-- tab 清华源 -->

```sh
conda config --remove-key channels
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
```

<!-- endtab -->

<!-- tab 科大源 -->

```sh
conda config --remove-key channels
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/
```

<!-- endtab -->

<!-- tab tsinghua -->

```sh
conda config --remove-key channels
conda config --add channels https://pypi.tuna.tsinghua.edu.cn/simple
```

<!-- endtab -->

<!-- tab 豆瓣源 -->

`conda config --add channels https://pypi.douban.com/simple/`

<!-- endtab -->

{% endtabs %}

验证镜像源配置成功否

`conda config --show channels`

设置搜索时显示通道地址

`conda config --set show_channel_urls yes`

删除所有源

`conda config --remove-key channels`

### 虚拟环境

{% tabs %}

<!-- tab 1、创建虚拟环境 -->

`conda create -n env_name python=3.11`

<!-- endtab -->

<!-- tab 2、激活虚拟环境 -->

`conda activate env_name`

<!-- endtab -->

<!-- tab 删除虚拟环境 -->

`conda remove -n env_name --all`

<!-- endtab -->

{% endtabs %}

#### 虚拟环境制作与导出

> 参考：https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#building-identical-conda-environments

#### 构建相同的 conda 环境

```sh
conda clean -p      //删除没有用的包
conda clean -t      //tar打包
conda clean -y -all //删除所有的安装包及cache
```

可以使用显式规范文件来构建相同的 Conda 环境(明确的规范文件通常不是跨平台的)。

<details close>
<summary>使用终端执行 `conda list --explicit > spec-file.txt` 运行以生成规范列表</summary>

```sh
# This file may be used to create an environment using:
# $ conda create --name <env> --file <this file>
# platform: osx-64
@EXPLICIT
https://repo.anaconda.com/pkgs/free/osx-64/mkl-11.3.3-0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/numpy-1.11.1-py35_0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/openssl-1.0.2h-1.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/pip-8.1.2-py35_0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/python-3.5.2-0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/readline-6.2-2.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/setuptools-25.1.6-py35_0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/sqlite-3.13.0-0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/tk-8.5.18-0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/wheel-0.29.0-py35_0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/xz-5.2.2-0.tar.bz2
https://repo.anaconda.com/pkgs/free/osx-64/zlib-1.2.8-3.tar.bz2
```

</details>

使用 spec 文件创建环境：`conda create --name myenv --file spec-file.txt`

#### pipenv创建python虚拟环境（打包时使用）

因为在打包环境下会引入了很多不必要的文件，一块打包会导致在生成exe文件过大，而在虚拟纯净环境里打包程序可以有效避免。

```sh
pip install pipenv
pipenv shell # 进入虚拟环境
```

安装库，但是使用 `pipenv install 库` 代替 `pip install 库`
安装pyinstaller：`pipenv install pyinstaller`
最后，在虚拟环境内使用 `pyinstaller -F 文件名.py`

## poetry

> [official url](https://python-poetry.org/)
