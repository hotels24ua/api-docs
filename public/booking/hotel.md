```
[POST] /booking/hotel
```
>
> [Здесь](https://github.com/hotels24ua/api-docs/issues/1) можно обсудить документацию, задать вопросы, добавить заявки на улучшения в сам метод бронирования, если вы не осмеливаетесь делать pull-request
>


Content
* Общее описание
* Специальные обозначения
    * Параметры
    * Типы данных
* Запрос бронирования
    * Параметры запроса
* Ответы от сервера
 * Поля ответов
 *



##Общее описание
 Метод для бронирования номеров в отеле.
 Бронирование может быть произведено в один запрос (за исключением случаев, где должна производится онлайн оплата).
 Бронирование состоит из нескольких этапов (см. [States](#booking.states)).
 Система может произвести остановку на каком-либо [этапе](#booking.states) в следующих случаях:

  * если не все необходимые данные присутствуют в запросе
  * данные не валидны
  * произошла непредвиденная ошибка
  * клиентская сторона сама инициировала остановку на нужном [этапе](#booking.states) под средством специального указателя [`next_state`](#f_next_state)

-------------------------

####Специальные обозначения
#####Обозначения параметров
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

#####Обозначения типов данных
Обозначение                 | Варианты использования        | Описание
---                         | ---                           | ---
`boolean`                   | `boolean`, `bool`             | `true`, `false`, `1`, `0` , `"yes"`, `"no"`
`string`                    | `string`, `str`               |  Строка
`integer`                   | `int`, `integer`              | Число. Целое
`object`                    | `object`, `obj`, `{}`         | объект с какими-то свойствами
<a name="t_date"></a>`date` | `date`                        | Строка даты дня. формат: `"yyyy-mm-dd"`
`dateTime`                  | `dateTime`                    | Дата. Дата-время (RFC 3339) `"yyyy-mm-ddTHH:MM:ss"`
`list`                      | `list`, `array`               | список
                            | `list[string]`, `[string]`,   | список состоящий из типов указаных в `[...]` (здесь - строк)
                            | [[`Entity`](#entity)]         | список состоящий из набора [`Entity`](#entity)
[`entity`](#entity)         | [`SomeEntity`](#someEntity)   | тип является какой-то сущностью и полностью соответствует её схеме (+ссылка на документацию)

--------------------------------------

### Запрос бронирования (Request)
Запрос принимает данные в форматах:
* `application/json` - json encoded post body
* `application/x-www-form-urlencoded` - url encoded

#### Параметры запроса (Request Parameters)

  Name | Type | Description
  ---                                                           | :---:             | --- | ---
  `arrival_date`<a name="f_ds"></a>                             | [date](#t_date)   | дата заезда
  `departure_date`<a name="f_de"></a>                           | [date](#t_date)   | дата выезда
  `blocks`<a name="f_blocks"></a>                               | list              | список бронируемых объектов (комнат/тарифов). Кол-во номеров с одинаковым тарифом регулируется кол-вом блоков.
  `blocks[].[id]`                                               | scalar            | любой уникальный id блока. Если указан, то в ответе от сервера в параметре [`booking`](#f_booking) в блоках этот id так же будет присутствовать (не используется системой, может понадобится только для привязки при валидации)
  `blocks[].tariff`                                             | string            |
  `blocks[].[conditions]`                                       | object            | объект с условиями бронирования `cancellation` и `booking_method`
  `blocks[].[conditions.cancellation]`                          | integer           | кол-во дней перед днем заезда после которого отмена не возможна
  `blocks[].[conditions.booking_method]`                        | integer           | метод бронирования для данного блока (опионален. если присутствует - будет свалидирован с фактическим методом указаным в базе. используется только в случае если для клиента важно где и как платить)
  `blocks[].[recovery]`<a name="f_recovery"></a>                | object            | предоплата (тот самый "**штраф**"), которая взымается за номер сразу при бронировании
  `blocks[].[recovery].unit`                                    | string            | единица измерения поля `recovery.amount` ("percent" - проценты% или "day" - дни)
  `blocks[].[recovery].amount`                                  | integer           | кол-во ед. предоплаты
  `blocks[].[recovery].sum`                                     | integer           | общая сумма пердоплаты в денежном эквиваленте (указывается за весь период по конкретному блоку)
  `blocks[].total_cost`                                         | integer           | стоимость проживания в блоке за весь период
  `blocks[].[living]`                                           | list              | список проживающих в конкретном номере (В случае, если проживающие не указаны, будет выбран тот, который указан в поле [`requisites`](#requisites))
  `blocks[].[living[]].first_name`                              | string            |  имя проживающего в блоке
  `blocks[].[living[].last_name]`                               | string            | фамилия проживающего в блоке
  `requisites`<a name="f_req"></a>                              | object            | объект реквизитов бронирующего клиента (обязателен для завершения стадии [`"preRequisites"`](#state.preRequisites))
  `requisites.first_name`                                       | string            | имя бронирующего
  `requisites.last_name`                                        | string            | фамилия бронирующего
  `requisites.email`                                            | string            | email адресс бронирующего
  `requisites.phone`                                            | string            | телефон бронирующего
  [[`request_id`](#request_id)]<a name="rqf_request_id"></a>    | string            | идентификатор запроса (в случае продолжения бронирования с определенной [стадии](#booking.states)), который был получен из поля ответа [`request_id`](#rsf_request_id)
  [[`next_state`](#f_next_state)]<a name="f_next_state"></a>    | string            | указатель следующей [стадии](#booking.states) на которой желательно провести остановку
  [[`accepted`](#accepted)]<a name="f_accepted"></a>            | mixed             | идентификатор приятия данных по предварительно остановленному [`next_state`](#f_next_state) [состоянию](#booking.states)
  `[payment_id]`<a name="f_payment_id"></a>                     | string            | идентификатор оплаты

###Ответы (Responses)
Результат запроса возвращается в формате `application/json`

<a name="response.statuses"></a>
####Response Parameters

[response example](#response_example1) - пример ответа

<a name="response.parameters"></a>
##### Параметры ответа
Name                                                | Type                      | Description
 ---                                                | ---                       | ---
<a name="f_state"></a>`state`                       | string                    | стадия бронирования на которой была завершена обработка запроса (см. [стадии](#booking.states))
<a name="f_status"></a>`status`                     | string                    | идентификатор состояния бронирования текущей [`state`](#f_state) см. [Response State Statuses](#response.statuses)
<a name="f_booking"></a>`booking`                   | object                    | объект с набором текущих свойств бронирования. Соответствует телу полного запроса (включая опциональные параметры для каждого блока) и с дополнительными полями `conditions` где указаны общие условия по всем блокам бронирования. В остальном данные такие же как в запросе за исключением управляющих директив ([`next_state`](#f_next_state),[`accepted`](#accepted))
```booking.arrival_date```                              | ([same](#f_ds))           | -//-
```booking.departure_date```                            | ([same](#f_de))           | -//-
<pre>booking.blocks[]</pre>                                  | ([same](#f_blocks))       | в все поля в блоках, в том числе `conditions` и `recovery` будут присутствовать обязательно и отображать реальные данные в базе
<pre>booking.requisites</pre>                                | ([same](#f_req))          | присутствуют только в случае если были переданы ранее
<pre>booking.conditions</pre>                                | object                    | общая калькуляция условий бронирования см (общие условия бронирования)[#f_general_conditions]
<pre>booking.conditions.booking_sum</pre>                    | integer                               | общая сумма по бронированию по всем блокам
<pre>booking.conditions.cancellation</pre>                   | integer                               | итоговый срок отмены
<pre>booking.conditions.[recoveries[]]<pre>                 | [[recovery](#f_recovery)]             | список всех предоплат со всех номеров каждый (структура повторяет [`recovery`](#f_recovery))
<pre>booking.conditions.[recovery_sum]</pre>      | integer                               |
<pre>booking.conditions.[payment_methods[]]</pre>            | list                                  |
<pre>booking.conditions.[payment_methods[]].id</pre>         | string                                | идентификатор ресурса (используется для проверки состояния оплаты) уникален для каждого бронирования
<pre>booking.conditions.[payment_methods[]].type</pre>       | integer                               | константа типа оплаты. см [Константы Типов Оплат](#const.payment_types)
<pre>booking.conditions.[payment_methods[]].rel</pre>        | string                                | абсолютный URL адресс для проведения оплаты
<pre>booking.conditions.[payment_methods[]].rel_type</pre>   | string                                | тип ресурса может быть `iframe`, `link`, `file`, `html`
<pre>booking.conditions.[payment_methods[]].title</pre>      | string                                | человекопонятное описание способа оплаты напр. `"Оплата по счету-фактуре"`
<a name="f_status_body"></a>`status_body`           | object                                | дополнительная информация о состоянии процесса бронирования
<a name="rsf_request_id"></a>`request_id`           | string                                | идентификатор запроса для того чтобы повторно не отправлять весь блок данных, а только тот, который необходим для текущей стадии или с параметром запроса accept. Необходим в запросах для стадий начиная с postConditions



<a name="booking.states"></a>
##### Стадии бронирования (States)
Name                                                    | Description
: ---                                                   | ---
`"preConditions"`<a name="state.preConditions"></a>     | этап без идентификатора брони. Может быть использован параметром [`next_state`](#f_next_state) только для проверки доступности брони по переданным данным.
`"postConditions"`<a name="state.postConditions"></a>   | демонстрация сложившихся условий бронирования
`"preRequisites"`<a name="state.preRequisites"></a>     | ожидание ввода реквизитов
`"postRequisites"`<a name="state.postRequisites"></a>   | демонстрация введенных реквизитов для подтверждения ... состояния касающиеся оплат
[`"prePayment"`]<a name="state.prePayment"></a>         | перед оплатой
[`"postPayment"`]<a name="state.postPayment"></a>       | после оплаты
`"complete"`<a name="state.complete"></a>               | бронирование завершено

<a name="response.statuses"></a>
##### Response State Statuses
State                                               | Description
 ---                                                | ---
`"ok"`<a name="status.ok"></a>                      | все хорошо. к состоянию пришли успешно
`"requirements"`<a name="status.requirements"></a>  | некоторые критические данные для продолжения процесса отсутствуют
`"rejected"`<a name="status.rejected"></a>          | ошибка интерполяции данных. Какой-то набор данных фактически (имеющийся в базе) не соответствует представленному в запросе. В этом случае в [`status_body`](#f_status_body) будет присутствовать набор данных которые были отвергнуты со структурой повторяющей структуру запроса. Для сравнения с фактическим набором данных можно использовать свойства поля [`booking`](#f_booking)
`"error"`<a name="status.error"></a>                | ошибка уровня приложения. в таких случаях лучше попробовать позже или известить мейнтейнера
`"failure"`<a name="status.failure"></a>            | переданы ошибочные данные (форматы, неверные ссылки и т.п.) в поле [`status_body`](#f_status_body) будет передан объект указывающий на некорректные данные с кодом ошибки вместо значений (см. [коды ошибок](#error_codes))

##### Booking Field
Field                   | Type                              |  Description
---                     | ---                               | ---


##### Общие условия бронирования
Field                   | Type
---     |   ---


##### Ресурсы (Типы) оплат (Payment Methods Resources)
JSON объект с спецификацией ресурса оплаты
Field       | Description
---         | ---
`id`        | идентификатор ресурса (используется для проверки состояния оплаты) уникален для каждого бронирования
`type`      | константа типа оплаты. см [Константы Типов Оплат](#const.payment_types)
`rel`       | абсолютный URL адресс для проведения оплаты
`rel_type`  | тип ресурса может быть `iframe`, `link`, `file`, `html`
`title`     | человекопонятное описание способа оплаты напр. `"Оплата по счету-фактуре"`



### Процесс бронирования (Request Flow)
#### Опциональные условия бронирования и ошибки при запросе
<a name="optional_conditions"></a>**[`optional_condition`]** - параметры, как опциональное условие бронирования.
Помечаются условия бронирования и другие параметры, которые могут быть проигнорированы в запросе если клиенту этот параметр не важен.
Если параметр предоставлен в запросе, но он не совпадает с фактическим параметром в базе - запрос будет возвращен обратно со [статусом](#statuses) [`"rejected"`](#status.rejected).
В случае с [`"rejected"`](#status.rejected) будет указаны:
 * ошибки в поле ответа [`status_body`](#f_status_body),
 * стадия в поле [`state`](#f_state) на которой была произведена остановка с соответствующим статусом (см. [статусы стадий](#statuses).

Этот функционал существует для того, чтобы клиентская сторона могла подтвердить изменения в условиях или произвести другие действия по корректировке запроса.
Если же эти опицональне условия упущены система произведет автоматическое принятие параметров на основе существующих в базе.

####Специальные параметры
<a name="request.accepted"></a>
#####Parameter `accepted`
Поле [`accepted`](#f_accepted) используется для продолжения бронирования с определенной стадии.

**Пример 1**:  приложению нужно продемонстрировать всю информацию по оформляемому бронированию клиенту и провести своеобразное подтверждение бронирования.
В этом случае приложение инициирует остановку процесса на стадиях [`postConditions`](#state.postConditions) или [`postRequisites`](#state.postRequisites),
Тогда, помимо других необходимых для бронирования данных, в запросе отправляется еще идентификатор остановки [`next_state`](#f_next_state) с указаной желаемой [стадией](#booking.states).
Если данные валидны процес остановится на указаной стадии и будет возвращен результат с полями:
 * [`state`](#f_state): "запрашиваемая стадия",
 * [`status`](#f_status): ["ok"](#status.ok)
 * [`booking`](#f_booking) : предварительная калькуляция, которая может быть продемонстрирована клиенту.
 * [`request_id`](#rsf_request_id) : сгенерированный идентификатор текущего запроса бронирования.

Далее приложение подтверждает бронирование. Для этого используется всего два поля в запросе: [`request_id`](#rf_request_id) и [`accepted`](#f_accepted) со значением `true`


**Пример 2**: когда система вывела требования по определенному этапу.
Здесь в ответе от сервера будет указана [стадия](#f_state) со статусом ([`status`](#f_status)) равным [`"requirements"`](#status.requirements).
К примеру, все стадии по бронированию (условия и реквизиты) были пройдены, но для завершения требуется произвести оплату.

В этом случае [`state`](#f_state) будет равен [`"prePayment"`](#state.prePayment) и в теле поля [`booking`](#f_booking) будет присутствовать секция `payment_methods` со списком возможных способов оплаты ([PaymentResource][#payment.resources]).
В каждом из способов оплаты, помимо [типа](#payment_types) есть идентификатор (`payment_id`)[#payment_id].
Тоесть приложение решает какие типы оплат оно поддерживает, дает (или не дает) клиенту выбор из этих типов оплат. Производит оплату, а затем делает повторный запрос с параметрами
* [`accepted`](#f_accepted): true
* [`payment_id`](#f_payment_id): <payment_id> id типа оплаты, которая была произведена
* [`request_id`](#rsf_request_id): <request_id> id нашего процесса бронирования


#### Константы
<a name="const.payment_types"></a>
##### Константы Типов Оплат
 NAME                   | Value | Description
 ---                    | ---   | ---
 PAY_INVOICE_HTML       | 810   | ...
 PAY_CARD_URL           | 844   | ...
 PAY_CARD_IFRAME        | 843   | ...
 PAY_SEND_CARD_IFRAME   | 853   | ...
 PAY_SEND_CARD_URL      | 854   | ...
 PAY_SEND_CARD_POST     | 852   | ...


##### Коды ошибок (Error Codes)
Code    | Constant Name | Description
 ---    | ---           | ---
31415   | PI            | Все плохо



<a name="error_codes"><a>

####Examples

#####Response Examples

######Response OK
<a name="response_example1"></a>
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
        },
        "status" : "ok",
        "status_body" : {},
        "request_id" : "60ef1a77fcf86cd791434021"
    },
    "msg" : "ok",
    "code" : 200
}
```





--------------------------------
(c)(r)(tm) sketch production 2014
