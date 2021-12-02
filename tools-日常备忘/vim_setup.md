### vim setup & usage

#### setup

```.vimrc
" set tag files, separate by ,
"set tags=/data11/haydengao/ads_dev/trunk/tags

" ignore case
set ic

" hight light search
set hlsearch

" encode style
set fileencodings=ucs-bom,utf-8,utf-16,gbk,big5,gb18030,latin1
set termencoding=utf-8
set encoding=utf-8

" show tabs and EOL space as '^I' and '$', respectively
set list
set listchars=trail:$

" auto indent
set autoindent

" set line number
set nu

" auto change dir.
set autochdir

" statusline setup
set laststatus=2
function! GitBranch()
  return system("git rev-parse --abbrev-ref HEAD 2>/dev/null | tr -d '\n'")
endfunction

function! StatuslineGit()
  let l:branchname = GitBranch()
  return strlen(l:branchname) > 0?'  '.l:branchname.' ':''
endfunction

"set statusline=
"set statusline+=%#PmenuSel#
"set statusline+=%{StatuslineGit()}
"set statusline+=%#LineNr#
set statusline+=\ %F
set statusline+=%m\
set statusline+=%=
set statusline+=%#CursorColumn#
set statusline+=\ %y
set statusline+=\ %{&fileencoding?&fileencoding:&encoding}
set statusline+=\[%{&fileformat}\]
set statusline+=\ %p%%
set statusline+=\ %l:%c
set statusline+=\
```

#### usage

* m + char	标记位置
  ’ + char	跳转到标记处
  ‘’	跳转回上一个光标位置
* % --- 匹配 "(", "{"
*   H、M、L：直接跳转到当前屏幕的顶部、中部、底部。
* tag:
  ctrl + ] : 跳转到声明/定义处
  ctrl + t : 跳转回来
  :ts ---当有多个可能时，列出所有的选项