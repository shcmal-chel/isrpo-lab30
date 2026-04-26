# Лабораторная работа №30. JSON и модели данных

**Выполнил:** Шмаль Иван Максимович
**Группа:** ИСП - 232
**Дата:** 26.04.26

---

## Цель работы:

- Разобраться с форматом JSON: структура, типы данных, вложенные объекты;
- Научиться настраивать формат JSON-ответов в ASP.NET Core;
- Применить атрибуты [JsonPropertyName] и [JsonIgnore];
- Понять как enum сериализуется в JSON и как это изменить;
- Увидеть разницу между форматом по умолчанию и настроенным.


---

## Краткая теория (вводная часть)

### Что такое JSON

**JSON** (JavaScript Object Notation) — текстовый формат для передачи данных.
Именно в этом формате наш сервер отвечает на запросы, а фронтенд читает ответы.
Когда вы делаете GET /api/heroes — сервер не отправляет C#-объекты. Он
превращает их в JSON-текст и отправляет этот текст. Браузер или JavaScript на другом
конце читает этот текст и превращает обратно в объекты.

### JSON и C#: соответствие типов

| JSON       |    C#                     |
-|-
| "строка"   |string                     |
| 42         | int                       |
| 9.8        | double                    |
| true/false | bool                      |
| null       | null (для nullable-типов) |
| ["a","b"] | List<string>              |
|{ "key": "value" }|Класс C#|

### Проблема с именами полей

В C# принято писать свойства с большой буквы: **HeroName**. В JSON принято писать с маленькой: **heroName**. Это называется **camelCase**.

System.Text.Json (используется в .NET 6+) по умолчанию сериализует свойства в camelCase. Но enum по умолчанию всё равно сериализуется как число: "universe": 0. Мы настроим явно и то, и другое — чтобы контролировать формат, а не зависеть от
умолчаний.

---

## Примеры кода:

```cs
[HttpGet("serialize")]
    public ActionResult GetSerialize() {
        var options = new JsonSerializerOptions {
            PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
            WriteIndented = true,
            Converters = { new JsonStringEnumConverter() }
        };
        var hero = new Hero {
            Id = 99,
            Name = "Тестовый Герой",
            RealName = "Шма-кадявка",
            Universe = Universe.DC,
            PowerLevel = 99,
            Powers = new () { "знание матов на разный языках", "пофигизм"},
            Weapon = new() { Name = "Слова", IsRanged = true},
            InternalNotes = "Это мысли шма"
        };
        string serialized = JsonSerializer.Serialize(hero, options);
        var deserialized = JsonSerializer.Deserialize<Hero>(serialized, options);
        return Ok(new {
            serializedJson = serialized,
            deserializedObject = deserialized,
            internalNotesAfterDeserialize = deserialized?.InternalNotes ?? "null - поле было проигнорировано"
        });
    }
```

## Итоговая таблица: что изучили в лабораторной

Концепция| Описание
|-|-|
JSON| Текстовый формат для передачи данных между сервером и клиентом
Сериализация | Объект C# → JSON-строка
Десериализация | JSON-строка → объект C#
[JsonPropertyName("name")] | Задать точное имя поля в JSON
[JsonIgnore] | Исключить поле из JSON
JsonNamingPolicy.CamelCase | Все поля автоматически в camelCase
JsonStringEnumConverter | Enum сериализуется как строка, не число
WriteIndented = true | JSON с отступами — читаемый вид
List<string> | В JSON становится массивом ["a","b","c"]
Вложенный объект | Класс внутри класса → объект внутри объекта в JSON
JsonSerializer.Serialize() | Ручная сериализация объекта в строку
JsonSerializer.Deserialize<T>() | Ручная десериализация строки в объект. <T> — тип, в который нужно преобразовать JSON. Например, Deserialize<Hero>(json) создаст объект типа Hero. <T> — это generics; тема, которую вы изучали в первом семестре

## Ссылка на репозиторий

[GitHub Repository](https://github.com/shcmal-chel/isrpo-lab30)
