---
title: Копирование данных в файловую систему и из нее с помощью фабрики данных Azure
description: Узнайте, как копировать данные в локальную файловую систему и из нее с помощью фабрики данных Azure.Узнайте, как перемещать данные в локальную файловую систему и из нее с помощью фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/13/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 9acd2adb68347978bb123d2239eedbb3e5595bc3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100387399"
---
# <a name="copy-data-to-and-from-an-on-premises-file-system-by-using-azure-data-factory"></a>Копирование данных в локальную файловую систему или из нее с помощью фабрики данных Azure
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-onprem-file-system-connector.md)
> * [Версия 2 (текущая)](../connector-file-system.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. статью о [соединителе файловой системы в службе "Фабрика данных Azure" версии 2](../connector-file-system.md).


В этой статье рассказывается, как с помощью действия копирования в фабрике данных Azure копировать данные в локальную файловую систему и обратно. Она основана на статье о [действиях перемещения данных](data-factory-data-movement-activities.md) , в которой представлен общий обзор перемещения данных с помощью действия копирования.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="supported-scenarios"></a>Поддерживаемые сценарии
Данные можно скопировать **из локальной файловой системы** в следующие хранилища данных:

[!INCLUDE [data-factory-supported-sink](../../../includes/data-factory-supported-sinks.md)]

Данные можно скопировать **в локальную файловую систему** из следующих хранилищ данных:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!NOTE]
> Действие копирования не удаляет исходный файл после его успешного копирования в место назначения. Если необходимо удалить исходный файл после успешного копирования, создайте настраиваемое действие для удаления файла и используйте это действие в конвейере.

## <a name="enabling-connectivity"></a>Включение соединения
Фабрика данных поддерживает подключение к локальной файловой системе и из нее через **шлюз управление данными**. Необходимо установить шлюз управления данными в локальной среде, чтобы фабрика данных могла подключаться к любому поддерживаемому локальному хранилищу данных, в том числе к файловой системе. В статье [Перемещение данных между локальными и облачными ресурсами](data-factory-move-data-between-onprem-and-cloud.md) приведены сведения о шлюзе управления данными и пошаговые инструкции по его настройке. Для обмена данными с файловой системой устанавливать какие-либо двоичные файлы, кроме шлюза управления данными, не требуется. Необходимо установить и использовать шлюз управления данными, даже если файловая система находится на виртуальной машине Azure IaaS. Дополнительные сведения о шлюзе см. в статье [Шлюз управления данными](data-factory-data-management-gateway.md).

