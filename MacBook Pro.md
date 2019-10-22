iTerm + oh-my-zsh

[打造 Mac 下高颜值好用的终端环境](https://blog.biezhi.me/2018/11/build-a-beautiful-mac-terminal-environment.html)

## 安装 zsh-syntax-highlighting

```sh
cd /Users/yuanfanbin/.oh-my-zsh/plugins
brew install zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
echo "source ${(q-)PWD}/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
source ./zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

## 安装 zsh-autosuggestions

```sh
cd /Users/yuanfanbin/.oh-my-zsh/plugins
brew install zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions
echo "source ${(q-)PWD}/zsh-autosuggestions/zsh-autosuggestions.zsh" >> ${ZDOTDIR:-$HOME}/.zshrc
source ./zsh-autosuggestions/zsh-autosuggestions.zsh
```

## 安装 colorls （配合 `powerlevel9k` 使用）

```sh
sudo gem install colorls
echo "source $(dirname $(gem which colorls))/tab_complete.sh" >> ${ZDOTDIR:-$HOME}/.zshrc
echo "alias lc='colorls -A --sd'" >> ${ZDOTDIR:-$HOME}/.zshrc
```
