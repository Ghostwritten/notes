#  python 模块 zipfile压缩文件管理



----
##  1. 简介
`Python zipfile`是一个用于操作ZIP 文件的标准库模块。在归档和压缩数字数据时，这种文件格式是一种广泛采用的行业标准。您可以使用它将几个相关文件打包在一起。它还允许您减小文件大小并节省磁盘空间。最重要的是，它促进了计算机网络上的数据交换。

zipfile作为 Python 开发人员或 DevOps 工程师，了解如何使用该模块创建、读取、写入、填充、提取和列出 ZIP 文件是一项有用的技能。

在本教程中，您将学习如何：

 - 使用 Python 从 ZIP 文件中读取、写入和提取文件zipfile
 - 读取有关 ZIP 文件内容的元数据，使用zipfile
 - 用于操作现有 ZIP 文件中的zipfile成员文件
 - 创建新的 ZIP 文件以存档和压缩文件

如果您经常处理 ZIP 文件，那么这些知识可以帮助您简化工作流程，从而自信地处理文件。

要充分利用本教程，您应该了解[使用文件](https://realpython.com/working-with-files-in-python/)、使用[with语句](https://realpython.com/python-with-statement/)、使用 处理文件系统路径pathlib以及使用类和面向对象编程的基础知识。

##  2. 什么是 ZIP 文件？
ZIP 文件是当今数字世界中广为人知的流行工具。这些文件相当流行并广泛用于计算机网络（尤其是 Internet）上的跨平台数据交换。

您可以使用 ZIP 文件将常规文件捆绑到一个存档中，压缩数据以节省一些磁盘空间，分发您的数字产品等等。在本教程中，您将学习如何使用 Python 的zipfile模块操作 ZIP 文件。

您可能已经遇到并使用过 ZIP 文件。是的，.zip文件扩展名无处不在！ZIP 文件，也称为ZIP 档案，是使用ZIP 文件格式的文件。

[PKWARE](https://en.wikipedia.org/wiki/PKWARE,_Inc.)是创建并首先实施此文件格式的公司。该公司整理并维护当前的[格式规范](https://pkware.cachefly.net/webdocs/casestudies/APPNOTE.TXT)，该规范是公开的，允许创建使用 ZIP 文件格式读写文件的产品、程序和流程。

ZIP 文件格式是一种跨平台、可互操作的文件存储和传输格式。它结合了[无损数据压缩](https://en.wikipedia.org/wiki/Lossless_compression)、文件管理和数据[加密](https://en.wikipedia.org/wiki/Encryption)。

数据压缩不是将存档视为 ZIP 文件的必要条件。因此，您可以在 ZIP 存档中包含压缩或未压缩的成员文件。ZIP 文件格式支持多种压缩算法，但[Deflate](https://en.wikipedia.org/wiki/Deflate)是最常见的。该格式还支持使用[CRC32](https://en.wikipedia.org/wiki/Cyclic_redundancy_check)进行信息完整性检查。

尽管还有其他类似的存档格式，例如[RAR](https://en.wikipedia.org/wiki/RAR_%28file_format%29)和[TAR](https://en.wikipedia.org/wiki/Tar_%28computing%29#File_format)文件，但 ZIP 文件格式已迅速成为高效数据存储和计算机网络上数据交换的通用标准。

ZIP 文件无处不在。例如，[Microsoft Office](https://en.wikipedia.org/wiki/Microsoft_Office)和[Libre Office](https://en.wikipedia.org/wiki/LibreOffice)等办公套件依赖 ZIP 文件格式作为其文[档容器文件](https://www.iso.org/standard/60101.html)。这意味着 , .docx, .xlsx, .pptx, .odt,.ods文件.odp实际上是 ZIP 档案，其中包含构成每个文档的多个文件和文件夹。其他使用 ZIP 格式的常见文件包括[.jar](https://en.wikipedia.org/wiki/JAR_%28file_format%29)、[.war](https://en.wikipedia.org/wiki/WAR_%28file_format%29)和[.epub](https://en.wikipedia.org/wiki/EPUB)文件。

您可能熟悉GitHub ，它为使用Git的软件开发和版本控制提供 Web 托管。当您将软件项目下载到本地计算机时，GitHub 使用 ZIP 文件打包软件项目。例如，您可以在 ZIP 文件中下载Python Basics: A [Practical Introduction to Python 3](https://realpython.com/products/python-basics-book/)书籍的练习解决方案，或者您可以下载您选择的任何其他项目。

ZIP 文件允许您将文件聚合、压缩和加密到单个可互操作且可移植的容器中。您可以流式传输 ZIP 文件、将它们分割成段、使它们自解压等等。

##  3. 为什么使用 ZIP 文件？
了解如何创建、读取、写入和提取 ZIP 文件对于使用计算机和数字信息的开发人员和专业人员来说可能是一项有用的技能。除其他好处外，ZIP 文件还允许您：

 - 在不丢失信息的情况下减小文件大小及其存储要求
 - 由于减小了大小和单文件传输，提高了网络传输速度
 - 将多个相关文件打包到一个存档中，以实现高效管理
 - 将您的代码捆绑到一个存档中以进行分发
 - 使用加密来保护您的数据，这是当今的普遍要求
 - 保证您信息的完整性，避免对您的数据进行意外和恶意更改

如果您正在寻找一种灵活、可移植且可靠的方式来归档您的数字文件，这些功能使 ZIP 文件成为 Python 工具箱的有用补充。

##  4. Python 可以操作 ZIP 文件吗？
是的！Python 有几个工具可以让你操作 ZIP 文件。其中一些工具在 [Python标准库](https://docs.python.org/3/library/index.html)中可用。[它们包括](https://realpython.com/python-zipfile/#using-other-libraries-to-manage-zip-files)[zlib](https://docs.python.org/3/library/zlib.html#module-zlib)用于使用特定压缩算法（例如、[bz2](https://docs.python.org/3/library/bz2.html#module-bz2)、[lzma](https://docs.python.org/3/library/lzma.html#module-lzma)等）压缩和解压缩数据的低级库。

Python 还提供了一个高级模块，称为zipfile专门用于创建、读取、写入、提取和列出 ZIP 文件的内容。在本教程中，您将了解 Pythonzipfile以及如何有效地使用它。


##  5. 使用 Python 操作现有的 ZIP 文件zipfile
Pythonzipfile提供了方便的类和函数，允许您创建、读取、写入、提取和列出 ZIP 文件的内容。以下是一些zipfile支持的附加功能：

 - 大于 4 GiB 的 ZIP 文件（[ZIP64 文件](https://en.wikipedia.org/wiki/ZIP_%28file_format%29#ZIP64)）
 - 数据解密
 - 多种压缩算法，例如 Deflate、[Bzip2](https://en.wikipedia.org/wiki/Bzip2)和[LZMA](https://en.wikipedia.org/wiki/Lempel%E2%80%93Ziv%E2%80%93Markov_chain_algorithm)
 - 使用 CRC32 进行信息完整性检查

请注意，它`zipfile`确实有一些限制。例如，当前的数据解密功能可能非常慢，因为它使用纯 Python 代码。该模块无法处理加密 ZIP 文件的创建。最后，也不支持使用多磁盘 ZIP 文件。尽管有这些限制，zipfile它仍然是一个伟大而有用的工具。继续阅读以探索其功能。

###  5.1 打开 ZIP 文件进行读写
在zipfile模块中，您会找到[ZipFile](https://docs.python.org/3/library/zipfile.html#zipfile.ZipFile)该类。这个类的工作方式很像 Python 的内置[open()](https://realpython.com/read-write-files-python/#opening-and-closing-a-file-in-python)函数，允许您使用不同的模式打开 ZIP 文件。读取模式 ( "r") 是默认值。您还可以使用写入 ( "w")、附加 ( "a") 和独占 ( "x") 模式。稍后您将了解更多有关这些内容的信息。

ZipFile实现**上下文管理器协议**，以便您可以在[with语句](https://realpython.com/read-write-files-python/#opening-and-closing-a-file-in-python)中使用该类。此功能允许您快速打开和使用 ZIP 文件，而无需担心在完成工作后关闭文件。

在编写任何代码之前，请确保您拥有将要使用的文件和存档的副本：
为了准备好您的工作环境，请将下载的资源放入`python-zipfile/`主文件夹中的一个目录中。将文件放在正确的位置后，移动到新创建的目录并在那里启动 Python 交互式会话。

要热身，首先要阅读名为 `.zip` 的 ZIP 文件`sample.zip`。为此，您可以ZipFile在阅读模式下使用：

```python
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     archive.printdir()
...
File Name                                        Modified             Size
hello.txt                                 2021-09-07 19:50:10           83
lorem.md                                  2021-09-07 19:50:10         2609
realpython.md                             2021-09-07 19:50:10          428
```
初始化程序的第一个参数ZipFile可以是一个字符串，表示您需要打开的 ZIP 文件的路径。这个参数也可以接受类似文件和类似路径的对象。在此示例中，您使用基于字符串的路径。

第二个参数ZipFile是一个单字母字符串，表示您将用于打开文件的模式。正如您在本节开头所了解的，ZipFile可以接受四种可能的模式，具体取决于您的需要。mode [位置参数](https://realpython.com/defining-your-own-python-function/#positional-arguments)默认为，因此"r"如果您想以只读方式打开存档，则可以去掉它。



在声明中with，您调用[.printdir()](https://docs.python.org/3/library/zipfile.html#zipfile.ZipFile.printdir). archive该archive [变量](https://realpython.com/python-variables/)现在保存其自身的实例ZipFile。此功能提供了一种在屏幕上显示底层 ZIP 文件内容的快捷方式。该函数的输出具有用户友好的表格格式，包含三个信息列：

 - File Name
 - Modified
 - Size

如果你想在尝试打开它之前确保你的目标是一个有效的 ZIP 文件，那么你可以ZipFile在一个[try...except](https://realpython.com/python-exceptions/#the-try-and-except-block-handling-exceptions)语句中包装并捕获任何[BadZipFile](https://docs.python.org/3/library/zipfile.html#zipfile.BadZipFile)异常：

```python
>>> import zipfile

>>> try:
...     with zipfile.ZipFile("sample.zip") as archive:
...         archive.printdir()
... except zipfile.BadZipFile as error:
...     print(error)
...
File Name                                        Modified             Size
hello.txt                                 2021-09-07 19:50:10           83
lorem.md                                  2021-09-07 19:50:10         2609
realpython.md                             2021-09-07 19:50:10          428

>>> try:
...     with zipfile.ZipFile("bad_sample.zip") as archive:
...         archive.printdir()
... except zipfile.BadZipFile as error:
...     print(error)
...
File is not a zip file
```
第一个示例成功打开`sample.zip`而没有引发`BadZipFile`异常。那是因为`sample.zip`具有有效的 ZIP 格式。另一方面，第二个示例没有成功打开`bad_sample.zip`，因为该文件不是有效的 ZIP 文件。

要检查有效的 ZIP 文件，您还可以使用以下`is_zipfile()`功能：

```python
>>> import zipfile

>>> if zipfile.is_zipfile("sample.zip"):
...     with zipfile.ZipFile("sample.zip", "r") as archive:
...         archive.printdir()
... else:
...     print("File is not a zip file")
...
File Name                                        Modified             Size
hello.txt                                 2021-09-07 19:50:10           83
lorem.md                                  2021-09-07 19:50:10         2609
realpython.md                             2021-09-07 19:50:10          428

>>> if zipfile.is_zipfile("bad_sample.zip"):
...     with zipfile.ZipFile("bad_sample.zip", "r") as archive:
...         archive.printdir()
... else:
...     print("File is not a zip file")
...
File is not a zip file
```
在这些示例中，您使用[条件语句](https://realpython.com/python-conditional-statements/)withis_zipfile()作为条件。此函数接受一个filename参数，该参数保存文件系统中 ZIP 文件的路径。此参数可以接受字符串、类似文件或类似路径的对象。True如果filename是有效的 ZIP 文件，该函数将返回。否则，它返回False.

现在假设您要`hello.txt`使用. 为此，您可以使用写入模式 ( )。此模式打开一个 ZIP 文件进行写入。如果目标 ZIP 文件存在，则该模式会截断它并写入您传入的任何新内容。hello.zipZipFile"w""w"

> 注意：如果您使用ZipFile现有文件，那么您应该小心使用该"w"模式。您可以截断 ZIP 文件并丢失所有原始内容。

如果目标 ZIP 文件不存在，则ZipFile在您关闭存档时为您创建它：

```python
>>> import zipfile

>>> with zipfile.ZipFile("hello.zip", mode="w") as archive:
...     archive.write("hello.txt")
...
>>> with  zipfile.ZipFile("hello.zip", mode="r") as archive:
...      archive.printdir()
... 
File Name                                             Modified             Size
hello.txt                                      2022-02-18 15:27:40           83

```
运行此代码后，您的目录中将有一个`hello.zip`文件`python-zipfile/`。如果您使用 列出文件内容`.printdir()`，那么您会注意到它`hello.txt`会在那里。`.write()`在此示例中，您调用ZipFile对象。此方法允许您将成员文件写入 ZIP 存档。请注意，参数 `to.write()`应该是现有文件。

> 注意： ZipFile当您在写作模式下使用课程并且目标存档不存在时，它足够聪明，可以创建新存档。但是，如果这些目录尚不存在，则该类不会在目标ZIP 文件的路径中创建新目录。

这就解释了为什么以下代码不起作用：

```python
>>> import zipfile

>>> with zipfile.ZipFile("missing/hello.zip", mode="w") as archive:
...     archive.write("hello.txt")
...
Traceback (most recent call last):
    ...
FileNotFoundError: [Errno 2] No such file or directory: 'missing/hello.zip'
```
因为missing/目标文件路径中的目录hello.zip不存在，所以会出现[FileNotFoundError](https://docs.python.org/3/library/exceptions.html#FileNotFoundError)异常

附加模式 ( "a") 允许您将新成员文件附加到现有 ZIP 文件。此模式不会截断存档，因此其原始内容是安全的。如果目标 ZIP 文件不存在，则该"a"模式会为您创建一个新文件，然后将您作为参数传递给`.write()`.

要尝试该"a"模式，请继续将`new_hello.txt`文件添加到新创建的hello.zip存档中：

```python
>>> import zipfile

>>> with zipfile.ZipFile("hello.zip", mode="a") as archive:
...     archive.write("new_hello.txt")
...

>>> with zipfile.ZipFile("hello.zip") as archive:
...     archive.printdir()
...
File Name                                        Modified             Size
hello.txt                                 2021-09-07 19:50:10           83
new_hello.txt                             2021-08-31 17:13:44           13
```
在这里，您使用附加模式添加n`ew_hello.txt`到`hello.zip`文件中。然后运行`.printdir()`以确认 ZIP 文件中存在新文件。

ZipFile还支持独占模式 ( "x")。此模式允许您专门创建新的 ZIP 文件并将新的成员文件写入其中。当您想要制作一个新的 ZIP 文件而不覆盖现有文件时，您将使用独占模式。如果目标文件已经存在，那么你会得到[FileExistsError](https://docs.python.org/3/library/exceptions.html#FileExistsError).

"w"最后，如果您使用、"a"或模式创建 ZIP 文件"x"，然后在不添加任何成员文件的情况下关闭存档，则会ZipFile创建一个具有适当 ZIP 格式的空存档。

###  5.2 从 ZIP 文件中读取元数据
你已经付诸`.printdir()`行动了。这是一种有用的方法，可用于快速列出 ZIP 文件的内容。除了.printdir()，ZipFile该类还提供了几种从现有 ZIP 文件中提取元数据的便捷方法。

以下是这些方法的摘要：
|方法|	描述|
|--|--|
|[.getinfo(filename)](https://docs.python.org/3/library/zipfile.html#zipfile.ZipFile.getinfo)|返回一个[ZipInfo](https://docs.python.org/3/library/zipfile.html#zipfile.ZipInfo)对象，其中包含由 提供的成员文件的信息filename。请注意，filename必须在底层 ZIP 文件中保存目标文件的路径。|
|[.infolist()](https://docs.python.org/3/library/zipfile.html#zipfile.ZipFile.infolist)|返回对象[列表](https://realpython.com/python-lists-tuples/)，[ZipInfo](https://docs.python.org/3/library/zipfile.html#zipfile.ZipInfo)每个成员文件一个。|
|[.namelist()](https://docs.python.org/3/library/zipfile.html#zipfile.ZipFile.namelist)|返回一个包含基础存档中所有成员文件名称的列表。此列表中的名称是 的有效参数.getinfo()。|

使用这三个工具，您可以检索有关 ZIP 文件内容的大量有用信息。例如，看看下面的例子，它使用`.getinfo()`：

```python

>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     info = archive.getinfo("hello.txt")
...

>>> info.file_size
83

>>> info.compress_size
83

>>> info.filename
'hello.txt'

>>> info.date_time
(2021, 9, 7, 19, 50, 10)
```
正如您在上表中了解到的，`.getinfo()`将成员文件作为参数并返回一个`ZipInfo`包含有关它的信息的对象。
> 注意：ZipInfo不打算直接实例化。`.getinfo()`and方法在您调用它们时会自动`.infolist()`返回对象。ZipInfo但是，ZipInfo它包含一个名为的类方法`.from_file()`，它允许您在需要时显式地实例化该类。

`ZipInfo`对象具有多个属性，可让您检索有关目标成员文件的有价值信息。例如，`.file_size`分别`.compress_size`保存原始文件和压缩文件的大小（以字节为单位）。该类还有一些其他有用的属性，例如`.filenameand .date_time`，它们返回文件名和最后修改日期。

> 注意：默认情况下，ZipFile不会压缩输入文件以将它们添加到最终存档中。这就是上例中大小和压缩大小相同的原因。您将在下面的压缩文件和目录部分了解有关此主题的更多信息。

使用`.infolist()`，您可以从给定存档中的所有文件中提取信息。这是一个使用此方法生成包含`sample.zip`存档中所有成员文件信息的最小报告的示例：

```python

>>> import datetime
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     for info in archive.infolist():
...         print(f"Filename: {info.filename}")
...         print(f"Modified: {datetime.datetime(*info.date_time)}")
...         print(f"Normal size: {info.file_size} bytes")
...         print(f"Compressed size: {info.compress_size} bytes")
...         print("-" * 20)
...
Filename: hello.txt
Modified: 2021-09-07 19:50:10
Normal size: 83 bytes
Compressed size: 83 bytes
--------------------
Filename: lorem.md
Modified: 2021-09-07 19:50:10
Normal size: 2609 bytes
Compressed size: 2609 bytes
--------------------
Filename: realpython.md
Modified: 2021-09-07 19:50:10
Normal size: 428 bytes
Compressed size: 428 bytes
--------------------
```
该[for循环](https://realpython.com/python-for-loop/)遍历来自 的ZipInfo对象.infolist()，检索文件名、最后修改日期、正常大小和每个成员文件的压缩大小。在此示例中，您习惯于[datetime](https://realpython.com/python-datetime/)以人类可读的方式格式化日期。
> 
> 注意：上面的示例改编自zipfile — [ZIP Archive Access](https://pymotw.com/3/zipfile/)。

如果您只需要对 ZIP 文件执行快速检查并列出其成员文件的名称，则可以使用`.namelist()`：

```python
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     for filename in archive.namelist():
...         print(filename)
...
hello.txt
lorem.md
realpython.md
```
由于此输出中的文件名是 的有效参数`.getinfo()`，因此您可以结合使用这两种方法来仅检索有关选定成员文件的信息。

例如，您可能有一个 ZIP 文件，其中包含不同类型的成员文件（.docx、.xlsx、.txt等）。无需使用 获取完整信息`.infolist()`，您只需获取有关`.docx`文件的信息。然后，您可以按扩展名过滤文件并仅调用`.getinfo()`您的.docx文件。来吧，试一试！

###  5.3 读取和写入成员文件
有时您有一个 ZIP 文件并且需要读取给定成员文件的内容而不解压缩它。为此，您可以使用[.read()](https://docs.python.org/3/library/zipfile.html#zipfile.ZipFile.read). 此方法采用成员文件name并将该文件的内容作为字节返回：

```python
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     for line in archive.read("hello.txt").split(b"\n"):
...         print(line)
...
b'Hello, Pythonista!'
b''
b'Welcome to Real Python!'
b''
b"Ready to try Python's zipfile module?"
b''
```
要使用`.read()`，您需要打开 ZIP 文件以进行读取或附加。请注意，.read()将目标文件的内容作为字节流返回。在此示例中，您使用换行符作为分隔符将`.split()`流拆分为行。因为是对字节对象进行操作，所以需要在用作参数的字符串中添加前导。"\n".split()b

`ZipFile.read()`还接受名为 `.` 的第二个位置参数pwd。此参数允许您提供用于读取加密文件的密码。要尝试此功能，您可以依赖`sample_pwd.zip`与本教程材料一起下载的文件：

```python

>>> import zipfile

>>> with zipfile.ZipFile("sample_pwd.zip", mode="r") as archive:
...     for line in archive.read("hello.txt", pwd=b"secret").split(b"\n"):
...         print(line)
...
b'Hello, Pythonista!'
b''
b'Welcome to Real Python!'
b''
b"Ready to try Python's zipfile module?"
b''

>>> with zipfile.ZipFile("sample_pwd.zip", mode="r") as archive:
...     for line in archive.read("hello.txt").split(b"\n"):
...         print(line)
...
Traceback (most recent call last):
    ...
RuntimeError: File 'hello.txt' is encrypted, password required for extraction
```
在第一个示例中，您提供了`secret`读取加密文件的密码。该pwd参数接受字节类型的值。如果您`.read()`在未提供所需密码的情况下使用加密文件，那么您会得到一个`RuntimeError`，正如您在第二个示例中所指出的那样。
> 
> 注意： Pythonzipfile支持解密。但是，它不支持创建加密的 ZIP 文件。这就是为什么您需要使用外部文件归档器来加密文件的原因。
> 
> 一些流行的文件归档器包括适用于 Windows的[7z](https://en.wikipedia.org/wiki/7-Zip)和[WinRAR](https://en.wikipedia.org/wiki/WinRAR) 、适用于 Linux 的[Ark](https://en.wikipedia.org/wiki/Ark_%28software%29)和[GNOME Archive
> Manager以及适用于 macOS](https://en.wikipedia.org/wiki/GNOME_Archive_Manager) 的[Archiver](https://archiverapp.com/)。

对于大型加密 ZIP 文件，请记住，解密操作可能非常缓慢，因为它是在纯 Python 中实现的。在这种情况下，请考虑使用专门的程序来处理您的档案，而不是使用zipfile.

如果您经常使用加密文件，那么您可能希望避免在每次调用.read()或其他接受pwd参数的方法时提供解密密码。如果是这种情况，您可以使用`ZipFile.setpassword()`设置全局密码：

```python
>>> import zipfile

>>> with zipfile.ZipFile("sample_pwd.zip", mode="r") as archive:
...     archive.setpassword(b"secret")
...     for file in archive.namelist():
...         print(file)
...         print("-" * 20)
...         for line in archive.read(file).split(b"\n"):
...             print(line)
...
hello.txt
--------------------
b'Hello, Pythonista!'
b''
b'Welcome to Real Python!'
b''
b"Ready to try Python's zipfile module?"
b''
lorem.md
--------------------
b'# Lorem Ipsum'
b''
b'Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    ...
```
使用`.setpassword()`，您只需提供一次密码。`ZipFile`使用该唯一密码解密所有成员文件。

相反，如果您的 ZIP 文件对各个成员文件具有不同的密码，那么您需要使用以下pwd参数为每个文件提供特定密码`.read()`：

```python

>>> import zipfile

>>> with zipfile.ZipFile("sample_file_pwd.zip", mode="r") as archive:
...     for line in archive.read("hello.txt", pwd=b"secret1").split(b"\n"):
...         print(line)
...
b'Hello, Pythonista!'
b''
b'Welcome to Real Python!'
b''
b"Ready to try Python's zipfile module?"
b''

>>> with zipfile.ZipFile("sample_file_pwd.zip", mode="r") as archive:
...     for line in archive.read("lorem.md", pwd=b"secret2").split(b"\n"):
...         print(line)
...
b'# Lorem Ipsum'
b''
b'Lorem ipsum dolor sit amet, consectetur adipiscing elit.
    ...
```
在此示例中，您使用`read`和`to readsecret1`作为密码。要考虑的最后一个细节是，当您使用该参数时，您将覆盖您可能使用`.hello.txtsecret2lorem.mdpwd.setpassword()`
> 
> 注意：调用`.read()`使用不受支持的压缩方法的 ZIP 文件会引发[NotImplementedError](https://docs.python.org/3/library/exceptions.html#NotImplementedError). 如果所需的压缩模块在您的Python 安装中不可用，您也会收到错误消息。

如果您正在寻找一种更灵活的方式来读取成员文件并创建新成员文件并将其添加到存档中，那么`ZipFile.open()`它适合您。与内置`open()`函数一样，此方法实现了上下文管理器协议，因此它支持以下with语句：

```python
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     with archive.open("hello.txt", mode="r") as hello:
...         for line in hello:
...             print(line)
...
b'Hello, Pythonista!\n'
b'\n'
b'Welcome to Real Python!\n'
b'\n'
b"Ready to try Python's zipfile module?\n"
```
在本例中，您打开hello.txt阅读。的第一个参数.open()是name，表示要打开的成员文件。第二个参数是模式，默认"r"为正常模式。`ZipFile.open()`还接受pwd打开加密文件的参数。此参数的作用与 中的等效pwd参数相同.read()。

您也可以使用.open()与"w"模式。此模式允许您创建一个新的成员文件，向其中写入内容，最后将该文件附加到底层存档，您应该以附加模式打开它：

```python

>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="a") as archive:
...     with archive.open("new_hello.txt", "w") as new_hello:
...         new_hello.write(b"Hello, World!")
...
13

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     archive.printdir()
...     print("------")
...     archive.read("new_hello.txt")
...
File Name                                        Modified             Size
hello.txt                                 2021-09-07 19:50:10           83
lorem.md                                  2021-09-07 19:50:10         2609
realpython.md                             2021-09-07 19:50:10          428
new_hello.txt                             1980-01-01 00:00:00           13
------
b'Hello, World!'
```
在第一个代码片段中，您`sample.zip`以附加模式 ( "a") 打开。然后你`new_hello.txt`通过调用模式`.open()`来创建。"w"该函数返回一个支持的类文件对象`.write()`，它允许您将字节写入新创建的文件。
> 
> 注意：您需要提供一个不存在的文件名到`.open()`,如果您使用基础存档中已经存在的文件名，那么您最终会得到一个重复的文件和一个[UserWarning](https://docs.python.org/3.9/library/exceptions.html#UserWarning)异常。

在此示例中，您b'Hello, World!'写入`new_hello.txt`. 当执行流程退出内部`with`语句时，Python 将输入字节写入成员文件。当外部with语句退出时，Python 将写入new_hello.txt底层 ZIP 文件sample.zip.

第二个代码片段确认它new_hello.txt现在是sample.zip. 在此示例的输出中要注意的一个细节是将新添加文件.write()的日期设置为，这是使用此方法时应牢记的奇怪行为。`Modified1980-01-01 00:00:00`

###  5.4 将成员文件的内容作为文本读取
正如您在上一节中所了解的，您可以使用`.read()`和`.write()`方法来读取和写入成员文件，而无需从包含的 ZIP 存档中提取它们。这两种方法都只适用于字节。

但是，当您有一个包含文本文件的 ZIP 存档时，您可能希望将其内容作为文本而不是字节来读取。至少有两种方法可以做到这一点。您可以使用：

 1. [bytes.decode()](https://docs.python.org/3/library/stdtypes.html#bytes.decode)
 2. [io.TextIOWrapper](https://docs.python.org/3/library/io.html#io.TextIOWrapper)

因为`ZipFile.read()`将目标成员文件的内容作为字节返回，`.decode()`所以可以直接对这些字节进行操作。该方法使用给定的字符编码格式将对象`.decode()`解码为字符串。bytes

以下是您可以用来从存档文件中`.decode()`读取文本的方法：`hello.txtsample.zip`

```python
>>> import zipfile

>>>  with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     text = archive.read("hello.txt").decode(encoding="utf-8")
...

>>> print(text)
Hello, Pythonista!

Welcome to Real Python!

Ready to try Python's zipfile module?
```
在此示例中，您读取`hello.txtas` 字节的内容。然后调用.decode()将字节解码为使用[UTF-8](https://en.wikipedia.org/wiki/UTF-8)作为[编码](https://realpython.com/python-encodings-guide/)的字符串。要设置`encoding`参数，请使用"utf-8"字符串。但是，您可以使用任何其他有效编码，例如UTF-16或cp1252，它们可以表示为不区分大小写的字符串。请注意，这"utf-8"是 的encoding参数的默认值.decode()。

请务必记住，您需要事先知道要使用.decode(). 如果您使用了错误的字符编码，那么您的代码将无法将底层字节正确解码为文本，最终您可能会得到大量无法辨认的字符。

从成员文件中读取文本的第二个选项是使用`io.TextIOWrapper`提供缓冲文本流的对象。这个时候你需要使用`.open()`而不是`.read()`. 下面是一个将成员文件的内容作为文本流io.TextIOWrapper读取的示例：hello.txt

```python
>>> import io
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     with archive.open("hello.txt", mode="r") as hello:
...         for line in io.TextIOWrapper(hello, encoding="utf-8"):
...             print(line.strip())
...
Hello, Pythonista!

Welcome to Real Python!

Ready to try Python's zipfile module?
```
在此示例的内部with语句中，您从存档中打开hello.txt成员文件。`sample.zip`然后将生成的二进制文件类对象 ,hello作为参数传递给`io.TextIOWrapper. hello`这通过解码使用 `UTF-8` 字符编码格式的内容来创建缓冲文本流。因此，您可以直接从目标成员文件中获得文本流。

就像 with 一样`.encode()`，io.TextIOWrapper该类接受一个encoding参数。您应该始终为此参数指定一个值，因为默认文本编码取决于运行代码的系统，并且可能不是您尝试解码的文件的正确值。


###  5.5 从您的 ZIP 档案中提取成员文件
提取给定存档的内容是您将对 ZIP 文件执行的最常见操作之一。根据您的需要，您可能希望一次提取一个文件或一次提取所有文件。

`ZipFile.extract()`让你完成第一个任务。此方法采用`member`文件名并将其提取到由 . 表示的给定目录path。目标path默认为当前目录：

```python
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     archive.extract("new_hello.txt", path="output_dir/")
...
'output_dir/new_hello.txt'
```

现在`new_hello.txt`将在您的`output_dir/`目录中。如果目标文件名已经存在于输出目录中，则`.extract()`覆盖它而不要求确认。如果输出目录不存在，`.extract()`则为您创建它。请注意，.extract()返回提取文件的路径。

成员文件的名称必须是返回的文件全名`.namelist()`。它也可以是ZipInfo包含文件信息的对象。

您还可以使用`.extract()`加密文件。在这种情况下，您需要提供所需的pwd参数或使用 设置存档级密码`.setpassword()`。

在从档案中提取所有成员文件时，您可以使用.extractall(). 顾名思义，此方法将所有成员文件提取到目标路径，默认为当前目录：

```python
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     archive.extractall("output_dir/")
...
```
运行此代码后，所有当前内容都`sample.zip`将在您的`output_dir/`目录中。如果您将不存在的目录传递给`.extractall()`，则此方法会自动创建该目录。最后，如果目标目录中已经存在任何成员文件，`.extractall()`则将覆盖它们而不要求您确认，因此请小心。

如果您只需要从给定存档中提取一些成员文件，那么您可以使用该members参数。此参数接受成员文件列表，该列表应该是手头存档中整个文件列表的子集。最后，就像 一样`.extract()`，该`.extractall()`方法也接受一个pwd参数来提取加密文件。

###  5.6 使用后关闭 ZIP 文件
有时，您可以方便地打开给定的 ZIP 文件而不使用`with`语句。在这些情况下，您需要在使用后手动关闭存档以完成任何写入操作并释放获取的资源。

为此，您可以调用`.close()`您的`ZipFile`对象：

```python
>>> import zipfile

>>> archive = zipfile.ZipFile("sample.zip", mode="r")

>>> # Use archive in different parts of your code
>>> archive.printdir()
File Name                                        Modified             Size
hello.txt                                 2021-09-07 19:50:10           83
lorem.md                                  2021-09-07 19:50:10         2609
realpython.md                             2021-09-07 19:50:10          428
new_hello.txt                             1980-01-01 00:00:00           13

>>> # Close the archive when you're done
>>> archive.close()
>>> archive
<zipfile.ZipFile [closed]>
```
呼叫为您`.close()`关闭archive。您必须.close()在退出程序之前调用。否则，可能无法执行某些写入操作。例如，如果您打开一个 ZIP 文件以附加 ( "a") 新成员文件，那么您需要关闭存档以写入文件。

##  6. 创建、填充和提取您自己的 ZIP 文件
到目前为止，您已经学习了如何使用现有的 ZIP 文件。您已经学会了使用ZipFile. 您还学习了如何读取相关元数据以及如何提取给定 ZIP 文件的内容。

在本节中，您将编写一些实际示例，帮助您了解如何使用`zipfile`和其他 Python 工具从多个输入文件和整个目录创建 ZIP 文件。您还将学习如何使用zipfile文件压缩等。

### 6.1 从多个常规文件创建 ZIP 文件
有时您需要从多个相关文件创建 ZIP 存档。这样，您可以将所有文件放在一个容器中，以便通过计算机网络分发它们或与朋友或同事共享它们。为此，您可以创建一个目标文件列表，并使用ZipFile循环将它们写入存档：

```python
>> import zipfile

>>> filenames = ["hello.txt", "lorem.md", "realpython.md"]

>>> with zipfile.ZipFile("multiple_files.zip", mode="w") as archive:
...     for filename in filenames:
...         archive.write(filename)
...
```
在这里，您创建一个ZipFile对象，并将所需的存档名称作为其第一个参数。该"w"模式允许您将成员文件写入最终的 ZIP 文件。

该for循环遍历您的输入文件列表，并使用.write(). 一旦执行流程退出with语句，ZipFile自动关闭存档，为您保存更改。现在您有了一个`multiple_files.zip`存档，其中包含原始文件列表中的所有文件。


###  6.2 从目录构建 ZIP 文件
将目录的内容捆绑到单个存档中是 ZIP 文件的另一个日常用例。Python 有几个工具可以用来zipfile完成这项任务。例如，您可以使用[pathlib](https://realpython.com/python-pathlib/)来读取给定目录的内容。有了这些信息，您可以使用ZipFile.

在该`python-zipfile/`目录中，您有一个名为 的子目录`source_dir/`，其中包含以下内容：

```python
source_dir/
│
├── hello.txt
├── lorem.md
└── realpython.md
```
在`source_dir/`中，您只有三个常规文件。因为该目录不包含子目录，您可以使用`pathlib.Path.iterdir()`它直接迭代其内容。考虑到这个想法，以下是如何从以下内容构建 ZIP 文件`source_dir/`：

```python
>>> import pathlib
>>> import zipfile

>>> directory = pathlib.Path("source_dir/")

>>> with zipfile.ZipFile("directory.zip", mode="w") as archive:
...    for file_path in directory.iterdir():
...        archive.write(file_path, arcname=file_path.name)
...

>>> with zipfile.ZipFile("directory.zip", mode="r") as archive:
...     archive.printdir()
...
File Name                                        Modified             Size
realpython.md                             2021-09-07 19:50:10          428
hello.txt                                 2021-09-07 19:50:10           83
lorem.md                                  2021-09-07 19:50:10         2609
```
[pathlib.Path](https://docs.python.org/3/library/pathlib.html#pathlib.Path)在此示例中，您从源目录创建一个对象。第一条with语句创建一个ZipFile准备好写入的对象。然后调用`.iterdir()`返回一个遍历底层目录中条目的迭代器。

因为您在 中没有任何子目录`source_dir/`，所以该`.iterdir()`函数只生成文件。for循环遍历文件并将它们写入存档。

在这种情况下，您传递`file_path.name`给 的第二个参数`.write()`。此参数被调用`arcname`并保存结果存档中成员文件的名称。到目前为止，您看到的所有示例都依赖于 的默认值arcname，这与您作为第一个参数传递给.write().

如果您不传递`file_path.nameto arcname`，那么您的源目录将位于 ZIP 文件的根目录，这也可能是一个有效的结果，具体取决于您的需要。

现在检查root_dir/工作目录中的文件夹。在这种情况下，您会发现以下结构：

```python
root_dir/
│
├── sub_dir/
│   └── new_hello.txt
│
├── hello.txt
├── lorem.md
└── realpython.md
```
您有通常的文件和一个包含单个文件的子目录。如果要创建具有相同内部结构的 ZIP 文件，则需要一个工具来[递归](https://realpython.com/python-recursion/)遍历.root_dir/

以下是如何使用模块中的zipfilewith压缩完整的目录树，如上面的目录树：`Path.rglob()pathlib`

```python
>>> import pathlib
>>> import zipfile

>>> directory = pathlib.Path("root_dir/")

>>> with zipfile.ZipFile("directory_tree.zip", mode="w") as archive:
...     for file_path in directory.rglob("*"):
...         archive.write(
...             file_path,
...             arcname=file_path.relative_to(directory)
...         )
...

>>> with zipfile.ZipFile("directory_tree.zip", mode="r") as archive:
...     archive.printdir()
...
File Name                                        Modified             Size
sub_dir/                                  2021-09-09 20:52:14            0
realpython.md                             2021-09-07 19:50:10          428
hello.txt                                 2021-09-07 19:50:10           83
lorem.md                                  2021-09-07 19:50:10         2609
sub_dir/new_hello.txt                     2021-08-31 17:13:44           13
```
在本例中，您使用递归[Path.rglob()](https://docs.python.org/3/library/pathlib.html#pathlib.Path.rglob)遍历. 然后将每个文件和子目录写入目标 ZIP 存档。root_dir/

这一次，您使用[Path.relative_to()](https://docs.python.org/3/library/pathlib.html#pathlib.PurePath.is_relative_to)获取每个文件的相对路径，然后将结果传递给.write(). 这样，生成的 ZIP 文件最终具有与原始源目录相同的内部结构。同样，如果您希望源目录位于 ZIP 文件的根目录，则可以去掉这个参数。


##  6.3 压缩文件和目录
如果您的文件占用了太多磁盘空间，那么您可能会考虑压缩它们。Pythonzipfile支持一些流行的压缩方法。但是，该模块默认情况下不会压缩您的文件。如果你想让你的文件更小，那么你需要显式地为ZipFile.

通常，您将使用术语存储来指代写入 ZIP 文件而不进行压缩的成员文件。这就是为什么默认压缩方法ZipFile称为`ZIP_STORED`的原因，它实际上是指未压缩的成员文件，这些文件只是存储在包含的存档中。

该`compression`方法是 的初始化程序的第三个参数ZipFile。如果您想在将文件写入 ZIP 存档时压缩文件，则可以将此参数设置为以下常量之一：
|持续的|	压缩方法|	所需模块|
|--|--|--|
|zipfile.ZIP_DEFLATED|	放气|	zlib|
|zipfile.ZIP_BZIP2|	压缩包|	bz2|
|zipfile.ZIP_LZMA|	LZMA|	lzma|

这些是您当前可以使用的压缩方法ZipFile。一种不同的方法会引发[NotImplementedError](https://docs.python.org/3/library/exceptions.html#NotImplementedError). zipfile从 `Python 3.10`开始，没有其他可用的压缩方法。

作为附加要求，如果您选择其中一种方法，则支持它的压缩模块必须在您的 Python 安装中可用。否则，您将得到一个[RuntimeError](https://docs.python.org/3/library/exceptions.html#RuntimeError)异常，并且您的代码将中断。

关于压缩文件的另一个相关论点ZipFile是`compresslevel`. 此参数控制您使用的压缩级别。

使用 `Deflate` 方法，compresslevel可以从中获取[整数](https://realpython.com/python-numbers/)。使用 `Bzip2` 方法，您可以从 传递整数。在这两种情况下，当压缩级别增加时，您会获得更高的压缩率和更低的压缩速度。0919

> 注意：二进制文件，例如 PNG、JPG、MP3 等，已经使用了某种压缩方式。因此，将它们添加到 ZIP文件可能不会使数据变得更小，因为它已经压缩到一定程度。

现在假设您要使用 `Deflate` 方法归档和压缩给定目录的内容，这是 ZIP 文件中最常用的方法。为此，您可以运行以下代码：

```python

>>> import pathlib
>>> from zipfile import ZipFile, ZIP_DEFLATED

>>> directory = pathlib.Path("source_dir/")

>>> with ZipFile("comp_dir.zip", "w", ZIP_DEFLATED, compresslevel=9) as archive:
...     for file_path in directory.rglob("*"):
...         archive.write(file_path, arcname=file_path.relative_to(directory))
...
```
在此示例中，您传递9到`compresslevel`以获得最大压缩。要提供此参数，请使用[关键字参数](https://realpython.com/defining-your-own-python-function/#keyword-arguments)。您需要这样做，因为compresslevel它不是初始化程序的第四个[位置参数](https://realpython.com/defining-your-own-python-function/#positional-arguments)。ZipFile
> 
> 注意：初始化程序ZipFile采用第四个参数，称为allowZip64. 它是一个布尔参数，告诉为大于 4 GB
> 的文件ZipFile创建扩展名为 ZIP 文件。.zip64


###  6.4 按顺序创建 ZIP 文件
按顺序创建 ZIP 文件可能是您日常编程中的另一个常见要求。例如，您可能需要创建一个包含或不包含内容的初始 ZIP 文件，然后在新成员文件可用时立即追加它们。在这种情况下，您需要多次打开和关闭目标 ZIP 文件。

要解决此问题，您可以使用ZipFile附加模式 ( "a")，就像您已经完成的那样。此模式允许您安全地将新成员文件附加到 ZIP 存档而不截断其当前内容：

```python
>>> import zipfile

>>> def append_member(zip_file, member):
...     with zipfile.ZipFile(zip_file, mode="a") as archive:
...         archive.write(member)
...

>>> def get_file_from_stream():
...     """Simulate a stream of files."""
...     for file in ["hello.txt", "lorem.md", "realpython.md"]:
...         yield file
...

>>> for filename in get_file_from_stream():
...     append_member("incremental.zip", filename)
...

>>> with zipfile.ZipFile("incremental.zip", mode="r") as archive:
...     archive.printdir()
...
File Name                                        Modified             Size
hello.txt                                 2021-09-07 19:50:10           83
lorem.md                                  2021-09-07 19:50:10         2609
realpython.md                             2021-09-07 19:50:10          428
```
在此示例中，`append_member()`是一个将文件 ( ) 附加到输入 ZIP 存档 ( ) 的函数。要执行此操作，该函数会在您每次调用时打开和关闭目标存档。使用函数来执行此任务允许您根据需要多次重用代码。`memberzip_file`

该`get_file_from_stream()`函数是一个生成器函数，用于模拟要处理的文件流。同时，for循环依次将成员文件添加到`incremental.zipusing append_number()`. 如果您在运行此代码后检查您的工作目录，那么您将找到一个`incremental.zip`包含您传递到循环中的三个文件的存档。

###  6.5 提取文件和目录
您对 ZIP 文件执行的最常见操作之一是将其内容提取到文件系统中的给定目录。您已经学习了使用`.extract()`和`.extractall()`从存档中提取一个或所有文件的基础知识。

再举一个例子，回到你的`sample.zip`文件。此时，存档包含四个不同类型的文件。你有两个`.txt`文件和两个`.md`文件。现在假设您只想提取.md文件。为此，您可以运行以下代码：

```python
>>> import zipfile

>>> with zipfile.ZipFile("sample.zip", mode="r") as archive:
...     for file in archive.namelist():
...         if file.endswith(".md"):
...             archive.extract(file, "output_dir/")
...
'output_dir/lorem.md'
'output_dir/realpython.md'
```
该`with`声明打开sample.zip供阅读。循环使用 遍历存档中的每个文件namelist()，而条件语句检查文件名是否以扩展名结尾.md。如果是这样，那么您将手头的文件提取到目标目录`output_dir/`，使用`.extract()`.

##  7. 探索其他类zipfile
到目前为止，您已经了解了`ZipFile`和`ZipInfo`，它们是 中可用的两个类zipfile。该模块还提供了另外两个在某些情况下可以派上用场的类。这些类是`zipfile.Path`和`zipfile.PyZipFile`。在以下两节中，您将了解这些类的基础知识及其主要功能。

###  7.1 Path在 ZIP 文件中查找
当您使用您最喜欢的存档应用程序打开 ZIP 文件时，您会看到存档的内部结构。您可能在存档的根目录中有文件。您可能还有包含更多文件的子目录。存档看起来像文件系统上的普通目录，每个文件都位于特定路径。

该类zipfile.Path允许您构造路径对象以快速创建和管理给定 ZIP 文件中成员文件和目录的路径。该类有两个参数：

 - root接受一个 ZIP 文件，可以是ZipFile对象，也可以是物理 ZIP 文件的基于字符串的路径。
 - at保存存档中特定成员文件或目录的位置。它默认为空字符串，表示存档的根。


以你的老朋友`sample.zip`为目标，运行以下代码：

```python
>>> import zipfile

>>> hello_txt = zipfile.Path("sample.zip", "hello.txt")

>>> hello_txt
Path('sample.zip', 'hello.txt')

>>> hello_txt.name
'hello.txt'

>>> hello_txt.is_file()
True

>>> hello_txt.exists()
True

>>> print(hello_txt.read_text())
Hello, Pythonista!

Welcome to Real Python!

Ready to try Python's zipfile module?
```
此代码显示`zipfile.Path`实现了对象共有的几个功能`pathlib.Path`。您可以使用`.name`. 您可以检查路径是否指向带有`.is_file()`. 您可以检查给定文件是否存在于特定 ZIP 文件中，等等。

Path还提供了`.open()`一种使用不同模式打开成员文件的方法。例如，下面的代码打开hello.txt以供阅读：

```python
>>> import zipfile

>>> hello_txt = zipfile.Path("sample.zip", "hello.txt")

>>> with hello_txt.open(mode="r") as hello:
...     for line in hello:
...         print(line)
...
Hello, Pythonista!

Welcome to Real Python!

Ready to try Python's zipfile module?
```
使用Path. 您可以快速创建指向给定 ZIP 文件中特定成员文件的路径对象，并使用.open().

就像使用pathlib.Path对象一样，您可以通过调用对象来列出 ZIP 文件的.iterdir()内容zipfile.Path：

```python
>>> import zipfile

>>> root = zipfile.Path("sample.zip")
>>> root
Path('sample.zip', '')

>>> root.is_dir()
True

>>> list(root.iterdir())
[
    Path('sample.zip', 'hello.txt'),
    Path('sample.zip', 'lorem.md'),
    Path('sample.zip', 'realpython.md')
]
```
很明显，它zipfile.Path提供了许多有用的功能，您可以使用这些功能几乎立即管理 ZIP 存档中的成员文件。

### 7.2 构建可导入的 ZIP 文件PyZipFile
另一个有用的类zipfile是[PyZipFile](https://docs.python.org/3.9/library/zipfile.html#zipfile.PyZipFile). 此类与 非常相似ZipFile，当您需要将 Python 模块和包捆绑到 ZIP 文件中时，它特别方便。两者的主要区别ZipFile在于初始化程序PyZipFile采用一个名为 的可选参数`optimize`，它允许您通过在归档之前将其编译为[字节码来优化 Python 代码](https://docs.python.org/3/glossary.html#term-bytecode)。

PyZipFile提供与 相同的接口ZipFile，但添加了[.writepy()](https://docs.python.org/3/library/zipfile.html#pyzipfile-objects). 此方法可以将 Python 文件 ( .py) 作为参数并将其添加到基础 ZIP 文件中。如果optimize是-1（默认值），则.py文件会自动编译为.pyc文件，然后添加到目标存档中。为什么会这样？

从 2.3 版本开始，Python 解释器支持[从 ZIP 文件导入 Python 代码](https://docs.python.org/3/whatsnew/2.3.html#pep-273-importing-modules-from-zip-archives)，这种功能称为[Zip 导入](https://realpython.com/python-zip-import/)。这个功能相当方便。它允许您创建可导入的 ZIP 文件，以将您的[模块和包](https://realpython.com/python-modules-packages/)分发为单个存档。

> 注意：您还可以使用 ZIP 文件格式创建和分发 Python 可执行应用程序，这些应用程序通常称为 Python Zip应用程序。要了解如何创建它们，请查看Python 的 zipapp：[构建可执行的 Zip 应用程序](https://realpython.com/python-zipapp/)。

PyZipFile当您需要生成可导入的 ZIP 文件时非常有用。打包.pyc文件而不是.py文件使导入过程更有效，因为它跳过了编译步骤。

在`python-zipfile/`目录中，您有一个`hello.py`包含以下内容的模块：

```python
"""Print a greeting message."""
# hello.py

def greet(name="World"):
    print(f"Hello, {name}! Welcome to Real Python!")
```
这段代码定义了一个名为 的函数`greet()`，它接受`name`一个参数并将问候消息打印到屏幕上。现在假设您想将此模块打包成一个 ZIP 文件以用于分发。为此，您可以运行以下代码：

```python
>>> import zipfile

>>> with zipfile.PyZipFile("hello.zip", mode="w") as zip_module:
...     zip_module.writepy("hello.py")
...

>>> with zipfile.PyZipFile("hello.zip", mode="r") as zip_module:
...     zip_module.printdir()
...
File Name                                        Modified             Size
hello.pyc                                 2021-09-13 13:25:56          311
```
在此示例中，对 的调用会.writepy()自动编译hello.py并hello.pyc存储在hello.zip. 当您使用 列出档案的内容时，这一点变得很清楚.printdir()。

hello.py捆绑到 ZIP 文件后，您可以使用 Python 的导入[系统](https://realpython.com/python-import/)从其包含的存档中导入此模块：

```python
>>> import sys

>>> # Insert the archive into sys.path
>>> sys.path.insert(0, "/home/user/python-zipfile/hello.zip")
>>> sys.path[0]
'/home/user/python-zipfile/hello.zip'

>>> # Import and use the code
>>> import hello

>>> hello.greet("Pythonista")
Hello, Pythonista! Welcome to Real Python!
```
从 ZIP 文件导入代码的第一步是将该文件以[sys.path](https://docs.python.org/3/library/sys.html#sys.path). 此[变量](https://realpython.com/python-variables/)包含一个字符串列表，用于指定 Python 的模块搜索路径。要向 中添加新项目sys.path，您可以使用.insert().

要使此示例正常工作，您需要更改占位符路径并将路径传递给hello.zip您的文件系统。一旦您的可导入 ZIP 文件在此列表中，您就可以像使用常规模块一样导入代码。

最后，考虑hello/工作文件夹中的子目录。它包含一个具有以下结构的小型 Python 包：

```python
hello/
|
├── __init__.py
└── hello.py
```
该`__init__.py`模块将hello/目录转换为 Python 包。该hello.py模块与您在上面的示例中使用的模块相同。现在假设您想将此包捆绑到一个 ZIP 文件中。如果是这种情况，那么您可以执行以下操作：

```python
>>> import zipfile

>>> with zipfile.PyZipFile("hello.zip", mode="w") as zip_pkg:
...     zip_pkg.writepy("hello")
...

>>> with zipfile.PyZipFile("hello.zip", mode="r") as zip_pkg:
...     zip_pkg.printdir()
...
File Name                                        Modified             Size
hello/__init__.pyc                        2021-09-13 13:39:30          108
hello/hello.pyc                           2021-09-13 13:39:30          317
```
调用`.writepy()`将hello包作为参数，在其中搜索.py文件，将它们编译为.pyc文件，最后将它们添加到目标 ZIP 文件中，hello.zip. 同样，您可以按照之前学习的步骤从该存档中导入您的代码：

```python
>>> import sys

>>> sys.path.insert(0, "/home/user/python-zipfile/hello.zip")

>>> from hello import hello

>>> hello.greet("Pythonista")
Hello, Pythonista! Welcome to Real Python!
```
因为您的代码现在在一个包中，所以您首先需要hello从包中导入模块hello。然后你就可以greet()正常访问你的功能了。

###  7.3 zipfile从命令行运行
Pythonzipfile还提供了一个最小的[命令行界面](https://realpython.com/command-line-interfaces-python-argparse/)，允许您快速访问模块的主要功能。例如，您可以使用-lor--list选项列出现有 ZIP 文件的内容：

```python
$ python -m zipfile --list sample.zip
File Name                                         Modified             Size
hello.txt                                  2021-09-07 19:50:10           83
lorem.md                                   2021-09-07 19:50:10         2609
realpython.md                              2021-09-07 19:50:10          428
new_hello.txt                              1980-01-01 00:00:00           13
```
`.printdir()`此命令显示与对`sample.zip`存档的等效调用相同的输出。

现在假设您要创建一个包含多个输入文件的新 ZIP 文件。在这种情况下，您可以使用`-cor--create`选项：

```python
$ python -m zipfile --create new_sample.zip hello.txt lorem.md realpython.md

$ python -m zipfile -l new_sample.zip
File Name                                         Modified             Size
hello.txt                                  2021-09-07 19:50:10           83
lorem.md                                   2021-09-07 19:50:10         2609
realpython.md                              2021-09-07 19:50:10          428
```
此命令创建一个`new_sample.zip`包含您的hello.txt, lorem.md,realpython.md文件。

如果您需要创建一个 ZIP 文件来归档整个目录怎么办？例如，您可能拥有source_dir/与上面示例相同的三个文件。您可以使用以下命令从该目录创建 ZIP 文件：

```python
$ python -m zipfile -c source_dir.zip source_dir/

$ python -m zipfile -l source_dir.zip
File Name                                         Modified             Size
source_dir/                                2021-08-31 08:55:58            0
source_dir/hello.txt                       2021-08-31 08:55:58           83
source_dir/lorem.md                        2021-08-31 09:01:08         2609
source_dir/realpython.md                   2021-08-31 09:31:22          428
```
使用此命令，zipfile将放置`source_dir/`在结果`source_dir.zip`文件的根目录中。像往常一样，您可以通过zipfile使用该-l选项运行来列出存档内容。

> 注意：当zipfile您从命令行创建存档时，库在存档文件时[隐式使用](https://github.com/python/cpython/blob/5c65834d801d6b4313eef0684a30e12c22ccfedd/Lib/zipfile.py#L2408)Deflate 压缩算法。

您还可以使用命令行中的`-eor--extract`选项提取给定 ZIP 文件的所有内容：

```python
$ python -m zipfile --extract sample.zip sample/
```
运行此命令后，您sample/的工作目录中将有一个新文件夹。新文件夹将包含存档中的当前文件sample.zip。

zipfile您可以在命令行中使用的最后一个选项是-t或--test。此选项允许您测试给定文件是否是有效的 ZIP 文件。来吧，试一试！

## 8. 使用其他库管理 ZIP 文件
Python 标准库中还有一些其他工具可用于在较低级别归档、压缩和解压缩文件。Python 在zipfile内部使用其中的一些，主要用于压缩目的。以下是其中一些工具的摘要：
|模块	|描述|
|--|--|
|zlib|	允许使用zlib库进行压缩和解压缩|
|bz2|	提供使用 Bzip2 压缩算法压缩和解压数据的接口|
|lzma|	提供使用 LZMA 压缩算法压缩和解压缩数据的类和函数|


与 不同zipfile的是，其中一些模块允许您从内存和数据流中压缩和解压缩数据，而不是常规文件和档案。

在 Python 标准库中，您还可以找到[tarfile](https://docs.python.org/3/library/tarfile.html)支持[TAR](https://en.wikipedia.org/wiki/Tar_%28computing%29)归档格式的 . 还有一个名为 的模块[gzip](https://docs.python.org/3/library/gzip.html)，它提供了一个压缩和解压缩数据的接口，类似于[GNU Gzip](https://www.gnu.org/software/gzip/)程序的做法。

例如，您可以使用gzip创建包含一些文本的压缩文件：

```python
>>> import gzip

>>> with gzip.open("hello.txt.gz", mode="wt") as gz_file:
...     gz_file.write("Hello, World!")
...
13
```
运行此代码后，您将在当前目录中拥有一个`hello.txt.gz`包含压缩版本的存档。hello.txt在里面hello.txt，你会发现文本Hello, World!。

在不使用的情况下创建 ZIP 文件的一种快速且高级的方法zipfile是使用[shutil](https://docs.python.org/3/library/shutil.html). 该模块允许您对文件和文件集合执行多项高级操作。当涉及到[归档操作](https://docs.python.org/3/library/shutil.html#archiving-operations)时，您[make_archive()](https://docs.python.org/3/library/shutil.html#shutil.make_archive)可以创建归档，例如 ZIP 或 TAR 文件：

```python
>>> import shutil

>>> shutil.make_archive("shutil_sample", format="zip", root_dir="source_dir/")
'/home/user/sample.zip'
```
`sample.zip`此代码在您的工作目录中创建一个名为的压缩文件。此 ZIP 文件将包含输入目录中的所有文件，`source_dir/. make_archive()`当您需要一种快速且高级的方式在 Python 中创建 ZIP 文件时，该功能非常方便。

##  9. 结论
当您需要从ZIP 存档zipfile中读取、写入、压缩、解压缩和提取文件时，Python是一个方便的工具。[ZIP 文件格式](https://en.wikipedia.org/wiki/ZIP_%28file_format%29)已成为行业标准，允许您打包和压缩您的数字数据。

使用 ZIP 文件的好处包括将相关文件归档在一起、节省磁盘空间、便于通过计算机网络传输数据、捆绑 Python 代码以进行分发等。
在本教程中，您学习了如何：

 - 使用 Python读取、写入和提取zipfile现有的 ZIP 文件
 - 阅读有关 ZIP 文件内容的元数据zipfile
 - 用于操作现有 ZIP 文件中的zipfile成员文件
 - 创建您自己的 ZIP 文件以存档和压缩您的数字数据
 - 命令行操作zip文件
 - 创建可导入包的zip文件
 - 其他压缩zip格式文件的包或模块

您还学习了如何zipfile从命令行使用来列出、创建和提取 ZIP 文件。有了这些知识，您就可以使用 ZIP 文件格式有效地归档、压缩和处理您的数字数据。


参考：

 - [zipfile --- 使用ZIP存档](https://docs.python.org/zh-cn/3/library/zipfile.html)
 - [Create and extract (zip and unzip) a ZIP file in Python](https://note.nkmk.me/en/python-zipfile/)

