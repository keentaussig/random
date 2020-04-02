
### Indent Line Plugin

https://github.com/Yggdroot/indentLine

```
git clone https://github.com/Yggdroot/indentLine.git ~/.vim/pack/vendor/start/indentLint
vim -u NONE -c "helptags  ~/.vim/pack/vendor/start/indentLint/doc" -c "q"
```

### .vimrc
```
set exrc
set secure
set paste
set number

filetype plugin indent on
" show existing tab with 4 spaces width
set tabstop=2
" when indenting with '>', use 4 spaces width
set shiftwidth=2
" On pressing tab, insert 4 spaces
set expandtab
set softtabstop=2


set colorcolumn=110
highlight ColorColumn ctermbg=darkgray

augroup project
  autocmd!
  autocmd BufRe
```
