





# Jupyterlab 的安装与配置

## 下载并安装 Anaconda

[Anaconda](https://www.anaconda.com/) 是目前最方便的 Python 发行版，搭载了很多我们终将必用的软件包，除了 Python 之外，还有 [R 语言](https://www.r-project.org/)，还包括 [Pandoc](https://pandoc.org/)，[NumPy](http://www.numpy.org/)，[SciPy](https://www.scipy.org/)，[Matplotlib](https://matplotlib.org/)…… 等等。

无论是图形化界面安装，还是命令行界面安装，建议都把 Anaconda 安装在本地用户目录内，`~/`。请下载并安装 Python 3.x 的版本。

图形化界面安装的教程，官方的很详细，各个操作平台的说明都有：

> <https://docs.anaconda.com/anaconda/install/>

两种方式安装

1. 在 MacOS 的 Terminal 命令行下，可以直接下载并安装：

   ```shell
   cd ~/Downloads/
   wget https://repo.anaconda.com/archive/Anaconda3-2018.12-MacOSX-x86_64.sh
   chmod +x Anaconda3-2018.12-MacOSX-x86_64.sh
   ./Anaconda3-2018.12-MacOSX-x86_64.sh
   ```

2. 使用 brew 安装

   ```
   brew cask install Anaconda
   ```

   

安装后可能会出现找不到 `zsh: command not found:` 问题。使用 zsh 。需要导入路径

```shell
cd ~
vim .zshrc
#ancoda environment
export PATH="/usr/local/anaconda3/bin:$PATH"
```

[ancoda 安装参考](https://medium.com/ayuth/install-anaconda-on-macos-with-homebrew-c94437d63a37)

基础命令 类似 `pip`  

> pip <install> <packname>

```
conda update conda  
conda update anaconda
conda install -c conda-forge nodejs
conda install -c conda-forge jupyterlab # 这是用来升级 jupyter lab 到最新版的方法
```

安装完毕之后，可以看看各个你将要用到的可执行命令都在什么地方，用 `which` 命令（windows下用 `where` 命令）：

```
which python
python --version
which node
node -v
which jupyter
jupyter lab --version
jupyter notebook --version
which pip
pip --version
```

## 第一次启动 Jupyter 

```
mkdir example # 新建项目
jupyter notebook  # 启动笔记
```

## 配置 Jupyter lab



```
jupyter lab
jupyter lab --version
conda install -c conda-forge jupyterlab # 这是用来升级 jupyter lab 到最新版的方法
jupyter notebook list                   # 查看正在运行的 jupyter lab/notebook
jupyter notebook stop                   # 停止 jupyter lab/notebook 服务
```



插件安装

-  [自动补全 **Hinterland**](https://jupyter-contrib-nbextensions.readthedocs.io/en/latest/nbextensions/hinterland/README.htm)

  

- [VIM 输入方式](https://github.com/lambdalisue/jupyter-vim-binding)

  ```shell
  # Create required directory in case (optional)
  mkdir -p $(jupyter --data-dir)/nbextensions
  # Clone the repository
  cd $(jupyter --data-dir)/nbextensions
  git clone https://github.com/lambdalisue/jupyter-vim-binding vim_binding
  # Activate the extension
  jupyter nbextension enable vim_binding/vim_binding
  ```

  

- [Jupyter Nbextensions Configurator ](https://github.com/Jupyter-contrib/jupyter_nbextensions_configurator)

  

  备注命令

  ```shell
  `jupyter --data-dir` # jupyter 的工作目录，插件文件夹。
   
   
   conda install -c conda-forge jupyter_contrib_nbextensions  
  ```

  





[conda的安装与使用](https://www.jianshu.com/p/edaa744ea47d)