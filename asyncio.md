### Оглавление
1. [Базовый пример](#example)
2. [Полезные ссылки](#links)
3. [Разное](#misc)
4. [Asyncio против других concurrency методов](#asyncvs)

### Базовый пример <a name="example"></a>
```python
import aiohttp
import asyncio

URL = 'https://swapi.dev/api/people/'
MAX_HEROES = 11


async def get_heroes(hero_id):
    async with aiohttp.ClientSession() as session:
        await session.get(f'{URL}{hero_id}')
        print(f'Got hero {hero_id}')

    return hero_id


async def main():
    coroutines = [get_heroes(i) for i in range(1, MAX_HEROES)]

    heroes_ids = await asyncio.gather(*coroutines)
    print(heroes_ids)

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
    async with aiohttp.ClientSession() as session:
        response = await session.get(f'{URL}{hero_id}')
```
Т.е. каждый раз, когда интерпретатор доходит до **await**, он будет возвращаться к **async with aiohttp.ClientSession() as session**.

Аутпут:
```python
Got hero 4
Got hero 8
Got hero 1
Got hero 2
Got hero 10
Got hero 6
Got hero 5
Got hero 7
Got hero 3
Got hero 9
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

Я добавил **print(heroes_ids)**, чтобы показать, как меняется код, если вместо asyncio.gather() использовать **asyncio.as_completed()**. asyncio.gather() возвращает итератор с результатами всех корутин, поэтому ему необходимо дождаться их окончания, чтобы он смог это сделать.

Если переписать функцию main() под asyncio.as_completed() таким образом:
```python
async def main():
    coroutines = [get_heroes(i) for i in range(1, MAX_HEROES)]

    for coroutine in asyncio.as_completed(coroutines):
        hero_id = await coroutine
        print(hero_id)
```

То поскольку метод as_completed() возвращает итератор с корутинами, это позволит работать с результатами каждой конкретной корутины по мере их поступления (т.е., например, это позволит обрабатывать результаты корутины, вернувшей результаты быстрее остальных). Аутпут в данном случае выглядит так:
```python
Got hero 8
Got hero 7
Got hero 2
Got hero 3
Got hero 6
Got hero 10
Got hero 1
Got hero 4
Got hero 9
Got hero 5
```

### Полезные ссылки <a name="links"></a>
1. https://habr.com/ru/post/337420/ (про asyncio в целом, с примерами)
2. http://sdiehl.github.io/gevent-tutorial/ (то же, что и #1, но на английском и про gevent)
3. https://docs.python.org/dev/library/asyncio-eventloop.html (документация по event loop)
4. https://docs.python.org/dev/library/asyncio-task.html#coroutines (документация по таскам)
5. https://docs.python.org/dev/library/asyncio-task.html#future (документация по футурам)

### Разное <a name="misc"></a>
1. [Event loops use cooperative scheduling: an event loop runs one Task at a time. While a Task awaits for the completion of a Future, the event loop runs other Tasks, callbacks, or performs IO operations.](https://docs.python.org/dev/library/asyncio-task.html#task-object)

### Asyncio против других concurrency методов <a name="asyncvs"></a>
1. **Процессы:** поскольку 1 процесс = 1 CPU, несколько процессов это хороший способ использовать больше CPU, однако ими сложнее управлять;
2. **Треды:** кол-во тредов, которое можно запустить на CPU, ограничено в отличие от кол-ва корутин. Для корректного взаимодействия между тредами необходимо добавлять lock-механизмы, которые не так просты в обращении. Так же треды потребляют гораздо больше памяти;
3. **Очереди:** очереди обычно используются в решениях бОльших маштабов, которые подразумевают интеграцию сразу нескольких серверов.
    
