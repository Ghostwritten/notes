```bash
$ git push orgin master
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 4 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (7/7), 313.63 MiB | 838.00 KiB/s, done.
Total 7 (delta 0), reused 0 (delta 0)
remote: warning: File docker/Docker容器与容器云（第2版）.pdf is 51.35 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB
remote: warning: File docker/Docker实战.pdf is 97.64 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB
remote: error: Trace: 1cfbf943c54b3452f96941481112e6535eb9db73b88495250362999ad6fbf534
remote: error: See http://git.io/iEPt8g for more information.
remote: error: File docker/《互联网企业容器技术实践》_龚曦_2019-01-01.pdf is 197.29 MB; this exceeds GitHub's file size limit of 100.00 MB
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.
To https://github.com/Ghostwritten/pdfbook.git
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'https://github.com/Ghostwritten/pdfbook.git'
```
## 解决方法
[https://git-lfs.github.com/](https://git-lfs.github.com/)

1.下载并安装 Git 命令行扩展。下载并安装后，运行以下命令为您的用户帐户设置 Git LFS：

```bash
git lfs install
```
2.在您要使用 Git LFS 的每个 Git 存储库中，选择您希望 Git LFS 管理的文件类型（或直接编辑您的 .gitattributes）。您可以随时配置其他文件扩展名

```css
git lfs migrate import --include="*.pdf"
git lfs track "*.pdf"
```
现在确保 .gitattributes 被跟踪：

```css
git add .gitattributes
```

> 请注意，定义 Git LFS 应该跟踪的文件类型本身不会将任何预先存在的文件转换为 Git LFS，例如其他分支上的文件或您之前的提交历史记录中的文件。为此，请使用git lfs migrate[1]命令，该命令具有一系列选项，旨在适应各种潜在用例。

3.没有第三步。只需像往常一样提交并推送到 GitHub；例如，如果您当前的分支名为main：

```css
git add --all
git commit -m "Add design file"
git push origin master --force
```

