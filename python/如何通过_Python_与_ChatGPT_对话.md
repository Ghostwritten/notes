
![在这里插入图片描述](https://img-blog.csdnimg.cn/66349d08bd764f8fb5aee00f4b7ab480.png)







> 预备条件: 1. 科学上网； 2. [注册 OpenAI 账号](https://blog.csdn.net/xixihahalelehehe/article/details/128494398)。

##  简介
ChatGPT 是 GPT-3 语言模型的变体，专为会话语言生成而设计。要在 Python 中使用 ChatGPT，您需要安装 `OpenAI API` 客户端并获取 API 密钥。当前提你需要知道如何获取一个openAI账号，访问：


在本文中，我们将设置一个简单的示例，教您在 Python 程序中使用 ChatGPT 所需的确切步骤。

让我们开始吧。首先创建一个新的空项目文件夹：

```bash
$ mkdir python-chatgpt
$ cd python-chatgpt
```
在下一步中，我们需要为 Python 安装 OpenAI API 客户端库。

## 安装 OpenAI API
要为 Python 安装 OpenAI API 客户端库，您需要在系统上安装 Python 和 pip（Python 包管理器）。

要安装该库，请打开终端或命令提示符并键入以下命令：

```bash
pip install openai
```

##  实例1
通过访问`https://beta.openai.com/account/api-keys`获取 `YOUR-APT-KEY`
![在这里插入图片描述](https://img-blog.csdnimg.cn/c4be8a84e100402d89468625e9e9dace.png)

让我们开始使用 Python 代码与人工智能进行交互：


```bash
import openai

# Set up the OpenAI API client
openai.api_key = "YOUR-APT-KEY"

# Set up the model and prompt
model_engine = "text-davinci-003"
prompt = "男生如何寻找适合自己女朋友?"

# Generate a response
completion = openai.Completion.create(
    engine=model_engine,
    prompt=prompt,
    max_tokens=1024,
    n=1,
    stop=None,
    temperature=0.5,
)

response = completion.choices[0].text
print(response)
```
执行输出：

```bash
$ python chat.py 


1、了解自己：首先，你要了解自己，知道自己的兴趣爱好，品味，性格，以及对未来的规划。这样你才能更好地知道自己想要什么样的女朋友
。

2、设定标准：其次，你要设定一些标准，比如你希望她的身高，体重，年龄，学历，性格，家庭状况等等。这些标准可以帮助你更好地找到最
适合你的女朋友。

3、多出去：最后，你要多出去走走，多多参加一些活动，多多交友，这样你才能更容易的遇到适合自己的女朋友。
```

参考：
- [How to use ChatGPT with Python](https://www.codingthesmartway.com/how-to-use-chatgpt-with-python/)
