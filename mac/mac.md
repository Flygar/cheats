## 替换homebrew源
地址: http://mirrors.ustc.edu.cn/
```sh
# Homebrew 源代码仓库
# 替换为USTC镜像
cd "$(brew --repo)"
git remote set-url origin https://mirrors.ustc.edu.cn/brew.git
# 重置为官方地址
cd "$(brew --repo)"
git remote set-url origin https://github.com/Homebrew/brew.git

# Homebrew 核心软件仓库
# 替换为USTC镜像
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git
# 重置为官方地址
cd "$(brew --repo)/Library/Taps/homebrew/homebrew-core"
git remote set-url origin https://github.com/Homebrew/homebrew-core

# Homebrew cask 软件仓库,提供 macOS 应用和大型二进制文件
# 替换为 USTC 镜像
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git
# 重置为官方地址
cd "$(brew --repo)"/Library/Taps/homebrew/homebrew-cask
git remote set-url origin https://github.com/Homebrew/homebrew-cask
```
