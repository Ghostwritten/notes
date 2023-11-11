

----
##  1. 简介
-- --High-level file operations  高级的文件操作模块。

　　os模块提供了对目录或者文件的新建/删除/查看文件属性，还提供了对文件以及目录的路径操作。比如说：绝对路径，父目录……  但是，os文件的操作还应该包含移动 复制  打包 压缩 解压等操作，这些os模块都没有提供。

　　而本章所讲的shutil则就是对os中文件操作的补充。--移动 复制  打包 压缩 解压，


##  2. 功能
### 2.1  shutil.copyfileobj(fsrc, fdst[, length=16*1024])    
copy文件内容到另一个文件，可以copy指定大小的内容

```python
#先来看看其源代码。
def copyfileobj(fsrc, fdst, length=16*1024):
    """copy data from file-like object fsrc to file-like object fdst"""
    while 1:
        buf = fsrc.read(length)
        if not buf:
            break
        fdst.write(buf)

#注意！ 在其中fsrc，fdst都是文件对象，都需要打开后才能进行复制操作
```
示例

```python
root@test1:~/python/shutil# cat name
hello world!

root@test1:~/python/shutil# cat test1.py 
import shutil
f1=open('name','r')
f2=open('name_copy','w+')
shutil.copyfileobj(f1,f2,length=16*1024)

root@test1:~/python/shutil# python test1.py

root@test1:~/python/shutil# cat name_copy 
hello world!

```
### 2.2 shutil.copyfile(src,dst) 
copy文件内容，是不是感觉上面的文件复制很麻烦？还需要自己手动用open函数打开文件，在这里就不需要了，事实上，copyfile调用了copyfileobj

源代码

```python
def copyfile(src, dst, *, follow_symlinks=True):
    """Copy data from src to dst.

    If follow_symlinks is not set and src is a symbolic link, a new
    symlink will be created instead of copying the file it points to.

    """
    if _samefile(src, dst):
        raise SameFileError("{!r} and {!r} are the same file".format(src, dst))

    for fn in [src, dst]:
        try:
            st = os.stat(fn)
        except OSError:
            # File most likely does not exist
            pass
        else:
            # XXX What about other special files? (sockets, devices...)
            if stat.S_ISFIFO(st.st_mode):
                raise SpecialFileError("`%s` is a named pipe" % fn)

    if not follow_symlinks and os.path.islink(src):
        os.symlink(os.readlink(src), dst)
    else:
        with open(src, 'rb') as fsrc:
            with open(dst, 'wb') as fdst:
                copyfileobj(fsrc, fdst)
    return dst
```
示例

```python
root@test1:~/python/shutil# cat name
hello world!
root@test1:~/python/shutil# cat test2.py 
import shutil

shutil.copyfile('name','name_copy_2')
root@test1:~/python/shutil# python test2.py
root@test1:~/python/shutil# cat name_copy_2 
hello world!
```
### 2.3 shutil.copymode(src,dst)
仅copy权限，不更改文件内容，组和用户。

源代码
```python
def copymode(src, dst, *, follow_symlinks=True):
    """Copy mode bits from src to dst.

    If follow_symlinks is not set, symlinks aren't followed if and only
    if both `src` and `dst` are symlinks.  If `lchmod` isn't available
    (e.g. Linux) this method does nothing.

    """
    if not follow_symlinks and os.path.islink(src) and os.path.islink(dst):
        if hasattr(os, 'lchmod'):
            stat_func, chmod_func = os.lstat, os.lchmod
        else:
            return
    elif hasattr(os, 'chmod'):
        stat_func, chmod_func = os.stat, os.chmod
    else:
        return

    st = stat_func(src)
    chmod_func(dst, stat.S_IMODE(st.st_mode))
```
示例

```python
#先看两个文件的权限
[root@slyoyo python_test]# ls -l
total 4
-rw-r--r--. 1 root root 79 May 14 05:17 test1
-rwxr-xr-x. 1 root root  0 May 14 19:10 test2

#运行命令
>>> import shutil
>>> shutil.copymode('test1','test2')

#查看结果
[root@slyoyo python_test]# ls -l
total 4
-rw-r--r--. 1 root root 79 May 14 05:17 test1
-rw-r--r--. 1 root root  0 May 14 19:10 test2

#当我们将目标文件换为一个不存在的文件时报错
>>> shutil.copymode('test1','test3')    
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/local/python/lib/python3.4/shutil.py", line 132, in copymode
    chmod_func(dst, stat.S_IMODE(st.st_mode))
FileNotFoundError: [Errno 2] No such file or directory: 'test233'
```
### 2.4  shutil.copystat(src,dst) 
复制所有的状态信息，包括权限，组，用户，时间等
源代码

