```bash
$ git push origin master
Uploading LFS objects: 100% (4/4), 367 MB | 0 B/s, done.
Enumerating objects: 15, done.
Counting objects: 100% (15/15), done.
Delta compression using up to 4 threads
Compressing objects: 100% (13/13), done.
error: RPC failed; curl 92 HTTP/2 stream 0 was not closed cleanly: CANCEL (err 8)
fatal: the remote end hung up unexpectedly
Writing objects: 100% (15/15), 313.63 MiB | 1.87 MiB/s, done.
Total 15 (delta 1), reused 0 (delta 0)
fatal: the remote end hung up unexpectedly
Everything up-to-date
```
解决方法
```c
XH@DESKTOP-2FKN21J MINGW64 /d/github/pdfbook (master)
$ git config --global http.postBuffer 1048576000

XH@DESKTOP-2FKN21J MINGW64 /d/github/pdfbook (master)
$ git push origin master
```

