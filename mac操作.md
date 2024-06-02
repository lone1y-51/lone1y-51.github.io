- [[#环境安装|环境安装]]
	- [[#环境安装#安装protobuf，并支持go|安装protobuf，并支持go]]
	- [[#环境安装#安装tig|安装tig]]
	- [[#环境安装#安装gitui|安装gitui]]
	- [[#环境安装#取消更新提示红点|取消更新提示红点]]
	- [[#环境安装#安装python|安装python]]
	- [[#环境安装#卸载python|卸载python]]
	- [[#环境安装#修改ideavim配置|修改ideavim配置]]
	- [[#环境安装#spacevim下修改vim的快捷键|spacevim下修改vim的快捷键]]
	- [[#环境安装#spacevim 解决`Tagbar: Ctags doesn't seem to be Exuberant Ctags!`|spacevim 解决`Tagbar: Ctags doesn't seem to be Exuberant Ctags!`]]
	- [[#环境安装#spacevim 修改默认的背景色为纯黑色|spacevim 修改默认的背景色为纯黑色]]
	- [[#环境安装#修改brew为清华源|修改brew为清华源]]
		- [[#修改brew为清华源#所有其他源列表|所有其他源列表]]
	- [[#环境安装#重置为官方源|重置为官方源]]
- [[#安装gitbook|安装gitbook]]
- [[#备用命令|备用命令]]
	- [[#备用命令#清理dns缓存|清理dns缓存]]

# 环境安装

## 安装protobuf，并支持go

1. 通过`homebrew` 安装protobuf`brew install protobuf`
2. 安装go插件

```bash
go get -u -v github.com/golang/protobuf/protogo get -u -v github.com/golang/protobuf/protoc-gen-go
```

1. 配置`$GOPATH/bin` 到环境变量 `export PATH=$GOPATH/bin:$PATH`

## 安装tig

`brew install tig`

## 安装gitui

`brew install gitui`,注意要`0.19`以上才支持自定义`key_config`

## 取消更新提示红点

1. 在`系统偏好设置->软件更新`中取消`自动保持我的mac最新`, 点击`高级`取消相应的勾选
2. 执行下面两行代码
   
    ```
    defaults write com.apple.systempreferences AttentionPrefBundleIDs 0
    Killall Dock
    ```
    

## 安装python

[淘宝仓库](http://npm.taobao.org/mirrors/python/) 寻找对应版本

## 卸载python

1. `sudo rm -rf /Library/Frameworks/Python.framework/Versions/{version}`
2. `sudo rm -rf /Applications/Python {version}`
3. 删除/usr/local/bin 目录下指向的Python3.7 的连接
   
    ```
    cd /usr/local/bin/
    ls -l /usr/local/bin
    rm Python3.7相关的文件和链接
    ```
    

## 修改ideavim配置

`jetbrains`的ideavim插件可以用`~/.ideavimrs`配置文件替换部分快捷键

比如，用`fd`替换`esc`退出编辑模式

```
" Quit insert mode
inoremap fd <Esc>
```

## spacevim下修改vim的快捷键

在`~/.SpaceVim.d/autoload`下创建`keymap`文件例如`myspacevim.vim`，内容如下

```
function! myspacevim#before() abort
  let g:neomake_enabled_c_makers = ['clang']
  inoremap fd <esc>
  vnoremap fd <esc>
endf
function! mluspacevim#after() abort
endf
```

然后修改`~/.Spacevim.d/init.toml`

```
[options]
    bootstrap_before = "myspacevim#before"
```

## spacevim 解决`Tagbar: Ctags doesn't seem to be Exuberant Ctags!`

错误全文

```
Tagbar: Ctags doesn't seem to be Exuberant Ctags!
BSD ctags will NOT WORK. Please download Exuberant Ctags from ctags.sourceforge.net and install it in a directory in your $PATH or set g:tagbar_ctags_bin.
Executed command: "'ctags' --version"
Command output:
/Library/Developer/CommandLineTools/usr/bin/ctags: illegal option -- -
usage: ctags [-BFadtuwvx] [-f tagsfile] file ...
Exit code: 1
Press ENTER or type command to continue
```

安装 `brew install ctags-exuberant`

然后在`.vimrc`中添加`let g:Tlist_Ctags_Cmd='/usr/local/Cellar/ctags/5.8_1/bin/ctags'`

## spacevim 修改默认的背景色为纯黑色

`~/.SpaceVim/bundle/gruvbox/colors`下的`gruvbox.vim`

设置 `s:gb.dark0 = ["#000000", 0]`

## 修改brew为清华源

```bash
git -C "$(brew --repo)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-cask.git
echo 'export HOMEBREW_BOTTLE_DOMAIN=https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles' >> ~/.zshrc
source ~/.zshrc
brew update
```

### 所有其他源列表

- `[https://mirrors.aliyun.com](https://mirrors.aliyun.com)` 阿里
- `[https://mirrors.ustc.edu.cn](https://mirrors.ustc.edu.cn)` 中科大

## 重置为官方源

```bash
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git
git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git
git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask
vim ~/.zshrc # 注释掉 HOMEBREW_BOTTLE_DOMAIN 配置
```

# 安装gitbook

如果出现

```
/usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
                 ^

TypeError: cb.apply is not a function
    at /usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287:18
    at FSReqCallback.oncomplete (node:fs:196:5)
```

可以直接vim `/usr/local/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js`

这个文件，在62-64行注释

```
//fs.stat = statFix(fs.stat)
//fs.fstat = statFix(fs.fstat)
//fs.lstat = statFix(fs.lstat)
```
# 备用命令
## 清理dns缓存

`sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder`
