---
title: 왕초보를 위한 Python에서 asyncio로 비동기 처리 구현하기[1]
author: csLee94
date: 2023-02-17 00:00:00 +0900
categories: [learning, data engineering]
tags: [python, async]
series:
    왕초보를 위한 Python에서 asyncio로 비동기 처리 구현하기[2] - coruntine & generator:
        url: python-async-2
---

분석을 위해 python을 활용해 데이터를 가공 처리하다 보면, 단순한 작업인데도 시간이 너무 오래 걸리는 경우들이 생깁니다. 이런 문제를 해결하기 위해 처리 방식을 **동기** 처리에서 **비동기** 처리로 변경하면서, 이참에 python의 `asyncio` library에 대해서 정리해 봅니다.

## Asynchronous Programming
Python은 기본적으로 동기 방식으로 동작하는 언어입니다. 즉, 기본적으로 코드가 반드시 작성된 순서 그대로 실행됩니다. 애초에 비동기 방식으로 동작하도록 설계된 언어인 JavaScript와 달리 Python은 `3.4 버전`부터 **asyncio** 라이브러리가 표준으로 채택되었고 `3.5 버전`부터 **async/await** 키워드가 문법으로 채택됐습니다.

**비동기라는 것은 쉽게 말해서 어떠한 작업이 완료되기를 기다리지 않고, 그 시간 동안 다른 작업을 하는 것**을 말합니다. 데이터 분석을 위해 Python을 공부했던 이에게는 꽤나 어색했던 개념입니다. 아주 간단한 예시를 들어보겠습니다.

```python
import time
import random 

def example_task(order_number, proceed_time):
    proceed_time = random.randint(1,5)
    time.sleep(proceed_time)
    print(f"Done | proceed time of task_{order_number} is {proceed_time}")

def main():
    start = time.time()
    example_task(1)
    example_task(2)
    example_task(3)
    end = time.time()
    print(f"All process is Done | proceed time is {end-start}")

if __name__ == "__main__":
    main()
```

`example_task` 함수는 random 하게 1~5초 정도 소요됩니다. 만약 일반적으로 `example_task`를 3회 반복하면 각각 소요되는 시간의 합 이상이 소요될 것입니다. 실제로 **주어진 작업 순서대로, 하나하나 처리됨**을 알 수 있습니다.

```terminal
Done | proceed time of task_1 is 2
Done | proceed time of task_2 is 4
Done | proceed time of task_3 is 1
All process is Done | proceed time is 7.011507987976074
```

반면에 간단하게 위 예시에 asyncio 라이브러리를 적용해 보고, 결과를 보면 다른 결과가 나옵니다.

```python
import asyncio
import time
import random

async def example_task(order_number):
    proceed_time = random.randint(1,5)
    await asyncio.sleep(proceed_time)
    print(f"Done | proceed time of task_{order_number} is {proceed_time}")

async def main():
    start = time.time()
    await asyncio.wait([
        example_task(1),
        example_task(2),
        example_task(3),
    ])
    end = time.time()
    print(f"All process is Done | proceed time is {end-start}")

if __name__ == "__main__":
    asyncio.run(main())
```

작업 결과는 주어진 작업 순서대로 진행되지도 않았으며, 전체 `main` 함수의 프로세스의 처리 시간이 가장 긴 처리 시간과 거의 비슷함을 알 수 있습니다.

```terminal
Done | proceed time of task_2 is 1
Done | proceed time of task_3 is 3
Done | proceed time of task_1 is 4
All process is Done | proceed time is 4.001569032669067
```

동기 처리 방식의 경우, `time.sleep()` 부분에서 결과를 기다리며 CPU를 놀리는 반면에, 비동기 처리 방식의 경우 이런 대기 시간을 낭비하지 않고 CPU가 다른 처리를 할 수 있도록 합니다.

## How to use asyncio?
기존 `def` 키워드 앞에 `async` 키워드를 추가하면 이 함수는 비동기 처리됩니다. 이런 비동기 함수는 일반적으로 `async`로 선언된 다른 비동기 함수 내에서 `await` 키워드를 붙여 호출해야 하며, `asyncio` 라이브러리를 이용해 호출할 수 있습니다.

```python
async def task_async():
    print("hello world")

async def main_async():
    await task()

asyncio.run(main_async()) # python 3.7 이상
```

혹은, `asyncio`의 event_loop를 이용해 호출할 수도 있습니다.

```python
async def task_async():
    print("hello world")

async def main_async():
    await task()

loop = asyncio.get_event_loop()
loop.run_until_complete(main_async())
loop.close()
```

하위 비동기 함수들을 제어하는 `main_async`에 여러 함수들을 리스트로 등록할 수 있습니다.

```python
async def task_async(txt):
    print(f"hello {txt}")

async def main_async():
    await asyncio.wait([
        task_async("world"),
        task_async("python"),
        task_async("async"),
    ])

asyncio.run(main_async()) # python 3.7 이상
```

각각의 task에 무작위로 time.sleep을 지정해 줘, task 처리에 걸리는 시간을 다르게 설정해보겠습니다. 또한 각 task에 소요된 시간을 함께 출력해, 비동기 처리가 반영되었는지 확인해 보겠습니다. 이때, time.sleep 함수 대신 **asyncio.sleep** 함수를 사용해 소요 시간을 발생시키는데, **asyncio.sleep** 자체도 비동기 함수이기 때문에 반드시 **await** 키워드를 붙여 호출해야 합니다.

> time.sleep 함수는 기다리는 동안 CPU를 놀리는 반면에, **asyncio.sleep**은 비동기 함수로서 기다리는 동안 다른 처리를 할 수 있도록 해줍니다.
{: .prompt-info }

```python
import asyncio
import random
import time

async def task_async(txt):
    start = time.time() # task 별 소요 시간 측정
    await asyncio.sleep(random.randint(1,5))
    print(f"hello {txt} | it takes {time.time() - start}")

async def main_async():
    start = time.time() # 전체 프로세스 소요 시간 측정
    await asyncio.wait([
        task_async("world"),
        task_async("python"),
        task_async("async"),
    ])
    print(f"All tasks are finished | it takes totally {time.time() - start}")

if __name__ == "__main__":
    asyncio.run(main_async()) # python 3.7 이상
```

결과를 확인해 보면, 잘 적용된 것을 확인할 수 있습니다.

```terminal
hello python | it takes 2.0014259815216064
hello async | it takes 3.001371145248413
hello world | it takes 5.000301122665405
All tasks are finished | it takes totally 5.000496864318848
```

## Wrap UP
어찌어찌, 비동기 구조가 반영된 코드는 작성을 마쳤습니다! 다만 python에서의 비동기 구조를 학습하다 보면, **코루틴(coroutine) / 이벤트 루프(event loop) / 제너레이터(generator) / 퓨처 객체와 태스크 객체** 등 다양한 용어가 나와 처음 학습하는 입장에서 혼동스럽습니다. 이번 포스트에서는 간단한 예시 코드로 한 사이클을 돌아봤다면, 다음 포스트에서는 각각 용어별로 정리해 보도록 하겠습니다.

## REFERENCE
> - [DaleSeo](https://www.daleseo.com/python-asyncio/)
> - [**IT 엘도라도**님의 블로그](https://it-eldorado.tistory.com/159)