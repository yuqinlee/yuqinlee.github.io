# Python 版本管理 —— Poetry


## poetry 是什么

主要针对虚拟环境下的 python 包管理工具

poetry 与常见框架对比

- conda: Python 版本管理，创建不同虚拟环境，管理不同 python 与 包 版本均
- Pyenv：Python 版本管理，注意这里主要是对于不同 python 版本需求

    ```bash
    # 安装
    curl https://pyenv.run | bash
    
    # 添加系统路径并初始化 Pyenv
    export PATH="$HOME/.pyenv/bin:$PATH"
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)"
    
    # 安装不同版本
    pyenv install 3.8.0
    
    # 全局启用
    pyenv global 3.8.0
    ```

- pip: Python 自带的包管理
- venv: Python 自带的虚拟环境管理

    ```bash
    python -m venv venv
    ```

小结：

1. conda: "重量级"环境管理，可以管理虚拟环境，其虚拟环境下可以独立使用不同版本 py 与不同包，互不干扰
2. pip+venv: 官网自带轻量化，但 pip 不检测依赖，容易出现"孤儿包"（删除下游包后上有依赖无法自动删除）
3. Pyenv: 管理不同的 python 版本

## 安装

archlinux 中直接通过 `pacman` 即可安装

```bash
# arch
sudo pacman -S python-poetry
```

[官网安装方式](https://python-poetry.org/docs/#installing-with-pipx)

## 常用命令

### 包管理

```bash
# 查看版本
poetry -V

# 创建新的基于poetry 的 py 项目，类似 vue-cli 创建新工程
poetry new <proj-name>

# 已有工程上初始化
poetry init

# 依赖稳定后生成lock文件
poetry lock

# 解析并安装 pyproject.toml 的依赖包
poetry install

# 添加包
poetry add <packag name>

# 删除包
poetry remove <packag name>

# 列出当前虚拟环境中的包
poetry show

# 树形结构查看项目安装的依赖
poetry show -t
```

### 虚拟环境

```bash
# 查看虚拟环境信息
poetry env info

# 显示虚拟环境列表
poetry env list

# 显示虚拟环境绝对路径
poetry env list --full-path

# 创建虚拟环境
poetry env use python3.X.X

# 删除虚拟环境
poetry env remove python3.X.X

# 查看python版本
poetry run python -V

# 启动环境
poetry shell
```

<font color='red'>这里创建虚拟环境是需要本地有对应 python 解释器的且在环境变量中找到的</font>

## 配置

### 镜像源配置

```bash
poetry source add tsinghua https://pypi.tuna.tsinghua.edu.cn/simple
```

也可以直接修改 `pyproject.toml`

```toml
[[tool.poetry.source]]
name = "tsinghua"
url = "https://pypi.tuna.tsinghua.edu.cn/simple"
```

### 虚拟环境存放路径

poetry 默认将虚拟环境统一存放

Windows：`C:\Users\<username>\AppData\Local\pypoetry\Cache\virtualenvs\`  
Linux：`.local\pypoetry\cache\virtualenvs\`

通过如下设置使 poetry 将虚拟环境放在项目 `.venv` 下

```bash
# 设置项目内 venv
poetry config virtualenvs.in-project true

# 查看配置
poetry config --list
```

---

*ref*

- [Poetry Doc](https://python-poetry.org/docs)
- [poetry 入门完全指南](https://notes.zhengxinonly.com/environment/use-poetry.html)
- [Python依赖和包管理工具Poetry](https://blog.agiexplained.com/2024/04/05/poetry-quickstart/index.html)
- [Poetry项目配置与环境管理-Poetry中文翻译(六)](https://www.tobyblogs.cn/PoetryCn/6/)
- [Python包管理之poetry的使用](https://www.cnblogs.com/-wenli/p/13337188.html)
- [Python版本管理入门指南：多个版本切换不再困难](https://www.w3cschool.cn/article/32429509.html)
- [poetry管理python开发环境学习小记](https://blog.csdn.net/wuzhongqiang/article/details/125861099)

