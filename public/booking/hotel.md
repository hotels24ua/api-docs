```
[POST] /booking/hotel
```
it should looks [like](https://developers.google.com/discovery/v1/reference/apis/list) but MD does not support colors :(

- [x] basic sketch
- [ ] draft done
- [ ] draft approved
- [ ] published

#####Общее описание
 Метод для бронирования номеров в отеле.
 Бронирование может быть произведено в один запрос (за исключением случаев, где должна производится онлайн оплата).
 Бронирование состоит из нескольких этапов (см. [States](#States)).
 Система может произвести остановку на каком-либо [этапе](#States) в следующих случаях:

  * если не все необходимые данные присутствуют в запросе
  * данные не валидны
  * произошла непредвиденная ошибка
  * клиентская сторона сама инициировала остановку на нужном [этапе](#States) под средством специального указателя [`next_state`](#next_state)

#####Специальные обозначения
######Параметры
Пример | Описание
--- | ---
`parameter` | обычный обязательный параметр
`[optional_parameter]` | опциональный параметр заключен в кв.скобки (может присутствовать, а может и отсутствовать. Детали поведеня должны быть приложены в описании)
`parameter.property` | `parameter` является объектом, а `property` его свойством
`[optional_parameter].property` | опциональный параметр `optional_parameter`, но если присутствует, то обязательно должен иметь свойство `property`
`[optional_parameter.property2]` | опциональный параметр `optional_parameter`, опционально может иметь свойство `property2`
[`parameter_doc_ref`](#parameter_reference) | ссылка на параметр в документации
[`parameter_doc_ref.`](#parameterRef)`property` | то же самое что и `parameter.property`, только на родительский параметр присутствует ссылка в документации
`parameter[]` | параметр со списком значений (известны так же как `list`, `enum`, `Array`, etc)
`parameter[].itemProperty` | параметр является списком объектов с обязательными свойствами `itemProperty`
`parameter[].[itemProperty2]` | параметр является списком объектов с опциональными свойствами `itemProperty`

######Типы данных
Обозначение | Варианты использования | Описание
--- | --- | ---
`boolean` | `boolean`, `bool` | `true`, `false`, `1`, `0` , `"yes"`, `"no"`
`string` | `string`, `str` |  Строка
`integer` | `int`, `integer` | Число. Целое
`object` | `object`, `obj`, `{}` | объект с какими-то свойствами
<a name="t_date"></a>`date` | `date` | Строка даты дня. формат: `"yyyy-mm-dd"`
`dateTime` | `dateTime` | Дата. Дата-время (RFC 3339) `"yyyy-mm-ddTHH:MM:ss"`
`list` | `list`, `array` | список
 | `list[string]`, `[string]`, | список состоящий из типов указаных в `[...]` (здесь - строк)
 | [[`Entity`](#entity)] | список состоящий из набора [`Entity`](#entity)
[`entity`](#entity) | [`SomeEntity`](#someEntity) | тип является какой-то сущностью и полностью соответствует её схеме (+ссылка на документацию)
...

####Request
Запрос принимает данные в форматах:
`application/json` - json encoded post body
`application/x-www-form-urlencoded` - url encoded

#####Request Flow
<a name="optional_conditions"></a>**[`optional_condition`]** - опциональное условие бронирования.
Помечаются условия бронирования и другие параметры, которые могут быть проигнорированы в запросе если клиенту этот параметр не важен.
Если параметр предоставлен в запросе, но он не совпадает с фактическим параметром в базе - запрос будет возвращен обратно со [статусом](#status) [`"rejected"`](#status_rejected).
В обратном запросе [`"rejected"`](#status_rejected) будет указаны:
 * несовпадения в поле [`status_body`](#status_body),
 * стадия на которой была произведена остановка [статусом](#status).
Этот функционал существует для того, чтобы клиентская сторона могла подтвердить изменения в условиях или произвести другие действия по корректировке запроса.
Если же [опицональне условия](#optional_conditions) упущены система произведет автоматическое принятие параметров на основе существующих в базе.

#####Request Parameters

  Name | Type | Description | Notes
  --- | --- | --- | ---
  `arrival_date` | [date](#t_date) | дата заезда
  `departure_date` | [date](#t_date) | дата выезда
  `blocks` | list | список бронируемых объектов (комнат/тарифов)
  `blocks[].tariff` | string |
  `blocks[].[conditions]` | object | объект с условиями бронирования `cancellation` и `booking_method`
  `blocks[].[conditions.cancellation]` | integer | кол-во дней перед днем заезда после которого отмена не возможна
  `blocks[].[conditions.booking_method]` | integer |метод бронирования для данного блока (опионален. если присутствует - будет свалидирован с фактическим методом указаным в базе. используется только в случае если для клиента важно где и как платить)
  `blocks[].[recovery]` | object | предоплата, которая взымается за номер сразу при бронировании
  `blocks[].[recovery].unit` | string | единица измерения поля `recovery.amount` (% - "percent" или дни - "day")
  `blocks[].[recovery].amount` | integer | кол-во ед. предоплаты (1-9 - дни 10-100 - проценты)
  `blocks[].[recovery].sum` | integer | общая сумма пердоплаты в денежном эквиваленте (указывается за весь период по конкретному блоку)
  `blocks[].total_cost` | integer | стоимость проживания в блоке за весь период
  `blocks[].quantity` | integer |  количество бронируемых блоков этого типа (тарифа)
  `blocks[].[living]` | list | список проживающих (на каждый `quantity` или один на все. В случае, если проживающих не достаточно, будет выбран тот, который указан в поле [`requisites`](#requisites)
  `blocks[].[living[]].first_name` | string |  имя проживающего в блоке
  `blocks[].[living[].last_name]` | string | фамилия проживающего в блоке
  `requisites` | object | объект реквизитов бронирующего клиента (обязателен для стадий после [`"preRequisites"`](#state.preRequisites))
  `requisites.first_name` | string | имя бронирующего
  `requisites.last_name` | string | фамилия бронирующего
  `requisites.email` | string | email адресс бронирующего
  `requisites.phone` | string | телефон бронирующего
  <a name="f_request_id"></a>[[`request_id`](#request_id)] | string | идентификатор запроса (в случае продолжения бронирования с определенной [стадии](#states))
  <a name="f_next_state"></a>[[`next_state`](#next_state)] | string | указатель следующей [стадии](#states) на которой желательно провести остановку
  <a name="f_accepted"></a>[[`accepted`](#accepted)] | boolean | идентификатор приятия данных по предварительно остановленному [`next_state`](#next_state) [состоянию](#states)

Параметр <a name="accepted"></a>**accepted**  - хитрый параметр. при передаче в теле запроса со значением `true` и дополнительным параметром `request_id` инициирует принятие данных указаных в поле [`booking`](#f_booking) ответа от сервера.

####Response
Результат запроса возвращается в формате `application/json`

Example:
```json
{
    "ok" : 1,
    "data" : {
        "state" : "complete",
        "booking" : {
            "arrival_date" : "2014-09-23",
            "departure_date" : "2014-09-24",
            "blocks" : [
                {
                    "tariff" :  "507f1f77bcf86cd799439011",
                    "total_cost" : 500,
                    "quantity" : 1,
                    "conditions" : {
                        "cancellation" : 3,
                        "booking_method" : 1
                    },
                    "recovery" : {
                        "unit" : "percent",
                        "amount" : 10,
                        "sum" : 50
                    },
                    "living" : [
                        {
                            "first_name" : "Booker",
                            "last_name" : "Da Mega"
                        }
                    ]
                }
            ],
            "requisites" : {
                "first_name" : "Booker",
                "last_name" : "Da Mega",
                "email" : "mega.booker@hotels24.ua",
                "phone" : "0800210017"
            },
            "general_conditions" : {
                "booking_sum" : 500,
                "cancellation" : 3,
                "booking_method" : 3,
                "recoveries" : [
                    {
                        "unit" : "percent",
                        "amount" : 10,
                        "sum" : 50
                    }
                ],
                "recovery_sum" : 50
            },
            "request_id" : "60ef1a77fcf86cd791434021"
        },
        "status" : "ok",
        "status_body" : {}
    },
    "msg" : "ok",
    "code" : 200
}
```

<a name="state_statuses"></a>
#####Response Parameters

Name | Type | Description | Notes
 --- | --- | --- | ---
<a name="f_state"></a>`state` | string | стадия бронирования на которой была завершена обработка запроса (см. [стадии](#states))
`booking` | object | объект с набором текущих свойств бронирования. Соответствует телу полного запроса (включая опциональные параметры для каждого блока) и с дополнительным полем [`general_conditions`](#general_conditions) где указаны общие условия по всем блокам бронирования. В остальном данные такие же как в запросе за исключением управляющих директив ([`next_state`](#next_state),[`accepted`](#accepted))
`status` | string | идентификатор состояния бронирования текущей [`state`](#f_state) см. [Response State Statuses](#state_statuses)
<a name ="f_status_body"></a>`status_body` | object | дополнительная информация о состоянии процесса бронирования
`request_id` | идентификатор запроса для того чтобы повторно не отправлять весь блок данных, а только тот, который необходим для текущей стадии или с параметром запроса accept. Необходим в запросах для стадий начиная с postConditions

<a name="state_statuses"></a>
######Response State Statuses

State | Description
 --- | ---
`"ok"` |  все хорошо. к состоянию пришли успешно
`"error"` |  ошибка уровня приложения. в таких случаях лучше попробовать позже или известить мейнтейнера
`"requirements"` | некоторые критические данные для продолжения процесса отсутствуют
`"rejected"` | ошибка интерполяции данных. Какой-то набор данных фактически (имеющийся в базе) не соответствует представленному в запросе. В этом случае в [`status_body`](#f_status_body) будет присутствовать набор данных которые были отвергнуты со структурой повторяющей структуру запроса. Для сравнения с фактическим набором данных можно использовать свойства поля [`booking`](#f_booking)
`"failure"` | переданы ошибочные данные (форматы, неверные ссылки и т.п.) в поле [`status_body`](#f_status_body) будет передан объект указывающий на некорректные данные с кодом ошибки вместо значений (см. [коды ошибок](#error_codes))

<a name="states"></a>
####Booking States
Name | Description
 --- | ---
<a name="state.preConditions"></a>`"preConditions"` | этап без идентификатора брони. Может быть использован параметром nextState только для проверки доступности брони по переданным данным.
<a name="state.postConditions"></a>`"postConditions"` | демонстрация сложившихся условий бронирования
<a name="state.preRequisites"></a>`"preRequisites"` | ожидание ввода реквизитов
<a name="state.postRequisites"></a>`"postRequisites"` | демонстрация введенных реквизитов для подтверждения ... состояния касающиеся оплат
 `todo payments` | ...
<a name="state.complete"></a>`"complete"` | бронирование завершено

<a name="error_codes"><a>
####Error Codes
Code | Constant Name | Description
 --- | --- | ---
31415 | PI | Все плохо



--------------------------------
(c)(r)(tm) sketch production 2014