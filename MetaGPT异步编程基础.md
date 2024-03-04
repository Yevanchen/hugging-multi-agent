1. ## 为什么采用异步的结构

对于多智能体协同这种场景

1. 使用异步或者说协程的资源开销是最低的，因为异步是单线程，无论消耗的资源或者协程切换的成本都是低于进程/线程的。
2. python 提供了足够方便的工具/语法糖来对异步进行支持，写起来就像同步代码一样简单，简化了处理并行任务和复杂工作流的编程模型
3. 人类之间的团队协作在社会中就是异步的，往往多个角色可以同时开启任务

1. ## 异步的优势

这个项目作为一个LLM (Large Language Model) API封装器的代理框架，涉及到大量的API访问请求，这正是使用异步编程的主要原因之一。在这个上下文中，异步编程的使用主要带来以下几个关键优势：

1. 提高并发性：异步编程允许程序在等待API响应时不会阻塞，能够同时处理多个API请求。这对于需要与服务器频繁交互的应用来说非常重要，因为它可以显著提高应用程序的并发处理能力，从而提高整体性能和响应速度。
2. 提高效率和性能：通过异步请求，程序可以在等待某个请求的响应时继续执行其他任务，而不是空闲等待。这意味着可以更有效地利用程序和服务器资源，减少等待时间，提高整体的运行效率和性能。
3. 改善用户体验：对于客户端应用而言，异步编程可以让界面保持响应状态，即使后台正在处理耗时的操作。这样，用户界面不会因为一个长时间运行的任务而冻结，从而大大改善了用户体验。
4. 简化复杂的网络交互：在处理复杂的网络请求和响应时，异步编程模型可以简化代码的编写。通过使用`async`和`await`，开发者可以用近似同步的方式编写代码，而实际上代码执行是非阻塞的，这使得代码更加简洁易读，同时保持了高效的执行性能。

在这个项目中，使用异步编程来处理API请求，意味着可以同时向LLM发送多个查询，而不必等待一个查询完成后再发送下一个。这对于提高数据处理速率、减少等待时间和提升用户交互的流畅性至关重要，特别是在需要快速响应和处理大量数据的应用场景中。

因此，对于这个LLM API封装器的代理框架，异步编程不仅是提高性能和效率的技术手段，也是实现高并发、高响应性服务的关键技术选择。

本文章介绍的异步/协程内容只针对学习者加深MetaGPT智能体开发当中的代码理解，更深入的协程学习请参考：

