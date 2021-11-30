### Оглавление
1. [Базовый пример](#example)
2. [Полезные ссылки](#links)
3. [Разное](#misc)

### Базовый пример <a name="example"></a>
```python
import aiohttp
import asyncio

URL = 'https://swapi.dev/api/people/'
MAX_HEROES = 11


async def get_heroes(hero_id):
    session = aiohttp.ClientSession()
    print(f'Getting hero with id {hero_id}')
    response = await session.get(f'{URL}{hero_id}')
    print(f'Got hero {hero_id}')
    await session.close()

    return response


async def main():
    coroutines = [get_heroes(i) for i in range(1, MAX_HEROES)]

    await asyncio.gather(*coroutines)


asyncio.run(main())
```

Порядок выполнения кода:
1. Добавить корутины в event loop:
```python
await asyncio.gather(*coroutines)
```

2. Пройтись по коду ниже указанное кол-во раз (10):
```python
async def get_heroes(hero_id):
    session = aiohttp.ClientSession()
    print(f'Getting hero with id {hero_id}')
    response = await session.get(f'{URL}{hero_id}')
```
Т.е. каждый раз, когда интерпретатор доходит до await session.get(f'{URL}{id}'), он будет возвращаться на session = aiohttp.ClientSession().

Аутпут:
```python
Getting hero #1
Getting hero #2
Getting hero #3
Getting hero #4
Getting hero #5
Getting hero #6
Getting hero #7
Getting hero #8
Getting hero #9
Getting hero #10
Got hero #8
Got hero #10
Got hero #5
Got hero #9
Got hero #3
Got hero #6
Got hero #2
Got hero #4
Got hero #7
Got hero #1
```

### Полезные ссылки <a name="links"></a>
1. https://habr.com/ru/post/337420/ (про asyncio в целом, с примерами)
2. http://sdiehl.github.io/gevent-tutorial/ (то же, что и #1, но на английском и про gevent)
3. https://docs.python.org/dev/library/asyncio-eventloop.html (документация по event loop)
4. https://docs.python.org/dev/library/asyncio-task.html#coroutines (документация по таскам)
5. https://docs.python.org/dev/library/asyncio-task.html#future (документация по футурам)

### Разное <a name="misc"></a>
1. [Event loops use cooperative scheduling: an event loop runs one Task at a time. While a Task awaits for the completion of a Future, the event loop runs other Tasks, callbacks, or performs IO operations.](https://docs.python.org/dev/library/asyncio-task.html#task-object)
