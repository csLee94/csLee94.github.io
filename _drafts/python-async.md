---
title: 왕초보를 위한 Python에서 asyncio로 비동기 처리 구현하기
author: csLee94
date: 2023-02-17 00:00:00 +0900
categories: [learning, data engineering]
tags: [python]
---

분석 위해 python을 활용해 데이터를 가공처리하다보면, 단순한 작업인데도 시간이 너무 오래 걸리는 경우들이 생깁니다. 이런 문제를 해결하기 위해 처리 방식을 **동기**처리에서 **비동기**처리로 변경하면서, 이 참에 python의 `asyncio` library에 대해서 정리해봅니다.

## Asynchronous Programming
Python은 기본적으로 동기 방식으로 동작하는 언어입니다. 즉, 기본적으로 코드가 반드시 작성된 순서 그대로 실행됩니다. 애초에 비동기 방식으로 동작하도록 설계된 언어인 JavaScript와 달리 Python은 `3.4 버전`부터 **asyncio** 라이브러리가 표준으로 채택되었고 `3.5 버전`부터 **async/await** 키워드가 문법으로 채택됐습니다.

**비동기라는 것은 쉽게 말해서 어떠한 작업이 완료되기를 기다리지 않고, 그 시간 동안 다른 작업을 하는 것**을 말합니다. 데이터 분석을 위해 Python을 공부했던 이에게는 꽤나 어색했던 개념입니다. 아주 간단한 예시를 들어보겠습니다.

```python
import time
import random 

def example_task(order_number, proceed_time):
    time.sleep(proceed_time)
    print(f"Done | proceed time of task_{order_number} is {proceed_time}")

def main():
    start = time.time()
    example_task(1, random.randint(1,5))
    example_task(2, random.randint(1,5))
    example_task(3, random.randint(1,5))
    end = time.time()
    print(f"All process is Done | proceed time is {end-start}")

if __name__ == "__main__":
    main()
```

`example_task` 함수는 random하게 1~5초 정도 소요됩니다. 만약 일반적으로 `example_task`를 3회 반복하면 각각 소요되는 시간의 합 이상이 소요될 것 입니다. 실제로 **주어진 작업 순서대로, 하나하나 처리됨**을 알 수 있습니다.

```terminal
Done | proceed time of task_1 is 2
Done | proceed time of task_2 is 4
Done | proceed time of task_3 is 1
All process is Done | proceed time is 7.011507987976074
```

반면에 간단하게 위 예시에 asyncio 라이브러리를 적용해보면 다른 결과가 나옵니다.

```python
import asyncio
import time
import random

async def example_task(order_number, proceed_time):
    await asyncio.sleep(proceed_time)
    print(f"Done | proceed time of task_{order_number} is {proceed_time}")

async def main():
    start = time.time()
    await asyncio.wait([
        example_task(1, random.randint(1,5)),
        example_task(2, random.randint(1,5)),
        example_task(3, random.randint(1,5)),
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



## REFERENCE
> - [DaleSeo](https://www.daleseo.com/python-asyncio/)
> - [**IT 엘도라도**님의 블로그](https://it-eldorado.tistory.com/159)