> API информационной системы Национального кредитного бюро

Термины и определения
=====================

Клиент API (Клиент) - организация заключившая договор с Национальным кредитным бюро 
на использование API.

ИС НКБ - информационная система Национального кредитного бюро.

ИС Клиента - информационная система Клиента.


API
===

Обмен между ИС НКБ и ИС Клиента осуществляется по протоколу HTTPS. 

Формат данных обмена - JSON. 

Параметры в запроса должны быть в кодировке UTF-8 и закодированы 
для передачи в URL (http://en.wikipedia.org/wiki/Percent-encoding).

ИС Клиента выступает инициатором обмена и является клиентом по отношению к 
серверу ИС НКБ.

При ответе на запрос, ИС НКБ может выставить один из перечисленных HTTP статусов:

* 200 OK: Запрос обработан успешно. Тело ответа содержит запрашиваемые данные.
* 400 Bad Request: Запрос содержит некорректные данные.
* 403 Forbidden: Сервер не может авторизовать клиента для выполнения запроса.
* 404 Not Found: Запрашиваемый ресурс не найден.
* 405 Method Not Allowed: Неподдерживаемый HTTP метод.
* 500 Internal Server Error: На сервере произошла ошибка.
* 502 Bad Gateway: Сервер недоступен или обновляется.
* 503 Service Unavailable: Сервер доступен, но перегружен запросами. Необходимо 
повторить запрос позже.


Аутентификация
--------------

HTTP Basic over HTTPS


Поиск организаций
-----------------

### Запрос

`GET /api/company`

### Параметры

* inn [строка 10 цифр, опционален] - ИНН организации
* ogrn [строка из 13 цифр, опционален] - ОГРН организации
* okpo [строка из 8 цифр, опционален] - ОГРН организации
* id [строка из цифр, опционален] - ID организации в системе

В запросе должен быть передан по крайней мере один идентификатор (ИНН, ОГРН, ОКПО или ID).
В случае передачи двух и более идентификаторов, поиск будет производиться по соответствию всех переданных идентификаторов.

Если в идентификаторе присутствует ведущий ноль, он также обязателен в запросе,
например поиск по ОКПО 00044428:

`?okpo=00044428`

* subsidiary [флаг, опционален] - учитывать филиалы.

Если значение равно `true`, то при поиске организации по ИНН или ОГРН также будут
найдены ее филиалы, представительства и обособленные подразделения.
Для однозначной идентификации организации, филиала, представительства или
подразделения необходимо использовать код ОКПО.

Если значение равно `false`, то при поиске организации исключаются ее филиалы,
представительства и подразделения. При поиске по ИНН или ОГРН будет найдена
только головная организация.

Значение по умолчанию `true`.

Вывод найденных организаций производится постранично, доступны параметры:

* page [целое число, опционален] - номер страницы

значение по умолчанию - 1.

* pageSize [целое число, опционален] - число организаций на одной странице

значение по умолчанию - 10.

Пример:

`?inn=7714611569&subsidiary=false&page=2&pageSize=5`

### Ответ

```json
{
    "total" : "<количество найденных организаций>",
    "pageSize" : "<размер страницы>",
    "pageNumber" : "<номер страницы>",
    "pageCount" : "<число страниц>",
    "firstNumber" : "<номер первой организации на странице>",
    "lastNumber" : "<номер последней организации на странице>",
    "list" : [
        {
            "inn" : "<ИНН>",
            "ogrn" : "<ОГРН>",
            "okpo" : "<ОКПО>",
            "id" : "уникальный идентификатор организации в системе",
            "address" : "<Адрес>",
            "chiefName" : "<ФИО руководителя или наименование управляющей компании>",
            "fullName" : "<Полное наименование>",
            "shortName" : "<Сокращенное наименование>"
        },
        {
            "inn" : "<ИНН>",
            "ogrn" : "<ОГРН>",
            "okpo" : "<ОКПО>",
            "id" : "уникальный идентификатор организации в системе",
            "address" : "<Адрес>",
            "chiefName" : "<ФИО руководителя или наименование управляющей компании>",
            "fullName" : "<Полное наименование>",
            "shortName" : "<Сокращенное наименование>"
        },
        ...
    ]
}
```

Ответ содержит количество найденных компаний (поле total), параметры страницы (поля pageSize,
pageNumber, pageCount, firstNumber, lastNumber) и набор карточек (поле list).

Каждая карточка содержит идентификаторы организации (ИНН, ОГРН, ОКПО), адрес,
ФИО руководителя или наименование управляющей компании, полное и сокращенное
наименование организации. В случае, если не найдено ни одной организации (значение total равно 0),
поле list представляет собой пустой массив.

### Пример №1: Поиск организации по ОГРН с учетом филиалов

`GET https://www.creditnet.ru/nkbrelation/api/company?ogrn=1027700333440`

`200 OK`

```json
{
    "total" : 2,
    "pageSize" : 10,
    "pageNumber" : 1,
    "pageCount" : 1,
    "firstNumber" : 1,
    "lastNumber" : 2,
    "list": [
        {
            "inn" : "7718083616",
            "ogrn" : "1027700333440",
            "okpo" : "17613675",
            "id" : "750397",
            "address" : "г Москва, ул 3-я Рыбинская, д 22",
            "chiefName" : "Дрибный Андрей Николаевич",
            "fullName" : "ОТКРЫТОЕ АКЦИОНЕРНОЕ ОБЩЕСТВО \"ЭКСТРА М\"",
            "shortName" : "ОАО \"ЭКСТРА М\""
        },
        {
            "inn" : "7718083616",
            "ogrn" : "1027700333440",
            "okpo" : "03559966",
            "id" : "9519004",
            "address" : "г Санкт-Петербург, ул Коли Томчака, 21",
            "chiefName" : "Ягмешин Юрий Валентинович",
            "fullName" : "Санкт-Петербургский филиал Открытого акционерного общества \"Экстра М\"",
            "shortName" : "Санкт-Петербургский филиал ОАО \"Экстра м\""
        }
    ]
}
```

### Пример №2: Поиск организации по ОГРН и ОКПО с учетом филиалов

`GET https://www.creditnet.ru/nkbrelation/api/company?ogrn=1027700333440&okpo=17613675`

`200 OK`

```json
{
    "total" : 1,
    "pageSize" : 10,
    "pageNumber" : 1,
    "pageCount" : 1,
    "firstNumber" : 1,
    "lastNumber" : 1,
    "list": [
        {
            "inn" : "7718083616",
            "ogrn" : "1027700333440",
            "okpo" : "17613675",
            "id" : "750397",
            "address" : "г Москва, ул 3-я Рыбинская, д 22",
            "chiefName" : "Дрибный Андрей Николаевич",
            "fullName" : "ОТКРЫТОЕ АКЦИОНЕРНОЕ ОБЩЕСТВО \"ЭКСТРА М\"",
            "shortName" : "ОАО \"ЭКСТРА М\""
        }
    ]
}
```

### Пример №3: Поиск организации по ИНН без учета филиалов

`GET https://www.creditnet.ru/nkbrelation/api/company?inn=7706107510&subsidiary=false`

`200 OK`

```json
{
    "total" : 1,
    "pageSize" : 10,
    "pageNumber" : 1,
    "pageCount" : 1,
    "firstNumber" : 1,
    "lastNumber" : 1,
    "list": [
        {
            "inn" : "7706107510",
            "ogrn" : "1027700043502",
            "okpo" : "00044428",
            "id" : "750397",
            "address" : "г Москва, наб Софийская, д 26/1",
            "chiefName" : "Сечин Игорь Иванович",
            "fullName" : "Открытое акционерное общество \"Нефтяная компания \"Роснефть\"",
            "shortName" : "ОАО \"НК \"Роснефть\""
        }
    ]
}
```

### Пример №4: Поиск организации по ОГРН с учетом филиалов вернул пустой результат поиска

`GET https://www.creditnet.ru/nkbrelation/api/company?ogrn=1237700123000`

`200 OK`

```json
{
    "total" : 0,
    "pageSize" : 10,
    "pageNumber" : 1,
    "pageCount" : 0,
    "firstNumber" : 0,
    "lastNumber" : 0,
    "list": [ ]
}
```



Проверка на аффилированность
----------------------------

### Запрос

`GET /api/connections`

### Параметры

* q [строка, опционален] - идентификатор организации: ОКПО, ИНН или ОГРН.
* i [строка, опционален] - идентификатор физического лица - Фамилия имя и отчество полностью (ФИО),
например *Костюк Андрей Григорьевич*. Значение параметра нечувствительно к регистру символов.

Для осуществления проверки необходимо передать два или более значения -
идентификатора организации или ФИО, при этом по каждой организации ожидается
только один идентификатор.

Пример:

`?q=60510227&q=1117746960704&q=7703561549&i=Костюк+Андрей+Григорьевич&i=Малеев+Владимир+Иванович`

Несколько идентификаторов организаций можно передать в одном параметре `q`, а
несколько ФИО - в одном параметре `i`. Для этого необходимо разделить их символом
перевода строки (ASCII код `0x0A`):

Пример:

`?q=60510227%0A1117746960704%0A7703561549&i=Костюк+Андрей+Григорьевич%0AМалеев+Владимир+Иванович`

* subsidiary [флаг, опционален] - учитывать филиалы.

Если значение равно `true`, то при поиске организации по ИНН или ОГРН также будут
найдены ее филиалы, представительства и обособленные подразделения.
Для однозначной идентификации организации, филиала, представительства или
подразделения необходимо использовать код ОКПО.

Если значение равно `false`, то при поиске организации исключаются ее филиалы,
представительства и подразделения. При поиске по ИНН или ОГРН будет найдена
только головная организация.

Значение по умолчанию `true`.

### Ответ

```json
{
    "connections": [
        ["<идентификатор11>", "<идентификатор12>", ..., "<идентификатор1N>"],
        ["<идентификатор21>", "<идентификатор22>", ..., "<идентификатор2M>"],
        ...
    ],
    "validation": {
        "<идентификаторX>": {"status": "<статусX>", "info": <инфоX>},
        "<идентификаторY>": {"status": "<статусY>", "info": <инфоY>},
        ...
    }
}
```

Ответ содержит набор групп аффилированных лиц `"connections"`.
В каждой группе перечислены идентификаторы, между которыми были выявлены признаки
аффилированности. Каждый идентификатор может принадлежать только одной группе
аффилированных лиц.

В примерах используются идентификаторы организаций:

* 60510227 - ООО "Газэнергоинформ", ОГРН 1097746173249, ИНН 7728696530, ОКПО *60510227*
* 1117746960704 - ЗАО "Спецстрой-Лизинг", ОГРН *1117746960704*, ИНН 7726687266, ОКПО 37276659
* 7703561549 - ООО "Фабрикант.ру", ОГРН: 1057748006139, ИНН *7703561549*, ОКПО 78522076
* 1047796089450 - ООО "ЮВЕНТА", ОГРН *1047796089450*, ИНН 7724505200, ОКПО 72066223
* 1127746519900 - ООО "НАЛПОИНТЕР", ОГРН *1127746519900*, ИНН 7704811150, ОКПО 9933928
* 1057747690890 - ЗАО "НКБ", ОГРН *1057747690890*, ИНН 7714611569, ОКПО 78486593

и идентификаторы физических лиц:

* *Малеев Владимир Иванович* - генеральный директор ЗАО "НКБ"
* *Костюк Андрей Григорьевич* - генеральный директор ООО "НАЛПОИНТЕР"

Если в запросе есть некорректные идентификаторы, ответ будет содержать объект
`"validation"` с описанием выявленных проблем. Ключами в объекте являются
некорректные идентификаторы, а значениями - описания проблем.

Для каждого некорректного идентификатора, описание проблемы представляет собой 
объект с двумя полями - обязательной строкой `"status"` и необязательным объектом
`"info"` и может принимать одно из следующих значений:

* `{"status": "NOT_FOUND"}` - идентификатор не найден в ИС НКБ. Идентификатор не участвует в проверке.
* `{"status": "MULTIPLE", "info": <количество совпадений>}` - по идентификатору найдено несколько записей в ИС НКБ.
Идентификатор не участвует в проверке. Объект `"info"` содержит количество записей с заданным идентификатором.
* `{"status": "ALIAS", "info": "<идентификатор>"}` - идентификатор встречается в
запросе больше одного раза или является иным уникальным идентификатором записи
участвующей в проверке. Объект `"info"` содержит строку-идентификатор этой записи.


### Пример №1: Выявлены признаки аффилированности среди всех участников

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=60510227&q=1117746960704&q=7703561549`

`200 OK`

```json
{
    "connections": [
        ["60510227", "1117746960704", "7703561549"]
    ]
}
```

Запрос на проверку аффилированности между организациями 

ООО "Газэнергоинформ", ЗАО "Спецстрой-Лизинг" и ООО "Фабрикант.ру"

выявил одну группу аффилированных лиц, содержащую все запрошенные идентификаторы.


### Пример №2: Выявлены признаки аффилированности среди некоторых участников

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=1117746960704&q=7703561549&q=1127746519900`

`200 OK`

```json
{
    "connections": [
        ["1117746960704", "7703561549"]
    ]
}
```

Запрос на проверку аффилированности между организациями 

ЗАО "Спецстрой-Лизинг", ООО "Фабрикант.ру" и ООО "НАЛПОИНТЕР"

выявил одну группу аффилированных лиц:

ЗАО "Спецстрой-Лизинг" и ООО "Фабрикант.ру"


### Пример №3: Выявлено несколько групп с признаками аффилированности

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=1117746960704&q=7703561549&q=1127746519900&q=1057747690890`

`200 OK`

```json
{
    "connections": [
        ["1117746960704", "7703561549"],
        ["1127746519900", "1057747690890"],
    ]
}
```

Запрос на проверку аффилированности между организациями 

ЗАО "Спецстрой-Лизинг", ООО "Фабрикант.ру", ООО "НАЛПОИНТЕР" и ЗАО "НКБ"

выявил две группы аффилированных лиц:

ЗАО "Спецстрой-Лизинг" и ООО "Фабрикант.ру"
ООО "НАЛПОИНТЕР" и ЗАО "НКБ"


### Пример №4: Признаки аффилированности не выявлены

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=1117746960704&q=1047796089450&q=1127746519900`

`200 OK`

```json
{
}
```

Запрос на проверку аффилированности между тремя организациями

ЗАО "Спецстрой-Лизинг", ООО "ЮВЕНТА" и ООО "НАЛПОИНТЕР"

не выявил признаков аффилированности - ответ не содержит массива `"connections"`


### Пример №5: Один идентификатор в запросе

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=1117746960704`

`400 Bad Request`

```json
{
}
```

Ответ имеет статус `400 Bad Request`, т.к. запрос на проверку аффилированности
должен содержать идентификаторы двух или более организаций.


### Пример №6: Несколько идентификаторов по одной организации

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=1117746960704&q=7726687266&q=37276659&q=1127746519900`

`200 OK`

```json
{
    "validation": {
        "7726687266": {"status": "ALIAS", "info": "1117746960704"},
        "37276659": {"status": "ALIAS", "info": "1117746960704"},
    }
}
```

Запрос на проверку аффилированности между организациями

ЗАО "Спецстрой-Лизинг" и ООО "НАЛПОИНТЕР"

не выявил признаков аффилированности - ответ не содержит массива `"connections"`

Запрос содержит несколько идентификаторов (ОГРН, ИНН и ОКПО) ЗАО "Спецстрой-Лизинг"
- ответ содержит объект `"validation"` с описанием проблем по идентификаторам
`"7726687266"` и `"37276659"` - эти идентификаторы являются иными идентификаторами
ЗАО "Спецстрой-Лизинг".


### Пример №7: Идентификатор не найден в ИС НКБ

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=987654321&q=1117746960704&q=7703561549`

`200 OK`

```json
{
    "connections": [
        ["1117746960704", "7703561549"],
    ],
    "validation": {
        "987654321": {"status": "NOT_FOUND"}
    }
}
```

Запрос на проверку аффилированности между организациями

с идентификатором `"987654321"`, ЗАО "Спецстрой-Лизинг" и ООО "Фабрикант.ру"

выявил одну группу аффилированных лиц:

ЗАО "Спецстрой-Лизинг" и ООО "Фабрикант.ру"

Ответ содержит объект `"validation"` с описанием проблемы по идентификатору
`"987654321"` - идентификатор не найден в ИС НКБ.


### Пример №8: Один корректный идентификатор в запросе

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=987654321&q=1127746519900`

`400 Bad Request`

```json
{
    "validation": {
        "987654321": {"status": "NOT_FOUND"}
    }
}
```

Запрос на проверку аффилированности между организациями

с идентификатором `"987654321"` и ООО "НАЛПОИНТЕР"

вернул статус `400 Bad Request` т.к. содержит только один корректный идентификатор.

Ответ содержит объект `"validation"` с описанием проблемы по идентификатору
`"987654321"` - идентификатор не найден в ИС НКБ.


### Пример №9: Неоднозначный идентификатор

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=7708503727&q=1047796089450&q=1127746519900`

`200 OK`

```json
{
    "validation": {
        "7708503727": {"status": "MULTIPLE", "info": 4480}
    }
}
```

Запрос на проверку аффилированности между организациями

ООО "ЮВЕНТА" и ООО "НАЛПОИНТЕР"

не выявил признаков аффилированности - ответ не содержит массива `"connections"`

Ответ содержит объект `"validation"` с описанием проблемы по идентификатору
`"7708503727"` - по идентификатору найдено 4480 организаций.


### Пример №10: Выявлены признаки аффилированности между участниками-физическими лицами

`GET https://www.creditnet.ru/nkbrelation/api/connections?i=Костюк+Андрей+Григорьевич&i=Малеев+Владимир+Иванович`

`200 OK`

```json
{
    "connections": [
        ["Костюк Андрей Григорьевич", "Малеев Владимир Иванович"]
    ]
}
```

Запрос на проверку аффилированности между ФИО

Костюк Андрей Григорьевич и Малеев Владимир Иванович

выявил признаки аффилированности между ними.


### Пример №11: Выявлены признаки аффилированности между организацией и физическим лицом

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=1057747690890&i=Малеев+Владимир+Иванович`

`200 OK`

```json
{
    "connections": [
        ["1057747690890", "Малеев Владимир Иванович"]
    ]
}
```

Запрос на проверку аффилированности между

ЗАО "НКБ" и Малеев Владимир Иванович

выявил признаки аффилированности между ними.


### Пример №12: Исключение филиалов

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=1027700043502&i=Сечин+Игорь+Иванович&subsidiary=false`

`200 OK`

```json
{
    "connections": [
        ["1027700043502", "Сечин Игорь Иванович"]
    ]
}
```

Запрос на проверку аффилированности между

ОАО "НК "Роснефть" (ОГРН *1027700043502*) и Сечин Игорь Иванович

выявил признаки аффилированности между ними.


### Пример №13: Исключение филиалов не выполняется

`GET https://www.creditnet.ru/nkbrelation/api/connections?q=1027700043502&i=Сечин+Игорь+Иванович&subsidiary=true`

`400 Bad Request`

```json
{
    "validation": {
        "1027700043502": {"status": "MULTIPLE", "info": 12}
    }
}
```

Запрос на проверку аффилированности между

ОАО "НК "Роснефть" (ОГРН *1027700043502*) и Сечин Игорь Иванович

вернул статус `400 Bad Request` т.к. содержит только один корректный идентификатор.

Ответ содержит объект `"validation"` с описанием проблемы по идентификатору
`"1027700043502"` - найдено несколько (12) записей в ИС НКБ.