[系列教程](https://www.bilibili.com/video/BV1y54y1Z7P9/?spm_id_from=pageDriver&vd_source=199360a57d436da9caa4d615e469c770)

1. ## 什么是协程

**协程，又称微线程，英文名****`Coroutine`**，是运行在单线程中的“并发”，协程相比多线程的一大优势就是省去了多线程之间的切换开销，获得了更高的运行效率。Python中的异步IO模块asyncio就是基本的协程模块。

协程是一种常规函数，能够在遇到可能需要一段时间才能完成的操作时暂停执行。当长时间运行的操作完成时，您可以恢复暂停的协程并执行该协程中的剩余代码。当协程等待长时间运行的操作时，您可以运行其他代码。通过这样做，您可以异步运行程序以提高其性能。

![img](https://deepwisdom.feishu.cn/space/api/box/stream/download/asynccode/?code=NWJkNGFhMTI2ODI5YzFhMzQxYTEyYWUxYTQ1ZTJkY2ZfdTNIQllDVmkzVFpPbklHeERVQ0JCSnExVTNDY2FtQUdfVG9rZW46S0JXZWJ4N1Jqb2Z6RHl4bTl5OGM4UE9pbm1iXzE3MDk1MTkyOTc6MTcwOTUyMjg5N19WNA)

1. ### 用例

#### 异步网络请求

在处理网络请求时，协程允许你并发地发送多个请求并等待它们的响应，而不会阻塞主线程。这对于开发高效的网络爬虫或处理大量并行API调用的服务尤为重要。

示例代码概念

```Python
import asyncio
import aiohttp

async def fetch_url(url):async with aiohttp.ClientSession() as session:async with session.get(url) as response:return await response.text()

async def main():
    urls = ["http://example.com", "http://example.org", "http://example.net"]
    tasks = [fetch_url(url) for url in urls]
    pages = await asyncio.gather(*tasks)处理获取的页面数据
asyncio.run(main())
```

#### Web应用和API服务

在开发Web应用和API服务时，协程允许服务器并发处理多个客户端请求。这对于构建高性能的Web服务和API非常有用，可以显著提高响应速度和吞吐量。

示例代码概念

假设使用基于协程的Web框架（如FastAPI）：

```Python
from fastapi import FastAPI
import httpx
app = FastAPI()

@app.get("/data")async def fetch_data():async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")return response.json()
```

#### 比较

让我们通过一个具体的例子来比较同步和异步编程的差异。我们将使用一个简单的场景：从多个URL获取数据。

##### 异步版本

这个版本使用`asyncio`和`aiohttp`来异步地从多个URL获取数据。

```Python
import asyncio
import aiohttp

async def fetch(url):async with aiohttp.ClientSession() as session:async with session.get(url) as response:return await response.text()

async def main():
    urls = ['http://example.com', 'http://example.org', 'http://example.net']
    tasks = [fetch(url) for url in urls]
    pages = await asyncio.gather(*tasks)for page in pages:print(len(page))  示例：打印每个页面的长度
asyncio.run(main())
```

##### 同步版本

下面是使用同步方法实现相同功能的代码，这里使用`requests`库来同步地获取数据。

```Python
import requests

def fetch(url):
    response = requests.get(url)return response.text

def main():
    urls = ['http://example.com', 'http://example.org', 'http://example.net']
    pages = [fetch(url) for url in urls]for page in pages:print(len(page))  示例：打印每个页面的长度
main()
```

- 异步版本：能够同时发起多个网络请求而不阻塞主线程。这意味着在等待一个请求的响应时，程序可以处理其他请求。这在处理大量网络请求时非常有效，能显著提高程序的整体性能。
- 同步版本：每个请求都是顺序发起的，必须等待当前请求完成后才能开始下一个请求。这会导致程序的运行时间主要由所有网络请求的总等待时间决定，效率较低。

通过代码我们发现二者写法主要在于import asyncio 引入的额外的关键字，需要理解`async`、`await`和事件循环的概念。

时间对比

```Python
import requests
import asyncio
import aiohttp
import time


async def fetch(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.text()


async def main():

    start = time.time()
    # 中间写上代码块
    urls = ['http://example.com', 'http://example.org', 'http://example.net']
    tasks = [fetch(url) for url in urls]
    pages = await asyncio.gather(*tasks)
    for page in pages:
        print(len(page))
    end = time.time()
    print('Running time: %s Seconds' % (end-start))

asyncio.run(main())
print("====================================================================")


def fetch(url):
    response = requests.get(url)
    return response.text


def main():
    start = time.time()
    urls = ['http://example.com', 'http://example.org', 'http://example.net']
    pages = [fetch(url) for url in urls]
    for page in pages:
        print(len(page))
    end = time.time()
    print('Running time: %s Seconds' % (end-start))


main()
```

1. ## 记忆：异步编程的基础知识（学习完第三章后阅读）

- The `async` keyword creates a coroutine. `async` 关键字创建一个协程。
- The `await` keyword pauses a coroutine. `await` 关键字暂停协程。

1. ### `async`

下面定义了一个返回整数平方数的简单函数：

```Python
def square(number: int) -> int:
    return number*number
```

您可以将一个整数传递给 `square()` 函数来获取其平方数：

```Python
def square(number: int) -> int:
    return number*number
result = square(10)
print(result)
```

Output: 输出：

```Python
100
```

当您将 `async` 关键字添加到函数中时，该函数将成为协程：

```Python
async def square(number: int) -> int:
    return number*number
async def square(number: int) -> int:
    return number*number

result = square(10)
print(result)
```

Output: 输出：

```Python
<coroutine object square at 0x00000185C31E7D80sys:1: RuntimeWarning: coroutine 'square' was never awaited
```

在这个例子中，我们调用 `square()` 协程，将返回值赋给 `result` 变量，并打印出来。

当你调用协程时，Python 不会立即执行协程内的代码。相反，它返回一个协程对象。

输出中的第二行还显示一条错误消息，指示从未等待过协程。有关详细信息，请参阅以下 `await` 部分：

```JavaScript
sys:1: RuntimeWarning: coroutine 'square' was never awaited
```

要运行协程，您需要在事件循环上执行它。在 Python 3.7 之前，您必须手动创建事件循环来执行协程并关闭事件循环。因为教程的版本是python3.9以上故我们可以用`asyncio.run()` 函数自动创建事件循环、运行协程并关闭它。

下面使用 `asyncio.run()` 函数执行 `square()` 协程并获取结果：

```Python
import asyncio


async def square(number: int) -> int:
    return number*number

result = asyncio.run(square(10))
print(result)
```

Output: 输出：

```Python
100
```

`asyncio.run()` 被设计为 `asyncio` 程序的主要入口点。

`asyncio.run()` 函数仅执行一个协程，该协程可能会调用程序中的其他协程和函数。

Jupyter笔记本或其他已经存在事件循环的环境中，可以使用`await`直接运行异步函数，或者使用`asyncio.create_task()`来在现有的事件循环中调度任务。这里有一个修改后的示例：

```Python
import asyncio

async def square(number):return number * number

#直接使用await运行异步函数，注意这行代码需要在异步环境中运行，比如在Jupyter的cell中
result = await square(10)
print(result)

#或者，如果你需要在一个异步函数中安排多个任务，可以这样做async def main():
    task = asyncio.create_task(square(10))
    result = await taskprint(result)

#然后使用await运行main函数await main()
```

在Jupyter笔记本中，你可以直接使用`await`来运行异步代码，而不需要使用`asyncio.run()`。如果你是在一个脚本中，确保整个脚本的执行入口是异步的，并且使用`if name == "__main__": asyncio.run(main())`来启动事件循环。

1. ### `await`

`await` 关键字暂停协程的执行。 `await` 关键字后面是对协程的调用，如下所示：

```Python
result = await my_coroutine()
```

`await` 关键字导致 `my_coroutine()` 执行，等待代码完成并返回结果。

需要注意的是 `await` 关键字仅在协程内部有效。换句话说，您必须在协程内使用 `await` 关键字。

```Python
import asyncio


async def square(number: int) -> int:
    return number*number


async def main() -> None:
    x = await square(10)
    print(f'x={x}')

    y = await square(5)
    print(f'y={y}')

    print(f'total={x+y}')

if __name__ == '__main__':
    asyncio.run(main())
```

How it works. 

首先，使用 `await` 关键字调用 `square()` 协程。 `await` 关键字将暂停 `main()` 协程的执行，等待 `square()` 协程完成，并返回结果：

```Python
x = await square(10)
print(f'x={x}')
```

其次，使用 `await` 关键字再次调用 `square()` 协程：

```Python
y = await square(5)
print(f'y={y}')
```

显示总计

```Python
print(f'total={x+y}')
```

以下语句使用 `run()` 函数执行 `main()` 协程并管理事件循环：

```Python
asyncio.run(main())
```

到目前为止，我们的程序就像同步程序一样执行。它没有揭示异步编程模型的强大功能。

1. ## 理解：异步编程如何提高性能（学习完第四章后阅读）

可以使用 `asyncio` 包的 `sleep()` 协程。 `sleep()` 函数延迟指定的秒数：

```Python
await asyncio.sleep(seconds)
```

因为 `sleep()` 是一个协程，所以需要使用 `await` 关键字。例如，以下使用 `sleep()` 协程来模拟 API 调用：

```Python
import asyncio


async def call_api(message, result=1000, delay=3):
    print(message)
    await asyncio.sleep(delay)
    return result
```

`call_api()` 是一个协程。它显示一条消息，暂停指定的秒数（默认为三秒），然后返回结果。

```Python
import asyncio
import time


async def call_api(message, result=1000, delay=3):
    print(message)
    await asyncio.sleep(delay)
    return result


async def main():
    start = time.perf_counter()

    price = await call_api('Get stock price of GOOG...', 300)
    print(price)

    price = await call_api('Get stock price of APPL...', 400)
    print(price)

    end = time.perf_counter()
    print(f'It took {round(end-start,0)} second(s) to complete.')

asyncio.run(main())
```

output:

```Python
Get stock price of GOOG...
300
Get stock price of APPL...
400
It took 6.0 second(s) to complete.
```

How it works

首先，使用 `time` 模块的 `perf_counter()` 函数启动一个计时器来测量时间：

```Python
 start = time.perf_counter()
```

其次，调用 `call_api()` 协程并显示结果：

```Python
price = await call_api('Get stock price of GOOG...', 300)
print(price)
```

第三，再次调用 `call_api()` ：

```Python
price = await call_api('Get stock price of APPL...', 400)
print(price)
```

最后，显示程序完成所需的时间：

```Python
end = time.perf_counter()
print(f'It took {round(end-start,0)} second(s) to complete.')
```

因为每个 `call_api()` 需要三秒，调用两次需要六秒。

在这个例子中，我们直接调用协程，不将其放在事件循环中运行。相反，我们获取一个协程对象并使用 `await` 关键字来执行它并获得结果。

![img](https://deepwisdom.feishu.cn/space/api/box/stream/download/asynccode/?code=OTJjMTAyYTcxODI3ODgzY2Q4YjVkZDU2N2Y3ODAyMDVfSU9mQjJVc3A2WGNtb2taWkowZUNPWGIxS25Hb3ZMZ1VfVG9rZW46VG1mc2JTNThobzNEYkN4enhDVmNWYlFwbk1oXzE3MDk1MTkyOTc6MTcwOTUyMjg5N19WNA)

我们使用 `async` 和 `await` 来编写异步代码，但不能同时运行它。要同时运行多个操作，我们需要使用称为任务的东西。

为了同时运行多个异步操作，你需要使用 `asyncio` 库中的 `Task`。`Task` 是 `asyncio` 中的一个对象，它封装了一个协程，并在事件循环中并发地运行。当你创建一个 `Task` 并将其提交给事件循环时，事件循环会开始运行这个任务，而不会阻塞当前协程的执行。这样，你就可以在一个协程中启动多个任务，它们将并行执行。

这里是一个简单的例子来说明如何在Python中使用 `asyncio` 和 `Task`：

```Python
import asyncio
import time

async def my_coroutine(id):
    print(f"Coroutine {id} is starting")
    # 模拟异步操作，例如网络请求或IO操作
    await asyncio.sleep(1)
    print(f"Coroutine {id} is finishing")
    return f"Result from coroutine {id}"

async def main():
    # 记录开始时间
    start_time = time.time()
    
    # 创建多个任务
    tasks = [asyncio.create_task(my_coroutine(i)) for i in range(3)]
    # 等待所有任务完成
    results = await asyncio.gather(*tasks)
    
    # 记录结束时间
    end_time = time.time()
    
    # 计算并打印耗时
    elapsed_time = end_time - start_time
    print(f"Total time taken: {elapsed_time} seconds")
    
    # 打印结果
    print(results)

# 运行主协程
asyncio.run(main())
```

在这个例子中，`main` 协程创建了三个任务，并且使用 `asyncio.gather` 来等待它们全部完成。这样，这三个协程 `my_coroutine` 可以并行执行，而不是串行等待每个协程完成。这种方式提高了程序的效率，特别是在处理多个IO密集型或网络请求时。（这里的`asyncio.gather`后面补充说明）

因为在MetaGPT中我们并没有显示(asyncio.create_task)的任务创建，因为我们在定义action以及SOP的时候都是非常自然的已经提供了细粒度的操作单元。且SOP常常需要按照特定步骤进行执行所以源码中往往直接使用 `async` 和 `await` 这样更为简洁和直接。

所以接下来我们并不展开讨论asyncio.create_task的创建和取消。如果感兴趣可以进一步查看：

[Python asyncio.create_task(): Run Multiple Tasks Concurrently](https://www.pythontutorial.net/python-concurrency/python-asyncio-create_task/)

但是已经跑完第四章的小伙伴如果有看env.run即classroom.run的源代码会发现

![img](https://deepwisdom.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjNhOGU0NmI4OTRkNzcyMGYwNTIxZDg4NDU3MmMyYjFfN0tVRHU1bjhNMGJIZDQ5NDB6WlBWMXhZVlA1Y0VGRllfVG9rZW46U2FacWJPS1pjb2NVc0l4eGZDM2N1QWRhbkJkXzE3MDk1MTkyOTc6MTcwOTUyMjg5N19WNA)

在这个run方法中什么是future？什么是gather呢？这里其实就是MG异步代码体现灵活性和拓展性的一个展示

1. ### Future

future 是一个在将来但不是现在返回值的对象。通常，未来对象是异步操作的结果。

例如，您可以从远程服务器调用 API，并期望稍后收到结果。 API 调用可能会返回一个 future 对象，以便您可以等待它。比如OpenAI或者微软的LLM服务，当然Agent的action除了访问这些服务还可以包括其他API比如airbnb，高德地图，墨迹天气等。

```Python
import asyncio
from asyncio import Future


async def main():
    my_future = Future()
    print(my_future.done())  # False

    my_future.set_result('Bright')

    print(my_future.done())  # True

    print(my_future.result())


asyncio.run(main())
```

新创造的feature没有任何价值，因为它还不存在。在这种状态下，未来被认为是不完整的、未解决的或未完成的。

调用 `done()` 方法来检查 future 对象的状态：

It returns `False`. 它返回 `False` 。

之后，通过调用 `set_result()` 方法为 future 对象设置一个值：

```Python
my_future.set_result('Bright')
```

一旦你设定了这个值，未来就完成了。在此阶段调用 future 对象的 `done()` 方法将返回 `True` 

最后，通过调用 future 对象的 `result()` 方法获取结果：

```Python
print(my_future.result())
```

你可能还疑惑以上 my_future = Future()，以及`set_result()`的过程似乎和源码不太一样。请看接下来的示例说明如何将 future 与 `await` 关键字一起使用：

```Python
from asyncio import Future
import asyncio


async def plan(my_future):
    print('Planning my future...')
    await asyncio.sleep(1)
    my_future.set_result('Bright')


def create() -> Future:
    my_future = Future()
    asyncio.create_task(plan(my_future))
    return my_future


async def main():
    my_future = create()
    result = await my_future

    print(result)


asyncio.run(main())
```

Output: 输出：

```Python
Planning my future...
Bright
```

How it works. 

首先，定义一个接受 future 并在 1 秒后设置其值的协程：

```Python
async def plan(my_future: Future):
    print('Planning my future...')
    await asyncio.sleep(1)
    my_future.set_result('Bright')
```

其次，定义一个 `create()` 函数，将 `plan()` 协程调度为任务并返回 future 对象：

```Python
def create() -> Future:
    my_future = Future()
    asyncio.create_task(plan(my_future))
    return my_future
```

第三，调用返回future的 `create()` 函数，使用await关键字等待future返回结果，并显示它：

```Python
async def main():
    my_future = create()
    result = await my_future

    print(result)
```

在实践中，您很少需要直接创建 `Future` 对象。但是，您将使用从 API 返回的 `Future` 对象。因此，了解 `Future` 的工作原理非常重要。

1. ### Gather

gather在上面提到asyncio.create_task的时候提到协程创建了三个任务，并且使用 `asyncio.gather` 来等待它们全部完成。

现在来展开了解

```Python
gather(*aws, return_exceptions=False) -> Future[tuple[()]]
```

`asyncio.gather()` 函数有两个参数：

- `aws` 是一系列可等待的对象。如果 `aws` 中的任何对象是协程，则 `asyncio.gather()` 函数会自动将其调度为任务。
- 默认情况下， `return_exceptions` 为 `False` 。如果可等待对象中发生异常，则会立即传播到等待 `asyncio.gather()` 的任务。其他等待项目将继续运行并且不会被取消。

`asyncio.gather()` 将可等待结果作为元组返回，其顺序与将可等待结果传递给函数的顺序相同

如果 `return_exceptions` 是 `True` 。 `asyncio.gather()` 会将异常添加到结果中（如果有），并且不会将异常传播给调用者。

以下示例使用 `asyncio.gather()` 运行两个异步操作并显示结果：

```Python
import asyncio


async def call_api(message, result, delay=3):
    print(message)
    await asyncio.sleep(delay)
    return result


async def main():
    a, b = await asyncio.gather(
        call_api('Calling API 1 ...', 1),
        call_api('Calling API 2 ...', 2)
    )
    print(a, b)


asyncio.run(main())
```

Output: 输出：

第一个协程需要 1 秒并返回 100，而第二个协程需要 2 秒并返回 100

2 秒后，gather 将结果作为元组返回，其中包含第一个和第二个协程的结果。

请注意，a 是 100，b 是 200，这是我们传递给 `asyncio.gather()` 函数的相应协程的结果。

```Python
Calling API 1 ...
Calling API 2 ...
100 200
```

1. ### env.run

至此我们弄明白了future和gather的原理现在我们重新审视代码

![img](https://deepwisdom.feishu.cn/space/api/box/stream/download/asynccode/?code=NzE2MmZiYWIwNTdjMTRmNjcwOGQxNjlhN2I0NGFjYzdfQ2NOeDh0eHNzblhjSDU2aUlLNXRVS1Qwd1ZSZTRNVWRfVG9rZW46TDNIS2JNc0pob1F1ZEN4UloxQ2MxbDJlbnpiXzE3MDk1MTkyOTc6MTcwOTUyMjg5N19WNA)

- 异步函数定义：`async def run(self, k=1)`定义了一个异步方法，这意味着这个方法内部可以使用`await`关键字等待其他异步操作完成。
- 循环执行：`for _ in range(k):`这个循环允许这个`run`方法执行多次（`k`次），每次执行都会处理所有角色的运行。
- 收集未来对象（futures）：`futures = []`初始化一个列表来存储每个角色的`run`方法返回的未来对象（future）。未来对象代表一个尚未完成的异步操作，可以被`await`。
- 并行执行角色的`run`方法：通过遍历`self.roles.values()`获取所有角色，并对每个角色调用`run`方法。每个`run`方法调用返回的未来对象被添加到`futures`列表中。
- 并发等待所有角色完成：`await asyncio.gather(*futures)`使用`asyncio.gather`来并发等待`futures`列表中所有未来对象的完成。`asyncio.gather`接受一个未来对象列表，并返回一个单一的未来对象，当列表中所有未来对象都完成时，这个单一的未来对象也就完成了。这允许程序并行执行多个异步任务。

其中虽然 `role.run()` 方法的调用在 `for` 循环中看起来是顺序执行的，实际上并发性的关键在于 `asyncio.gather(*futures)` 的使用。

#### 顺序调用异步函数并不等于顺序执行

- 当 `for` 循环遍历每个角色并调用其异步的 `run()` 方法时，这些调用本身是顺序的。但重要的是理解，这里的 `run()` 方法是异步函数（定义为 `async def run(...):`），这意味着它们返回的是一个协程对象，而不是直接执行。
- 这些协程对象（或称为“future”）被添加到 `futures` 列表中，此时它们尚未执行。实际的执行发生在 `await asyncio.gather(*futures)` 被调用时。

#### `asyncio.gather` 实现并发

- `asyncio.gather(*futures)` 接收一个协程对象的列表，然后并发执行这些协程。`gather` 是一个异步操作，它等待所有传入的协程并发运行并完成。
- 通过 `await` 关键字等待 `asyncio.gather` 的结果，你实际上是在等待所有并发运行的协程完成。这里的并发是指，尽管协程的启动是顺序发生的，但它们的执行是同时进行的，这是因为每个协程在遇到 `await` 表达式时会让出控制权，从而允许其他协程运行。

在这个具体示例中，即使 `role.run()` 的调用在代码中是顺序的，所有的 `run()` 方法（或者更准确地说，它们创建的协程）实际上是通过 `asyncio.gather` 并发执行的。这意味着，所有角色的 `run` 操作可以同时进行，而不需要等待每个单独的 `run` 方法顺序完成。

1. ## 应用和思考：

以斯坦福小镇类似的生成式沙盒为例，这表达了复杂的社会关系而不是software 软件公司类型的 PRD - 架构 - 开发程序- 测试的监听流程。这时env.run就会体现出巨大的效率优势。

以头脑风暴作为场景试着给出一个简单的usecase作为额外的作业：

A:Alex 老师

B:Bob  学生

C:Chalice 助教

三者在环境中彼此互相监听{a 关注 b c的输出，b关注 c a，c关注a b，在启动的时候a接受用户信息先往环境说了一句话init_action}，其中由老师发起头脑风暴

第一轮运行：a b c 都执行了观察，b c从环境中都拿到自己关注的a内容，从而b c都会运行think act然后输出

a 观察用户信息 输出a0

b观察到a0 输出b1

c观察到a0 输出c1

此时bc同时观察到a0（已完成），b1和c1的请求是同时的，await会完整的等待 b1和c1的结果后进入到下一个round

第二轮运行 a b c 都执行了观察此时future = [a0,b1,c1] 

a 观察到 b1 c1 输出a1

b观察到c1  输出b2

c观察到b1 输出c2

此时协程将同时运行 role（Alex）， role（Bob）， role（Chalice ）并发执行这些Role的run请求此时在下一轮开始前future = [a1, b2, c2 ]

1. ### 作业

请尝试完成这个多智能体SOP，并用合适的方法计算这个任务的耗时，在一个round内 abc彼此监听并轮流发言，

可以在`main`函数的开始和结束处使用`time`模块来记录时间。下面是如何实现的示例：

```Python
import time

async def main():
    start_time = time.time()
    env = Environment()环境和角色的初始化代码...await env.run()
    end_time = time.time()print(f"耗时: {end_time - start_time}秒")
asyncio.run(main())
```

将这个耗时和每个角色的行动会依次执行，每个行动必须等待前一个完成后才能开始进行比较，可以被抽象为role观察到环境里的一个触发信息然后发起一个API请求，将这个耗时*3倍。尝试多次运行求平均值观察异步带来的效率提升

同时有兴趣深入了解异步的同学还可以尝试画出程序运行图类似：

![img](https://deepwisdom.feishu.cn/space/api/box/stream/download/asynccode/?code=MmVjYzE5MmNiODVlMGJkYzdkYTIyYjdjNTExZTk3MDBfVE1Hc3c4U3VUNnk2QVVuYmtKZWo4VmFYUmlrQjZlRkFfVG9rZW46VXY5UGJ2R1dnb2daVll4MmtzY2NKNU9DbnVTXzE3MDk1MTkyOTc6MTcwOTUyMjg5N19WNA)

如果觉得这个教程对理解MG的异步过程有帮助可以帮我们的项目仓库点一个star，学习者的关注是我们最大的动力
MetaGPT:https://github.com/geekan/MetaGPT
Hugging-Muti-Agent:https://github.com/datawhalechina/hugging-multi-agent