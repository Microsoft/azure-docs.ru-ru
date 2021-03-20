---
title: Копирование данных из HDFS с помощью фабрики данных Azure
description: Узнайте, как копировать данные из облачного или локального источника HDFS в поддерживаемые хранилища данных-приемники с помощью действия копирования в конвейере фабрики данных Azure.
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 03/17/2021
ms.author: jingwang
ms.openlocfilehash: 9c274bdfb5854529dbb82bd2d8b7cefdf07390b1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104588908"
---
# <a name="copy-data-from-the-hdfs-server-by-using-azure-data-factory"></a>Копирование данных с сервера HDFS с помощью фабрики данных Azure

> [!div class="op_single_selector" title1="Выберите версию используемой службы фабрики данных:"]
> * [Версия 1](v1/data-factory-hdfs-connector.md)
> * [Текущая версия](connector-hdfs.md)

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как скопировать данные с сервера Hadoop распределенная файловая система (HDFS). Дополнительные сведения о Фабрике данных Azure см. во [вводной статье](introduction.md).

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Соединитель HDFS поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с [поддерживаемой матрицей источника и приемника](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)
- [Действие удаления](delete-activity.md)

В частности, соединитель HDFS поддерживает:

- Копирование файлов с помощью *Windows* (Kerberos) или *анонимной* проверки подлинности.
- Копирование файлов с помощью протокола *HDFS* или *встроенной поддержки DistCp* .
- Копирование файлов как есть или путем анализа или создания файлов с [поддерживаемыми форматами файлов и кодеками сжатия](supported-file-formats-and-compression-codecs.md).

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [data-factory-v2-integration-runtime-requirements](../../includes/data-factory-v2-integration-runtime-requirements.md)]

> [!NOTE]
> Убедитесь, что среда выполнения интеграции может обращаться ко *всем* [серверу узла имен]: [порт узла] и [серверы узлов данных]: [порт узла данных] кластера Hadoop. По умолчанию [порт узла имени] имеет значение 50070, а значение по умолчанию [порт узла данных] — 50075.

## <a name="get-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, относящихся к HDFS.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы HDFS поддерживаются следующие свойства:

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Для свойства *Type* необходимо задать значение *HDFS*. | Да |
| url |URL-адрес HDFS |Да |
| authenticationType | Допустимые значения: *anonymous* или *Windows*. <br><br> Сведения о настройке локальной среды см. в разделе [использование проверки подлинности Kerberos для СОЕДИНИТЕЛЯ HDFS](#use-kerberos-authentication-for-the-hdfs-connector) . |Да |
| userName |Имя пользователя для проверки подлинности Windows. Для проверки подлинности Kerberos укажите **\<username> @ \<domain> . com**. |Да (для проверки подлинности Windows) |
| password |Пароль для проверки подлинности Windows. Пометьте это поле как SecureString, чтобы безопасно хранить его в фабрике данных, или [сослаться на секрет, хранящийся в хранилище ключей Azure](store-credentials-in-key-vault.md). |Да (для проверки подлинности Windows) |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites) . Если среда выполнения интеграции не указана, служба использует Azure Integration Runtime по умолчанию. |Нет |

**Пример: использование анонимной проверки подлинности**

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "authenticationType": "Anonymous",
            "userName": "hadoop"
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

**Пример: использование проверки подлинности Windows**

```json
{
    "name": "HDFSLinkedService",
    "properties": {
        "type": "Hdfs",
        "typeProperties": {
            "url" : "http://<machine>:50070/webhdfs/v1/",
            "authenticationType": "Windows",
            "userName": "<username>@<domain>.com (for Kerberos auth)",
            "password": {
                "type": "SecureString",
                "value": "<password>"
            }
        },
        "connectVia": {
            "referenceName": "<name of Integration Runtime>",
            "type": "IntegrationRuntimeReference"
        }
    }
}
```

## <a name="dataset-properties"></a>Свойства набора данных

Полный список разделов и свойств, которые можно использовать для определения наборов данных, см. [в статье DataSets in Azure Data фабрика](concepts-datasets-linked-services.md). 

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Следующие свойства поддерживаются для HDFS в `location` параметрах набора данных на основе формата:

