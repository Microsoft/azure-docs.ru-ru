---
title: Создание конфигурации набора ресурсов с заданной областью
description: Узнайте, как создать правило конфигурации набора ресурсов с заданной областью для перезаписи ресурсов, сгруппированных в наборы ресурсов.
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 02/17/2021
ms.openlocfilehash: 10e925a84dbe187ccdf5e444cb8b3dd4b7bb4676
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102608008"
---
# <a name="create-scoped-resource-set-configuration-rules"></a>Создание правил конфигурации набора ресурсов с заданной областью

В системах обработки данных с масштабированием обычно одна таблица хранится на диске в виде нескольких файлов. Эта концепция представлена в Azure зрения с помощью наборов ресурсов. Набор ресурсов — это отдельный объект в каталоге данных, который представляет большое количество ресурсов в хранилище. Дополнительные сведения см. в разделе [Общие сведения о наборах ресурсов](concept-resource-sets.md).

При сканировании учетной записи хранения Azure зрения использует набор определенных шаблонов, чтобы определить, является ли группа ресурсов набором. В некоторых случаях Группирование набора ресурсов Azure зрения может неточно отражать ваше место в данных. Правила набора ресурсов с заданной областью позволяют настраивать или переопределять, как Azure зрения определяет, какие ресурсы группируются в виде наборов ресурсов и как они отображаются в каталоге.

## <a name="how-to-create-a-scoped-resource-set-configuration"></a>Создание конфигурации набора ресурсов с заданной областью

Выполните следующие действия, чтобы создать новую конфигурацию набора ресурсов с областью действия.

1. Перейдите в центр управления. Выберите **наборы ресурсов** с заданной областью в меню. Выберите **+ создать** , чтобы создать новый набор правил конфигурации.

   :::image type="content" source="media/how-to-scoped-resource-sets/create-new-scoped-resource-set-rule.png" alt-text="Создать новое правило набора ресурсов с областью действия" border="true":::

1. Введите область конфигурации набора ресурсов с заданной областью. Выберите тип учетной записи хранения и имя учетной записи хранения, для которой нужно создать правило. Каждый набор правил применяется относительно области пути к папке, указанной в поле **путь к папке** .

   :::image type="content" source="media/how-to-scoped-resource-sets/create-new-scoped-resource-set-scope.png" alt-text="Создание конфигураций набора ресурсов с заданной областью" border="true":::

1. Чтобы ввести правило для области конфигурации, выберите **+ создать правило**.

