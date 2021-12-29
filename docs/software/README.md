# Реалізація інформаційного та програмного забезпечення

## Створення проекту

- Використовую **IDE Visual Studio**
- Запустіть Visual Studio. У меню **File** виберіть **New > Project**. Виберіть шаблон проекту **ASP.NET Core Web Application (.NET Core)**. Назвіть проект ```TodoApi```, зніміть позначку **Host in the cloud** та натисніть **OK**.
- У вікні **New ASP.NET Core Web Application (.NET Core) - TodoApi** виберіть шаблон **Web API**. Натисніть **O**K.

## Додавання класу моделі

Модель – це об'єкт, який представляє дані у нашому додатку. У разі єдина модель — це елемент списку справ.

Додати каталог з ім'ям «Models». У браузері рішень натисніть праву кнопку миші на проекті. Виберіть **Add > New Folder**. Введіть назву каталогу **Models**.

Примітка: класи моделі можуть знаходитись у будь-якому місці проекту, але зазвичай їх розміщують у каталозі **Models**.

Додати клас ```TodoItem```. Натисніть праву кнопку миші на каталозі **Models** та виберіть **Add > Class**. Введіть ім'я класу ```TodoItem``` та натисніть **Add**.

Замініть сформований код наступним:

```cs
namespace TodoApi.Models
{
    public class TodoItem
    {
        public string Key { get; set; }
        public string Name { get; set; }
        public bool IsComplete { get; set; }
    }
}
```

## Додавання класу репозиторію
**Репозиторій** - це об'єкт, який інкапсулює рівень даних і містить логіку для отримання даних і напрямках їх до моделі. Хоча у цьому додатку немає бази даних, має сенс показати, як можна впроваджувати репозиторії в контролери. Створіть код репозиторію у каталозі **Models**.

Почніть з визначення інтерфейсу репозиторію під назвою ```ITodoRepository```. Використовуйте шаблон класу (**Add New Item > Class**).

``` cs
using System.Collections.Generic;

namespace TodoApi.Models
{
    public interface ITodoRepository
    {
        void Add(TodoItem item);
        IEnumerable<TodoItem> GetAll();
        TodoItem Find(string key);
        TodoItem Remove(string key);
        void Update(TodoItem item);
    }
}
```

Цей інтерфейс визначає основні операції CRUD.

Потім додайте клас ```TodoRepository```, який реалізує ```ITodoRepository```:

```cs
using System;
using System.Collections.Generic;
using System.Collections.Concurrent;

namespace TodoApi.Models
{
    public class TodoRepository : ITodoRepository
    {
        private static ConcurrentDictionary<string, TodoItem> _todos =
              new ConcurrentDictionary<string, TodoItem>();

        public TodoRepository()
        {
            Add(new TodoItem { Name = "Item1" });
        }

        public IEnumerable<TodoItem> GetAll()
        {
            return _todos.Values;
        }

        public void Add(TodoItem item)
        {
            item.Key = Guid.NewGuid().ToString();
            _todos[item.Key] = item;
        }

        public TodoItem Find(string key)
        {
            TodoItem item;
            _todos.TryGetValue(key, out item);
            return item;
        }

        public TodoItem Remove(string key)
        {
            TodoItem item;
            _todos.TryRemove(key, out item);
            return item;
        }

        public void Update(TodoItem item)
        {
            _todos[item.Key] = item;
        }
    }
}
```

## Регістрація репозиторію
При визначенні інтерфейсу репозиторію ми можемо відокремити клас репозиторію від контролера MVC, який його використовує. Замість реалізації ```TodoRepository``` усередині контролера ми впровадимо ```TodoRepository```, використовуючи вбудовану в ASP.NET Core підтримку [запровадження залежностей](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-6.0).

Такий підхід полегшує модульне тестування контролерів. Модульні тести впроваджують фіктивну або імітаційну версію ```ITodoRepository```. У цьому випадку тест орієнтований на логіку контролера, а не на рівень доступу до даних.

Для застосування репозиторію в контролер необхідно зареєструвати його за допомогою контейнерів DI. Відкрийте файл **Startup.cs**. Додайте наступну директиву using:
```cs
using TodoApi.Models;
```

