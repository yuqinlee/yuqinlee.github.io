# zsh 简记


## zsh 安装与基础配置

arch

```bash
pacman -S zsh
```

切换默认 shell

```bash
# 列出当前 shell
cat /etc/shells 

# 切换 /bin/zsh 是连接，这里需要二进制路径
chsh -s /usr/bin/zsh
```

### zsh 快捷键

||zsh|bash|
|---|---|---|
|光标移动到行首|ctrl+a|home|
|光标移动到行尾|ctrl+e|end|
|删除光标后的所有字符|ctrl+k||
|删除光标前的所有字符|ctrl+h||
|搜索历史|ctrl+r||

## zsh 插件

### zsh-autosuggestions

命令历史自动提示插件

```bash
# arch
sudo pacman -Sy zsh-autosuggestions
echo "source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh" >> ~/.zshrc

# mac
brew install zsh-autosuggestions
echo "source $(brew --prefix)/share/zsh-autosuggestions/zsh-autosuggestions.zsh" >>  ${ZDOTDIR:-$HOME}/.zshrc
```

### zsh-syntax-highlighting

语法高亮插件

```bash
# arch
pacman -Sy zsh-syntax-highlighting
echo "source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ~/.zshrc

# mac
brew install zsh-syntax-highlighting
echo "source $(brew --prefix)/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
```

### autojump

跳转到历史记录里的目录

### incr

系统目录自动提示插件

## zsh 主题

---

`~/.zshrc`: <https://github.com/yuqinlee/uchin-config/blob/master/zsh/.zshrc>

```bash
# ================================================
#             uchin zsh config
# ================================================
# ================================================
#             uchin zsh config
# ================================================
# alias
alias ls='ls --color=auto'
alias ll='ls -lah --color=auto'

# enable color
autoload -U colors && colors

# prompt config
PROMPT="%{$fg[red]%}%n%{$reset_color%}@%{$fg[blue]%}%m %{$fg[green]%}%1|%~ %{$reset_color%}%#>"
# return last command status on line tail
RPROMPT="[%{$fg_bold[yellow]%}%?%{$reset_color%}]"

#  plugins
source /usr/share/zsh/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
source /usr/share/zsh/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh

```

---

*ref:*

- [zsh 入门](https://linux.cn/article-11378-1.html)
- [配置一个简洁高效的 Zsh](https://linux.cn/article-13030-1.html)
- [arch linux安装并简单配置zsh](https://www.cnblogs.com/lookfeel/p/17839867.html)
- [配置 Zsh 终端：Prompt 命令提示符](https://www.newverse.wiki/knows/prompt/)