```python
def copystat(src, dst, *, follow_symlinks=True):
    """Copy all stat info (mode bits, atime, mtime, flags) from src to dst.

    If the optional flag `follow_symlinks` is not set, symlinks aren't followed if and
    only if both `src` and `dst` are symlinks.

    """
    def _nop(*args, ns=None, follow_symlinks=None):
        pass

    # follow symlinks (aka don't not follow symlinks)
    follow = follow_symlinks or not (os.path.islink(src) and os.path.islink(dst))
    if follow:
        # use the real function if it exists
        def lookup(name):
            return getattr(os, name, _nop)
    else:
        # use the real function only if it exists
        # *and* it supports follow_symlinks
        def lookup(name):
            fn = getattr(os, name, _nop)
            if fn in os.supports_follow_symlinks:
                return fn
            return _nop

    st = lookup("stat")(src, follow_symlinks=follow)
    mode = stat.S_IMODE(st.st_mode)
    lookup("utime")(dst, ns=(st.st_atime_ns, st.st_mtime_ns),
        follow_symlinks=follow)
    try:
        lookup("chmod")(dst, mode, follow_symlinks=follow)
    except NotImplementedError:
        # if we got a NotImplementedError, it's because
        #   * follow_symlinks=False,
        #   * lchown() is unavailable, and
        #   * either
        #       * fchownat() is unavailable or
        #       * fchownat() doesn't implement AT_SYMLINK_NOFOLLOW.
        #         (it returned ENOSUP.)
        # therefore we're out of options--we simply cannot chown the
        # symlink.  give up, suppress the error.
        # (which is what shutil always did in this circumstance.)
        pass
    if hasattr(st, 'st_flags'):
        try:
            lookup("chflags")(dst, st.st_flags, follow_symlinks=follow)
        except OSError as why:
            for err in 'EOPNOTSUPP', 'ENOTSUP':
                if hasattr(errno, err) and why.errno == getattr(errno, err):
                    break
            else:
                raise
    _copyxattr(src, dst, follow_symlinks=follow)
```
示例

```python
root@test1:~/python/shutil# cat test4.py 
import shutil

shutil.copystat('name','name_copy4') 


root@test1:~/python/shutil# python test4.py
root@test1:~/python/shutil# stat name_copy4
  File: name_copy4
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: fd00h/64768d	Inode: 146632      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2021-07-20 08:02:27.455574000 +0000
Modify: 2021-07-20 08:02:19.403713000 +0000
Change: 2021-07-20 08:17:31.152624136 +0000
 Birth: -
root@test1:~/python/shutil# stat name
  File: name
  Size: 13        	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 136569      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2021-07-20 08:02:27.455574893 +0000
Modify: 2021-07-20 08:02:19.403713825 +0000
Change: 2021-07-20 08:02:19.403713825 +0000
 Birth: -
```
###  2.5 shutil.copy(src,dst) 
复制文件的内容以及权限，先copyfile后copymode

```python
def copy(src, dst, *, follow_symlinks=True):
    """Copy data and mode bits ("cp src dst"). Return the file's destination.

    The destination may be a directory.

    If follow_symlinks is false, symlinks won't be followed. This
    resembles GNU's "cp -P src dst".

    If source and destination are the same file, a SameFileError will be
    raised.

    """
    if os.path.isdir(dst):
        dst = os.path.join(dst, os.path.basename(src))
    copyfile(src, dst, follow_symlinks=follow_symlinks)
    copymode(src, dst, follow_symlinks=follow_symlinks)
    return dst
```
示例

```python
root@test1:~/python/shutil# ls -l name
-rw-r--r-- 1 root root 13 Jul 20 08:02 name
root@test1:~/python/shutil# cat name
hello world!
root@test1:~/python/shutil# cat test5.py 
import shutil
shutil.copy('name','name_copy5') 
root@test1:~/python/shutil# python test5.py 
root@test1:~/python/shutil# ls -l name_copy5
-rw-r--r-- 1 root root 13 Jul 20 08:20 name_copy5
root@test1:~/python/shutil# cat  name_copy5
hello world!
```
### 2.6 shutil.copy2(src,dst)
复制文件的内容以及文件的所有状态信息。先copyfile后copystat

```python
def copy2(src, dst, *, follow_symlinks=True):
    """Copy data and all stat info ("cp -p src dst"). Return the file's
    destination."

    The destination may be a directory.

    If follow_symlinks is false, symlinks won't be followed. This
    resembles GNU's "cp -P src dst".

    """
    if os.path.isdir(dst):
        dst = os.path.join(dst, os.path.basename(src))
    copyfile(src, dst, follow_symlinks=follow_symlinks)
    copystat(src, dst, follow_symlinks=follow_symlinks)
    return dst
```
示例

