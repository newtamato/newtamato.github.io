# VIM配色

标签（空格分隔）： vim 配色 

---

> 1. 首先将Solarized的配色库download下来
    git clone git://github.com/altercation/solarized.git
> 2. 将solarized.vim文件copy 到 ~/.vim/colors/  文件夹下面
>3. 配置.vimrc
```
syntax on  
set t_Co=256  
let g:solarized_termcolors=256  
set background=dark  
colorscheme solarized  
```
    有两处配置颜色的地方，一处在/usr/share/vim/colors/,一处在~/.vim/colors。前一处是给所有user使用的，后一处仅仅是给当前登录的用户使用的配色 
        