1. Введите следующие поля, чтобы создать правило:

   1. **Имя правила:** Имя правила конфигурации. Это поле не влияет на активы, к которым применяется правило.

   1. **Полное имя:** Полный путь, который использует сочетание Text, Dynamic реплацерс и static реплацерс для сопоставления ресурсов с правилом конфигурации. Этот путь задается относительно области действия правила конфигурации. Подробные инструкции по указанию полных имен см. в разделе [синтаксис](#syntax) ниже.

   1. **Отображаемое имя:** Отображаемое имя ресурса. Это поле является необязательным. Используйте обычный текст и статический реплацерс для настройки отображения ресурса в каталоге. Более подробные инструкции см. в разделе [синтаксис](#syntax) ниже.

   1. **Не группировать как набор ресурсов:** Если параметр включен, соответствующий ресурс не будет сгруппирован в набор ресурсов.

      :::image type="content" source="media/how-to-scoped-resource-sets/scoped-resource-set-rule-example.png" alt-text="Создать новое правило конфигурации." border="true":::

1. Сохраните правило, нажав кнопку **Добавить**.

## <a name="scoped-resource-set-syntax"></a><a name="syntax"></a> Синтаксис набора ресурсов с заданной областью

При создании правил набора ресурсов с заданной областью используйте следующий синтаксис для указания правил, к которым применяются правила активов.

### <a name="dynamic-replacers-single-brackets"></a>Динамический реплацерс (одиночные скобки)

Одинарные скобки используются в качестве **динамических реплацерс** в правиле набора ресурсов с заданной областью. Укажите динамический заменяемый элемент в полном имени, используя формат `{<replacerName:<replacerType>}` . При сопоставлении динамические реплацерс используются в качестве условия группирования, которое указывает, что ресурсы должны быть представлены в виде набора ресурсов. Если ресурсы сгруппированы в набор ресурсов, то полный путь к набору ресурсов будет содержать `{replacerName}` место, где был указан указанный элемент.

Например, если два актива `folder1/file-1.csv` и `folder2/file-2.csv` соответствуют правилу `{folder:string}/file-{NUM:int}.csv` , набор ресурсов будет единственной сущностью `{folder}/file-{NUM}.csv` .

#### <a name="special-case-dynamic-replacers-when-not-grouping-into-resource-set"></a>Особый случай: Dynamic реплацерс при отсутствии группирования в наборе ресурсов

Если параметр *не группировать в качестве набора ресурсов* включен для правила набора ресурсов с заданной областью, имя замены является необязательным полем. `{:<replacerType>}` является допустимым синтаксисом. Например, `file-{:int}.csv` будет успешно сопоставляться для `file-1.csv` и `file-2.csv` и создавать два разных ресурса вместо набора ресурсов.

### <a name="static-replacers-double-brackets"></a>Static реплацерс (двойные скобки)

Двойные скобки используются в качестве **статических реплацерс** в полном имени правила набора ресурсов с областью действия. Укажите статический заменяемый элемент в полном имени, используя формат `{{<replacerName>:<replacerType>}}` . При совпадении каждый набор уникальных статических значений замены будет создавать различные группирования наборов ресурсов.

Например, если два `folder1/file-1.csv` `folder2/file-2.csv` ресурса и соответствует правилу, будут `{{folder:string}}/file-{NUM:int}.csv` созданы два набора ресурсов `folder1/file-{NUM}.csv` и `folder2/file-{NUM}.csv` .

Статический реплацерс можно использовать для указания отображаемого имени ресурса, соответствующего правилу набора ресурсов с областью действия. `{{<replacerName>}}`При использовании в отображаемом имени правила будет использоваться совпадающее значение в имени ресурса.

### <a name="available-replacement-types"></a>Доступные типы замены

Ниже приведены доступные типы, которые можно использовать в статических и динамических реплацерс:

| Type | structure |
| ---- | --------- |
| строка | Последовательность из 1 или более символов Юникода, включая разделители, такие как пробелы. |
| INT | Последовательность из 1 или более 0-9 символов ASCII, это может быть 0 с префиксом (например, 0001). |
| guid | Серия 32 или 8-4-4-4-12 строкового представления UUID в виде дефинеддефа в [RFC 4122](https://tools.ietf.org/html/rfc4122). |
| Дата | Последовательность из 6 или 8 0-9 символов ASCII с неопределенными разделителями: ГГГГММДД, гггг-мм-дд, YYMMDD, YY-MM-ДД, указанные в [RFC 3339](https://tools.ietf.org/html/rfc3339). |
| time | Последовательность из 4 или 6 0-9 символов ASCII с необязательными разделителями: ЧЧММ, чч: мм, ЧЧММСС, чч: мм: СС, указанные в [RFC 3339](https://tools.ietf.org/html/rfc3339). |
| TIMESTAMP | Последовательность из 12 или 14 0-9 символов ASCII с неопределенными разделителями: гггг-мм-ddTHH: mm, ГГГГММДДЧЧмм, гггг-мм-ddTHH: mm: SS, yyyymmddHHmmss, указанный в [RFC 3339](https://tools.ietf.org/html/rfc3339). |
| Логическое | Может содержать значение "true" или "false", без учета регистра. |
| Число | Последовательность из 0 или более 0-9 символов ASCII, может иметь значение 0 (например, 0001), а при необходимости — точку «.» и последовательность из 1 или более 0-9 символов ASCII. это может быть 0 (например,. 100). |
| hex | Последовательность из 1 или более символов ASCII из набора 0-1 и A-F, может иметь значение 0 с префиксом |
| локаль | Строка, соответствующая синтаксису, указанному в [RFC 5646](https://tools.ietf.org/html/rfc5646). |

## <a name="order-of-scoped-resource-set-rules-getting-applied"></a>Порядок применения правил набора ресурсов с областью действия

Ниже приведен порядок операций по применению правил набора ресурсов с заданной областью.

1. Более конкретные области будут иметь приоритет, если ресурс соответствует двум правилам. Например, правила в области `container/folder` будут применяться перед правилами в области действия `container` .

1. Порядок правил в определенной области. Это можно изменить в UX.

1. Если ресурс не соответствует ни одному определенному правилу, применяется эвристика набора ресурсов по умолчанию.

## <a name="examples"></a>Примеры

### <a name="example-1"></a>Пример 1

Извлечение данных SAP в полные и разностные нагрузки

#### <a name="inputs"></a>Входные данные

Файлы:

- `https://myazureblob.blob.core.windows.net/bar/customer/full/2020/01/13/saptable_customer_20200101_20200102_01.txt`
- `https://myazureblob.blob.core.windows.net/bar/customer/full/2020/01/13/saptable_customer_20200101_20200102_02.txt`
- `https://myazureblob.blob.core.windows.net/bar/customer/delta/2020/01/15/saptable_customer_20200101_20200102_01.txt`
- `https://myazureblob.blob.core.windows.net/bar/customer/full/2020/01/17/saptable_customer_20200101_20200102_01.txt`
- `https://myazureblob.blob.core.windows.net/bar/customer/full/2020/01/17/saptable_customer_20200101_20200102_02.txt`

#### <a name="scoped-resource-set-rule"></a>Правило набора ресурсов с заданной областью

**Область действия:**`https://myazureblob.blob.core.windows.net/bar/`

**Отображаемое имя:** "Внешний клиент"

**Полное имя:**`customer/{extract:string}/{year:int}/{month:int}/{day:int}/saptable_customer_{date_from:date}_{date_to:time}_{sequence:int}.txt`

**Набор ресурсов:** true

#### <a name="output"></a>Выходные данные

Один ресурс набора ресурсов

**Отображаемое имя:** Внешний клиент

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/customer/{extract}/{year}/{month}/{day}/saptable_customer_{date_from}_{date_to}_{sequence}.txt`

### <a name="example-2"></a>Пример 2

Данные IoT в формате Avro

#### <a name="inputs"></a>Входные данные

Файлы:

- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-002.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/02-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/01-01-2020/22:33:22-001.avro`

#### <a name="scoped-resource-set-rules"></a>Правила набора ресурсов с заданной областью

**Область действия:**`https://myazureblob.blob.core.windows.net/bar/`

Правило 1

**Отображаемое имя:** "Machine-89"

**Полное имя:**`raw/machinename-89/{date:date}/{time:time}-{id:int}.avro`

**Набор ресурсов:** true

Правило 2

**Отображаемое имя:** "Machine-90"

**Полное имя:**`raw/machinename-90/{date:date}/{time:time}-{id:int}.avro`

#### <a name="resource-set-true"></a>*Набор ресурсов: true*

#### <a name="outputs"></a>Выходные данные

2 набора ресурсов

Набор ресурсов 1

**Отображаемое имя:** machine-89

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/{date}/{time}-{id}.avro`

Набор ресурсов 2

**Отображаемое имя:** machine-90

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/{date}/{time}-{id}.avro`

### <a name="example-3"></a>Пример 3

Данные IoT в формате Avro

#### <a name="inputs"></a>Входные данные

Файлы:

- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-002.avro`
- `https://myazureblob.blob.core.windows.netbar/raw/machinename-89/02-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/01-01-2020/22:33:22-001.avro`

#### <a name="scoped-resource-set-rule"></a>Правило набора ресурсов с заданной областью

**Область действия:**`https://myazureblob.blob.core.windows.net/bar/`

**Отображаемое имя:** "Machine-{{MachineId}}"

**Полное имя:**`raw/machinename-{{machineid:int}}/{date:date}/{time:time}-{id:int}.avro`

**Набор ресурсов:** true

#### <a name="outputs"></a>Выходные данные

Набор ресурсов 1

**Отображаемое имя:** machine-89

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/{date}/{time}-{id}.avro`

Набор ресурсов 2

**Отображаемое имя:** machine-90

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/{date}/{time}-{id}.avro`

### <a name="example-4"></a>Пример 4

Не группировать в наборы ресурсов

#### <a name="inputs"></a>Входные данные

Файлы:

- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-002.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/02-01-2020/22:33:22-001.avro`
- `https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/01-01-2020/22:33:22-001.avro`

#### <a name="scoped-resource-set-rule"></a>Правило набора ресурсов с заданной областью

**Область действия:**`https://myazureblob.blob.core.windows.net/bar/`

**Отображаемое имя:**`Machine-{{machineid}}`

**Полное имя:**`raw/machinename-{{machineid:int}}/{{:date}}/{{:time}}-{{:int}}.avro`

**Набор ресурсов:** false

#### <a name="outputs"></a>Выходные данные

4 отдельных активов

Ресурс 1

**Отображаемое имя:** machine-89

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-001.avro`

Ресурс 2

**Отображаемое имя:** machine-89

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/01-01-2020/22:33:22-002.avro`

Ресурс 3

**Отображаемое имя:** machine-89

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/raw/machinename-89/02-01-2020/22:33:22-001.avro`

Ресурс 4

**Отображаемое имя:** machine-90

**Полное имя:**`https://myazureblob.blob.core.windows.net/bar/raw/machinename-90/01-01-2020/22:33:22-001.avro`

## <a name="next-steps"></a>Дальнейшие действия

Приступите [к работе, зарегистрировав и проверяя учетную запись хранения Azure Data Lake Gen2](register-scan-adls-gen2.md).