```python
root@test1:~/python/shutil# stat name
  File: name
  Size: 13        	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 136569      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2021-07-20 08:02:27.455574893 +0000
Modify: 2021-07-20 08:02:19.403713825 +0000
Change: 2021-07-20 08:02:19.403713825 +0000
 Birth: -

root@test1:~/python/shutil# cat test6.py
import shutil
shutil.copy2('name','name_copy6') 
root@test1:~/python/shutil# python test6.py 
root@test1:~/python/shutil# stat name_copy6
  File: name_copy6
  Size: 13        	Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d	Inode: 146633      Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2021-07-20 08:02:27.455574000 +0000
Modify: 2021-07-20 08:02:19.403713000 +0000
Change: 2021-07-20 08:22:27.266375293 +0000
 Birth: -
root@test1:~/python/shutil# cat name_copy6
hello world!
```
###  2.7 shutil.copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2,ignore_dangling_symlinks=False) 
递归的复制文件内容及状态信息

源代码

```python
def copytree(src, dst, symlinks=False, ignore=None, copy_function=copy2,
             ignore_dangling_symlinks=False):

    names = os.listdir(src)
    if ignore is not None:
        ignored_names = ignore(src, names)
    else:
        ignored_names = set()

    os.makedirs(dst)
    errors = []
    for name in names:
        if name in ignored_names:
            continue
        srcname = os.path.join(src, name)
        dstname = os.path.join(dst, name)
        try:
            if os.path.islink(srcname):
                linkto = os.readlink(srcname)
                if symlinks:
                    # We can't just leave it to `copy_function` because legacy
                    # code with a custom `copy_function` may rely on copytree
                    # doing the right thing.
                    os.symlink(linkto, dstname)
                    copystat(srcname, dstname, follow_symlinks=not symlinks)
                else:
                    # ignore dangling symlink if the flag is on
                    if not os.path.exists(linkto) and ignore_dangling_symlinks:
                        continue
                    # otherwise let the copy occurs. copy2 will raise an error
                    if os.path.isdir(srcname):
                        copytree(srcname, dstname, symlinks, ignore,
                                 copy_function)
                    else:
                        copy_function(srcname, dstname)
            elif os.path.isdir(srcname):
                copytree(srcname, dstname, symlinks, ignore, copy_function)
            else:
                # Will raise a SpecialFileError for unsupported file types
                copy_function(srcname, dstname)
        # catch the Error from the recursive copytree so that we can
        # continue with other files
        except Error as err:
            errors.extend(err.args[0])
        except OSError as why:
            errors.append((srcname, dstname, str(why)))
    try:
        copystat(src, dst)
    except OSError as why:
        # Copying file access times may fail on Windows
        if getattr(why, 'winerror', None) is None:
            errors.append((src, dst, str(why)))
    if errors:
        raise Error(errors)
    return dst
```

示例

```python
[root@slyoyo python_test]# tree copytree_test/
copytree_test/
└── test
    ├── test1
    ├── test2
    └── hahaha

[root@slyoyo test]# ls -l
total 0
-rw-r--r--. 1 python python 0 May 14 19:36 hahaha
-rw-r--r--. 1 python python 0 May 14 19:36 test1
-rw-r--r--. 1 root   root   0 May 14 19:36 test2

>>> shutil.copytree('copytree_test','copytree_copy')
'copytree_copy'

[root@slyoyo python_test]# ls -l
total 12
drwxr-xr-x. 3 root   root   4096 May 14 19:36 copytree_copy
drwxr-xr-x. 3 root   root   4096 May 14 19:36 copytree_test
-rw-r--r--. 1 python python   79 May 14 05:17 test1
-rw-r--r--. 1 root   root      0 May 14 19:10 test2
[root@slyoyo python_test]# tree copytree_copy/
copytree_copy/
└── test
    ├── hahaha
    ├── test1
    └── test2
```
###  2.8 shutil.rmtree(path, ignore_errors=False, onerror=None) 
递归地删除文件

