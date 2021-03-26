---
title: Настройка условий соответствия для набора правил "Стандартный" или "Премиум" для передней дверцы Azure
description: В этой статье приводится список различных условий соответствия, доступных в наборе правил "Стандартный" или "Премиум" для передней дверцы Azure.
services: frontdoor
author: duongau
ms.service: frontdoor
ms.topic: conceptual
ms.date: 03/24/2021
ms.author: yuajia
ms.openlocfilehash: 039effb885463c1c53085535a6980601be890340
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105561513"
---
# <a name="azure-front-door-standardpremium-preview-rule-set-match-conditions"></a>Условия соответствия набора правил "Стандартный" или "Премиум" (Предварительная версия) для передней дверцы Azure

> [!Note]
> Эта документация предназначена для Azure Front Door категории "Стандартный" или "Премиум" (предварительная версия). Сведения об Azure Front Door см. [здесь](../front-door-overview.md).

В [наборе правил](concept-rule-set.md)"Стандартный" или "Премиум" передней дверцы Azure правило состоит из одного или нескольких условий соответствия и действия. В этой статье приводятся подробные описания условий соответствия, которые можно использовать в наборе правил "Стандартный" или "Премиум" передней дверцы Azure.

Первая часть правила — это одно или несколько условий соответствия. Правило может содержать до 10 условий соответствия. Условие соответствия определяет конкретные типы запросов, для которых выполняются определенные действия. При использовании нескольких условий соответствия они группируются с помощью логического компонента AND. Для всех условий соответствия, поддерживающих несколько значений, или используется логика.

Условие соответствия можно использовать для:

* Отфильтровать запросы на основе определенного IP-адреса, страны или региона.
* Отфильтровать запросы по информации из заголовка.
* Отфильтровать запросы с мобильных или классических устройств.
* Фильтрация запросов по имени файла запроса и расширению файла.
* Фильтрация запросов из URL-адреса запроса, протокола, пути, строки запроса, аргументов POST и т. д.

> [!IMPORTANT]
> Azure Front Door уровня "Стандартный" или "Премиум" (предварительная версия) в настоящее время доступен в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="device-type"></a><a name="IsDevice"></a> Тип устройства

Используйте условие соответствия " **тип устройства** " для обнаружения запросов, выполненных с мобильного устройства или с настольного устройства.  

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-------|------------------|
| Оператор | <ul><li>В портал Azure: `Equal` , `Not Equal`</li><li>В шаблонах ARM: `Equal` ; Используйте `negateCondition` свойство, чтобы указать _не равно_ |
| Значение | `Mobile`, `Desktop` |

### <a name="example"></a>Например, .

В этом примере мы сопоставлены все запросы, которые были обнаружены на мобильном устройстве как поступающие.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/device-type.png" alt-text="Снимок экрана портала с условием соответствия типа устройства.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "IsDevice",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "Mobile"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleIsDeviceConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'IsDevice'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'Mobile'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleIsDeviceConditionParameters'
  }
}
```

---

## <a name="post-args"></a><a name="PostArgs"></a> Аргументы POST

Используйте условие совпадения **аргументов POST** для распознавания запросов на основе аргументов, указанных в тексте запроса POST. Одно условие соответствия сопоставляет один аргумент из тела запроса POST. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

> [!NOTE]
> Условие соответствия " **POST args** " работает с `application/x-www-form-urlencoded` типом содержимого.

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Аргументы POST | Строковое значение, представляющее имя аргумента POST. |
| Оператор | Любой оператор из [списка стандартных операторов](#operator-list). |
| Значение | Одно или несколько строковых или целочисленных значений, представляющих значение аргумента POST для сопоставления. Если указано несколько значений, они оцениваются с помощью логики или. |
| Преобразование регистра | `Lowercase`, `Uppercase` |

### <a name="example"></a>Например, .

В этом примере мы сопоставлены все запросы POST `customerName` , где аргумент указан в тексте запроса и где значение `customerName` начинается с буквы `J` или `K` . Мы используем преобразование регистра для преобразования входных значений в верхний регистр, чтобы значения, `J` `j` , `K` и `k` были согласованы.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/post-args.png" alt-text="Снимок экрана портала с условием соответствия &quot;POST args&quot;.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "PostArgs",
  "parameters": {
    "selector": "customerName",
    "operator": "BeginsWith",
    "negateCondition": false,
    "matchValues": [
        "J",
        "K"
    ],
    "transforms": [
        "Uppercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRulePostArgsConditionParameters"
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'PostArgs'
  parameters: {
    selector: 'customerName'
    operator: 'BeginsWith'
    negateCondition: false
    matchValues: [
      'J'
      'K'
    ]
    transforms: [
      'Uppercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRulePostArgsConditionParameters'
  }
}
```

