---
title: Формат Excel в фабрике данных Azure
description: В этом разделе описывается, как работать с форматом Excel в фабрике данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 12/08/2020
ms.author: jingwang
ms.openlocfilehash: bef29bc958253be0498442f842dda67105ce799b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100386532"
---
# <a name="excel-format-in-azure-data-factory"></a>Формат Excel в фабрике данных Azure
[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Если вы хотите **проанализировать файлы Excel**, следуйте этой статье. Фабрика данных Azure поддерживает как "xls", так и xlsx.

Формат Excel поддерживается для следующих соединителей: [Amazon S3](connector-amazon-simple-storage-service.md), [большой двоичный объект Azure](connector-azure-blob-storage.md), [Azure Data Lake Storage 1-го поколения](connector-azure-data-lake-store.md), [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md), [хранилище файлов Azure](connector-azure-file-storage.md), [Файловая система](connector-file-system.md), [FTP](connector-ftp.md), [Google Cloud Storage](connector-google-cloud-storage.md), [HDFS](connector-hdfs.md), [http](connector-http.md)и [SFTP](connector-sftp.md). Он поддерживается в качестве источника, но не для приемника. 

**Примечание**. формат ". xls" не поддерживается при использовании [http](connector-http.md). 

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных Excel.

| Свойство.         | Описание                                                  | Обязательно |
| ---------------- | ------------------------------------------------------------ | -------- |
| type             | Свойство Type набора данных должно иметь значение **Excel**.   | Да      |
| location         | Параметры расположения файлов. Каждый файловый соединитель имеет собственный тип расположения и поддерживаемые свойства в разделе `location` . | Да      |
| sheetName        | Имя листа Excel для чтения данных.                       | Укажите `sheetName` или `sheetIndex` |
| шитиндекс | Индекс листа Excel для чтения данных, начиная с 0. | Укажите `sheetName` или `sheetIndex` |
| range            | Диапазон ячеек на данном листе для нахождение выборочных данных, например:<br>-Не указано: считывает весь лист в виде таблицы из первой непустой строки и столбца<br>- `A3`: считывает таблицу, начиная с заданной ячейки, динамически обнаруживает все строки ниже и все столбцы справа.<br>- `A3:H5`: считывает этот фиксированный диапазон в виде таблицы<br>- `A3:A3`: считывает эту единственную ячейку | Нет       |
| firstRowAsHeader | Указывает, следует ли рассматривать первую строку в заданном листе или диапазоне как строку заголовка с именами столбцов.<br>Допустимые значения: **true** и **false** (по умолчанию). | Нет       |
| nullValue        | Задает строковое представление значения NULL. <br>Значение по умолчанию — **пустая строка**. | Нет       |
| compression | Группа свойств для настройки сжатия файла. Настройте этот раздел, если требуется сжатие и распаковка во время выполнения действия. | Нет |
| type<br/>(*в `compression` разделе*) | Кодек сжатия, используемый для чтения и записи JSON-файлов. <br>Допустимые значения: **bzip2**, **gzip**, **Deflate**, **ZipDeflate**, **таргзип**, **tar**, **привязки** или **lz4**. Значение по умолчанию не сжато.<br>**Примечание** . в настоящее время действие копирования не поддерживает "привязку" & "lz4", а поток данных сопоставления не поддерживает "ZipDeflate", "таргзип" и "tar".<br>**Примечание** . при использовании действия копирования для распаковки файлов **ZipDeflate** и записи в хранилище файловых данных приемника файлы извлекаются в папку: `<path specified in dataset>/<folder named as source zip file>/` . | Нет.  |
| уровень<br/>(*в `compression` разделе*) | Коэффициент сжатия. <br>Допустимые значения: **оптимальный** или **самый быстрый**.<br>- **Самый быстрый:** Операция сжатия должна завершиться как можно быстрее, даже если итоговый файл не сжимается оптимально.<br>- **Оптимально**. операция сжатия должна быть оптимально сжата, даже если выполнение операции займет больше времени. Дополнительные сведения см. в разделе [Уровень сжатия](/dotnet/api/system.io.compression.compressionlevel). | Нет       |

Ниже приведен пример набора данных Excel в хранилище BLOB-объектов Azure.

```json
{
    "name": "ExcelDataset",
    "properties": {
        "type": "Excel",
        "linkedServiceName": {
            "referenceName": "<Azure Blob Storage linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, retrievable during authoring > ],
        "typeProperties": {
            "location": {
                "type": "AzureBlobStorageLocation",
                "container": "containername",
                "folderPath": "folder/subfolder",
            },
            "sheetName": "MyWorksheet",
            "range": "A3:H5",
            "firstRowAsHeader": true
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником Excel.

### <a name="excel-as-source"></a>Excel в качестве источника 

В разделе ***\* Источник \**** действия копирования поддерживаются следующие свойства.

| Свойство.      | Описание                                                  | Обязательно |
| ------------- | ------------------------------------------------------------ | -------- |
| type          | Свойство Type источника действия копирования должно иметь значение **ексцелсаурце**. | Да      |
| сторесеттингс | Группа свойств для чтения данных из хранилища данных. Каждый файловый соединитель имеет собственные Поддерживаемые параметры чтения в разделе `storeSettings` . | Нет       |

```json
"activities": [
    {
        "name": "CopyFromExcel",
        "type": "Copy",
        "typeProperties": {
            "source": {
                "type": "ExcelSource",
                "storeSettings": {
                    "type": "AzureBlobStorageReadSettings",
                    "recursive": true
                }
            },
            ...
        }
        ...
    }
]
```

## <a name="mapping-data-flow-properties"></a>Свойства потока данных для сопоставления

В статье сопоставление потоков данных можно прочитать формат Excel в следующих хранилищах данных: [хранилище BLOB-объектов Azure](connector-azure-blob-storage.md#mapping-data-flow-properties), [Azure Data Lake Storage 1-го поколения](connector-azure-data-lake-store.md#mapping-data-flow-properties)и [Azure Data Lake Storage 2-го поколения](connector-azure-data-lake-storage.md#mapping-data-flow-properties). Можно указывать на файлы Excel либо с помощью набора данных Excel, либо с помощью [встроенного набора данных](data-flow-source.md#inline-datasets).

### <a name="source-properties"></a>Свойства источника

В таблице ниже перечислены свойства, поддерживаемые источником Excel. Эти свойства можно изменить на вкладке **Параметры источника** . При использовании встроенного набора данных вы увидите дополнительные параметры файла, которые совпадают со свойствами, описанными в разделе [Свойства набора данных](#dataset-properties) .

| Имя                      | Описание                                                  | Обязательно | Допустимые значения                                            | Свойство сценария потока данных         |
| ------------------------- | ------------------------------------------------------------ | -------- | --------------------------------------------------------- | --------------------------------- |
| Подстановочные пути           | Будут обработаны все файлы, соответствующие пути с подстановочными знаками. Переопределяет папку и путь к файлу, заданные в наборе данных. | Нет       | String[]                                                  | вилдкардпасс                     |
| Корневой путь раздела       | Для секционированных файловых данных можно ввести корневой путь к разделу, чтобы считывать секционированные папки в виде столбцов. | Нет       | Строка                                                    | партитионрутпас                 |
| Список файлов             | Указывает, указан ли источник на текстовый файл, в котором перечислены обрабатываемые файлы | Нет       | `true` или `false`                                         | fileList                          |
| Столбец для хранения имени файла | Создать новый столбец с именем и путем к исходному файлу       | Нет       | Строка                                                    | ровурлколумн                      |
| После завершения          | Удалите или переместите файлы после обработки. Путь к файлу начинается с корня контейнера | Нет       | Delete: `true` или `false` <br> Поместить `['<from>', '<to>']` | пуржефилес <br> мовефилес         |
| Фильтровать по дате последнего изменения   | Выберите фильтр файлов в зависимости от времени последнего изменения | Нет       | Отметка времени                                                 | modifiedAfter <br> modifiedBefore |
| Разрешить не найдены файлы | Если значение — true, ошибка не возникает, если файлы не найдены | Нет | `true` или `false` | игноренофилесфаунд |

### <a name="source-example"></a>Пример исходного кода

Ниже приведен пример исходной конфигурации Excel в сопоставлении потоков данных с помощью режима набора данных.

![Источник Excel](media/data-flow/excel-source.png)

Связанный сценарий потока данных:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    wildcardPaths:['*.xls']) ~> ExcelSource
```

При использовании встроенного набора данных в поле сопоставление потока данных отображаются следующие параметры источника.

![Исходный встроенный набор данных Excel](media/data-flow/excel-source-inline-dataset.png)

Связанный сценарий потока данных:

```
source(allowSchemaDrift: true,
    validateSchema: false,
    format: 'excel',
    fileSystem: 'container',
    folderPath: 'path',
    fileName: 'sample.xls',
    sheetName: 'worksheet',
    firstRowAsHeader: true) ~> ExcelSourceInlineDataset
```

## <a name="next-steps"></a>Дальнейшие действия

- [Действие копирования в фабрике данных Azure](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Действие получения метаданных в Фабрике данных Azure](control-flow-get-metadata-activity.md)