#  Python 应用程序布局




在本文中，我想为您提供一个可靠的 Python 应用程序布局参考指南，您可以参考它来解决您的绝大多数用例。

您将看到常见 Python 应用程序结构的示例，包括[命令行应用程序](https://realpython.com/python-command-line-arguments/)（CLI 应用程序）、一次性脚本、可安装包以及使用[Flask](https://flask.palletsprojects.com/en/2.0.x/)和[Django](https://www.djangoproject.com/)等流行框架的 Web 应用程序布局。

> 注意：本参考指南假定您具备 Python 模块和包的工作知识。如果您觉得有点生疏，请查看我们对 [Python 模块和包的介绍以复习](https://realpython.com/python-modules-packages/)。

##  1. 命令行应用程序布局
我们中的很多人主要使用[通过命令行界面](https://realpython.com/comparing-python-command-line-parsing-libraries-argparse-docopt-click/) (CLI)运行的 Python 应用程序。

从一个空的项目文件夹开始可能会令人生畏，并且不会导致编码器块不足。在本节中，我想分享一些经过验证的布局，我个人将这些布局用作我所有 Python CLI 应用程序的起点。

我们将从一个非常基本的用例的非常基本的布局开始：一个独立运行的简单脚本。然后，您将看到如何随着用例的推进来构建布局。

###  1.1 一次性脚本
如果您只是制作一个供自己使用的脚本，或者没有任何外部[依赖项](https://realpython.com/courses/managing-python-dependencies/)的脚本，那很好，但是如果您必须分发它怎么办？特别是对于不太懂技术的用户？

以下布局适用于所有这些情况，并且可以轻松修改以反映您在工作流程中使用的任何安装或其他工具。无论您是创建纯 Python 脚本（即没有依赖项的脚本）还是使用诸如pip或Pipenv 之类的工具，此布局都将涵盖您。

在阅读本参考指南时，请记住，文件在布局中的确切位置比它们放置在何处的原因更重要。所有这些文件都应位于以您的项目命名的项目目录中。对于这个例子，我们将使用（还有什么？）helloworld作为项目名称和根目录。

这是我通常用于 CLI 应用程序的 Python 项目结构：

```bash
helloworld/
│
├── .gitignore
├── helloworld.py
├── LICENSE
├── README.md
├── requirements.txt
├── setup.py
└── tests.py
```
这非常简单：所有内容都在同一个目录中。此处显示的文件不一定详尽，但如果您打算使用这样的基本布局，我建议将文件数量保持在最低限度。其中一些文件对您来说是新的，所以让我们快速了解一下它们的作用。

 - `gitignore`：这是一个告诉 Git 忽略哪些类型的文件的文件，例如 IDE 混乱文件或本地配置文件。[我们的 Git 教程包含所有详细信息](https://realpython.com/python-git-github-intro/#gitignore)，您可以[在此处](https://github.com/github/gitignore).gitignore找到Python 项目的示例文件。
 - `helloworld.py`: 这是您分发的脚本。至于主脚本文件的命名，我建议您使用项目的名称（与顶级目录的名称相同）。
 - `LICENSE`：此纯文本文件描述了您用于项目的许可证。如果您要分发代码，拥有一个总是一个好主意。按照惯例，文件名全部大写。

> 注意：需要帮助为您的项目选择许可证？查看选择[许可证](https://choosealicense.com/)。

 - `README.md`：这是一个Markdown（或reStructuredText）文件，记录了您的应用程序的目的和用法。制作一件物品是一门艺术，但您可以在这里README找到精通的捷径。
 - `requirements.txt`：此文件为您的应用程序定义外部 Python 依赖项及其版本。
 - `setup.py`: 这个文件也可以用来定义依赖关系，但是对于其他需要在安装过程中完成的工作来说，它确实很出色。您可以在我们的 [Pipenv 指南](https://realpython.com/pipenv-guide/)中阅读更多关于两者setup.py的信息。requirements.txt
 - `tests.py`：这个脚本包含你的测试，如果你有的话。[你应该有一些](https://realpython.com/python-testing/)。


###  1.2 安装单包
假设它`helloworld.py`仍然是要执行的主脚本，但是您已将所有辅助方法移动到一个名为`helpers.py`.

我们将把helloworld Python 文件打包在一起，但将所有杂项文件（例如 your README、.gitignore等）保存在顶层目录中。

让我们看一下更新后的结构：

```kotlin
helloworld/
│
├── helloworld/
│   ├── __init__.py
│   ├── helloworld.py
│   └── helpers.py
│
├── tests/
│   ├── helloworld_tests.py
│   └── helpers_tests.py
│
├── .gitignore
├── LICENSE
├── README.md
├── requirements.txt
└── setup.py
```
这里唯一的区别是你的应用程序代码现在都保存在helloworld子目录中——这个目录以你的包命名——并且我们添加了一个名为`__init__.py.` 让我们介绍这些新文件：

 - `helloworld/__init__.py`: 这个文件有很多功能，但是为了我们的目的，它告诉 Python 解释器这个目录是一个包目录。您可以设置此`__init__.py`文件，使您能够从整个包中导入类和方法，而不是了解内部模块结构并从`helloworld.helloworld`或导入`helloworld.helpers`。

> 注意：对于内部包和 的更深入讨论__init__.py，[我们的 Python 模块和包概述已涵盖](https://realpython.com/python-modules-packages/)。

 - `helloworld/helpers.py`: 如上所述，我们已经将helloworld.py's 的大部分业务逻辑移到了这个文件中。多亏了`__init__.py`，外部模块将能够通过从`helloworld`包中导入来访问这些帮助程序。
 - `tests/`：我们已经将我们的测试移到了他们自己的目录中，随着我们的程序结构变得越来越复杂，您将继续看到这种模式。我们还将测试拆分为单独的模块，以反映我们包的结构。

[此布局是Kenneth Reitz 的 samplemod 应用程序结构的精简版](https://github.com/navdeep-G/samplemod)。这是您的 CLI 应用程序的另一个很好的起点，尤其是对于更广泛的项目。

###  1.3 带有内部包的应用程序
在较大的应用程序中，您可能有一个或多个内部包，它们要么与主运行程序脚本绑定在一起，要么为您正在打包的较大库提供特定功能。我们将扩展上述约定以适应这种情况：

```bash
helloworld/
│
├── bin/
│
├── docs/
│   ├── hello.md
│   └── world.md
│
├── helloworld/
│   ├── __init__.py
│   ├── runner.py
│   ├── hello/
│   │   ├── __init__.py
│   │   ├── hello.py
│   │   └── helpers.py
│   │
│   └── world/
│       ├── __init__.py
│       ├── helpers.py
│       └── world.py
│
├── data/
│   ├── input.csv
│   └── output.xlsx
│
├── tests/
│   ├── hello
│   │   ├── helpers_tests.py
│   │   └── hello_tests.py
│   │
│   └── world/
│       ├── helpers_tests.py
│       └── world_tests.py
│
├── .gitignore
├── LICENSE
└── README.md
```
这里还有更多内容需要消化，但只要您记住它遵循之前的布局，您就会更轻松地跟随。我将按顺序介绍添加和修改、它们的用途以及您可能需要它们的原因。

 - `bin/`：此目录包含所有可执行文件。我改编自[Jean-Paul Calderone 的经典结构帖子](http://as.ynchrono.us/2007/12/filesystem-structure-of-python-project_21.html)，他对使用bin/目录的规定仍然很重要。要记住的最重要的一点是，您的可执行文件不应该有很多代码，只需要导入和调用运行程序脚本中的[main函数](https://realpython.com/python-main-function/)。如果您使用的是纯 Python 或没有任何可执行文件，则可以省略此目录。
 - `/docs`：对于更高级的应用程序，您需要维护其所有部分的良好文档。我喜欢将内部模块的任何文档放在这里，这就是为什么您会看到hello和world包的单独文档。如果您在内部模块中使用[文档字符串](https://realpython.com/documenting-python-code/#documenting-your-python-code-base-using-docstrings)（并且您应该这样做！），您的整个模块文档至少应该对模块的目的和功能提供一个整体视图。
 - `helloworld/`:这和以前的结构类似helloworld/，但现在有子目录。随着您增加更多复杂性，您将需要使用“分而治之”的策略，并将应用程序逻辑的部分拆分成更易于管理的块。请记住，目录名称是指整个包名称，因此子目录名称（hello/和world/）应该反映它们的包名称。
 - `data/`:有这个目录对测试很有帮助。它是您的应用程序将摄取或生成的任何文件的中心位置。根据您部署应用程序的方式，您可以将“生产级”输入和输出指向此目录，或者仅将其用于内部测试。
 - `tests/`：在这里，您可以放置​​所有测试——单元测试、执行测试、集成测试等等。随意以最方便的方式为您的测试策略、导入策略等构建此目录。有关使用Python 测试命令行应用程序的复习，请查看[我的文章4 测试 Python 命令行 (CLI) 应用程序的技术。](https://realpython.com/python-cli-testing/)

顶级文件与之前的布局基本相同。这三种布局应该涵盖命令行应用程序的大多数用例，甚至是 GUI 应用程序，但需要注意的是，您可能需要根据所使用的 GUI 框架修改一些东西。

> 注意：请记住，这些只是布局。如果目录或文件对您的特定用例没有意义（例如tests/，如果您不使用代码分发测试），请随时将其排除在外。但尽量不要遗漏docs/。记录您的工作总是一个好主意。


## 2. Web 应用程序布局
Python 的另一个主要用例是[Web 应用程序](https://realpython.com/python-web-applications/)。[Django](https://www.djangoproject.com/)和[Flask](https://flask.palletsprojects.com/en/2.0.x/)可以说是 Python 最流行的 Web 框架，幸运的是，在应用程序布局方面它们更加固执己见。

为了确保这篇文章是一个完整的、成熟的布局参考，我想强调这些框架共有的结构。

###  2.1 Django
让我们按字母顺序，从 Django 开始。Django 的优点之一是它会在运行后为您创建一个项目骨架`django-admin startproject project`，其中project是您的项目名称。这将在您当前的工作目录中创建一个名为project以下内​​部结构的目录：

```python
project/
│
├── project/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
└── manage.py
```
这似乎有点空，不是吗？所有的逻辑都到哪里去了？观点？连测试都没有！

在 Django 中，这是一个项目，它将 Django 的另一个概念、应用程序联系在一起。应用程序是逻辑、模型、视图等全部存在的地方，并且在此过程中它们执行某些任务，例如维护博客。

Django 应用程序可以导入项目并跨项目使用，其结构类似于专门的 Python 包。

与项目一样，Django 使生成 Django 应用程序布局变得非常容易。设置项目后，您所要做的就是导航到manage.py并运行的位置`python manage.py startapp app`，这app是您的应用程序的名称。

这将产生一个app使用以下布局调用的目录：

```python
app/
│
├── migrations/
│   └── __init__.py
│
├── __init__.py
├── admin.py
├── apps.py
├── models.py
├── tests.py
└── views.py
```
然后可以将其直接导入到您的项目中。有关这些文件的作用、如何将它们用于您的项目等的详细信息超出了本参考的范围，但您可以在我们的 Django 教程[添加链接描述](https://realpython.com/django-setup/)和[官方 Django 文档](https://docs.djangoproject.com/en/2.0/intro/tutorial01/)中获得所有这些信息和更多信息。

这种文件和文件夹结构非常简单，是 Django 的基本要求。对于任何开源 Django 项目，您可以（并且应该）从命令行应用程序布局调整结构。我通常在外部project/目录中得到类似这样的东西：

```python
project/
│
├── app/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   │
│   ├── migrations/
│   │   └── __init__.py
│   │
│   ├── models.py
│   ├── tests.py
│   └── views.py
│
├── docs/
│
├── project/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── static/
│   └── style.css
│
├── templates/
│   └── base.html
│
├── .gitignore
├── manage.py
├── LICENSE
└── README.md
```
要深入讨论更高级的 Django 应用程序布局，请[参阅 Stack Overflow 线程](https://stackoverflow.com/questions/22841764/best-practice-for-django-project-working-directory-structure)。[django-project-skeleton项目文档](https://django-project-skeleton.readthedocs.io/en/latest/structure.html)解释了您将在 Stack Overflow 线程中找到的一些目录。可以在 Django 的两个独家新闻的页面中找到对 Django 的全面深入了解，该页面将向您介绍 Django 开发的所有最新最佳实践。

如需更多 Django 教程，请[访问 Real Python 中的 Django 部分](https://realpython.com/tutorials/django/)。

###  2.2 Flask
Flask 是一个 Python 网络“微框架”。主要卖点之一是它可以非常快速地以最小的开销进行设置。Flask 文档有一个不到 10 行代码和单个脚本的 Web 应用程序示例。当然，在实践中，您不太可能编写这么小的 Web 应用程序。

幸运的是[，Flask 文档](http://flask.pocoo.org/docs/1.0/)突然出现，为他们的教程项目（一个名为 Flaskr 的博客 Web 应用程序）提供了建议的布局，我们将在主项目目录中检查它：

```python
flaskr/
│
├── flaskr/
│   ├── ___init__.py
│   ├── db.py
│   ├── schema.sql
│   ├── auth.py
│   ├── blog.py
│   ├── templates/
│   │   ├── base.html
│   │   ├── auth/
│   │   │   ├── login.html
│   │   │   └── register.html
│   │   │
│   │   └── blog/
│   │       ├── create.html
│   │       ├── index.html
│   │       └── update.html
│   │ 
│   └── static/
│       └── style.css
│
├── tests/
│   ├── conftest.py
│   ├── data.sql
│   ├── test_factory.py
│   ├── test_db.py
│   ├── test_auth.py
│   └── test_blog.py
│
├── venv/
│
├── .gitignore
├── setup.py
└── MANIFEST.in
```
flaskr在这个布局中，除了你的测试、[虚拟环境](https://realpython.com/python-virtual-environments-a-primer/)的目录和通常的顶级文件之外，所有东西都在包中。与其他布局一样，您的测试将大致匹配位于flaskr包中的各个模块。您的模板也驻留在主项目包中，这在 Django 布局中不会发生。

请务必访问我们的[Flask Boilerplate Githu](https://github.com/realpython/flask-boilerplate)b 页面以查看更完整的 Flask 应用程序，并在此处查看样板文件。

有关 Flask 的更多信息，请在此[处查看我们所有的 Flask 教程](https://realpython.com/tutorials/flask/)。


参考：

 - [Python Application Layouts: A Reference](https://realpython.com/python-application-layouts/#application-with-internal-packages)