---

## <a name="query-string"></a><a name="QueryString"></a> Строка запроса

Используйте условие соответствия **строки запроса** для обнаружения запросов, содержащих определенную строку запроса. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

> [!NOTE]
> Вся строка запроса сопоставляется как одна строка без начального символа `?` .

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | Любой оператор из [списка стандартных операторов](#operator-list). |
| Строка запроса | Одно или несколько строковых или целочисленных значений, представляющих собой значение строки запроса для сопоставления. Не включайте в `?` начало строки запроса. Если указано несколько значений, они оцениваются с помощью логики или. |
| Преобразование регистра | `Lowercase`, `Uppercase` |

### <a name="example"></a>Например, .

В этом примере мы будем сопоставлять все запросы, в которых строка запроса содержит строку `language=en-US` . Мы хотим, чтобы условие соответствия было чувствительно к регистру, поэтому мы не преобразуем регистр.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/query-string.png" alt-text="Снимок экрана портала с условием соответствия строки запроса.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "QueryString",
  "parameters": {
    "operator": "Contains",
    "negateCondition": false,
    "matchValues": [
      "language=en-US"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleQueryStringConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'QueryString'
  parameters: {
    operator: 'Contains'
    negateCondition: false
    matchValues: [
      'language=en-US'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleQueryStringConditionParameters'
  }
}
```

---

## <a name="remote-address"></a><a name="RemoteAddress"></a> Удаленный адрес

Условие сопоставления **удаленных адресов** определяет запросы на основе расположения или IP-адреса запрашивающей стороны. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

* Используйте нотацию CIDR при указании блоков IP-адресов. Это означает, что синтаксис блока IP-адресов — это базовый IP-адрес, за которым следует косая черта и размер префикса. Например:
    * **Пример IPv4**. `5.5.5.64/26` соответствует всем запросам, поступающим из адресов 5.5.5.64 через 5.5.5.127.
    * **Пример IPv6**. `1:2:3:/48` соответствует всем запросам, поступающим с адресов 1:2:3:0:0:0:0:0 по 1:2:3: FFFF: FFFF: FFFF: FFFF: FFFF.
* При указании нескольких IP-адресов и блоков IP-адресов применяется логика "или".
    * **Пример IPv4**. Если добавить два IP-адреса `1.2.3.4` и `10.20.30.40` , условие будет сопоставляться для любых запросов, поступивших из адреса 1.2.3.4 или 10.20.30.40.
    * **Пример для IPv6**. Если добавить два IP-адреса `1:2:3:4:5:6:7:8` и `10:20:30:40:50:60:70:80` , условие будет сопоставляться для любых запросов, поступивших из адреса 1:2:3:4:5:6:7:8 или 10:20:30:40:50:60:70:80.

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | <ul><li>В портал Azure: `Geo Match` , `Geo Not Match` , `IP Match` или `IP Not Match`</li><li>В шаблонах ARM: `GeoMatch` , `IPMatch` ; Используйте `negateCondition` свойство, чтобы указать _географическое несоответствие_ или _IP-адрес не соответствует_ .</li></ul> |
| Значение | <ul><li>Для `IP Match` операторов или `IP Not Match` : укажите один или несколько диапазонов IP-адресов. Если указано несколько диапазонов IP-адресов, они оцениваются с помощью логики или.</li><li>Для `Geo Match` операторов или `Geo Not Match` : Укажите одно или несколько расположений, используя код страны.</li></ul> |

### <a name="example"></a>Например, .

В этом примере мы сопоставлены все запросы, в которых запрос не был получен из США.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/remote-address.png" alt-text="Снимок экрана портала с условием соответствия &quot;удаленный адрес&quot;.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RemoteAddress",
  "parameters": {
    "operator": "GeoMatch",
    "negateCondition": true,
    "matchValues": [
      "US"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRemoteAddressConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RemoteAddress'
  parameters: {
    operator: 'GeoMatch'
    negateCondition: true
    matchValues: [
      'US'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRemoteAddressConditionParameters'
  }
}
```

---

## <a name="request-body"></a><a name="RequestBody"></a> Текст запроса

Условие соответствия **тексту запроса** определяет запросы на основе конкретного текста, отображаемого в тексте запроса. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

> [!NOTE]
> Если текст запроса превышает 64 КБ, для условия соответствия **текста запроса** будет учитываться только первый 64 КБ.

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | Любой оператор из [списка стандартных операторов](#operator-list). |
| Значение | Одно или несколько строковых или целочисленных значений, представляющих собой значение текста запроса для сопоставления. Если указано несколько значений, они оцениваются с помощью логики или. |
| Преобразование регистра | `Lowercase`, `Uppercase` |

### <a name="example"></a>Например, .

В этом примере мы будем сопоставлять все запросы, в которых текст запроса содержит строку `ERROR` . Перед вычислением соответствия текст запроса преобразуется в верхний регистр, поэтому `error` и другие вариации вариантов также активируют это условие соответствия.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-body.png" alt-text="Снимок экрана портала с условием соответствия &quot;текст запроса&quot;.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestBody",
  "parameters": {
    "operator": "Contains",
    "negateCondition": false,
    "matchValues": [
      "ERROR"
    ],
    "transforms": [
      "Uppercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestBodyConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestBody'
  parameters: {
    operator: 'Contains'
    negateCondition: false
    matchValues: [
      'ERROR'
    ]
    transforms: [
      'Uppercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestBodyConditionParameters'
  }
}
```

---

## <a name="request-file-name"></a><a name="UrlFileName"></a> Имя файла запроса

Условие соответствия " **имя файла запроса** " определяет запросы, включающие в себя указанное имя файла в URL-адресе запроса. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | Любой оператор из [списка стандартных операторов](#operator-list). |
| Значение | Одно или несколько строковых или целочисленных значений, представляющих собой значение имени файла запроса для сопоставления. Если указано несколько значений, они оцениваются с помощью логики или. |
| Преобразование регистра | `Lowercase`, `Uppercase` |

### <a name="example"></a>Например, .

В этом примере мы сопоставлены все запросы, в которых имя файла запроса — `media.mp4` . Перед вычислением совпадения имя файла преобразуется в нижний регистр, поэтому `MEDIA.MP4` и другие вариации вариантов также активируют это условие соответствия.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-file-name.png" alt-text="Снимок экрана портала с условием соответствия &quot;имя файла запроса&quot;.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "UrlFileName",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "media.mp4"
    ],
    "transforms": [
      "Lowercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlFilenameConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'UrlFileName'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'media.mp4'
    ]
    transforms: [
      'Lowercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlFilenameConditionParameters'
  }
}
```

---

## <a name="request-file-extension"></a><a name="UrlFileExtension"></a> Запросить расширение файла

Условие соответствия " **расширение файла запроса** " определяет запросы, которые включают указанное расширение файла, в имя файла в URL-адресе запроса. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

> [!NOTE]
> Не включайте начальную точку. Например, используйте `html` вместо `.html`.

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | Любой оператор из [списка стандартных операторов](#operator-list). |
| Значение | Одно или несколько строковых или целочисленных значений, представляющих собой значение расширения файла запроса для сопоставления. Не включайте начальную точку. Если указано несколько значений, они оцениваются с помощью логики или. |
| Преобразование регистра | `Lowercase`, `Uppercase` |

### <a name="example"></a>Например, .

В этом примере мы сопоставлены все запросы, где расширение файла запроса — `pdf` или `docx` . Перед вычислением совпадения расширение файла запроса преобразуется в нижний регистр, поэтому `PDF` , `DocX` и другие вариации вариантов также активируют это условие соответствия.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-file-extension.png" alt-text="Снимок экрана портала, в котором отображается условие соответствия &quot;расширение файла запроса&quot;.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "UrlFileExtension",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "pdf",
      "docx"
    ],
    "transforms": [
      "Lowercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlFileExtensionMatchConditionParameters"
  }
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'UrlFileExtension'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'pdf'
      'docx'
    ]
    transforms: [
      'Lowercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlFileExtensionMatchConditionParameters'
  }
}
```

---

## <a name="request-header"></a><a name="RequestHeader"></a> Заголовок запроса

Условие соответствия **заголовка запроса** определяет запросы, включающие в запрос конкретный заголовок. Это условие соответствия можно использовать для проверки существования заголовка, независимого от его значения, или для проверки соответствия заголовка заданному значению. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Имя заголовка | Строковое значение, представляющее имя аргумента POST. |
| Оператор | Любой оператор из [списка стандартных операторов](#operator-list). |
| Значение | Одно или несколько строковых или целочисленных значений, представляющих собой значение заголовка запроса для сопоставления. Если указано несколько значений, они оцениваются с помощью логики или. |
| Преобразование регистра | `Lowercase`, `Uppercase` |

### <a name="example"></a>Например, .

В этом примере мы сопоставлены все запросы, в которых запрос содержит заголовок с именем `MyCustomHeader` , независимо от его значения.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-header.png" alt-text="Снимок экрана портала с условием соответствия заголовка запроса.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestHeader",
  "parameters": {
    "selector": "MyCustomHeader",
    "operator": "Any",
    "negateCondition": false,
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestHeaderConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestHeader'
  parameters: {
    selector: 'MyCustomHeader',
    operator: 'Any'
    negateCondition: false
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestHeaderConditionParameters'
  }
}
```

---

## <a name="request-method"></a><a name="RequestMethod"></a> Метод запроса

Условие соответствия **методу запроса** определяет запросы, использующие указанный метод HTTP-запроса. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | <ul><li>В портал Azure: `Equal` , `Not Equal`</li><li>В шаблонах ARM: `Equal` ; Используйте `negateCondition` свойство, чтобы указать _не равно_ |
| Метод запроса | Один или несколько методов HTTP из: `GET` , `POST` , `PUT` , `DELETE` , `HEAD` , `OPTIONS` , `TRACE` . Если указано несколько значений, они оцениваются с помощью логики или. |

### <a name="example"></a>Например, .

В этом примере мы будем сопоставлять все запросы, в которых запрос использует `DELETE` метод.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-method.png" alt-text="Снимок экрана портала, в котором отображается условие соответствия &quot;метод запроса&quot;.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestMethod",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "DELETE"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestMethodConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestMethod'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'DELETE
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestMethodConditionParameters'
  }
}
```

---

## <a name="request-path"></a><a name="UrlPath"></a> Путь запроса

Условие соответствия **пути запроса** определяет запросы, содержащие указанный путь в URL-адресе запроса. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

> [!NOTE]
> Путь является частью URL-адреса после имени узла и косой черты. Например, в URL-адресе `https://www.contoso.com/files/secure/file1.pdf` это путь `files/secure/file1.pdf` .

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | Любой оператор из [списка стандартных операторов](#operator-list). |
| Значение | Одно или несколько строковых или целочисленных значений, представляющих собой значение пути запроса для сопоставления. Не включайте начальную косую черту. Если указано несколько значений, они оцениваются с помощью логики или. |
| Преобразование регистра | `Lowercase`, `Uppercase` |

### <a name="example"></a>Например, .

В этом примере мы будем сопоставлять все запросы, где путь к файлу запроса начинается с `files/secure/` . Перед вычислением совпадения расширение файла запроса преобразуется в нижний регистр, поэтому запросы `files/SECURE/` и другие вариации вариантов также будут активировать это условие соответствия.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-path.png" alt-text="Снимок экрана портала с условием соответствия &quot;путь запроса&quot;.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "UrlPath",
  "parameters": {
    "operator": "BeginsWith",
    "negateCondition": false,
    "matchValues": [
      "files/secure/"
    ],
    "transforms": [
      "Lowercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlPathMatchConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'UrlPath'
  parameters: {
    operator: 'BeginsWith'
    negateCondition: false
    matchValues: [
      'files/secure/'
    ]
    transforms: [
      'Lowercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleUrlPathMatchConditionParameters'
  }
}
```

---

## <a name="request-protocol"></a><a name="RequestScheme"></a> Протокол запроса

Условие соответствия **протокола запроса** определяет запросы, использующие указанный протокол (HTTP или HTTPS).

> [!NOTE]
> *Протокол* иногда также называется *схемой*.

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | <ul><li>В портал Azure: `Equal` , `Not Equal`</li><li>В шаблонах ARM: `Equal` ; Используйте `negateCondition` свойство, чтобы указать _не равно_ |
| Метод запроса | `HTTP`, `HTTPS` |

### <a name="example"></a>Например, .

В этом примере мы будем сопоставлять все запросы, в которых запрос использует `HTTP` протокол.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-protocol.png" alt-text="Снимок экрана портала с условием соответствия протокола запроса.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestScheme",
  "parameters": {
    "operator": "Equal",
    "negateCondition": false,
    "matchValues": [
      "HTTP"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestSchemeConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestScheme'
  parameters: {
    operator: 'Equal'
    negateCondition: false
    matchValues: [
      'HTTP
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestSchemeConditionParameters'
  }
}
```

---

## <a name="request-url"></a><a name="RequestUrl"></a> URL-адрес запроса

Определяет запросы, соответствующие указанному URL-адресу. Вычисляется весь URL-адрес. Можно указать несколько значений для сопоставления, которые будут сочетаться с помощью логики или.

> [!TIP]
> При использовании этого условия правила не забудьте включить протокол. Например, `https://www.contoso.com` вместо просто используйте `www.contoso.com` .

### <a name="properties"></a>Свойства

| Свойство | Поддерживаемые значения |
|-|-|
| Оператор | Любой оператор из [списка стандартных операторов](#operator-list). |
| Значение | Одно или несколько строковых или целочисленных значений, представляющих собой значение URL-адреса запроса для сопоставления. Если указано несколько значений, они оцениваются с помощью логики или. |
| Преобразование регистра | `Lowercase`, `Uppercase` |

### <a name="example"></a>Например, .

В этом примере мы сопоставлены все запросы, начинающиеся с URL-адреса запроса `https://api.contoso.com/customers/123` . Перед вычислением совпадения расширение файла запроса преобразуется в нижний регистр, поэтому запросы `https://api.contoso.com/Customers/123` и другие вариации вариантов также будут активировать это условие соответствия.

# <a name="portal"></a>[Портал](#tab/portal)

:::image type="content" source="../media/concept-rule-set-match-conditions/request-url.png" alt-text="Снимок экрана портала с условием соответствия URL-адреса запроса.":::

# <a name="json"></a>[JSON](#tab/json)

```json
{
  "name": "RequestUri",
  "parameters": {
    "operator": "BeginsWith",
    "negateCondition": false,
    "matchValues": [
      "https://api.contoso.com/customers/123"
    ],
    "transforms": [
      "Lowercase"
    ],
    "@odata.type": "#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestUriConditionParameters"
  }
}
```

# <a name="bicep"></a>[Bicep](#tab/bicep)

```bicep
{
  name: 'RequestUri'
  parameters: {
    operator: 'BeginsWith'
    negateCondition: false
    matchValues: [
      'https://api.contoso.com/customers/123'
    ]
    transforms: [
      'Lowercase'
    ]
    '@odata.type': '#Microsoft.Azure.Cdn.Models.DeliveryRuleRequestUriConditionParameters'
  }
}
```

---

## <a name="operator-list"></a><a name = "operator-list"></a> Список операторов

Для правил, которые принимают значения из списка стандартных операторов, допустимы следующие операторы:

| Оператор                   | Описание                                                                                                                    | Поддержка шаблонов ARM                                            |
|----------------------------|--------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| Любой                        | Соответствует, если имеется какое-либо значение, независимо от его значения.                                                                     | `operator`: `Any`                                               |
| Равно                      | Соответствует, если значение точно соответствует указанной строке.                                                                   | `operator`: `Equal`                                             |
| Содержит                   | Соответствует, если значение содержит указанную строку.                                                                          | `operator`: `Contains`                                          |
| Меньше                  | Соответствует, если длина значения меньше заданного целого числа.                                                       | `operator`: `LessThan`                                          |
| Больше чем               | Соответствует, если длина значения больше заданного целого числа.                                                    | `operator`: `GreaterThan`                                       |
| Меньше или равно         | Соответствует, если длина значения меньше или равна указанному целому числу.                                           | `operator`: `LessThanOrEqual`                                   |
| Больше или равно      | Соответствует, если длина значения больше или равна указанному целому числу.                                        | `operator`: `GreaterThanOrEqual`                                |
| Начинается с                | Соответствует, когда значение начинается с указанной строки.                                                                       | `operator`: `BeginsWith`                                        |
| Заканчивается на                  | Соответствует, когда значение заканчивается на указанную строку.                                                                         | `operator`: `EndsWith`                                          |
| Регулярные                      | Соответствует, если значение соответствует указанному регулярному выражению. [Дополнительные сведения см. ниже.](#regular-expressions)        | `operator`: `RegEx`                                             |
| Не все                    | Соответствует, если значение отсутствует.                                                                                                | `operator`: `Any` и `negateCondition` : `true`                |
| Не равно                  | Соответствует, если значение не совпадает с указанной строкой.                                                                    | `operator`: `Equal` и `negateCondition` : `true`              |
| Не содержит               | Соответствует, если значение не содержит указанную строку.                                                                  | `operator`: `Contains` и `negateCondition` : `true`           |
| Не меньше              | Соответствует, если длина значения не меньше заданного целого числа.                                                   | `operator`: `LessThan` и `negateCondition` : `true`           |
| Не больше           | Соответствует, если длина значения не превышает указанное целое число.                                                | `operator`: `GreaterThan` и `negateCondition` : `true`        |
| Не меньше или равно     | Соответствует, если длина значения не меньше заданного целого числа или равно ему.                                       | `operator`: `LessThanOrEqual` и `negateCondition` : `true`    |
| Не больше или равно | Соответствует, если длина значения не больше заданного целого числа или равно ему.                                    | `operator`: `GreaterThanOrEqual` и `negateCondition` : `true` |
| Не начинается с            | Соответствует, если значение не начинается с указанной строки.                                                               | `operator`: `BeginsWith` и `negateCondition` : `true`         |
| Не заканчивается на              | Соответствует, если значение не оканчивается на указанную строку.                                                                 | `operator`: `EndsWith` и `negateCondition` : `true`           |
| Не регулярное выражение                  | Соответствует, если значение не соответствует указанному регулярному выражению. [Дополнительные сведения см. ниже.](#regular-expressions) | `operator`: `RegEx` и `negateCondition` : `true`              |

> [!TIP]
> Для числовых операторов, таких как *меньше чем* и *больше или равно*, используется сравнение, основанное на длине. Значение в условии соответствия должно быть целым числом, указывающим длину, которую требуется сравнить.

### <a name="regular-expressions"></a><a name="regular-expressions"></a> Регулярные выражения

Регулярные выражения не поддерживают следующие операции:

* Обратные ссылки и запись частей выражений.
* Произвольные утверждения нулевой ширины.
* Ссылки на подпрограммы и рекурсивные шаблоны.
* Условные шаблоны.
* Команды управления поиска с возвратом.
* `\C`Однобайтовая директива.
* `\R`Директива соответствия новой строки.
* `\K`Директива начала сопоставления с сбросом.
* Выноски и внедренный код.
* Атомарное группирование и более обладающие Кванторы.

## <a name="arm-template-support"></a>Поддержка шаблонов ARM

Наборы правил можно настроить с помощью шаблонов Azure Resource Manager. [См. Пример шаблона](https://github.com/Azure/azure-quickstart-templates/tree/master/201-front-door-standard-premium-rule-set). Условия соответствия можно добавить с помощью фрагментов кода JSON или Бицеп, входящих в приведенные выше примеры.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о [наборе правил](concept-rule-set.md).
* Узнайте, как [настроить первый набор правил](how-to-configure-rule-set.md).
* Дополнительные сведения о [действиях набора правил](concept-rule-set-actions.md).
