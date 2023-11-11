#  Linux Command Alias 别名
alias命令别名用法：
```bash
# save fingers!
alias l='ls'
# long listing of ls
alias ll='ls -l'
# colors and file types
alias lf='ls -CF'		
# sort by filename extension
alias lx='ls -lXB'              
# sort by size
alias lk='ls -lSr'              
# show hidden files
alias la='ls -A'               
# sort by date
alias lt='ls -ltr'              

# other aliases

# launch webpages from terminal
alias bbc='firefox http://www.bbc.co.uk/ &'
alias sd='firefox http://slashdot.org/ &'
alias www='firefox'

# ssh to common destinations by just typing their name
# log in to 'declan'
alias declan='ssh declan'
# log in to work using a non-standard port (222)
alias work='ssh work.example.com -p 222'
# log in to work and tunnel the internal proxy to localhost:80
alias workweb='ssh work.example.com -p 222 -L 80:proxy.example.com:8080'
```

