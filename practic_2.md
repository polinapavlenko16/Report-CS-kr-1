# Практичекая работа 2  

*Павленко П. УИБО 02-24*  

1. Базовый валидный запрос

```json
{
    "birthday": "13.06.1999",
    "name": "Иван",
    "passport": "4444 № 44444444",
    "patronymic": "Иванович",
    "phone": "+7 (999)-999-99-99",
    "surname": "Иванов"
}
```

2. Тестирование с невалидными данными (фокус на ИБ)

A. SQL-инъекции

```json
{
    "birthday": "13.06.1999'; DROP TABLE users; --",
    "name": "Иван' OR '1'='1",
    "passport": "4444 № 44444444",
    "patronymic": "Иванович",
    "phone": "+7 (999)-999-99-99",
    "surname": "Иванов"
}
```

Проблема ИБ: Отсутствие санитизации входных данных может привести к выполнению произвольных SQL-команд на сервере.

B. XSS-атаки

```json
{
    "birthday": "13.06.1999",
    "name": "<script>alert('XSS')</script>",
    "passport": "4444 № 44444444",
    "patronymic": "Иванович", 
    "phone": "+7 (999)-999-99-99",
    "surname": "Иванов"
}
```

Проблема ИБ: Возможность внедрения JavaScript кода, который выполнится при отображении данных в административной панели.

C. Buffer Overflow атаки

```json
{
    "birthday": "13.06.1999",
    "name": "A".repeat(10000),
    "passport": "4444 № 44444444",
    "patronymic": "Иванович",
    "phone": "+7 (999)-999-99-99",
    "surname": "A".repeat(10000)
}
```

Проблема ИБ: Переполнение буфера может привести к отказу в обслуживании (DoS) или выполнению произвольного кода.

D. Path Traversal

```json
{
    "birthday": "../../etc/passwd",
    "name": "Иван",
    "passport": "4444 № 44444444", 
    "patronymic": "Иванович",
    "phone": "+7 (999)-999-99-99",
    "surname": "Иванов"
}
```

Проблема ИБ: Попытка доступа к системным файлам через манипуляцию параметрами.

E. JSON-инъекции

```json
{
    "birthday": "13.06.1999",
    "name": "Иван\", \"admin\": true, \"x\": \"",
    "passport": "4444 № 44444444",
    "patronymic": "Иванович",
    "phone": "+7 (999)-999-99-99",
    "surname": "Иванов"
}
```

Проблема ИБ: Возможность модификации структуры JSON и добавления несанкционированных полей.

F. Неправильные типы данных

```json
{
    "birthday": 13061999,
    "name": 12345,
    "passport": 444444444444,
    "patronymic": 67890,
    "phone": 79999999999,
    "surname": 54321
}
```

Проблема ИБ: Отсутствие строгой типизации может привести к ошибкам обработки и потенциальным уязвимостям.

3. Дополнительные тесты безопасности

Отсутствие обязательных полей

```json
{
    "birthday": "13.06.1999",
    "name": "Иван"
}
```

Специальные символы и управляющие последовательности

```json
{
    "birthday": "13.06.1999",
    "name": "Иван\n\r\t\0",
    "passport": "4444 № 44444444",
    "patronymic": "Иванович\\\"",
    "phone": "+7 (999)-999-99-99", 
    "surname": "Иванов"
}
```

4. Тесты в Postman для проверки безопасности

```javascript
// Проверка на SQL-инъекции в ответе
pm.test("No SQL errors in response", function () {
    const responseText = pm.response.text();
    pm.expect(responseText).to.not.include("SQL");
    pm.expect(responseText).to.not.include("syntax");
    pm.expect(responseText).to.not.include("database");
});

// Проверка времени ответа
pm.test("Response time acceptable", function () {
    pm.expect(pm.response.responseTime).to.be.below(2000);
});

// Проверка на раскрытие системной информации
pm.test("No sensitive data exposure", function () {
    const responseText = pm.response.text();
    pm.expect(responseText).to.not.include("at line");
    pm.expect(responseText).to.not.include("stack trace");
    pm.expect(responseText).to.not.include("file path");
});

// Проверка корректности статус кода
pm.test("Status code is appropriate", function () {
    pm.expect(pm.response.code).to.be.oneOf([200, 400, 422]);
});
```

5. Критические уязвимости:
- Инъекции кода - SQL, XSS, JSON  
- Отсутствие валидации входных данных  
- Раскрытие системной информации в ошибках  
- Отсутствие ограничений на размер данных  
