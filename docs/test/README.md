# Тестування працездатності системи

Тестувати та переглядати HTTP-відповіді за допомогою [Postman](https://www.postman.com/).

## Реалізація інших операцій CRUD

Додамо методи ```Create```, ```Update``` та ```Delete```. Цей процес аналогічний тому, про що йшлося раніше, тому тут буде показаний код та виділено основні відмінності. Створіть проект після додавання або зміни коду.

### Create

```cs
[HttpPost]
public IActionResult Create([FromBody] TodoItem item)
{
    if (item == null)
    {
        return BadRequest();
    }
    TodoItems.Add(item);
    return CreatedAtRoute("GetTodo", new { id = item.Key }, item);
}
```

Це метод HTTP POST, вказаний у атрибуті [HttpPost]. Атрибут [FromBody] посилає команду MVC отримати значення елемента списку справ із тіла HTTP-запиту.

Метод CreatedAtRoute повертає відповідь 201, яка є стандартною відповіддю для методу HTTP POST, що створює новий ресурс на сервері. ```CreateAtRoute``` також додає у відповідь заголовок Location. Заголовок Location вказує URL-адресу створеного елемента списку справ. Опис: [10.2.2 201 Created](https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html).

### Використання Postman для надсилання запиту Create

<center>
    <img src="/postman1.png" style="
    width:60%;
    margin:1em 0;
    border-radius:4px;
    border: 1px solid #cfd7e6;
    box-shadow: 0 1px 3px 0 rgba(89,105,129,.05), 0 1px 1px 0 rgba(0,0,0,.025);
    ">
</center>

- Встановіть ```POST``` як метод HTTP.
- Виберіть **Body**.
- Виберіть **raw**.
- Виберіть тип JSON.
- У редакторі пар ключ-значення вкажіть елемент Todo так: ```{"Name":"<your to-do item>"}```.
- Натисніть кнопку **Send**.

Виберіть закладку Headers та скопіюйте заголовок **Location**:

<center>
    <img src="/postman2.png" style="
    width:60%;
    margin:1em 0;
    border-radius:4px;
    border: 1px solid #cfd7e6;
    box-shadow: 0 1px 3px 0 rgba(89,105,129,.05), 0 1px 1px 0 rgba(0,0,0,.025);
    ">
</center>

Для доступу до ресурсу, який щойно створено, можна використовувати URL із заголовка Location. Повторно викличте спосіб ```GetById```, який створив іменований маршрут ```"GetTodo"```:

```cs
[HttpGet("{id}", Name = "GetTodo")]
public IActionResult GetById(string id)
```

### Update

```cs
[HttpPut("{id}")]
public IActionResult Update(string id, [FromBody] TodoItem item)
{
    if (item == null || item.Key != id)
    {
        return BadRequest();
    }

    var todo = TodoItems.Find(id);
    if (todo == null)
    {
        return NotFound();
    }

    TodoItems.Update(item);
    return new NoContentResult();
}
```

```Update``` подібний до ```Create```, але використовує HTTP PUT. Відповідь [204 (Немає вмісту)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html). Відповідно до специфікації HTTP на запит PUT потрібно, щоб клієнт відправив оновлений об'єкт повністю, а не тільки дельти. Для підтримки часткових оновлень використовуйте HTTP PATCH.

<center>
    <img src="/postman3.png" style="
    width:60%;
    margin:1em 0;
    border-radius:4px;
    border: 1px solid #cfd7e6;
    box-shadow: 0 1px 3px 0 rgba(89,105,129,.05), 0 1px 1px 0 rgba(0,0,0,.025);
    ">
</center>

### Update з використанням Patch
Аналогічно ```Update```, але за допомогою HTTP PATCH. Відповідь [204 (Немає вмісту)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).

```cs
[HttpPatch("{id}")]
public IActionResult Update([FromBody] TodoItem item, string id)
{
    if (item == null)
    {
        return BadRequest();
    }

    var todo = TodoItems.Find(id);
    if (todo == null)
    {
        return NotFound();
    }

    item.Key = todo.Key;

    TodoItems.Update(item);
    return new NoContentResult();
}
```

<center>
    <img src="/postman4.png" style="
    width:60%;
    margin:1em 0;
    border-radius:4px;
    border: 1px solid #cfd7e6;
    box-shadow: 0 1px 3px 0 rgba(89,105,129,.05), 0 1px 1px 0 rgba(0,0,0,.025);
    ">
</center>

### Delete
```cs
[HttpDelete("{id}")]
public IActionResult Delete(string id)
{
    var todo = TodoItems.Find(id);
    if (todo == null)
    {
        return NotFound();
    }

    TodoItems.Remove(id);
    return new NoContentResult();
}
```
Відповідь [204 (Немає вмісту)](https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html).

<center>
    <img src="/postman5.png" style="
    width:60%;
    margin:1em 0;
    border-radius:4px;
    border: 1px solid #cfd7e6;
    box-shadow: 0 1px 3px 0 rgba(89,105,129,.05), 0 1px 1px 0 rgba(0,0,0,.025);
    ">
</center>
