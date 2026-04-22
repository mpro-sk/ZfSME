Úprava bash skriptu texloop pre beh s xdvi v Ubuntu 20.4 (a novšie)

V pôvodnom skripte texloop z https://petr.olsak.net/texinunix.html
je pre vetvu xdvi s aktualizáciou prehliadača použitý signál
kill -USR1 $VIEWPID (riadok 72, pôvodne riadok 71). V dnešných systémoch UBUNTU,
alebo DEBIAN sa však prehliadač nespúšťa priamo, ale cez wrapper 
"/usr/bin/perl -w /usr/bin/xdvi jemny", kde jemny značí meno TeXového súboru:

$ps -a
 4407 pts/2    S      0:00 /bin/bash /home/marian/bin/texloop csplain jemny A
 4415 pts/2    S      0:00 /usr/bin/perl -w /usr/bin/xdvi jemny
 4419 pts/2    S      0:01 xdvi.bin -name xdvi jemny

Problém je, že zaslaním signálu USR1 na proces s wrapperom, nedôjde 
k aktualizovaniu prehliadača xdvi, ale k jeho ukončeniu -- samotný proces 
s prehliadačom ma iné PID. Upravený skript má preto zakomentovaný riadok

# kill -USR1 $VIEWPID

a nad ním je odkaz na podrobný popis problému:

# https://bugs.launchpad.net/ubuntu/+source/texlive-bin/+bug/399148

V upravenej verzii som použil dirty hack, kde namiesto premennej $VIEWPID 
v ktorej sa nachádza PID wrappera sa na riadku 72 nachádza toto:

kill -USR1 $(ps a | grep xdvi.bin | awk 'NR==1{print $1}')

Výraz v zátvorke má túto funkciu:

ps a | grept xdvi.bin | vypíše všetky procesy, kde sa nachádza reťazec xdvi.bin
a pošle ich rúrou programu awk, ktorý vyberie z tohoto zoznamu 1. riadok a 
v prvom stĺpci riadku sa nachádza požadované PID. Problém môže nastať, ak sa už
aj predtým spustil prehliadač xdvi.bin -- vtedy však nenastane ukončenie 
procesu, len sa náš prehliadač xdvi spustený skriptom texloop neaktualizuje 
(na to však stačí myšou ťuknúť do okna s dvi obrazom, alebo stlačiť Shift-R 
(Reread dvi file), alebo Ctrl-R (Redisplays the current page).

===

Natavenia pre xdvi forward a reverse search:

- preklad z tex do dvi robiť s parametrom "-src-specials"*) -- viď riadky
  23, 24, 62 a 63 texloop; v tomto prípade sa v xdvi po zadaní Ctrl+V
  zobrazia malé červené štvorčeky na začiatku odstavca, kde sa premiestni
  kurzor vim-u v zdrojovom texte po zadaní Ctrl+ľavé tlačidlo myši v xdvi.

- okrem generovania súboru  ~/.tlpid sa generuje aj súbor ~/.tlfname s názvom
  zdrojového súboru, aby dochádzalo len k prekladu tohoto súboru aj v prípade
  rozdelenia zdroja do viacerých súborov -- viď riadok 92 texloop

- pre forward search (zo zdrojového tex editovanom vo vim-e do dvi zobrazenom 
  v xdvi) nastavené makro vim-u na klávesu F3 -- vo .vimrc je to táto časť:

""" povodny variant z https://vim.fandom.com/wiki/Vim_can_interact_with_xdvi
""" (funguje len pre rovnaky nazov suboru *.tex a *.dvi)
"map <F3> :execute "!xdvi -sourceposition " . line(".") . expand("%") . " " .  expand("%:r") . ".dvi"<CR><CR>
""" nazov suboru sa prebera z obsahu suboru ~/.tlfname, ktory vytvara
""" skript texloop
let fname = readfile(glob('~/.tlfname'))[0]
map <F3> :execute "!xdvi -sourceposition " . line(".") . expand("%") . " " . fname . ".dvi"<CR><CR>
imap <F3> <ESC>:execute "!xdvi -sourceposition " . line(".") . expand("%") . " " . fname . ".dvi"<CR><CR> i

- pre reverse search je potrebné spustiť xdvi s definovaním parametra -editor:
  xdvi -editor "uxterm -e vim --servername xdvi --remote" $MAINFILEBASE &
  viď riadok 46 texloop

- editor vim je potrebné spustiť s parametrom vim --servername xdvi; pre toto 
  mám v ~/.bash_aliases vytvorený alias pre tvim -- riadok:
  alias tvim='vim --servername xdvi'

Pre takto spustený vim sa bude premiestňovať kurzor z miesta v xdvi do
zdrojového textu tex otvoreného vo vim. Pokiaľ nie je spustená ani jedna
inštancia takéhoto vim-u, otvorí sa nové okno UXterm-u a v ňom sa bude
presúvať kurzor na požadované miesto.

---
*) Poznámka k použitiu parametra -src-specials:
Výstupom je dvi súbor, ktorý sa v niektorých prípadoch {\em môže} líšiť od
dvi súboru, keby preklad prebehol bez tohto parametra aj v mieste zlomu
strany, takže tento môže mať iný počet strán. Príkladom je tento text vo
formáte tex (bez tejto poznámky a následného textu má pri použití veľkosti
strany A5 dve presne strany, ale ak sa použije ""-src-specials" budú to strany
tri). To isté platí aj pre pdf súbor. Pre tento formát nemá toto rozšírenie
žiadny význam, preto sa jeho použitie v texloop pre formát pdf automaticky
potlačí. Pokiaľ chcete použitie -src-specials potlačiť aj pre dvi formát,
stačí ak posledný parameter skriptu texloop (t. j. tretí alebo štvrtý) je
"n".

===

Do pôvodného skriptu bola pri otváraní aplikácie xdvi doplnená aj možnosť
výberu formátu strany -- okrem prednastavenej hodnoty A4 ke možné zadať aj
iné formáty, ktoré pozná aplikácia xdvi (letter, A5, A6, B5, B6, ...)
