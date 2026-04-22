Úprava bash skriptu vimura

Pôvodný skript vimura z https://gist.github.com/vext01/16df5bd48019d451e078
používa pdf prehliadač zathura a textový editor vim, pomocou ktorých je
možné používať obojsmernú synchronizáciu medzi zdrojovým textom editovaným
editorom Vim, ktorý je doplnený o niekoľko makier a výsledným pdf súborom
zobrazeným prehliadačom Zathura, ktorý túto synchronizáciu podporuje pomocou
rozšírenia Synctex. Obmedzením je jeho použitie len na jednoduché projekty,
kedy zdrojový tex súbor a výsledný pdf súbor musia mať rovnaký názov súboru,
rozdielna je len ich prípona. Pre väčšie projekty zložené z viacerých
zdrojových súborov sa preto nehodí.

Pre odstránenie tohto obmedzenia vznikol upravený skript, ktorý používa pre
názov hlavného súboru (bez jeho prílohy je rovnaký ako zobrazovaný pdf
súbor) premennú uloženú v súbore ~/.tlfname a číslo procesu príslušného
terminálu, kde beží skript vimura v súbore ~/.ttynum -- tento slúži na
zobrazenie logu z behu TeX-u pri jeho preklade. Pre ich načítanie vo vim-e
je potrebné doplniť konfiguračný súbor ~/.vimrc o tieto riadky:

let fname = readfile(glob('~/.tlfname'))[0]
let ttynum = readfile(glob('~/.ttynum'))[0]

Tieto súbory vytvára skript vimura, preto je ho vhodné spustiť už pred prvým
spustením takto doplneného súbou .vimrc, kde je potrebné doplniť aj funkciu
Synctex:

function! Synctex()
   execute "silent !zathura --synctex-forward " . line('.') . ":" . col('.') . ":" . bufname('%') . " " . g:syncpdf 
  redraw! 
endfunction

Pre použitie tejto funkcie namapujeme jej spustenie v príkazovom (map) aj vo
vkladacom režime (imap) na kombináciu kláves Ctrl-Enter doplnením týchto 
riadkov do .vimrc:

map <C-enter> :call Synctex()<CR>
imap <C-enter> <ESC> :call Synctex()<CR>i

Obdobne pre obidva režimy namapujeme pre kombináciu kláves Ctrl-F5 preklad
zdrojového tex súboru pomocou pdfcsplain do výsledného pdf súboru doplnením
týchto riadkov do .vimrc:

map <C-F5> <ESC>:wa<CR>:exec ":redir! > " . ttynum <CR>:exec "!pdfcsplain -synctex=1 " . fname <CR><CR> :exec ":redir END" <CR>
imap <C-F5> <ESC>:wa<CR>:exec ":redir! > " . ttynum <CR>:exec "!pdfcsplain -synctex=1 " . fname <CR><CR>:exec ":redir END" <CR> i

kde sa počas prekladu zobrazuje jeho priebeh v okne samotného vim-u (či
podľa jeho konfigurácie v skripte vimura jeho grafickom variante gvim), kde
sa po preklade znovu objaví editovaný text. Po preklade je možné jeho
priebeh skontrolovať v okne terminálu, odkiaľ bol spustený bash skript vimura.
