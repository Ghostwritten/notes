![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/221587a1916049fdb74ba505197c9f92.png)


## 1. 前言
FastAPI 是一个进一步提升 Python Web 开发效率和性能的框架，具有高性能，易于使用，并提供一次性文档。本文将通过一些实战案例来认识 FastAPI 的基本功能和高级应用。

## 2. FastAPI 的优势
1. 带有完善的自动文档生成：根据模型定义，自动生成丰富的 API 文档，兼存 Swagger UI 和 ReDoc。
2. 高效性：基于 Starlette 和 Pydantic ，提供优秀性能和数据校验。
3. 完善的支持性：支持 WebSocket，GraphQL，OAuth2。
4. 最新高级功能：完全保持 Python 代码的高级引擎和控制。

---

## 3. FastAPI 快速入门
### 3.1  安装
确保已安装 Python 3.7+：

```bash
pip install fastapi uvicorn
```
注：Uvicorn 是一个高性能的 ASGI 引擎，用于运行 FastAPI 应用。

### 3.2 最简单的 API 案例
创建文件 `main.py`：

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "欢迎使用 FastAPI!"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```
启动服务器：

```bash
uvicorn main:app --reload
```
访问 `http://127.0.0.1:8000` 和 Swagger UI：`http://127.0.0.1:8000/docs`。

---

## 4. 基础功能应用
### 4.1 模型验证和参数校验
使用 Pydantic 模型来校验数据：

```python
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    description: str = None
    price: float
    tax: float = None

@app.post("/items/")
def create_item(item: Item):
    return {"item": item}
```

提供备选参数，进行文档化。

### 4.2 实现高级计划：用于实时功能和快速发布
这里例如使用 BackgroundTasks：

```python
from fastapi import BackgroundTasks

def write_log(message: str):
    with open("log.txt", "a") as log:
        log.write(message + "\n")

@app.post("/log/")
def log_message(message: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_log, message)
    return {"message": "Log recorded!"}
```

---

## 5. 高级应用

### 5.1 实现 OAuth2 认证
FastAPI 支持 OAuth2 和 JWT：

```python
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.get("/users/me")
def read_users_me(token: str = Depends(oauth2_scheme)):
    return {"token": token}
```

### 5.2 提供 WebSocket 支持

```python
from fastapi import WebSocket

@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    while True:
        data = await websocket.receive_text()
        await websocket.send_text(f"Message text was: {data}")
```

---

## 6. 总结
FastAPI 是完善的选择，适合构建低带宽、高性能的 API。通过本文中的案例，希望你能够快速上手并创建自己的实现。


- [fastapi 教程](https://fastapi.tiangolo.com/zh/tutorial/)


