# 2022/11/10

## 1 python与js的同步异步

生活中的异步：打开洗衣机，等待洗衣服的过程中去打开电饭煲，在煮饭的过程中可以去看书。

### 1.1 js的异步

js的异步和生活中一样，每个任务都有个等待状态和结束状态，等待状态下可以先执行其他任务。

等待状态：pending

结束状态：fulfilled或者fejected，即已完成（成功）和已拒绝（失败）

如果是嵌套await的时候，需要判断一下状态（通过返回值）

```javadoclike
function test_async(){
    $.ajax({type: 'GET',
            contentType: 'application/json; charset=utf-8',
            url: 'http://httpbin.org/delay/5',
            success: function (response) {
                console.log('5秒请求返回：', response)
            }
        })
    var a = 1 + 1
    a = a * 2
    console.log(a)
    $.ajax({type: 'GET',
            contentType: 'application/json; charset=utf-8',
            url: 'http://httpbin.org/ip',
            success: function (response) {
                console.log('查询 IP 返回：', response)

            }
        })
    console.log('这里是代码的末尾')
}
```

### 1.2 python的异步

```python
import asyncio
import aiohttp

async def main():
    async with aiohttp.ClientSession() as client:
        response = await client.get('http://httpbin.org/delay/5')
        result = await response.json()
        print('5秒请求返回：', result)
        a = 1 + 1
        a = a * 2
        print(a)
        new_response = await client.get('http://httpbin.org/ip')
        new_result = await new_response.json()
        print('查询 IP 返回：', new_result)
        print('这里是代码的末尾')

asyncio.run(main())
```

上面的那段程序是顺序执行的，要让程序异步执行，徐亚凑够一批任务提交给asyncio，让它自己通过事件循环来调度这些任务

```python
import asyncio
import aiohttp


async def do_plus():
    a = 1 + 1
    a = a * 2
    print(a)


async def test_delay(client):
    print("开始test_delay")
    response = await client.get('http://httpbin.org/delay/5')
    result = await response.json()
    print('5秒请求返回：', result)


async def test_ip(client):
    print("开始test_ip")
    response = await client.get('http://httpbin.org/ip')
    result = await response.json()
    print('查询 IP 返回：', result)


async def test_print():
    print('这里是代码的末尾')


async def main():
    async with aiohttp.ClientSession() as client:
        tasks = [
                asyncio.create_task(test_delay(client),name='n1'),
                asyncio.create_task(do_plus(),name='n2'),
                asyncio.create_task(test_ip(client),name='n3'),
                asyncio.create_task(test_print(),name='n4')
                ]
        await asyncio.gather(*tasks)
        # done, pending = await asyncio.await(tasks, timeout=2)

print("main开始")
asyncio.run(main())
print("main结束") 

"""
main开始
开始test_delay
4
开始test_ip
这里是代码的末尾
查询 IP 返回： {'origin': '27.38.193.141'}
5秒请求返回： {'args': {}, 'data': '', 'files': {}, 'form': {}, 'headers': {'Accept': '*/*', 'Accept-Encoding': 'gzip, deflate', 'Host': 'httpbin.org', 'User-Agent': 'Python/3.8 aiohttp/3.8.3', 'X-Amzn-Trace-Id': 'Root=1-636c2930-22fa90080595411818e176c3'}, 'origin': '27.38.193.141', 'url': 'http://httpbin.org/delay/5'}
main结束
"""
```

asyncio.create_task(异步函数())并没有真正运行，当调用await asyncio.gather(*tasks)时，这些任务才被作为参数传入asyncio.gather函数中，python的时间循环才开始调度他们。异步async异步函数中，await表示可能有IO等待，可以挂起去检查时间循环里面其他异步任务是否已经结束等待可以往继续他们await后面的程序。没有await的地方依然是串行的。

```python
"""
desc:测试讲解python[版本3.8***]的异步任务调度执行
"""
import asyncio
import time

task_count = 0  # 系统支持最大任务并发数
task_list = [{'name': 'task a', 'time': 1},
             {'name': 'task b', 'time': 2},
             {'name': 'task c', 'time': 3},
             {'name': 'task d', 'time': 4},
             {'name': 'task e', 'time': 5},
             {'name': 'task f', 'time': 6},
             {'name': 'task g', 'time': 7},
             {'name': 'task h', 'time': 8},
             {'name': 'task i', 'time': 9},
             {'name': 'task j', 'time': 10},
             {'name': 'task k', 'time': 11},
             {'name': 'task l', 'time': 12}]


async def delay_sleep(sleep_time):
    await asyncio.sleep(sleep_time)
    return sleep_time


async def request(unit):
    print("开始执行任务单元:", unit['name'])
    global task_count
    sleep_time = await delay_sleep(unit['time'])
    task_count -= 1  # 当前任务单元执行完成 系统正在执行任务数 -1
    print("任务单元执行完成:", unit['name'], sleep_time)


async def init():
    global task_count
    temp = task_list[:5]
    for each in temp:
        task_list.remove(each)  # 删除任务单元
        asyncio.create_task(request(each))
        task_count += 1


async def main():
    global task_count
    await init()  # 创建初始化任务 执行并发
    while True:
        print("系统当前正在执行任务数量:{}".format(task_count))
        if task_count < 5:
            if task_list:
                each = task_list[0]
                del task_list[0]
                print("添加新的任务单元:{}".format(each))
                asyncio.create_task(request(each))
                task_count += 1
            else:
                if not task_count:
                    print("所有任务单元执行完成...终止程序")
                    break
                else:
                    #  休眠等待所有任务执行完成
                    await asyncio.sleep(0.5)
        else:
            await asyncio.sleep(0.5) # 完成以后执行loop中的其他任务，也就是执行新加入的任务单元


if __name__ == '__main__':
    start_time = time.time()
    asyncio.run(main())
    print("总共消耗时间:{}".format(time.time() - start_time))
```

总消耗时间为21s

其中await asyncio.sleep()会在await完成后要求事件循环loop运行其他内容。

如果在新加入任务前使用await，即await asyncio.create_task(request(each))会导致这个任务独占线程。而不是与其他任务异步。

#### 大部分情况下使用async协程方式异步执行，但是遇到第三方库SQL（不支持async）则使用进程池或线程池包装为async的方式

```python
import asyncio
import time
import concurrent.futures

def func1():
    time.sleep(2)
    return "SB"

async def main():
    loop = asyncio.get_running_loop()
    with concurrent.futures.ThreadPlloExecutor() as pool:
        result = await loop.run_in_executor(pool,func1)
        print(result) 

```



loop.run_in_executor(pool,func1)执行步骤

- pool调用submit方法去申请一个线程执行func1，并返回一个concurrent.futures.Future对象

- 调用asyncio.wrap_future将concurrent.futures.Future对象包装为asyncio.Future对象

- 这样await才能使用
