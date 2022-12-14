# Rest design

В подходах к дизайну Rest API разными командами используются различные методы. Эта вариация была разработал для pet-проекта, после чего была перенесена в мою рабочую команду, где применяется в production уже несколько лет. 

HTTP-статусы ответов API используются в клиенте (для уточнения контекста ситуации) и в бэкенде (все логируется, а статус отправляется в Prometheus вместе с остальными данными). Ошибки валидации или примитивные ошибки бизнес-логики, которые не оставляют двоякого понимания ситуации пользователю, игнорируются, чтобы не забивать логи и мониторинг.

## Success responses

### item

```
HTTP 200
{
    "success": true,
    "results": {
        "id": 1
    }
}
```

### collection

```
HTTP 200
{
    "success": true,
    "results": [
        {
            "id": 1
        }
    ]
}
```

### paginated collection

```
HTTP 200
{
    "success": true,
    "meta": {
        "from": 1,
        "to": 1,
        "per_page": 25,
        "current_page": 1,
        "has_more_pages": false
    },
    "results": [
        {
            "id": 1
        }
    ]
}
```

### success message

Сообщение к выполненному бэкендом действию. Может быть отображено в форме или нотификации UI. 

```
HTTP 200
{
    "success": true,
    "message": "На указанный Email отправлено письмо"
}
```

### message of business logic invariant

Бэкенд принял все входные данные, не увидел в них проблем, но остановился на инварианте бизнес-логики. Сообщение может быть отображено в форме или нотификации после UI.

```
HTTP 200
{
    "success": false,
    "message": "Такой Email уже зарегистрирован"
}
```

## Error responses

### validation error

```
HTTP 400
{
  "success": false,
  "validation_errors": [
    "email": [
      "Отсутствует поле Email"
    ]
}
```

### authorization error

Клиент не авторизован.

```
HTTP 401
{
  "success": false
}
```

### forbidden error

Клиент авторизован, но у него не хватает прав доступа к ресурсу API.

```
HTTP 403
{
  "success": false
}
```

### not found error

Ресурс не найден. Запрос в слой хранения данных вернул пустой результат. Относится только к получению сущности. Запрос коллекции вернет коллекцию без элементов.

```
HTTP 404
{
  "success": false
}
```

### internal server error

Внутренняя ошибка сервера. Логируется, клиенту отправляется код ошибки. Данные код пишется в логи для дальнейшего разбора ситуации. Детализация ошибки (caller, trace) является опциональной и используется только для dev-окружения.

```
HTTP 500
{
  "success": false,
  "error": {
    "code": "123456789",
    "message": "Ошибка чтения файла",
    "caller": "ReadFile.php (line)",
    "trace": [
    
    ]
  }
}
```