源代码
```python
def rmtree(path, ignore_errors=False, onerror=None):

    if ignore_errors:
        def onerror(*args):
            pass
    elif onerror is None:
        def onerror(*args):
            raise
    if _use_fd_functions:
        # While the unsafe rmtree works fine on bytes, the fd based does not.
        if isinstance(path, bytes):
            path = os.fsdecode(path)
        # Note: To guard against symlink races, we use the standard
        # lstat()/open()/fstat() trick.
        try:
            orig_st = os.lstat(path)
        except Exception:
            onerror(os.lstat, path, sys.exc_info())
            return
        try:
            fd = os.open(path, os.O_RDONLY)
        except Exception:
            onerror(os.lstat, path, sys.exc_info())
            return
        try:
            if os.path.samestat(orig_st, os.fstat(fd)):
                _rmtree_safe_fd(fd, path, onerror)
                try:
                    os.rmdir(path)
                except OSError:
                    onerror(os.rmdir, path, sys.exc_info())
            else:
                try:
                    # symlinks to directories are forbidden, see bug #1669
                    raise OSError("Cannot call rmtree on a symbolic link")
                except OSError:
                    onerror(os.path.islink, path, sys.exc_info())
        finally:
            os.close(fd)
    else:
        return _rmtree_unsafe(path, onerror)

# Allow introspection of whether or not the hardening against symlink
# attacks is supported on the current platform
rmtree.avoids_symlink_attacks = _use_fd_functions
```
示例

```python
root@test1:~/python/shutil# mkdir copy_tree/test1/a
root@test1:~/python/shutil# cat test7.py 
import shutil
shutil.rmtree('./copy_tree/test1/a', ignore_errors=False, onerror=None) 
root@test1:~/python/shutil# python test7.py
root@test1:~/python/shutil# ls copy_tree/
test1
root@test1:~/python/shutil# ls copy_tree/test1/
```
### 2.9 shutil.move(src, dst)
递归的移动文件
源代码

```python
def move(src, dst):

    real_dst = dst
    if os.path.isdir(dst):
        if _samefile(src, dst):
            # We might be on a case insensitive filesystem,
            # perform the rename anyway.
            os.rename(src, dst)
            return

        real_dst = os.path.join(dst, _basename(src))
        if os.path.exists(real_dst):
            raise Error("Destination path '%s' already exists" % real_dst)
    try:
        os.rename(src, real_dst)
    except OSError:
        if os.path.islink(src):
            linkto = os.readlink(src)
            os.symlink(linkto, real_dst)
            os.unlink(src)
        elif os.path.isdir(src):
            if _destinsrc(src, dst):
                raise Error("Cannot move a directory '%s' into itself '%s'." % (src, dst))
            copytree(src, real_dst, symlinks=True)
            rmtree(src)
        else:
            copy2(src, real_dst)
            os.unlink(src)
    return real_dst
```
示例

```python
root@test1:~/python/shutil# ls copy_tree/test1/
root@test1:~/python/shutil# mkdir a
root@test1:~/python/shutil# cat test8.py 
import shutil
shutil.move('a','copy_tree/test1')
root@test1:~/python/shutil# python test8.py
root@test1:~/python/shutil# ls copy_tree/test1/
a
```
###  2.10 make_archive(base_name, format, root_dir=None, base_dir=None, verbose=0,dry_run=0, owner=None, group=None, logger=None) 

压缩打包

源代码

```python
def make_archive(base_name, format, root_dir=None, base_dir=None, verbose=0,
                 dry_run=0, owner=None, group=None, logger=None):

    save_cwd = os.getcwd()
    if root_dir is not None:
        if logger is not None:
            logger.debug("changing into '%s'", root_dir)
        base_name = os.path.abspath(base_name)
        if not dry_run:
            os.chdir(root_dir)

    if base_dir is None:
        base_dir = os.curdir

    kwargs = {'dry_run': dry_run, 'logger': logger}

    try:
        format_info = _ARCHIVE_FORMATS[format]
    except KeyError:
        raise ValueError("unknown archive format '%s'" % format)

    func = format_info[0]
    for arg, val in format_info[1]:
        kwargs[arg] = val

    if format != 'zip':
        kwargs['owner'] = owner
        kwargs['group'] = group

    try:
        filename = func(base_name, base_dir, **kwargs)
    finally:
        if root_dir is not None:
            if logger is not None:
                logger.debug("changing back to '%s'", save_cwd)
            os.chdir(save_cwd)

    return filename
```

 - base_name：    压缩打包后的文件名或者路径名
 - format：          压缩或者打包格式    "zip", "tar", "bztar"or "gztar"
 - root_dir :         将哪个目录或者文件打包（也就是源文件）

示例

```python
>>> shutil.make_archive('tarball','gztar',root_dir='copytree_test')

[root@slyoyo python_test]# ls -l
total 12
drwxr-xr-x. 3 root   root   4096 May 14 19:36 copytree_copy
drwxr-xr-x. 3 root   root   4096 May 14 19:36 copytree_test
-rw-r--r--. 1 root   root      0 May 14 21:12 tarball.tar.gz
-rw-r--r--. 1 python python   79 May 14 05:17 test1
-rw-r--r--. 1 root   root      0 May 14 19:10 test2
```

参考链接：
[https://www.cnblogs.com/MnCu8261/p/5494807.html](https://www.cnblogs.com/MnCu8261/p/5494807.html)