| Свойство.   | Описание                                                  | Обязательно |
| ---------- | ------------------------------------------------------------ | -------- |
| type       | Свойство *Type* в `location` наборе данных должно иметь значение *хдфслокатион*. | Да      |
| folderPath | Путь к папке. Если вы хотите использовать подстановочный знак для фильтрации папки, пропустите этот параметр и укажите путь в параметрах источника действия. | Нет       |
| fileName   | Имя файла с указанным параметром folderPath. Если вы хотите использовать подстановочный знак для фильтрации файлов, пропустите этот параметр и укажите имя файла в параметрах источника действия. | Нет       |

**Пример**.

```json
{
    "name": "DelimitedTextDataset",
    "properties": {
        "type": "DelimitedText",
        "linkedServiceName": {
            "referenceName": "<HDFS linked service name>",
            "type": "LinkedServiceReference"
        },
        "schema": [ < physical schema, optional, auto retrieved during authoring > ],
        "typeProperties": {
            "location": {
                "type": "HdfsLocation",
                "folderPath": "root/folder/subfolder"
            },
            "columnDelimiter": ",",
            "quoteChar": "\"",
            "firstRowAsHeader": true,
            "compressionCodec": "gzip"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, которые можно использовать для определения действий, см. [в статье конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). В этом разделе содержится список свойств, поддерживаемых источником HDFS.

### <a name="hdfs-as-source"></a>HDFS в качестве источника

[!INCLUDE [data-factory-v2-file-formats](../../includes/data-factory-v2-file-formats.md)] 

Следующие свойства поддерживаются для HDFS в `storeSettings` параметрах в источнике копирования на основе формата:

| Свойство.                 | Описание                                                  | Обязательно                                      |
| ------------------------ | ------------------------------------------------------------ | --------------------------------------------- |
| type                     | Свойство *Type* в разделе `storeSettings` должно иметь значение **хдфсреадсеттингс**. | Да                                           |
| ***Размещение файлов для копирования*** |  |  |
| ВАРИАНТ 1. Статический путь<br> | Скопируйте из папки или пути к файлу, указанному в наборе данных. Если вы хотите скопировать все большие файлы из папки, дополнительно укажите `wildcardFileName` как `*`. |  |
| ВАРИАНТ 2. Подстановочный знак<br>- wildcardFolderPath | Путь к папке с подстановочными знаками для фильтрации исходных папок. <br>Допустимые знаки подстановки: `*` (соответствует нулю или нескольким символам) и `?` (соответствует нулю или одному символу). Используйте `^` для экранирования, если фактическое имя папки содержит подстановочный знак или этот escape-символ внутри. <br>Дополнительные примеры см. в разделе [Примеры фильтров файлов и папок](#folder-and-file-filter-examples). | Нет                                            |
| ВАРИАНТ 2. Подстановочный знак<br>- wildcardFileName | Имя файла с подстановочными знаками в указанной folderPath/Вилдкардфолдерпас для фильтрации исходных файлов. <br>Допустимые подстановочные знаки: `*` (соответствует нулю или большему числу символов) и `?` (соответствует нулю или одиночному символу); используйте `^` для экранирования, если фактическое имя файла содержит подстановочный знак или escape-символ внутри.  Дополнительные примеры см. в разделе [Примеры фильтров файлов и папок](#folder-and-file-filter-examples). | Да |
| ВАРИАНТ 3. Список файлов<br>- fileListPath | Указывает, что нужно скопировать указанный набор файлов. Укажите текстовый файл, содержащий список файлов, которые необходимо скопировать (по одному файлу в строке с относительным путем к пути, заданному в наборе данных).<br/>При использовании этого параметра не указывайте имя файла в наборе данных. Дополнительные примеры см. в разделе [примеры списка файлов](#file-list-examples). |Нет |
| ***Дополнительные параметры*** |  | |
| recursive | Указывает, следует ли читать данные рекурсивно из вложенных папок или только из указанной папки. Если `recursive` для параметра задано значение *true* и приемник является хранилищем на основе файлов, пустая папка или подпапка не копируется и не создается в приемнике. <br>Допустимые значения: *true* (по умолчанию) и *false*.<br>Это свойство не применяется при настройке `fileListPath`. |Нет |
| делетефилесафтеркомплетион | Указывает, будут ли удалены двоичные файлы из исходного хранилища после успешного перемещения в целевое хранилище. Файл удаляется для каждого файла, поэтому при сбое действия копирования вы увидите, что некоторые файлы уже скопированы в место назначения и удалены из источника, а остальные остаются в исходном хранилище. <br/>Это свойство допустимо только в сценарии копирования двоичных файлов. Значение по умолчанию: false. |Нет |
| modifiedDatetimeStart    | Файлы фильтруются с учетом *последнего изменения* атрибута. <br>Файлы выбираются, если время последнего изменения находится в диапазоне от `modifiedDatetimeStart` до `modifiedDatetimeEnd` . Время применяется к часовому поясу UTC в формате *2018-12-01T05:00:00Z*. <br> Свойства могут иметь значение NULL, что означает, что фильтр атрибутов файла не применяется к набору данных.  Если `modifiedDatetimeStart` имеет значение DateTime, но `modifiedDatetimeEnd` равно null, это означает, что выбраны файлы, атрибут последнего изменения которых больше или равен значению DateTime.  Если `modifiedDatetimeEnd` имеет значение DateTime, но `modifiedDatetimeStart` равно null, это означает, что выбраны файлы, атрибут последнего изменения которых меньше значения DateTime.<br/>Это свойство не применяется при настройке `fileListPath`. | Нет                                            |
| modifiedDatetimeEnd      | То же, что и выше.  
| енаблепартитиондисковери | Для секционированных файлов укажите, следует ли анализировать секции из пути к файлу и добавить их в качестве дополнительных исходных столбцов.<br/>Допустимые значения: **false** (по умолчанию) и **true**. | Нет                                            |
| партитионрутпас | Если обнаружение секций включено, укажите абсолютный корневой путь для чтения секционированных папок в виде столбцов данных.<br/><br/>Если значение не указано, по умолчанию<br/>— При использовании пути к файлу в наборе данных или списке файлов в источнике корневой путь к разделу — это путь, настроенный в наборе данных.<br/>— При использовании фильтра папки с подстановочными знаками корневой путь к разделу является вложенным путем перед первым подстановочным знаком.<br/><br/>Например, предположим, что вы настроили путь в наборе данных как «root/папка/year = 2020/месяц = 08/Day = 27»:<br/>— Если указать корневой путь к разделу "root/папка/year = 2020", действие копирования создаст еще два столбца `month` и `day` со значением "08" и "27" соответственно в дополнение к столбцам внутри файлов.<br/>— Если корневой путь секции не указан, лишний столбец не будет создан. | Нет                                            |
| maxConcurrentConnections | Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет                                            |
| ***Параметры DistCp*** |  | |
| distcpSettings | Группа свойств, используемая при использовании HDFS DistCp. | Нет |
| resourceManagerEndpoint | Конечная точка YARN (еще другая Resource Negotiator) | Да, при использовании DistCp |
| tempScriptPath | Путь к папке, используемый для хранения командного сценария Temp DistCp. Файл скрипта создается фабрикой данных и будет удален после завершения задания копирования. | Да, при использовании DistCp |
| distcpOptions | Дополнительные параметры для команды DistCp. | Нет |

**Пример**.

```json
"activities":[
    {
        "name": "CopyFromHDFS",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Delimited text input dataset name>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset name>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DelimitedTextSource",
                "formatSettings":{
                    "type": "DelimitedTextReadSettings",
                    "skipLineCount": 10
                },
                "storeSettings":{
                    "type": "HdfsReadSettings",
                    "recursive": true,
                    "distcpSettings": {
                        "resourceManagerEndpoint": "resourcemanagerendpoint:8088",
                        "tempScriptPath": "/usr/hadoop/tempscript",
                        "distcpOptions": "-m 100"
                    }
                }
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="folder-and-file-filter-examples"></a>Примеры фильтров папок и файлов

В этом разделе описывается результирующее поведение при использовании фильтра с подстановочными знаками с путем к папке и именем файла.

| folderPath | fileName             | recursive | Структура исходной папки и результат фильтрации (извлекаются файлы, выделенные **полужирным** шрифтом) |
| :--------- | :------------------- | :-------- | :----------------------------------------------------------- |
| `Folder*`  | (пусто, используйте по умолчанию) | false     | ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Файл2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5.csv<br/>Другая_папкаB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл6.csv |
| `Folder*`  | (пусто, используйте по умолчанию) | Да      | ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Файл2.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл4.json**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл5.csv**<br/>Другая_папкаB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл6.csv |
| `Folder*`  | `*.csv`              | false     | ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3.csv<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5.csv<br/>Другая_папкаB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл6.csv |
| `Folder*`  | `*.csv`              | Да      | ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл5.csv**<br/>Другая_папкаB<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл6.csv |

### <a name="file-list-examples"></a>Примеры списков файлов

В этом разделе описывается поведение, полученное в результате использования пути к списку файлов в источнике действия копирования. Предполагается, что у вас есть следующая исходная структура папок и вы хотите скопировать файлы, которые имеют полужирный тип:

| Пример исходной структуры                                      | Содержимое файла FileListToCopy.txt                             | Настройка фабрики данных Azure                                            |
| ------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| root<br/>&nbsp;&nbsp;&nbsp;&nbsp;ПапкаA<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл1.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл2.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл3.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4.json<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Файл5.csv**<br/>&nbsp;&nbsp;&nbsp;&nbsp;Метаданные<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FileListToCopy.txt | Файл1.csv<br>Вложенная_папка1/Файл3.csv<br>Вложенная_папка1/Файл5.csv | **В наборе данных:**<br>— Путь к папке: `root/FolderA`<br><br>**В источнике действия копирования:**<br>— Путь к списку файлов: `root/Metadata/FileListToCopy.txt` <br><br>Путь к списку файлов указывает на текстовый файл в том же хранилище данных, который содержит список файлов, которые необходимо скопировать (по одному файлу в строке с относительным путем к пути, настроенному в наборе данных). |

## <a name="use-distcp-to-copy-data-from-hdfs"></a>Использовать DistCp для копирования данных из HDFS

[DistCp](https://hadoop.apache.org/docs/current3/hadoop-distcp/DistCp.html) — это собственная программа командной строки Hadoop для выполнения распределенного копирования в кластере Hadoop. При выполнении команды в DistCp сначала выводится список всех копируемых файлов, а затем в кластере Hadoop создается несколько заданий Map. Каждое задание сопоставлений выполняет двоичную копию из источника в приемник.

Действие копирования поддерживает использование DistCp для копирования файлов в хранилище BLOB-объектов Azure (включая [промежуточное копирование](copy-activity-performance.md)) или Azure Data Lake Store. В этом случае DistCp может воспользоваться возможностями кластера, а не работать в локальной среде выполнения интеграции. Использование DistCp обеспечивает лучшую пропускную способность копирования, особенно если ваш кластер очень мощный. На основе конфигурации в фабрике данных действие копирования автоматически создаст команду DistCp, отправляет ее в кластер Hadoop и отслеживает состояние копирования.

### <a name="prerequisites"></a>Предварительные требования

Чтобы использовать DistCp для копирования файлов в виде из HDFS в хранилище BLOB-объектов Azure (включая промежуточную копию) или Azure Data Lake Store, убедитесь, что кластер Hadoop соответствует следующим требованиям.

* Службы MapReduce и YARN включены.  
* YARN версия 2,5 или более поздняя.  
* Сервер HDFS интегрирован с целевым хранилищем данных: **хранилище BLOB-объектов Azure** или **Azure Data Lake Store (ADLS 1-го поколения)**: 

    - Начиная с версии Hadoop 2.7, изначально поддерживается файловая система больших двоичных объектов Azure. Необходимо только указать путь JAR в конфигурации среды Hadoop.
    - Файловая система Azure Data Lake Store упаковывается, начиная с Hadoop версии 3.0.0-alpha1. Если версия кластера Hadoop более ранняя, чем версия, необходимо вручную импортировать связанные с Azure Data Lake Store пакеты JAR (Азуре-даталаке-Сторе. jar) в кластер [отсюда](https://hadoop.apache.org/releases.html), а также указать путь к JAR-файлу в конфигурации среды Hadoop.

* Подготовьте временную папку в HDFS. Эта временная папка используется для хранения сценария оболочки DistCp, поэтому она займет пространство уровня KB.
* Убедитесь, что учетная запись пользователя, предоставленная в связанной службе HDFS, имеет разрешение на:
   * Отправьте приложение в YARN.
   * Создайте вложенную папку и файлы для чтения и записи в папке Temp.

### <a name="configurations"></a>Конфигурации

Для конфигураций и примеров, связанных с DistCp, перейдите в раздел [HDFS в качестве источника](#hdfs-as-source) .

## <a name="use-kerberos-authentication-for-the-hdfs-connector"></a>Использовать проверку подлинности Kerberos для соединителя HDFS

Существует два варианта настройки локальной среды для использования проверки подлинности Kerberos для соединителя HDFS. Вы можете выбрать тот, который лучше подходит для вашей ситуации.
* Вариант 1. [Присоединение компьютера с локальной средой выполнения интеграции в области Kerberos](#kerberos-join-realm)
* Вариант 2. [Включение взаимного доверия между доменом Windows и областью Kerberos](#kerberos-mutual-trust).

Для любого из этих вариантов убедитесь, что вы включаете для кластера Hadoop поддержку:

1. Создайте субъект HTTP и keytab для HDFS.

    > [!IMPORTANT]
    > Участник HTTP Kerberos должен начинаться с "**http/**" в соответствии со спецификацией Kerberos HTTP SPNEGO. Дополнительные сведения см. [здесь](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/WebHDFS.html#HDFS_Configuration_Options).

    ```bash
    Kadmin> addprinc -randkey HTTP/<namenode hostname>@<REALM.COM>
    Kadmin> ktadd -k /etc/security/keytab/spnego.service.keytab HTTP/<namenode hostname>@<REALM.COM>
    ```

2. Параметры конфигурации HDFS: добавьте следующие три свойства в `hdfs-site.xml` .
    ```xml
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.web.authentication.kerberos.principal</name>
        <value>HTTP/_HOST@<REALM.COM></value>
    </property>
    <property>
        <name>dfs.web.authentication.kerberos.keytab</name>
        <value>/etc/security/keytab/spnego.service.keytab</value>
    </property>
    ```

### <a name="option-1-join-a-self-hosted-integration-runtime-machine-in-the-kerberos-realm"></a><a name="kerberos-join-realm"></a>Вариант 1. Присоединение компьютера с локальной средой выполнения интеграции в области Kerberos

#### <a name="requirements"></a>Требования

* Локальная среда выполнения интеграции должна быть присоединена к области Kerberos и не может присоединяться к какому-либо домену Windows.

#### <a name="how-to-configure"></a>Порядок настройки

**На сервере KDC сделайте следующее:**

Создайте субъект, который будет использоваться фабрикой данных Azure, и укажите пароль.

> [!IMPORTANT]
> Имя пользователя не должно содержать имя узла.

```bash
Kadmin> addprinc <username>@<REALM.COM>
```

**На компьютере, где размещена локальная среда выполнения интеграции:**

1.  Запустите служебную программу Ksetup, чтобы настроить сервер центра распространения ключей (KDC) и область Kerberos.

    Компьютер должен быть настроен в качестве члена Рабочей группы, так как область Kerberos отличается от домена Windows. Эту конфигурацию можно достичь, задав область Kerberos и добавив сервер KDC, выполнив следующие команды. Замените *realm.com* именем своей области.

    ```cmd
    C:> Ksetup /setdomain REALM.COM
    C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
    ```

    После выполнения этих команд перезапустите компьютер.

2.  Проверьте конфигурацию с помощью `Ksetup` команды. Результат должен выглядеть примерно так:

    ```cmd
    C:> Ksetup
    default realm = REALM.COM (external)
    REALM.com:
        kdc = <your_kdc_server_address>
    ```

**В фабрике данных:**

* Настройте соединитель HDFS с помощью проверки подлинности Windows вместе с именем и паролем участника Kerberos для подключения к источнику данных HDFS. Дополнительные сведения о конфигурации см. в разделе [Свойства связанной службы HDFS](#linked-service-properties) .

### <a name="option-2-enable-mutual-trust-between-the-windows-domain-and-the-kerberos-realm"></a><a name="kerberos-mutual-trust"></a>Вариант 2. Включение взаимного доверия между доменом Windows и областью Kerberos

#### <a name="requirements"></a>Требования

*   Компьютер, на котором размещена локальная среда выполнения интеграции, должен быть присоединен к домену Windows.
*   Требуется разрешение на обновление параметров контроллера домена.

#### <a name="how-to-configure"></a>Порядок настройки

> [!NOTE]
> Замените REALM.COM и AD.COM в следующем руководстве на имя своей области и контроллер домена.

**На сервере KDC сделайте следующее:**

1. Измените конфигурацию KDC в файле *krb5. conf* , чтобы разрешить KDC доверять домену Windows, обратившись к следующему шаблону конфигурации. По умолчанию файл конфигурации содержится в папке */etc/krb5.conf*.

   ```config
   [logging]
    default = FILE:/var/log/krb5libs.log
    kdc = FILE:/var/log/krb5kdc.log
    admin_server = FILE:/var/log/kadmind.log
            
   [libdefaults]
    default_realm = REALM.COM
    dns_lookup_realm = false
    dns_lookup_kdc = false
    ticket_lifetime = 24h
    renew_lifetime = 7d
    forwardable = true
            
   [realms]
    REALM.COM = {
     kdc = node.REALM.COM
     admin_server = node.REALM.COM
    }
   AD.COM = {
    kdc = windc.ad.com
    admin_server = windc.ad.com
   }
            
   [domain_realm]
    .REALM.COM = REALM.COM
    REALM.COM = REALM.COM
    .ad.com = AD.COM
    ad.com = AD.COM
            
   [capaths]
    AD.COM = {
     REALM.COM = .
    }
    ```

   После настройки файла перезапустите службу KDC.

2. Подготовьте субъект с именем *KRBTGT/realm. COM \@ AD.com* на сервере KDC с помощью следующей команды:

    ```cmd
    Kadmin> addprinc krbtgt/REALM.COM@AD.COM
    ```

3. В файл конфигурации службы HDFS *hadoop.security.auth_to_local* добавьте `RULE:[1:$1@$0](.*\@AD.COM)s/\@.*//`.

**На контроллере домена сделайте следующее:**

1.  `Ksetup`Чтобы добавить запись области, выполните следующие команды:

    ```cmd
    C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
    C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM
    ```

2.  Установите доверие между доменом Windows и областью Kerberos. [password] представляет собой пароль для субъекта *krbtgt/REALM.COM\@AD.COM*.

    ```cmd
    C:> netdom trust REALM.COM /Domain: AD.COM /add /realm /password:[password]
    ```

3.  Выберите алгоритм шифрования, используемый в Kerberos.

    а. Выберите **Диспетчер сервера**  >  **Групповая политика**  >  **домен** управления  >  **Групповая политика**  >  **политики по умолчанию или активная политика домена**, а затем щелкните **изменить**.

    b. На панели **редактор "Управление групповыми политиками"** выберите **Конфигурация компьютера**  >  **политики** параметры  >  **Windows настройки**  >  **безопасности**  >  **Локальные политики**  >  **Параметры безопасности**, а затем настройте **безопасность сети: Настройка типов шифрования, разрешенных для Kerberos**.

    c. Выберите алгоритм шифрования, который будет использоваться при подключении к серверу KDC. Можно выбрать все параметры.

    ![Снимок экрана: панель "Сетевая безопасность. Настройка типов шифрования, разрешенных для Kerberos"](media/connector-hdfs/config-encryption-types-for-kerberos.png)

    d. Используйте `Ksetup` команду, чтобы указать алгоритм шифрования, используемый в указанной области.

    ```cmd
    C:> ksetup /SetEncTypeAttr REALM.COM DES-CBC-CRC DES-CBC-MD5 RC4-HMAC-MD5 AES128-CTS-HMAC-SHA1-96 AES256-CTS-HMAC-SHA1-96
    ```

4.  Создайте сопоставление между учетной записью домена и участником Kerberos, чтобы можно было использовать субъект Kerberos в домене Windows.

    а. Выберите **Администрирование**  >  **Active Directory пользователи и компьютеры**.

    b. Настройте дополнительные функции, выбрав **Вид** > **Дополнительные параметры**.

    c. На панели **Дополнительные функции** щелкните правой кнопкой мыши учетную запись, для которой нужно создать сопоставления, и на панели **сопоставления имен** выберите вкладку **имена Kerberos** .

    d. Добавьте субъект из области.

       ![Панель "сопоставление идентификаторов безопасности"](media/connector-hdfs/map-security-identity.png)

**На компьютере, где размещена локальная среда выполнения интеграции:**

* Выполните следующие `Ksetup` команды, чтобы добавить запись области.

   ```cmd
   C:> Ksetup /addkdc REALM.COM <your_kdc_server_address>
   C:> ksetup /addhosttorealmmap HDFS-service-FQDN REALM.COM
   ```

**В фабрике данных:**

* Настройте соединитель HDFS с помощью проверки подлинности Windows вместе с учетной записью домена или субъекта Kerberos для подключения к источнику данных HDFS. Дополнительные сведения о конфигурации см. в разделе [Свойства связанной службы HDFS](#linked-service-properties) .

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Сведения о свойствах действия поиска см. [в разделе действие поиска в фабрике данных Azure](control-flow-lookup-activity.md).

## <a name="delete-activity-properties"></a>Свойства действия удаления

Сведения о свойствах действия удаления см. [в разделе действие удаления в фабрике данных Azure](delete-activity.md).

## <a name="legacy-models"></a>Устаревшие модели

>[!NOTE]
>Следующие модели по-прежнему поддерживаются в соответствии с обратной совместимостью. Рекомендуется использовать ранее описанную новую модель, так как пользовательский интерфейс создания фабрики данных Azure переключился на создание новой модели.

### <a name="legacy-dataset-model"></a>Устаревшая модель набора данных

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство *Type* набора данных должно *иметь значение общая* папка |Да |
| folderPath | Путь к папке. Поддерживается фильтр с подстановочными знаками. Разрешенные подстановочные знаки `*` (соответствует нулю или нескольким символам) и `?` (соответствует нулю или одному символу); используйте `^` для экранирования, если фактическое имя файла содержит подстановочный знак или escape-символ внутри. <br/><br/>Примеры: rootfolder/subfolder/. Дополнительные примеры см. в разделе [Примеры фильтров папок и файлов](#folder-and-file-filter-examples). |Да |
| fileName |  Имя или фильтр с подстановочными знаками для файлов с указанной "folderPath". Если этому свойству не присвоить значение, набор данных будет указывать на все файлы в папке. <br/><br/>Для фильтра разрешенные подстановочные знаки `*` (соответствует нулю или более) и `?` (соответствует нулю или одному символу).<br/>Пример 1. `"fileName": "*.csv"`<br/>Пример 2. `"fileName": "???20180427.txt"`<br/>Используйте `^` для экранирования, если фактическое имя папки содержит подстановочный знак или этот escape-символ внутри. |Нет |
| modifiedDatetimeStart | Файлы фильтруются с учетом *последнего изменения* атрибута. Файлы выбираются, если время последнего изменения находится в диапазоне от `modifiedDatetimeStart` до `modifiedDatetimeEnd` . Время применяется к часовому поясу UTC в формате *2018-12-01T05:00:00Z*. <br/><br/> Имейте в виду, что включение этого параметра влияет на общую производительность перемещения данных, если требуется применить фильтр файлов к большому числу файлов. <br/><br/> Свойства могут иметь значение NULL, что означает, что фильтр атрибутов файла не применяется к набору данных.  Если `modifiedDatetimeStart` имеет значение DateTime, но `modifiedDatetimeEnd` равно null, это означает, что выбраны файлы, атрибут последнего изменения которых больше или равен значению DateTime.  Если `modifiedDatetimeEnd` имеет значение DateTime, но `modifiedDatetimeStart` равно null, это означает, что выбраны файлы, атрибут последнего изменения которых меньше значения DateTime.| Нет |
| modifiedDatetimeEnd | Файлы фильтруются с учетом *последнего изменения* атрибута. Файлы выбираются, если время последнего изменения находится в диапазоне от `modifiedDatetimeStart` до `modifiedDatetimeEnd` . Время применяется к часовому поясу UTC в формате *2018-12-01T05:00:00Z*. <br/><br/> Имейте в виду, что включение этого параметра влияет на общую производительность перемещения данных, если требуется применить фильтр файлов к большому числу файлов. <br/><br/> Свойства могут иметь значение NULL, что означает, что фильтр атрибутов файла не применяется к набору данных.  Если `modifiedDatetimeStart` имеет значение DateTime, но `modifiedDatetimeEnd` равно null, это означает, что выбраны файлы, атрибут последнего изменения которых больше или равен значению DateTime.  Если `modifiedDatetimeEnd` имеет значение DateTime, но `modifiedDatetimeStart` равно null, это означает, что выбраны файлы, атрибут последнего изменения которых меньше значения DateTime.| Нет |
| format | Если требуется скопировать файлы между файловыми хранилищами "как есть" (двоичное копирование), можно пропустить раздел форматирования в определениях входного и выходного наборов данных.<br/><br/>Если необходимо проанализировать файлы определенного формата, поддерживаются следующие форматы файлов: *TextFormat*, *JsonFormat*, *AvroFormat*, *OrcFormat*, *ParquetFormat*. Свойству *type* в разделе format необходимо присвоить одно из этих значений. Дополнительные сведения см. в разделах о [текстовом формате](supported-file-formats-and-compression-codecs-legacy.md#text-format), [формате JSON](supported-file-formats-and-compression-codecs-legacy.md#json-format), [формате Avro](supported-file-formats-and-compression-codecs-legacy.md#avro-format), [формате ORC](supported-file-formats-and-compression-codecs-legacy.md#orc-format) и [формате Parquet](supported-file-formats-and-compression-codecs-legacy.md#parquet-format). |Нет (только для сценария двоичного копирования) |
| compression | Укажите тип и уровень сжатия данных. Дополнительные сведения см. в разделе [Поддержка сжатия](supported-file-formats-and-compression-codecs-legacy.md#compression-support).<br/>Поддерживаются следующие типы: *gzip*, *Deflate*, *bzip2* и *ZipDeflate*.<br/>Поддерживаемые уровни: *Оптимальный* и *Самый быстрый*. |Нет |

>[!TIP]
>Чтобы скопировать все файлы в папке, укажите только **folderPath**.<br>Чтобы скопировать отдельный файл с указанным именем, укажите параметр **FolderPath** с именем файла, который является частью папки, а **имя_файла** .<br>Чтобы скопировать подмножество файлов в папке, укажите **folderPath** с частью папки и **fileName** с фильтром подстановочных знаков.

**Пример**.

```json
{
    "name": "HDFSDataset",
    "properties": {
        "type": "FileShare",
        "linkedServiceName":{
            "referenceName": "<HDFS linked service name>",
            "type": "LinkedServiceReference"
        },
        "typeProperties": {
            "folderPath": "folder/subfolder/",
            "fileName": "*",
            "modifiedDatetimeStart": "2018-12-01T05:00:00Z",
            "modifiedDatetimeEnd": "2018-12-01T06:00:00Z",
            "format": {
                "type": "TextFormat",
                "columnDelimiter": ",",
                "rowDelimiter": "\n"
            },
            "compression": {
                "type": "GZip",
                "level": "Optimal"
            }
        }
    }
}
```

### <a name="legacy-copy-activity-source-model"></a>Исходная модель действия копирования прежних версий

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство *Type* источника действия копирования должно иметь значение *HdfsSource*. |Да |
| recursive | Указывает, следует ли читать данные рекурсивно из вложенных папок или только из указанной папки. Если параметру recursive присвоено значение *true* , а приемник является хранилищем на основе файлов, то пустая папка или подпапка не будет скопирована или создана в приемнике.<br/>Допустимые значения: *true* (по умолчанию) и *false*. | Нет |
| distcpSettings | Группа свойств, когда вы используете HDFS DistCp. | Нет |
| resourceManagerEndpoint | Конечная точка диспетчер ресурсов YARN | Да, при использовании DistCp |
| tempScriptPath | Путь к папке, используемый для хранения командного сценария Temp DistCp. Файл скрипта создается фабрикой данных и будет удален после завершения задания копирования. | Да, при использовании DistCp |
| distcpOptions | Для команды DistCp предоставляются дополнительные параметры. | Нет |
| maxConcurrentConnections | Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет |

**Пример: источник HDFS в действии копирования с помощью DistCp**

```json
"source": {
    "type": "HdfsSource",
    "distcpSettings": {
        "resourceManagerEndpoint": "resourcemanagerendpoint:8088",
        "tempScriptPath": "/usr/hadoop/tempscript",
        "distcpOptions": "-m 100"
    }
}
```

## <a name="next-steps"></a>Дальнейшие действия
Список хранилищ данных, которые поддерживаются в качестве источников и приемников действием копирования в фабрике данных Azure, см. в разделе [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats).
