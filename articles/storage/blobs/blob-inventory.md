---
title: Использование инвентаризации хранилища Azure для управления данными большого двоичного объекта (Предварительная версия)
description: Инвентаризация службы хранилища Azure — это средство, помогающее получить общие сведения о всех данных больших двоичных объектов в учетной записи хранения.
services: storage
author: mhopkins-msft
ms.service: storage
ms.date: 03/05/2021
ms.topic: conceptual
ms.author: mhopkins
ms.reviewer: yzheng
ms.subservice: blobs
ms.custom: references_regions
ms.openlocfilehash: 8310de465a6416102a7ce4e614ead7029e6be87a
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104950932"
---
# <a name="use-azure-storage-blob-inventory-to-manage-blob-data-preview"></a>Использование инвентаризации BLOB-объектов службы хранилища Azure для управления данными больших двоичных объектов (Предварительная версия)

Функция инвентаризации BLOB-объектов службы хранилища Azure предоставляет общие сведения о данных больших двоичных объектов в учетной записи хранения. Используйте отчет об инвентаризации, чтобы понять общий размер данных, возраст, состояние шифрования и т. д. Отчет содержит общие сведения о данных для бизнеса и требований соответствия. После включения отчет инвентаризации автоматически создается ежедневно.

## <a name="availability"></a>Доступность

Инвентаризация BLOB-объектов поддерживается как для общего назначения версии 2 (GPv2), так и для учетных записей хранения блочных BLOB-объектов класса Premium. Эта функция поддерживается с включенной функцией [иерархического пространства имен](data-lake-storage-namespace.md) или без нее.

