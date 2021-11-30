### Оглавление
1. [Базовый пример](#example)
2. [Полезные ссылки](#links)

### Базовый пример <a name="example"></a>
```python
import aiohttp
import asyncio

URL = 'https://swapi.dev/api/people/'
MAX_HEROES = 11


async def fetch_heroes(id):
    session = aiohttp.ClientSession()
    print(f'Getting hero with id {id}')
    response = await session.get(f'{URL}{id}')
    print(f'Got hero {id}')
    await session.close()

    return response


async def main():
    tasks = []
    for i in range(1, MAX_HEROES):
        task = asyncio.create_task(fetch_heroes(i))
        tasks.append(task)

    await asyncio.gather(*tasks)


asyncio.run(main())

```

### Полезные ссылки <a name="links"></a>
1. https://habr.com/ru/post/337420/
2. http://sdiehl.github.io/gevent-tutorial/ (то же, что и #1, но на английском и про gevent)
3. https://docs.python.org/dev/library/asyncio-eventloop.html (документация по event loop)
4. https://docs.python.org/3.5/library/asyncio-task.html#coroutines (документация по таскам)
5. https://docs.python.org/3.5/library/asyncio-task.html#future (документация по футурам)
