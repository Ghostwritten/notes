
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1c03f675404f4f9393a120088a4c485e.png)





[Jupyter Notebook](https://jupyter.org/) 是一款非常流行的交互式开发工具，尤其适合数据科学、机器学习和教学场景。借助 Conda，我们可以方便地安装和管理 Jupyter Notebook 及其依赖。

#### 1. 安装 Conda

在安装 Jupyter Notebook 之前，确保系统已安装 Conda。Conda 可以通过 Anaconda 或 Miniconda 提供。

- **Anaconda**: 完整的 Python 数据科学平台，包含许多常用包。
- **Miniconda**: 精简版，仅包含 Conda 和 Python，适合自定义环境。

##### 下载与安装步骤：
1. 访问 [Miniconda 下载页面](https://docs.conda.io/en/latest/miniconda.html) 或 [Anaconda 下载页面](https://www.anaconda.com/products/distribution)。
2. 下载适合您操作系统的安装包。
3. 执行安装脚本：

   ```bash
   # 示例：在 Linux 系统上安装 Miniconda
   bash Miniconda3-latest-Linux-x86_64.sh
   ```
4. 按提示完成安装。

安装完成后，运行以下命令验证 Conda 是否安装成功：

```bash
conda --version
```

#### 2. 创建虚拟环境

使用 Conda 创建独立的 Python 环境，可以避免不同项目间的依赖冲突。

```bash
conda create -n jupyter_env python=3.9 -y
```

激活环境：

```bash
conda activate jupyter_env
```

#### 3. 安装 Jupyter Notebook

在激活的虚拟环境中，运行以下命令安装 Jupyter Notebook：

```bash
conda install -c conda-forge notebook -y
```

安装完成后，验证安装：

```bash
jupyter notebook --version
```

#### 4. 启动 Jupyter Notebook

启动 Notebook 服务：

```bash
jupyter notebook
```

成功启动后，您将在终端看到类似以下的输出：

```
http://localhost:8888/tree
```

复制链接到浏览器，即可访问 Jupyter Notebook 界面。

> 注意：这只允许本地访问

如果实现远程访问

```bash
jupyter notebook --allow-root --ip=0.0.0.0 --port=8888
```
输出：

```bash
[I 2025-01-03 15:08:19.708 ServerApp] notebook | extension was successfully linked.
[I 2025-01-03 15:08:19.799 ServerApp] notebook_shim | extension was successfully linked.
[I 2025-01-03 15:08:19.805 ServerApp] notebook_shim | extension was successfully loaded.
[I 2025-01-03 15:08:19.806 ServerApp] jupyter_lsp | extension was successfully loaded.
[I 2025-01-03 15:08:19.806 ServerApp] jupyter_server_terminals | extension was successfully loaded.
[I 2025-01-03 15:08:19.807 LabApp] JupyterLab extension loaded from /root/miniconda3/envs/python3.13.1/lib/python3.13/site-packages/jupyterlab
[I 2025-01-03 15:08:19.807 LabApp] JupyterLab application directory is /root/miniconda3/envs/python3.13.1/share/jupyter/lab
[I 2025-01-03 15:08:19.807 LabApp] Extension Manager is 'pypi'.
[I 2025-01-03 15:08:19.825 ServerApp] jupyterlab | extension was successfully loaded.
[I 2025-01-03 15:08:19.826 ServerApp] notebook | extension was successfully loaded.
[I 2025-01-03 15:08:19.826 ServerApp] Serving notebooks from local directory: /root/python
[I 2025-01-03 15:08:19.826 ServerApp] Jupyter Server 2.15.0 is running at:
[I 2025-01-03 15:08:19.826 ServerApp] http://registry.ocp.local:8888/tree?token=83ba987b3e4b4e42270650bb2b32c0d8b39eef8dacab3d7e
[I 2025-01-03 15:08:19.826 ServerApp]     http://127.0.0.1:8888/tree?token=83ba987b3e4b4e42270650bb2b32c0d8b39eef8dacab3d7e
[I 2025-01-03 15:08:19.826 ServerApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[W 2025-01-03 15:08:19.828 ServerApp] No web browser found: Error('could not locate runnable browser').
[C 2025-01-03 15:08:19.828 ServerApp] 
    
    To access the server, open this file in a browser:
        file:///root/.local/share/jupyter/runtime/jpserver-1149459-open.html
    Or copy and paste one of these URLs:
        http://192.168.21.184:8888/tree?token=83ba987b3e4b4e42270650bb2b32c0d8b39eef8dacab3d7e
        http://127.0.0.1:8888/tree?token=83ba987b3e4b4e42270650bb2b32c0d8b39eef8dacab3d7e
[I 2025-01-03 15:08:19.834 ServerApp] Skipped non-installed server(s): bash-language-server, dockerfile-language-server-nodejs, javascript-typescript-langserver, jedi-language-server, julia-language-server, pyright, python-language-server, python-lsp-server, r-languageserver, sql-language-server, texlab, typescript-language-server, unified-language-server, vscode-css-languageserver-bin, vscode-html-languageserver-bin, vscode-json-languageserver-bin, yaml-language-server
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3227b8d7a7a34889ac3cf5c2af071340.png)


#### 5. 安装扩展功能（可选）

为提升使用体验，可以安装 Jupyter Notebook 扩展工具。

```bash
conda install -c conda-forge jupyter_contrib_nbextensions
jupyter contrib nbextension install --user
```

启用常用扩展：

```bash
jupyter nbextension enable <extension_name>
```

#### 6. 更新与维护

定期更新 Jupyter Notebook 以获取最新功能和修复：

```bash
conda update notebook
```

删除虚拟环境（如果不再需要）：

```bash
conda remove -n jupyter_env --all
```

#### 7. 总结

通过 Conda 安装 Jupyter Notebook 是一种快速而高效的方式，尤其适合需要管理多个 Python 环境的用户。您可以根据需求创建独立环境，并灵活扩展 Jupyter 的功能，从而提升开发效率。


