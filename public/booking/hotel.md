```
[POST] /booking/hotel
```
>
> [Здесь](https://github.com/hotels24ua/api-docs/issues/1) можно обсудить документацию, задать вопросы, добавить заявки на улучшения в сам метод бронирования, если вы не осмеливаетесь делать pull-request
>


Content
* [Общее описание](#overall)
* [Специальные обозначения](#special.notation)
    * [Параметры](#parameters.desc)
    * [Типы данных]([#types.desc)
* [Запрос бронирования](#request)
    * [Параметры запроса](#request.parameters)
* [Ответы от сервера](#response)
    * [Параметры ответа](#response.parameters)
    * [Response Status body](#response.status.body)
* [Booking Methods (Booking Types)](#bookingMethods)
* [Коды ошибок](#errorCodes)
* [Examples](#examples)



<a name="overall"></a>
##Общее описание
 Метод для бронирования номеров в отеле.
 Бронирование может быть произведено в один запрос (за исключением случаев, где должна производится онлайн оплата).
 Бронирование состоит из нескольких этапов (см. [States](#booking.states)).
 Система может произвести остановку на каком-либо [этапе](#booking.states) в следующих случаях:

  * если не все необходимые данные присутствуют в запросе
  * данные не валидны
  * произошла непредвиденная ошибка
  * клиентская сторона сама инициировала остановку на нужном [этапе](#booking.states) под средством специального указателя [`nextState`](#fNextState)

-------------------------

####Специальные обозначения
<a name="special.notation"></a>
<a name="parameters.desc"></a>
#####Обозначения параметров

Пример                                          | Описание
---                                             | ---
`parameter`                                     | обычный обязательный параметр
`[optionalParameter]`                           | опциональный параметр заключен в кв.скобки (может присутствовать, а может и отсутствовать. Детали поведеня должны быть приложены в описании)
`parameter.property`                            | `parameter` является объектом, а `property` его свойством
`[optionalParameter].property`                  | опциональный параметр `optionalParameter`, но если присутствует, то обязательно должен иметь свойство `property`
`[optionalParameter.property2]`                 | опциональный параметр `optionalParameter`, опционально может иметь свойство `property2`
[`parameterDocRef`](#parameterReference)        | ссылка на параметр в документации
[`parameterDocRef.`](#parameterRef)`property`   | то же самое что и `parameter.property`, только на родительский параметр присутствует ссылка в документации
`parameter[]`                                   | параметр со списком значений (известны так же как `list`, `enum`, `Array`, etc)
`parameter[].itemProperty`                      | параметр является списком объектов с обязательными свойствами `itemProperty`
`parameter[].[itemProperty2]`                   | параметр является списком объектов с опциональными свойствами `itemProperty`

<a name="types.desc"></a>
#####Обозначения типов данных

Обозначение                 | Варианты использования        | Описание
---                         | ---                           | ---
`boolean`                   | `boolean`, `bool`             | `true`, `false`, `1`, `0` , `"yes"`, `"no"`
`string`                    | `string`, `str`               |  Строка
`integer`                   | `int`, `integer`              | Число. Целое
`object`                    | `object`, `obj`, `{}`         | объект с какими-то свойствами
<a name="tDate"></a>`date`  | `date`                        | Строка даты дня. формат: `"yyyy-mm-dd"`
`dateTime`                  | `dateTime`                    | Дата. Дата-время (RFC 3339) `"yyyy-mm-ddTHH:MM:ss"`
`list`                      | `list`, `array`               | список
                            | `list[string]`, `[string]`,   | список состоящий из типов указаных в `[...]` (здесь - строк)
                            | [[`Entity`](#entity)]         | список состоящий из набора [`Entity`](#entity)
[`entity`](#entity)         | [`SomeEntity`](#someEntity)   | тип является какой-то сущностью и полностью соответствует её схеме (+ссылка на документацию)

--------------------------------------
<a name="request"></a>
### Запрос бронирования (Request)
Запрос принимает данные в форматe: `application/x-www-form-urlencoded` 
Единственный параметр запроса это `request` который является сериализированной json строкой. Структура json объекта описана ниже.

<a name="request.parameters"></a>
#### Параметры запроса (Request Parameters)

  Name | Type | Description
  ---                                                           | :---:             | --- | ---
  `arrivalDate`<a name="fDs"></a>                               | [date](#tDate)    | дата заезда
  `departureDate`<a name="fDe"></a>                             | [date](#tDate)    | дата выезда
  `blocks`<a name="fBlocks"></a>                                | list              | список бронируемых объектов (комнат/тарифов). Кол-во номеров с одинаковым тарифом регулируется кол-вом блоков.
  `blocks[].[id]`                                               | string            | любой уникальный id блока. Если указан, то в ответе от сервера в параметре [`booking`](#fBooking) в блоках этот id так же будет присутствовать (не используется системой, может понадобится только для привязки при валидации)
  `blocks[].tariff`                                             | string            |
  `blocks[].[conditions]`                                       | object            | объект с условиями бронирования `cancellation` и `bookingMethod`
  `blocks[].[conditions.cancellation]`                          | integer           | кол-во дней перед днем заезда после которого отмена не возможна
  `blocks[].[conditions.bookingMethod]`                         | integer           | метод бронирования для данного блока (опионален. если присутствует - будет свалидирован с фактическим методом указаным в базе. используется только в случае если для клиента важно где и как платить)
  `blocks[].[recovery]`<a name="fRecovery"></a>                 | object            | предоплата (тот самый "**штраф**"), которая взымается за номер сразу при бронировании
  `blocks[].[recovery].unit`                                    | string            | единица измерения поля `recovery.amount` ("percent" - проценты% или "day" - дни)
  `blocks[].[recovery].amount`                                  | integer           | кол-во ед. предоплаты
  `blocks[].[recovery].sum`                                     | integer           | общая сумма пердоплаты в денежном эквиваленте (указывается за весь период по конкретному блоку)
  `blocks[].[discount].percent`                                 | integer           | % скидки на блок. может быть упущен если скиндки нет или если для запроса валидация скидки не важна. в этом случае реальная скидка будет отображена в блоке в ответе от сервера
  `blocks[].[discount].originPrice`                             | integer           | стоимость проживания в блоке за весь период без скидки
  `blocks[].totalCost`                                          | integer           | стоимость проживания в блоке за весь период
  `blocks[].[living]`                                           | list              | список проживающих в конкретном номере (В случае, если проживающие не указаны, будет выбран тот, который указан в поле [`requisites`](#requisites))
  `blocks[].[living[]].firstName`                               | string            |  имя проживающего в блоке
  `blocks[].[living[].lastName]`                                | string            | фамилия проживающего в блоке
  `requisites`<a name="fReq"></a>                               | object            | объект реквизитов бронирующего клиента (обязателен для завершения стадии [`"requisites"`](#state.requisites))
  `requisites.firstName`                                        | string            | имя бронирующего
  `requisites.lastName`                                         | string            | фамилия бронирующего
  `requisites.email`                                            | string            | email адресс бронирующего
  `requisites.phone`                                            | string            | телефон бронирующего
  [[`requestId`](#rsfRequestId)]<a name="rqfRequestId"></a>     | string            | идентификатор запроса (в случае продолжения бронирования с определенной [стадии](#booking.states)), который был получен из поля ответа [`requestId`](#rsfRequestId)
  [[`nextState`](#fNextState)]<a name="fNextState"></a>         | string            | указатель следующей [стадии](#booking.states) на которой желательно провести остановку
  [[`continue`](#request.continue)]<a name="fContinue"></a>     | mixed             | идентификатор приятия данных по предварительно остановленному [`nextState`](#fNextState) [состоянию](#booking.states)
  `[paymentId]`<a name="fPaymentId"></a>                        | string            | идентификатор оплаты

<a name="response"></a>
###Ответы (Responses)
Результат запроса возвращается в формате `application/json`

<a name="response.statuses"></a>
####Response Parameters
[response example](#responseExample1) - пример ответа

<a name="response.parameters"></a>
##### Параметры ответа
Name                                                                    | Type                      | Description
 ---                                                                    | ---                       | ---
<a name="fState"></a>`state.type`                                       | string                    | стадия бронирования на которой была завершена обработка запроса (см. [стадии](#booking.states))
<a name="fStatus"></a>`state.status`                                    | string                    | идентификатор состояния бронирования текущей [`state.type`](#fState) см. [Response State Statuses](#response.statuses)
<a name="fBooking"></a>`booking`                                        | object                    | объект с набором текущих свойств бронирования. Соответствует телу полного запроса (включая опциональные параметры для каждого блока) и с дополнительными полями `conditions` где указаны общие условия по всем блокам бронирования. В остальном данные такие же как в запросе за исключением управляющих директив ([`nextState`](#fNextState),[`continue`](#continue))
`booking.arrivalDate`                                                   | ([same](#fDs))           | -//-
`booking.departureDate`                                                 | ([same](#fDe))           | -//-
`booking.hotel`                                                         | integer                   | id бронируемого отеля
`booking.blocks[]`                                                      | ([same](#fBlocks))       | в все поля в блоках, в том числе `conditions` и `recovery` будут присутствовать обязательно и отображать реальные данные в базе
`booking.requisites`                                                    | ([same](#fReq))          | присутствуют только в случае если были переданы ранее
`booking.conditions`                                                    | object                    | общая калькуляция условий бронирования см (общие условия бронирования)[#fGeneralConditions]
`booking.conditions.bookingSum`                                         | integer                   | общая сумма по бронированию по всем блокам
`booking.conditions.cancellation`                                       | integer                   | итоговый срок отмены
`booking.conditions.bookingMethod`                                      | integer                  | Тип (метод) бронирования, который был установлен для текущего заказа/брони (один из [методов бронирования](#bookingMethods))
`booking.conditions.selfConfirm`                                        | boolean | Флаг автоподтверждения. Подтверждает ли отель автоматически бронирование или бронирование будет обработано оператором `true` или `false`
`booking.conditions.[recoveries[]]`                                     | [[recovery](#fRecovery)] | список всех предоплат со всех номеров (структура повторяет [`recovery`](#fRecovery)). Может отсутствовать, если бронирование не предпологает предоплаты в этот отель
`booking.conditions.[recoverySum]`                                      | integer                   | общая сумма по предоплатам. так же может отсутствовать если бронирование не требует предоплат
`booking.conditions.[paymentMethods[]]`<a name="payment.resources"></a> | list                      | список объектов - ресурсов (типов) оплат возможных для данного бронирования. если бронирование не требует предоплат, то этот элемент будет отсутствовать
`booking.conditions.[paymentMethods[]].id`                              | string                    | идентификатор ресурса (используется для проверки состояния оплаты) уникален для каждого бронирования
`booking.conditions.[paymentMethods[]].type`<a name="pType"></a>        | integer                   | константа типа оплаты. см [Константы Типов Оплат](#const.paymentTypes)
`booking.conditions.[paymentMethods[]].rel`                             | string                    | абсолютный URL адресс для проведения оплаты
`booking.conditions.[paymentMethods[]].relType`                         | string                    | тип ресурса может быть `iframe`, `link`, `file`, `html`
`booking.conditions.[paymentMethods[]].title`<a name="b.c.pm.t"></a>                           | string                    | человекопонятное описание способа оплаты напр. `"Оплата по счету-фактуре"`
`bookingId`                                                             | integer                   | ID бронирования. В случае успешного завершения. Может присутствовать начиная с [`requisites`](#state.requisites) стадии
`statusBody`<a name="fStatusBody"></a>                                  | object                    | дополнительная информация о состоянии процесса бронирования
`requestId`<a name="rsfRequestId"></a>                                  | string                    | идентификатор запроса для того чтобы повторно не отправлять весь блок данных, а только тот, который необходим для текущей стадии или с параметром запроса accept. Необходим в запросах для стадий начиная с `conditions`



<a name="booking.states"></a>
##### Стадии бронирования (States)

Name                                            | Description
 ---                                            | ---
`"conditions"`<a name="state.conditions"></a>   | этап без идентификатора брони. Может быть использован параметром [`nextState`](#fNextState) только для проверки доступности брони по переданным данным.
`"requisites"`<a name="state.requisites"></a>   | ожидание ввода реквизитов
`"payment"`<a name="state.payment"></a>         | перед оплатой. отсутствовует если бронирование предполагает расчет в отеле
`"complete"`<a name="state.complete"></a>       | бронирование завершено

<a name="response.statuses"></a>
##### Response State Statuses
State Status                                        | Description
 ---                                                | ---
`"requirements"`<a name="status.requirements"></a>  | некоторые критические данные для продолжения процесса отсутствуют
`"rejection"`<a name="status.rejected"></a>         | ошибка интерполяции данных. Какой-то набор данных фактически (имеющийся в базе) не соответствует представленному в запросе. В этом случае в [`statusBody`](#fStatusBody) будет присутствовать набор данных которые были отвергнуты со структурой повторяющей структуру запроса. Для сравнения с фактическим набором данных можно использовать свойства поля [`booking`](#fBooking)
`"failure"`<a name="status.failure"></a>            | переданы ошибочные данные (форматы, неверные ссылки и т.п.) в поле [`statusBody`](#fStatusBody) будет передан объект указывающий на некорректные данные с кодом ошибки вместо значений (см. [коды ошибок](#errorCodes))
`"error"`<a name="status.error"></a>                | ошибка уровня приложения. в таких случаях лучше попробовать позже или известить мейнтейнера
`"ok"`<a name="status.ok"></a>                      | все хорошо. к состоянию пришли успешно

<a name="response.status.body"></a>
##### Response Status Body
 Поле [`statusBody`](#fStatusBody) в ответе предназначено для дополнительной информации о бронировании на определенных стадиях или состояниях.
В случае с статусами ([status](#response.status)) отличными от [`ok`](#status.ok) обычно отдаются причины ошибок, поля с ошибками или пустое поле.
В случае со стадией [`complete`](#state.complete) и статусом [`ok`](#status.ok) - будет передана дополнительная информация о бронировании от провайдера (сервиса, который занимается непосредственно бронированием).
###### CRM PROVIDER BOOKING STATUS BODY
Field               |   Description
---                 |   ---
`callStatus`        | состояние обращения (`open`, `closed`, `inProgress`, `none`, `new`)     
`bookingStatus`     | `bookingSubmit` \| `notConfirmOral` \|  `cancelOral` \| `invoiceSend` \| `clientPay` \| `notConfirmed` \| `cancel` \| `waitPayment` (так же возможны другие статусы. но они здесь не используются)
`hasBills`          | есть ли выставленные счета в текущем бронировании (boolaean)
`paymentType`       | `invoice` \| `cardPay` \| `cardTransfer` \| `interkassa` \| `unknown`
`hasClientPayments` | boolean
`invoiceAvailable`  | boolean   
`invoiceHtmlUrl`    | url string
`voucherAvailabl`   | boolean
`voucherHtmlUrl`    | url string

### Процесс бронирования (Request Flow)
#### Опциональные условия бронирования и ошибки при запросе
<a name="optionalConditions"></a>**[`optionalCondition`]** - параметры, как опциональное условие бронирования.
Помечаются условия бронирования и другие параметры, которые могут быть проигнорированы в запросе если клиенту этот параметр не важен.
Если параметр предоставлен в запросе, но он не совпадает с фактическим параметром в базе - запрос будет возвращен обратно со [статусом](#statuses) [`"rejected"`](#status.rejected).
В случае с [`"rejected"`](#status.rejected) будет указаны:
 * ошибки в поле ответа [`statusBody`](#fStatusBody),
 * стадия в поле [`state.type`](#fState) на которой была произведена остановка с соответствующим статусом (см. [статусы стадий](#statuses).

Этот функционал существует для того, чтобы клиентская сторона могла подтвердить изменения в условиях или произвести другие действия по корректировке запроса.
Если же эти опицональне условия упущены система произведет автоматическое принятие параметров на основе существующих в базе.

####Специальные параметры
<a name="request.continue"></a>
#####Parameter `continue`
Поле [`continue`](#fContinue) используется для продолжения бронирования с определенной стадии.

**Пример 1**:  приложению нужно продемонстрировать всю информацию по оформляемому бронированию клиенту и провести своеобразное подтверждение бронирования.
В этом случае приложение инициирует остановку процесса на стадиях [`conditions`](#state.conditions) или [`requisites`](#state.requisites),
Тогда, помимо других необходимых для бронирования данных, в запросе отправляется еще идентификатор остановки [`nextState`](#fNextState) с указаной желаемой [стадией](#booking.states).
Если данные валидны процес остановится на указаной стадии и будет возвращен результат с полями:
 * [`state.type`](#fState): "запрашиваемая стадия",
 * [`state.status`](#fStatus): ["ok"](#status.ok)
 * [`booking`](#fBooking) : предварительная калькуляция, которая может быть продемонстрирована клиенту.
 * [`requestId`](#rsfRequestId)<a name=""></a> : сгенерированный идентификатор текущего запроса бронирования.

Далее приложение подтверждает бронирование. Для этого используется всего два поля в запросе: [`requestId`](#rfRequestId) и [`continue`](#fContinue) со значением `true`


**Пример 2**: когда система вывела требования по определенному этапу.
Здесь в ответе от сервера будет указана [стадия](#fState) со статусом ([`status`](#fStatus)) равным [`"requirements"`](#status.requirements).
К примеру, все стадии по бронированию (условия и реквизиты) были пройдены, но для завершения требуется произвести оплату.

В этом случае [`state.type`](#fState) будет равен [`"payment"`](#state.payment) и в теле поля [`booking`](#fBooking) будет присутствовать секция `paymentMethods` со списком возможных способов оплаты ([PaymentResource][#payment.resources]).
В каждом из способов оплаты, помимо [типа](#pType) есть идентификатор (`paymentId`)[#paymentId].
Тоесть ваше приложение выбирает из списка какие типы оплат оно поддерживает, дает (или не дает) клиенту выбор из этих типов оплат. Производит оплату, а затем делает повторный запрос с параметрами
* [`continue`](#fContinue): true
* [`paymentId`](#fPaymentId): <paymentId> id типа оплаты, которая была произведена
* [`requestId`](#rsfRequestId): <requestId> id нашего процесса бронирования



#### Константы
<a name="const.paymentTypes"></a>

##### Константы Типов Оплат
 NAME                   | Value | Description
 ---                    | ---   | ---
 PAY_INVOICE_HTML       | 810   | Счет фактура. Ссылка на её html
 PAY_INVOICE_POST       | 812   | Счет фактура. Ссылка на action постом. Дополнительно принимается redirect аргумент (куда перенаправлять после успешного поста).
 PAY_CARD_URL           | 844   | ...
 PAY_CARD_IFRAME        | 843   | ...
 PAY_SEND_CARD_IFRAME   | 853   | ...
 PAY_SEND_CARD_URL      | 854   | ...
 PAY_SEND_CARD_POST     | 852   | ...


##### Коды ошибок (Error Codes)
<a name="errorCodes"><a>

Code    | Constant Name                         | Description
 ---    | ---                                   | ---
31415   | PI                                    | Все плохо. На все случаи жизни.
1401    | FIELD_COULD_NOT_BE_EMPTY              | -//-
14015   | FIELD_COULD_BE_OMITTED_OR_FULFILLED   | optional fields with non optional properties must be omitted or fulfilled with its requirements
1406    | FIELD_INVALID_FORMAT                  | -//-
1403    | FIELD_DIFFERS_FROM_ORIGIN             | rejection error. has to have origin property in response=>booking
1404    | FIELD_CONTAINS_MISSED_REFERENCE       | failure error. reference which represents this field is not exists

####Booking Types (Booking Methods)
<a name="bookingMethods"></a>

Constant    | Constant Name | Description
 ---        | ---           | ---
0           | INVOICE       | Бронирование по предоплате
3           | NO_GUARANTEES | Бронирование без финансовой гарантии (оплата в отеле)
4           | SEND_CARD     | Передача данных кридитной карточки в отель


####Examples
<a name="examples"></a>


#####Case 1
######Request
```JSON
{
    "arrivalDate": "2014-11-25",
    "departureDate": "2014-11-26",
    "blocks": [
        {
            "id": "1",
            "tariff": "5425476b6b06784d498b4567",
            "totalCost": 400,
            "living": [],
            "conditions": {"cancellation": 1001, "bookingMethod": 3}
        },
        {
            "id": "2",
            "tariff": "5425476b6b06784d498b4567",
            "totalCost": 400,
            "living": [],
            "conditions": {"cancellation": 1001, "bookingMethod": 3}
        }
    ],
    "requisites": {
        "fistName": "TestFirstName",
        "lastName": "TestLastName",
        "email": "test.email@hotels24.ua",
        "phone": "+3805555555"
    }
}
```
######Response OK
<a name="responseExample1"></a>

```JSON
{
   "state": { 
      "type": "complete",
      "status": "ok"
   },
   "booking": {
         "conditions": {"bookingSum": 800, "cancellation": 1001, "bookingMethod": 3},
         "arrivalDate": "2014-11-25",
         "departureDate": "2014-11-26",
         "blocks": [
            {
                "id": "1",
                "tariff": "5425476b6b06784d498b4567",
                "totalCost": 400,
                "living": [],
                "conditions": {"cancellation": 1001, "bookingMethod": 3}
            },
            {
                "id": "2",
                "tariff": "5425476b6b06784d498b4567",
                "totalCost": 400,
                "living": [],
                "conditions": {"cancellation": 1001, "bookingMethod": 3}
            }
        ],
        "bookingId": 654654,
        "requisites": {
            "fistName": "TestFirstName",
            "lastName": "TestLastName",
            "email": "test.email@hotels24.ua",
            "phone": "+3805555555"
        }
    },
    "requestId": "5425476b6b06784d498b456a"
}

```





--------------------------------
(c)(r)(tm) sketch production 2014