Чтобы использовать общую папку Linux, установите [Samba](https://www.samba.org/) на сервере Linux, а шлюз управления данными — на сервере Windows. Установка шлюза управления данными на сервер Linux не поддерживается.

## <a name="getting-started"></a>Начало работы
Можно создать конвейер с действием копирования, которое перемещает данные из файловой системы и обратно с помощью различных средств и API-интерфейсов.

Самый простой способ создать конвейер — использовать **Мастер копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены краткие пошаговые указания по созданию конвейера с помощью мастера копирования данных.

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **фабрику данных**. Фабрика данных может содержать один или несколько конвейеров.
2. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных. Например, при копировании данных из хранилища BLOB-объектов Azure в локальную файловую систему создайте две связанные службы, чтобы связать локальную файловую систему и учетную запись хранения Azure с фабрикой данных. Сведения о свойствах связанной службы, относящихся к локальной файловой системе, см. в разделе [Свойства связанной службы](#linked-service-properties).
3. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования. В примере, упомянутом на последнем этапе, создайте набор данных, чтобы указать контейнер BLOB-объектов и папку, содержащую входные данные. Создайте другой набор данных, укажите имя папки и файла (необязательно) в файловой системе. Сведения о свойствах набора данных, относящихся к локальной файловой системе, см. в разделе [Свойства набора данных](#dataset-properties).
4. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных. В примере выше BlobSource используется как источник, а FileSystemSink — как приемник для действия копирования. Точно так же при копировании из локальной файловой системы в хранилище BLOB-объектов Azure в действии копирования используются FileSystemSource и BlobSink. Сведения о свойствах действия копирования, относящихся к локальной файловой системе, см. в разделе [Свойства действия копирования](#copy-activity-properties). Для получения сведений о том, как использовать хранилище данных в качестве источника или приемника, щелкните ссылку в предыдущем разделе об источнике данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON.  Примеры с определениями JSON для сущностей фабрики данных, которые используются для копирования данных из файловой системы и обратно, см. в разделе [Примеры JSON для копирования данных в файловую систему и обратно](#json-examples-for-copying-data-to-and-from-file-system) этой статьи.

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, характерных для файловой системы.

## <a name="linked-service-properties"></a>Свойства связанной службы
Локальную файловую систему можно связать с фабрикой данных Azure с помощью связанной службы **локального файлового сервера** . В таблице ниже приведены описания элементов JSON, которые относятся к связанной службе локального файлового сервера.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| type |Убедитесь, что свойству type присвоено значение **OnPremisesFileServer**. |Да |
| host |Задает корневой путь к папке, которую необходимо скопировать. Используйте escape-символ "\" для специальных символов в строке. Примеры приведены в разделе [Примеры определений связанной службы и набора данных](#sample-linked-service-and-dataset-definitions). |Да |
| userid |Укажите идентификатор пользователя, имеющего доступ к серверу. |Нет (если выбрать encryptedcredential) |
| password |Укажите пароль для пользователя (userid). |Нет (если выбрать encryptedcredential) |
| encryptedCredential |Укажите зашифрованные учетные данные, которые можно получить, выполнив командлет New-AzDataFactoryEncryptValue. |Нет (если имя пользователя и пароль указываются в виде обычного текста) |
| gatewayName |Задает имя шлюза, который следует использовать фабрике данных для подключения к локальному файловому серверу. |Да |


### <a name="sample-linked-service-and-dataset-definitions"></a>Примеры определений связанной службы и набора данных
| Сценарий | Размещение в определении связанной службы | Путь к файлу в определении набора данных |
| --- | --- | --- |
| Локальная папка на компьютере со шлюзом управления данными. <br/><br/>Примеры: D:\\\* или D:\folder\subfolder\\\* |"D:\\\\" (для шлюза управления данными 2.0 и более поздних версий) <br/><br/> localhost (для версий, предшествующих версии 2.0 шлюза управления данными) |.\\\\ или "папка\\\\вложенная_папка" (для шлюза управления данными 2.0 и более поздних версий) <br/><br/>"D:\\\\" или "D:\\\\папка\\\\вложенная_папка" (для шлюза версии ниже 2.0) |
| Удаленная общая папка: <br/><br/>Примеры: "\\\\сервер\\общая_папка\\\\*" или "\\\\сервер\\общая_папка\\папка\\вложенная_папка\\\\*" |\\\\\\\\сервер\\\\общая_папка |.\\\\ или "папка\\\\вложенная_папка" |

>[!NOTE]
>При создании и настройке с помощью пользовательского интерфейса не нужно вводить две обратные косые черты (`\\`) в качестве escape-символа, как при использовании JSON. Укажите одну обратную косую черту.

### <a name="example-using-username-and-password-in-plain-text"></a>Пример указания имени пользователя и пароля в виде обычного текста

```JSON
{
  "Name": "OnPremisesFileServerLinkedService",
  "properties": {
    "type": "OnPremisesFileServer",
    "typeProperties": {
      "host": "\\\\Contosogame-Asia",
      "userid": "Admin",
      "password": "123456",
      "gatewayName": "mygateway"
    }
  }
}
```

### <a name="example-using-encryptedcredential"></a>Пример использования encryptedcredential

```JSON
{
  "Name": " OnPremisesFileServerLinkedService ",
  "properties": {
    "type": "OnPremisesFileServer",
    "typeProperties": {
      "host": "D:\\",
      "encryptedCredential": "WFuIGlzIGRpc3Rpbmd1aXNoZWQsIG5vdCBvbmx5IGJ5xxxxxxxxxxxxxxxxx",
      "gatewayName": "mygateway"
    }
  }
}
```

## <a name="dataset-properties"></a>Свойства набора данных
Полный список разделов и свойств, используемых для определения наборов данных, см. в статье [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md). Разделы структуры, доступности и политики JSON набора данных одинаковы для всех типов наборов данных.

Разделы typeProperties для каждого типа набора данных отличаются. В этом разделе указываются такие сведения, как расположение и формат данных в хранилище данных. Раздел typeProperties набора данных типа **FileShare** содержит следующие свойства.

| Свойство. | Описание | Обязательно |
| --- | --- | --- |
| folderPath |Указывает вложенный путь к папке. Чтобы указать специальные знаки в строке, используйте escape-символ '\'. Фильтр подстановочных знаков не поддерживается. Примеры приведены в разделе [Примеры определений связанной службы и набора данных](#sample-linked-service-and-dataset-definitions).<br/><br/>Вы можете использовать это свойство вместе с параметром **partitionBy**, чтобы в путях к папкам учитывались дата и время начала и окончания среза. |Да |
| fileName |Укажите имя файла в свойстве **folderPath**, если требуется создать таблицу, ссылающуюся на конкретный файл в папке. Если этому свойству не присвоить значение, таблица будет указывать на все файлы в папке.<br/><br/>Если значение **fileName** не указано для выходного набора данных, а значение **preserveHierarchy** не указано в приемнике действия, имя созданного файла будет иметь следующий формат: <br/><br/>`Data.<Guid>.txt` (пример: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt) |Нет |
| fileFilter |Укажите фильтр для выбора подмножества файлов из folderPath. Фильтр дает возможность выбирать только некоторые файлы, а не все. <br/><br/>Допустимые значения: `*` (несколько знаков) и `?` (один знак).<br/><br/>Пример 1: "fileFilter": "*.log"<br/>Пример 2: "fileFilter": 2014-1-?.txt"<br/><br/>Обратите внимание, что свойство fileFilter применяется к входному набору данных FileShare. |Нет |
| partitionedBy |С помощью свойства partitionedBy можно указать динамические значения folderPath и fileName для данных временного ряда. Например, можно параметризовать значение folderPath для каждого часа получения данных. |Нет |
| format | Поддерживаются следующие типы формата: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat**, **ParquetFormat**. Свойству **type** в разделе format необходимо присвоить одно из этих значений. Дополнительные сведения см. в разделах о [текстовом формате](data-factory-supported-file-and-compression-formats.md#text-format), [формате Json](data-factory-supported-file-and-compression-formats.md#json-format), [формате Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [формате Orc](data-factory-supported-file-and-compression-formats.md#orc-format) и [ формате Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). <br><br> Если требуется скопировать файлы между файловыми хранилищами **как есть** (двоичное копирование), можно пропустить раздел форматирования в определениях входного и выходного наборов данных. |Нет |
| compression | Укажите тип и уровень сжатия данных. Поддерживаемые типы: **GZip**, **Deflate**, **BZip2** и **ZipDeflate**. Поддерживаемые уровни: **Оптимальный** и **Самый быстрый**. См. сведения о [форматах файлов и сжатия данных в фабрике данных Azure](data-factory-supported-file-and-compression-formats.md#compression-support). |Нет |

> [!NOTE]
> Свойства filename и fileFilter невозможно использовать одновременно.

### <a name="using-partitionedby-property"></a>Использование свойства partitionedBy
Как сказано выше, для данных временных рядов путь к папке (folderPath) и имя файла (fileName) можно указывать динамически. Это делается с помощью свойства **partitionedBy**, [функций фабрики данных и системных переменных](data-factory-functions-variables.md).

Дополнительные сведения о наборах данных временных рядов, планировании и срезах см. в статьях [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md), [Планирование и исполнение с использованием фабрики данных](data-factory-scheduling-and-execution.md) и [Конвейеры и действия в фабрике данных Azure: создание конвейеров, цепочки действий и расписаний для них](data-factory-create-pipelines.md).

#### <a name="sample-1"></a>Пример 1

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

В этом примере {Slice} заменяется значением SliceStart (системная переменная фабрики данных) в формате ГГГГММДДЧЧ. SliceStart указывает время начала среза. Значение folderPath отличается для каждого среза. Например: wikidatagateway/wikisampledataout/2014100103 или wikidatagateway/wikisampledataout/2014100104.

#### <a name="sample-2"></a>Пример 2

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Year}/{Month}/{Day}",
"fileName": "{Hour}.csv",
"partitionedBy":
[
    { "name": "Year", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyy" } },
    { "name": "Month", "value": { "type": "DateTime", "date": "SliceStart", "format": "MM" } },
    { "name": "Day", "value": { "type": "DateTime", "date": "SliceStart", "format": "dd" } },
    { "name": "Hour", "value": { "type": "DateTime", "date": "SliceStart", "format": "hh" } }
],
```

В этом примере год, месяц, день и время SliceStart извлекаются в отдельные переменные, используемые в свойствах folderPath и fileName.

## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Создание конвейеров](data-factory-create-pipelines.md). Свойства, например имя, описание, входные и выходные таблицы, политики и т. д., доступны для всех типов действий. В свою очередь свойства, доступные в разделе **typeProperties** действия, зависят от конкретного типа действия.

Для действия копирования они различаются в зависимости от типов источников и приемников. При перемещении данных из локальной файловой системы в действии копирования задается тип источника **FileSystemSource**. Точно так же при перемещении данных из локальной файловой системы в действии копирования задается тип приемника **FileSystemSink**. Этот раздел содержит список свойств, поддерживаемых типами FileSystemSource и FileSystemSink.

**FileSystemSource** поддерживает следующие свойства:

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| recursive |Указывает, следует ли читать данные рекурсивно из вложенных папок или только из указанной папки. |True, False (по умолчанию) |Нет |

**FileSystemSink** поддерживает следующие свойства.

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| copyBehavior |Это свойство определяет поведение функции копирования, когда в качестве источника используется BlobSource или FileSystem. |**PreserveHierarchy:** Сохраняет иерархию файлов в целевой папке. То есть относительный путь исходного файла в исходной папке идентичен относительному пути целевого файла в целевой папке.<br/><br/>**FlattenHierarchy**: все файлы из исходной папки создаются на первом уровне в целевой папке. Целевые файлы создаются с автоматически сформированными именами.<br/><br/>**MergeFiles:** Объединяет все файлы из исходной папки в один файл. Если указано имя большого двоичного объекта или имя файла, то оно присваивается объединенному файлу. В противном случае присваивается автоматически созданное имя файла. |Нет |

### <a name="recursive-and-copybehavior-examples"></a>Примеры recursive и copyBehavior
В данном разделе описываются результаты выполнения операции копирования при использовании различных сочетаний значений свойств recursive и copyBehavior.

| Значение recursive | Значение copyBehavior | Результаты выполнения операции |
| --- | --- | --- |
| Да |preserveHierarchy |Если у исходной папки "Папка1" следующая структура:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается с такой же структурой, как и исходная папка.<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5 |
| Да |flattenHierarchy |Если у исходной папки "Папка1" следующая структура:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой: <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл1"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл2"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл3"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл4"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл5" |
| Да |mergeFiles |Если у исходной папки "Папка1" следующая структура:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой: <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Содержимое файлов "Файл1", "Файл2", "Файл3", "Файл4" и "Файл5" объединяется в один файл с автоматически созданным именем. |
| false |preserveHierarchy |Если у исходной папки "Папка1" следующая структура:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/><br/>Папка "Вложенная_папка1" с файлами "Файл3", "Файл4" и "Файл5" не будет включена в эту папку. |
| false |flattenHierarchy |Если у исходной папки "Папка1" следующая структура:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл1"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл2"<br/><br/>Папка "Вложенная_папка1" с файлами "Файл3", "Файл4" и "Файл5" не будет включена в эту папку. |
| false |mergeFiles |Если у исходной папки "Папка1" следующая структура:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Содержимое файлов "Файл1" и "Файл2" объединяется в один файл с автоматически созданным именем.<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл1"<br/><br/>Папка "Вложенная_папка1" с файлами "Файл3", "Файл4" и "Файл5" не будет включена в эту папку. |

## <a name="supported-file-and-compression-formats"></a>Поддерживаемые форматы файлов и сжатия
Дополнительные сведения см. в статье [Форматы файлов и сжатия данных, поддерживаемые фабрикой данных Azure](data-factory-supported-file-and-compression-formats.md).

## <a name="json-examples-for-copying-data-to-and-from-file-system"></a>Примеры JSON для копирования данных в файловую систему и обратно
Следующие примеры содержат образцы определений JSON, которые можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). В них показано, как копировать данные в локальную файловую систему и хранилище BLOB-объектов Azure и из них. Тем не менее вы можете скопировать данные *непосредственно* из любых источников в любые из приемников, перечисленных в разделе [Поддерживаемые источники и приемники](data-factory-data-movement-activities.md#supported-data-stores-and-formats), с помощью действия копирования в фабрике данных Azure.

### <a name="example-copy-data-from-an-on-premises-file-system-to-azure-blob-storage"></a>Пример. Копирование данных из локальной файловой системы в хранилище BLOB-объектов Azure
В этом примере показано, как скопировать данные из локальной файловой системы в хранилище BLOB-объектов Azure. Пример содержит следующие сущности фабрики данных.

* Связанная служба типа [OnPremisesFileServer](#linked-service-properties).
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Входной [набор данных](data-factory-create-datasets.md) типа общая [Папка.](#dataset-properties)
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, в котором используются [FileSystemSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В следующем примере данные временных рядов каждый час копируются из локальной файловой системы в хранилище BLOB-объектов Azure. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

В первую очередь настройте шлюз управления данными, как указано в статье [Перемещение данных между локальными источниками и облаком с помощью шлюза управления данными](data-factory-move-data-between-onprem-and-cloud.md).

**Связанная служба локального файлового сервера**

```JSON
{
  "Name": "OnPremisesFileServerLinkedService",
  "properties": {
    "type": "OnPremisesFileServer",
    "typeProperties": {
      "host": "\\\\Contosogame-Asia.<region>.corp.<company>.com",
      "userid": "Admin",
      "password": "123456",
      "gatewayName": "mygateway"
    }
  }
}
```

Мы рекомендуем использовать свойство **encryptedCredential** вместо свойств **userid** и **password**. Подробные сведения о связанной службе файлового сервера см. в [соответствующем разделе](#linked-service-properties).

**Связанная служба хранилища Azure:**

```JSON
{
  "name": "StorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

**Входной набор данных локальной файловой системы**

Данные берутся из нового файла каждый час. Свойства folderPath и fileName определяются с учетом времени начала среза.

Если задано `"external": "true"`, то фабрика данных считает эту таблицу внешней, то есть не созданной в результате какого-либо действия в фабрике данных.

```JSON
{
  "name": "OnpremisesFileSystemInput",
  "properties": {
    "type": " FileShare",
    "linkedServiceName": " OnPremisesFileServerLinkedService ",
    "typeProperties": {
      "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ]
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Выходной набор данных хранилища BLOB-объектов Azure**

Данные записываются в новый BLOB-объект каждый час (frequency: hour, interval: 1). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ],
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "\t",
        "rowDelimiter": "\n"
      }
    },
    "availability": {
      "frequency": "Hour",
      "interval": 1
    }
  }
}
```

**Действие копирования в конвейере с файловой системой в качестве источника и BLOB-объектом в качестве приемника**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **Source** установлено значение **FileSystemSource**, а для типа **sink** — значение **BlobSink**.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2015-06-01T18:00:00",
    "end":"2015-06-01T19:00:00",
    "description":"Pipeline for copy activity",
    "activities":[
      {
        "name": "OnpremisesFileSystemtoBlob",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "OnpremisesFileSystemInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "FileSystemSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```

### <a name="example-copy-data-from-azure-sql-database-to-an-on-premises-file-system"></a>Пример. Копирование данных из базы данных SQL Azure в локальную файловую систему
В примере ниже используется следующее:

* Связанная служба типа [AzureSqlDatabase.](data-factory-azure-sql-connector.md#linked-service-properties)
* Связанная служба типа [OnPremisesFileServer](#linked-service-properties).
* Входной набор данных типа [AzureSqlTable](data-factory-azure-sql-connector.md#dataset-properties).
* Выходной набор данных типа общая [Папка.](#dataset-properties)
* Конвейер с действием копирования, в котором используются [SqlSource](data-factory-azure-sql-connector.md#copy-activity-properties) и [FileSystemSink](#copy-activity-properties).

В этом примере данные временного ряда каждый час копируются из таблицы SQL Azure в локальную файловую систему. Используемые в этих примерах свойства JSON описаны в разделах, следующих за примерами.

**Связанная служба базы данных SQL Azure:**

```JSON
{
  "name": "AzureSqlLinkedService",
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
    }
  }
}
```

**Связанная служба локального файлового сервера**

```JSON
{
  "Name": "OnPremisesFileServerLinkedService",
  "properties": {
    "type": "OnPremisesFileServer",
    "typeProperties": {
      "host": "\\\\Contosogame-Asia.<region>.corp.<company>.com",
      "userid": "Admin",
      "password": "123456",
      "gatewayName": "mygateway"
    }
  }
}
```

Мы рекомендуем использовать свойство **encryptedCredentia** вместо свойств **userid** и **password**. Подробные сведения о связанной службе файловой системы см. в [соответствующем разделе](#linked-service-properties).

**Входной набор данных SQL Azure**

В этом примере предполагается, что вы создали таблицу MyTable в SQL Azure и она содержит столбец с именем «timestampcolumn» для данных временных рядов.

Если задано ``"external": "true"``, то фабрика данных считает эту таблицу внешней, то есть не созданной в результате какого-либо действия в фабрике данных.

```JSON
{
  "name": "AzureSqlInput",
  "properties": {
    "type": "AzureSqlTable",
    "linkedServiceName": "AzureSqlLinkedService",
    "typeProperties": {
      "tableName": "MyTable"
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Выходной набор данных локальной файловой системы**

Данные копируются в новый файл каждый час. Свойства folderPath и fileName для большого двоичного объекта определяются с учетом времени начала среза.

```JSON
{
  "name": "OnpremisesFileSystemOutput",
  "properties": {
    "type": "FileShare",
    "linkedServiceName": " OnPremisesFileServerLinkedService ",
    "typeProperties": {
      "folderPath": "mysharedfolder/yearno={Year}/monthno={Month}/dayno={Day}",
      "fileName": "{Hour}.csv",
      "partitionedBy": [
        {
          "name": "Year",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "yyyy"
          }
        },
        {
          "name": "Month",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "MM"
          }
        },
        {
          "name": "Day",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "dd"
          }
        },
        {
          "name": "Hour",
          "value": {
            "type": "DateTime",
            "date": "SliceStart",
            "format": "HH"
          }
        }
      ]
    },
    "external": true,
    "availability": {
      "frequency": "Hour",
      "interval": 1
    },
    "policy": {
      "externalData": {
        "retryInterval": "00:01:00",
        "retryTimeout": "00:10:00",
        "maximumRetry": 3
      }
    }
  }
}
```

**Действие копирования в конвейере с базой данных SQL в качестве источника и файловой системой в качестве приемника**

Конвейер содержит действие копирования, которое использует входной и выходной наборы данных и выполняется каждый час. В определении JSON конвейера для типа **source** установлено значение **SqlSource**, а для типа **sink** — значение **FileSystemSink**. SQL-запрос, указанный для свойства **SqlReaderQuery**, выбирает для копирования данные за последний час.

```JSON
{
  "name":"SamplePipeline",
  "properties":{
    "start":"2015-06-01T18:00:00",
    "end":"2015-06-01T20:00:00",
    "description":"pipeline for copy activity",
    "activities":[
      {
        "name": "AzureSQLtoOnPremisesFile",
        "description": "copy activity",
        "type": "Copy",
        "inputs": [
          {
            "name": "AzureSQLInput"
          }
        ],
        "outputs": [
          {
            "name": "OnpremisesFileSystemOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "SqlSource",
            "SqlReaderQuery": "$$Text.Format('select * from MyTable where timestampcolumn >= \\'{0:yyyy-MM-dd}\\' AND timestampcolumn < \\'{1:yyyy-MM-dd}\\'', WindowStart, WindowEnd)"
          },
          "sink": {
            "type": "FileSystemSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 3,
          "timeout": "01:00:00"
        }
      }
    ]
  }
}
```

Также можно сопоставить столбцы из набора данных, используемого в качестве источника, со столбцами из приемника в определении действия копирования. Дополнительные сведения см. в статье [Сопоставление столбцов исходного набора данных со столбцами целевого набора данных](data-factory-map-columns.md).

## <a name="performance-and-tuning"></a>Производительность и настройка
 Ознакомьтесь с [руководством по настройке производительности действия копирования](data-factory-copy-activity-performance.md), в котором описываются ключевые факторы, влияющие на производительность перемещения данных (действия копирования) в фабрике данных Azure, и различные способы оптимизации этого процесса.