До методу ```ConfigureServices``` додайте виділений код:

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc();

    services.AddSingleton<ITodoRepository, TodoRepository>();
}
```

## Додавання контролеру
У браузері рішень натисніть праву кнопку миші на каталозі **Controllers**. Виберіть **Add > New Item**. У вікні **Add New Item** виберіть шаблон** Web API Controller Class**. Введіть назву класу ```TodoController```.

```cs
using System.Collections.Generic;
using Microsoft.AspNetCore.Mvc;
using TodoApi.Models;

namespace TodoApi.Controllers
{
    [Route("api/[controller]")]
    public class TodoController : Controller
    {
        public TodoController(ITodoRepository todoItems)
        {
            TodoItems = todoItems;
        }
        public ITodoRepository TodoItems { get; set; }
    }
}
```

Отже визначається клас порожнього контролера. У наступних розділах описується додавання методів реалізації API.

## Отримання елементів списку справ
Щоб отримати елементи списку справ, додайте такі методи класу ```TodoController```:

```cs
public IEnumerable<TodoItem> GetAll()
{
    return TodoItems.GetAll();
}

[HttpGet("{id}", Name = "GetTodo")]
public IActionResult GetById(string id)
{
    var item = TodoItems.Find(id);
    if (item == null)
    {
        return NotFound();
    }
    return new ObjectResult(item);
}
```

Ці методи реалізують два методи GET:

- ```GET /api/todo```
- ```GET /api/todo/{id}```

У цьому випадку HTTP-відповідь для методу GetAll буде такою:

```cs
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Server: Microsoft-IIS/10.0
Date: Thu, 18 Jun 2015 20:51:10 GMT
Content-Length: 82

[{"Key":"4f67d7c5-a2a9-4aae-b030-16003dd829ae","Name":"Item1","IsComplete":false}]
```

### Маршрутизація та URL-шляхи

Атрибут ```HttpGet``` (**HttpGetAttribute**) визначає метод HTTP GET. URL-шлях для кожного методу будується так:

- Візьміть рядок шаблону з атрибута route контролера: ```[Route("api/[controller]")]```
- Замініть [Controller] на ім'я контролера, яке ви отримаєте, взявши ім'я класу контролера та прибравши суфікс Controller. У прикладі ім'я класу контролера — **TodoController**, а кореневе ім'я — todo. Маршрутизація ASP.NET Core не враховує регістр.
- Якщо атрибут ```[HttpGet]``` має рядок шаблону, додайте його до шляху. У цьому прикладі рядок шаблону не використовується.

У методі ```GetById```:

```cs
[HttpGet("{id}", Name = "GetTodo")]
public IActionResult GetById(string id)
```

```"{id}"``` - це величина, яка замінюється на ідентифікатор елемента ```todo```. Коли ```GetById``` викликається, значення “{id}” URL присвоюється параметру ```id``` методу.

```Name = "GetTodo"``` створює іменований маршрут, що дозволяє посилатися на нього в HTTP-відповіді. Надалі це буде показано на прикладі.

### Значення, що повертаються
Метод ```GetAll``` повертає ```IEnumerable```. MVC автоматично серіалізує об'єкт у JSON та записує [JSON](https://www.json.org/json-en.html) у тіло відповіді. Код відповіді для цього методу — 200, якщо немає необроблених винятків (необроблені винятки переводяться в помилки 5xx.)

У свою чергу метод ```GetById``` повертає значення більш загального типу ```IActionResult```, який представлений великою кількістю типів значень, що повертаються. ```GetById``` має два різні типи значень, що повертаються:

- Якщо немає відповідності запитуваному ідентифікатору, метод повертає помилку 404. Це відбувається при поверненні ```NotFound```.
- В інших випадках метод повертає код 200 та тіло відповіді у форматі JSON. Це відбувається при поверненні ```ObjectResult```.

### Запуск програми
Натисніть клавіші CTRL+F5 у Visual Studio, щоб запустити програму. Запуститься браузер і відкриється веб-сторінка ```http://localhost:port/api/values```, де **port** є довільним номером порту. Якщо використовується Chrome, Edge або Firefox, буде відображено дані. У разі використання IE буде запропоновано відкрити або зберегти файл **values.json**.