> [!IMPORTANT]
> Инвентаризация BLOB-объектов сейчас доступна в **предварительной версии**. Ознакомьтесь с [дополнительными условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не включены в общедоступную версию.

### <a name="preview-regions"></a>Регионы предварительной версии

Предварительная версия инвентаризации BLOB-объектов доступна в учетных записях хранения в следующих регионах:

- Центральная Франция
- Центральная Канада
- Восточная Канада
- Восточная часть США
- восточная часть США 2

### <a name="pricing-and-billing"></a>Цены и выставление счетов

Плата за отчеты об инвентаризации не взимается в период действия предварительной версии. Цены будут определены, если эта функция общедоступна.

## <a name="enable-inventory-reports"></a>Включение отчетов об инвентаризации

Включите отчеты об инвентаризации BLOB-объектов, добавив политику в учетную запись хранения. Добавление, изменение или удаление политики с помощью [портал Azure](https://portal.azure.com/).

1. Перейдите на [портал Azure](https://portal.azure.com/).
1. Выберите одну из учетных записей хранения
1. В разделе **Служба BLOB-объектов** выберите пункт **Инвентаризация BLOB-объектов** .
1. Установите флажок **Включить инвентаризацию BLOB-объектов** .
1. Выберите **Добавить правило** .
1. Назовите новое правило
1. Выберите **типы больших двоичных объектов** для отчета об инвентаризации
1. Добавление сопоставления префикса для фильтрации отчета об инвентаризации
1. Укажите, следует ли **включать версии BLOB-объектов** и **включать моментальные снимки** в отчет инвентаризации. Для учетной записи необходимо включить версии и моментальные снимки, чтобы сохранить новое правило с включенным соответствующим параметром.
1. Нажмите кнопку **Сохранить**.

:::image type="content" source="./media/blob-inventory/portal-blob-inventory.png" alt-text="Снимок экрана, показывающий, как добавить правило инвентаризации BLOB-объектов с помощью портал Azure":::

Политики инвентаризации считываются или записываются полностью. Частичные обновления не поддерживаются.

> [!IMPORTANT]
> Если вы настроили для учетной записи хранения правила брандмауэра, запросы на инвентаризацию могут быть заблокированы. Их можно разблокировать, предоставив исключения для доверенных служб Майкрософт. Дополнительные сведения см. в разделе "исключения" раздела [Настройка брандмауэров и виртуальных сетей](../common/storage-network-security.md#exceptions).

Выполнение инвентаризации BLOB-объектов автоматически планируется каждый день. Выполнение инвентаризации может занять до 24 часов. Отчет об инвентаризации настраивается путем добавления политики инвентаризации с одним или несколькими правилами.

## <a name="inventory-policy"></a>Политика инвентаризации

Политика инвентаризации — это коллекция правил в документе JSON.

```json
{
    "destination": "destinationContainer",
    "enabled": true,
    "rules": [
    {
        "enabled": true,
        "name": "inventoryrule1",
        "definition": {. . .}
    },
    {
        "enabled": true,
        "name": "inventoryrule2",
        "definition": {. . .}
    }]
}
```

Просмотрите JSON для политики инвентаризации, выбрав вкладку **представление кода** в разделе **Инвентаризация Blob-объектов** портал Azure.

| Имя параметра | Тип параметра        | Примечания | Необходим? |
|----------------|-----------------------|-------|-----------|
| ресурс destination    | Строковый тип                | Целевой контейнер, в котором будут создаваться все файлы инвентаризации. Целевой контейнер уже должен существовать. | Да |
| Включено        | Логическое               | Используется для отключения всей политики. Если задано **значение true**, то поле с включенным уровнем правила переопределяет этот параметр. Если этот параметр отключен, запасы для всех правил будут отключены. | Да |
| правила          | Массив объектов правил | В политике требуется по крайней мере одно правило. Поддерживается до 10 правил. | Да |

## <a name="inventory-rules"></a>Правила инвентаризации

Правило захватывает условия фильтрации и выходные параметры для создания отчета об инвентаризации. Каждое правило создает отчет об инвентаризации. Правила могут иметь перекрывающиеся префиксы. Большой двоичный объект может отображаться в нескольких запасах в зависимости от определений правил.

Каждое правило в политике имеет несколько параметров:

| Имя параметра | Тип параметра                 | Примечания | Необходим? |
|----------------|--------------------------------|-------|-----------|
| name           | Строка                         | Имя правила может содержать до 256 буквенно-цифровых символов с учетом регистра. Имя должно быть уникальным в пределах политики. | Да |
| Включено        | Логическое                        | Флаг, позволяющий включать или отключать правило. Значение по умолчанию — **true** | Да |
| Определение     | Определение правила инвентаризации JSON | Каждое определение состоит из набора фильтров правил. | Да |

Флаг глобального хранилища **BLOB-объектов** имеет приоритет над параметром, *включенным* в правило.

### <a name="rule-filters"></a>Фильтры правила

Для настройки отчета по инвентаризации BLOB-объектов доступны несколько фильтров:

| Имя фильтра         | Тип фильтра                     | Примечания | Необходим? |
|---------------------|---------------------------------|-------|-----------|
| blobTypes           | Массив предопределенных значений перечисления | Допустимые значения: `blockBlob` и `appendBlob` для иерархических учетных записей с включенным пространством имен,, `blockBlob` `appendBlob` и `pageBlob` для других учетных записей. | Да |
| prefixMatch         | Массив из 10 строк для сопоставления префиксов. Префикс должен начинаться с имени контейнера, например "container1/foo" | Если не определить *префиксматч* или предоставить пустой префикс, правило применяется ко всем BLOB-объектам в учетной записи хранения. | Нет |
| инклудеснапшотс    | Логическое                         | Указывает, должны ли запасы включать моментальные снимки. Значение по умолчанию — **false**. | Нет |
| инклудеблобверсионс | Логическое                         | Указывает, должны ли запасы включать версии больших двоичных объектов. Значение по умолчанию — **false**. | Нет |

Просмотрите JSON для правил инвентаризации, выбрав вкладку **представление кода** в разделе **Инвентаризация Blob-объектов** портал Azure. Фильтры указываются в определении правила.

```json
{
    "destination": "destinationContainer",
    "enabled": true,
    "rules": [
    {
        "enabled": true,
        "name": "inventoryrule1",
        "definition":
        {
            "filters":
            {
                "blobTypes": ["blockBlob", "appendBlob", "pageBlob"],
                "prefixMatch": ["inventorycontainer1", "inventorycontainer2/abcd", "etc"]
            }
        }
    },
    {
        "enabled": true,
        "name": "inventoryrule2",
        "definition":
        {
            "filters":
            {
                "blobTypes": ["pageBlob"],
                "prefixMatch": ["inventorycontainer-disks-", "inventorycontainer4/"],
                "includeSnapshots": true,
                "includeBlobVersions": true
            }
        }
    }]
}
```

## <a name="inventory-output"></a>Выходные данные инвентаризации

Каждый прогон инвентаризации создает набор файлов в формате CSV в указанном контейнере назначения инвентаризации. Выходные данные инвентаризации формируются по следующему `https://<accountName>.blob.core.windows.net/<inventory-destination-container>/YYYY/MM/DD/HH-MM-SS/` пути:

- *AccountName* — имя учетной записи хранилища BLOB-объектов Azure
- *Inventory — Destination — контейнер* — это целевой контейнер, указанный в политике инвентаризации.
- *Гггг/мм/дд/чч-мм-СС* — время начала инвентаризации

### <a name="inventory-files"></a>Файлы инвентаризации

Каждый запуск инвентаризации создает следующие файлы:

- **CSV-файл инвентаризации**: файл значений с разделителями-запятыми (CSV) для каждого правила инвентаризации. Каждый файл содержит совпадающие объекты и их метаданные. Первая строка каждого файла в формате CSV всегда является строкой схемы. На следующем рисунке показан CSV-файл инвентаризации, Открытый в Microsoft Excel.

   :::image type="content" source="./media/blob-inventory/csv-file-excel.png" alt-text="Снимок экрана CSV-файла инвентаризации, открытого в Microsoft Excel":::

- **Файл манифеста**: manifest.jsфайла, содержащего сведения о файлах инвентаризации, созданных для каждого правила в этом запуске. Файл манифеста также захватывает определение правила, предоставленное пользователем, и путь к инвентаризации для этого правила.

- **Файл контрольной суммы**: файл манифеста. CHECKSUM, содержащий контрольную сумму MD5 содержимого manifest.jsв файле. Создание файла манифеста. CHECKSUM отмечает завершение выполнения инвентаризации.

## <a name="inventory-completed-event"></a>Событие завершения инвентаризации

Подпишитесь на событие завершении инвентаризации, чтобы получать уведомления о завершении выполнения инвентаризации. Это событие создается при создании файла контрольной суммы манифеста. Событие "Инвентаризация" также возникает, если при выполнении инвентаризации возникла ошибка пользователя перед запуском. Например, недопустимая политика или контейнер назначения, не имеющий ошибки, будут вызывать событие. Это событие Опубликовано в разделе Инвентаризация BLOB-объектов.

Пример события:

```json
{
  "topic": "/subscriptions/3000151d-7a84-4120-b71c-336feab0b0f0/resourceGroups/BlobInventory/providers/Microsoft.EventGrid/topics/BlobInventoryTopic",
  "subject": "BlobDataManagement/BlobInventory",
  "eventType": "Microsoft.Storage.BlobInventoryPolicyCompleted",
  "id": "c99f7962-ef9d-403e-9522-dbe7443667fe",
  "data": {
    "scheduleDateTime": "2020-10-13T15:37:33Z",
    "accountName": "inventoryaccountname",
    "policyRunStatus": "Succeeded",
    "policyRunStatusMessage": "Inventory run succeeded, refer manifest file for inventory details.",
    "policyRunId": "b5e1d4cc-ee23-4ed5-b039-897376a84f79",
    "manifestBlobUrl": "https://inventoryaccountname.blob.core.windows.net/inventory-destination-container/2020/10/13/15-37-33/manifest.json"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "eventTime": "2020-10-13T15:47:54Z"
}
```

## <a name="next-steps"></a>Дальнейшие действия

- [Вычисление количества и общего размера больших двоичных объектов на контейнер](calculate-blob-count-size.md)
- [Управление жизненным циклом хранилища BLOB-объектов Azure](storage-lifecycle-management-concepts.md)
