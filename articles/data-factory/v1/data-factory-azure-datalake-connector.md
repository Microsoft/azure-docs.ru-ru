---
title: Копирование данных в Azure Data Lake Storage 1-го поколения и из них
description: Узнайте, как с помощью фабрики данных Azure копировать данные в Data Lake Store и обратно
author: linda33wj
ms.service: data-factory
ms.topic: conceptual
ms.date: 01/22/2018
ms.author: jingwang
ms.custom: devx-track-csharp
robots: noindex
ms.openlocfilehash: 8bdfceff562ae2501a9d95774f7134669e929b7e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100379409"
---
# <a name="copy-data-to-and-from-data-lake-storage-gen1-by-using-data-factory"></a>Копирование данных в Azure Data Lake Storage Gen1 и обратно с помощью фабрики данных
> [!div class="op_single_selector" title1="Выберите используемую версию службы "Фабрика данных":"]
> * [Версия 1](data-factory-azure-datalake-connector.md)
> * [Версия 2 (текущая)](../connector-azure-data-lake-store.md)

> [!NOTE]
> В этой статье рассматривается служба "Фабрика данных Azure" версии 1. Если вы используете текущую версию Фабрики данных, см. раздел [Соединитель Azure Data Lake Storage Gen1 версии 2](../connector-azure-data-lake-store.md).

В этой статье описывается, как с помощью действия копирования в фабрике данных Azure перемещать данные из Azure Data Lake Storage Gen1 (ранее Azure Data Lake Store) и в него. Это продолжение статьи о [действиях перемещения данных](data-factory-data-movement-activities.md), в которой приведены общие сведения о перемещении данных с помощью действия копирования.

## <a name="supported-scenarios"></a>Поддерживаемые сценарии
Данные можно скопировать **из Azure Data Lake Store** в следующие хранилища данных:

[!INCLUDE [data-factory-supported-sinks](../../../includes/data-factory-supported-sinks.md)]

Данные можно скопировать **в Azure Data Lake Store** из следующих хранилищ данных:

[!INCLUDE [data-factory-supported-sources](../../../includes/data-factory-supported-sources.md)]

> [!NOTE]
> Прежде чем создавать конвейер с действием копирования, создайте учетную запись Data Lake Store. Дополнительные сведения см. в статье [Начало работы с хранилищем озера данных Azure с помощью портала Azure](../../data-lake-store/data-lake-store-get-started-portal.md).

## <a name="supported-authentication-types"></a>Поддерживаемые типы аутентификации
Соединитель Data Lake Store поддерживает следующие типы проверки подлинности:
* Проверка подлинности субъекта-службы
* Проверка подлинности учетных данных пользователя (OAuth)

Мы рекомендуем использовать проверку подлинности субъекта-службы, особенно для копирования данных по расписанию. При проверке подлинности учетных данных пользователя может истечь срок действия маркера. Сведения о настройке см. в разделе [Свойства связанной службы](#linked-service-properties).

## <a name="get-started"></a>Начало работы
Можно создать конвейер с действием копирования, которое перемещает данные из Azure Data Lake Store или обратно с помощью различных инструментов и интерфейсов API.

Проще всего создать конвейер для копирования данных с помощью **мастера копирования**. В статье [Руководство. Создание конвейера с действием копирования с помощью мастера копирования фабрики данных](data-factory-copy-data-wizard-tutorial.md) приведены указания по созданию конвейера с помощью мастера копирования данных.

Для создания конвейера можно также использовать следующие средства: **Visual Studio**, **Azure PowerShell**, **Azure Resource Manager шаблон**, **API .NET** и **REST API**. Пошаговые инструкции по созданию конвейера с действием копирования см. в разделе [учебник по действиям копирования](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md) .

Независимо от используемого средства или API-интерфейса, для создания конвейера, который перемещает данные из источника данных в приемник, выполняются следующие шаги:

1. Создайте **фабрику данных**. Фабрика данных может содержать один или несколько конвейеров.
2. Создайте **связанные службы** , чтобы связать входные и выходные данные хранилища с фабрикой данных. Например, при копировании данных из хранилища BLOB-объектов Azure в Azure Data Lake Store создайте две связанные службы, чтобы связать учетную запись хранения Azure и Azure Data Lake Store с фабрикой данных. Сведения о свойствах связанной службы, относящихся к Azure Data Lake Store, см. в разделе [Свойства связанной службы](#linked-service-properties).
2. Создайте **наборы данных**, которые представляют входные и выходные данные для операции копирования. В примере, упомянутом на последнем этапе, создайте набор данных, чтобы указать контейнер BLOB-объектов и папку, содержащую входные данные. Также создайте другой набор данных, чтобы указать папку, содержащую данные, скопированные из хранилища BLOB-объектов, и путь к ней в Data Lake Store. Сведения о свойствах набора данных, относящихся к Azure Data Lake Store, см. в разделе [Свойства набора данных](#dataset-properties).
3. Создайте **конвейер** с действием копирования, который принимает входной набор данных и возвращает выходной набор данных. В примере выше BlobSource используется как источник, а AzureDataLakeStoreSink — как приемник для действия копирования. Аналогично, при копировании из Azure Data Lake Store в хранилище BLOB-объектов Azure в действии копирования используются AzureDataLakeStoreSource и BlobSink. Сведения о свойствах действия копирования, относящихся к Azure Data Lake Store, см. в разделе [Свойства действия копирования](#copy-activity-properties). Для получения сведений о том, как использовать хранилище данных в качестве источника или приемника, щелкните ссылку в предыдущем разделе об источнике данных.

Если вы используете мастер, то он автоматически создает определения JSON для сущностей фабрики данных (связанных служб, наборов данных и конвейера). При использовании инструментов и интерфейсов API (за исключением API .NET) вы самостоятельно определяете эти сущности фабрики данных в формате JSON. Примеры с определениями JSON для сущностей фабрики данных, которые используются для копирования данных из Azure Data Lake Store, см. в разделе [Примеры определений JSON](#json-examples-for-copying-data-to-and-from-data-lake-store) этой статьи.

Следующие разделы содержат сведения о свойствах JSON, которые используются для определения сущностей фабрики данных, характерных для Data Lake Store.

## <a name="linked-service-properties"></a>Свойства связанной службы
Связанная служба привязывает хранилище данных к фабрике данных. Для связи Data Lake Store с фабрикой данных следует создать связанную службу типа **AzureDataLakeStore**. В приведенной ниже таблице описываются элементы JSON, которые относятся к связанным службам Data Lake Store. Вы можете выбирать между проверкой подлинности субъекта-службы и учетных данных пользователя.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| **type** | Для свойства type следует задать значение **AzureDataLakeStore** | Да |
| **dataLakeStoreUri** | Сведения об учетной записи Azure Data Lake Store. Эти данные принимают один из следующих форматов: `https://[accountname].azuredatalakestore.net/webhdfs/v1` или `adl://[accountname].azuredatalakestore.net/`. | Да |
| **subscriptionId** | Идентификатор подписки Azure, к которой принадлежит учетная запись Data Lake Store. | Необходимо для приемника |
| **resourceGroupName** | Имя группы ресурсов Azure, к которой принадлежит учетная запись Data Lake Store. | Необходимо для приемника |

### <a name="service-principal-authentication-recommended"></a>Проверка подлинности на основе субъекта-службы (рекомендуется)
При использовании проверки подлинности на основе субъекта-службы необходимо зарегистрировать сущность приложения в Azure Active Directory (Azure AD) и предоставить ей доступ к Data Lake Store. Подробные инструкции см. в статье [Аутентификация между службами в Data Lake Store с помощью Azure Active Directory](../../data-lake-store/data-lake-store-service-to-service-authenticate-using-active-directory.md). Запишите следующие значения, которые используются для определения связанной службы:
* Идентификатор приложения
* Ключ приложения
* Tenant ID

> [!IMPORTANT]
> Убедитесь, что в Azure Data Lake Store субъекту-службе предоставлено правильное разрешение.
>- **Чтобы использовать Data Lake Store в качестве источника**, предоставьте разрешения на доступ к данным по крайней мере на **чтение и выполнение** , чтобы перечислить и скопировать содержимое папки, или разрешение на **Чтение** для копирования одного файла. Управление доступом на уровне учетной записи не требуется.
>- **Чтобы использовать Data Lake Store в качестве приемника**, предоставьте разрешения на доступ к данным по крайней мере на **запись и выполнение** для создания дочерних элементов в папке. Если для расширения возможностей копирования используется среда IR Azure (источник и приемник находятся в облаке), чтобы фабрика данных смогла определить регион Data Lake Store, назначьте в Системе управления идентификацией и доступом (IAM) роль по крайней мере **читателя**. Чтобы не назначать эту роль IAM, определите [executionLocation](data-factory-data-movement-activities.md#global) с расположением Data Lake Store в действии копирования.
>- Если **для создания конвейеров используется мастер копирования**, предоставьте как минимум роль **читателя** в контроле доступа к учетным записям (IAM). Кроме того, предоставьте разрешение по крайней мере на **чтение и выполнение** в корневом каталоге Data Lake Store (/) и его дочерних элементах. Иначе может появиться сообщение "Указаны недопустимые учетные данные".

Используйте проверку подлинности на основе субъекта-службы, указав следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| **servicePrincipalId** | Укажите идентификатора клиента приложения. | Да |
| **servicePrincipalKey** | Укажите ключ приложения. | Да |
| **tenant** | Укажите сведения о клиенте (доменное имя или идентификатор клиента), в котором находится приложение. Эти сведения можно получить, наведя указатель мыши на правый верхний угол страницы портала Azure. | Да |

**Пример. Проверка подлинности субъекта-службы**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

### <a name="user-credential-authentication"></a>Использование проверки подлинности на основе учетных данных пользователя
Также для копирования данных в Data Lake Store и обратно можно использовать проверку подлинности на основе учетных данных пользователя, указав приведенные ниже свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| **проверки** | Нажмите кнопку **Авторизовать** в редакторе фабрики данных и введите учетные данные. URL-адрес авторизации будет создан автоматически и присвоен этому свойству. | Да |
| **sessionId** | Идентификатор сеанса OAuth из сеанса авторизации OAuth. Каждый идентификатор сеанса является уникальным и используется только один раз. Этот параметр создается автоматически при использовании редактора фабрики данных. | Да |

> [!IMPORTANT]
> Убедитесь, что в Azure Data Lake Store пользователю предоставлено правильное разрешение.
>- **Чтобы использовать Data Lake Store в качестве источника**, предоставьте разрешения на доступ к данным по крайней мере на **чтение и выполнение** , чтобы перечислить и скопировать содержимое папки, или разрешение на **Чтение** для копирования одного файла. Управление доступом на уровне учетной записи не требуется.
>- **Чтобы использовать Data Lake Store в качестве приемника**, предоставьте разрешения на доступ к данным по крайней мере на **запись и выполнение** для создания дочерних элементов в папке. Если для расширения возможностей копирования используется среда IR Azure (источник и приемник находятся в облаке), чтобы фабрика данных смогла определить регион Data Lake Store, назначьте в Системе управления идентификацией и доступом (IAM) роль по крайней мере **читателя**. Чтобы не назначать эту роль IAM, определите [executionLocation](data-factory-data-movement-activities.md#global) с расположением Data Lake Store в действии копирования.
>- Если **для создания конвейеров используется мастер копирования**, предоставьте как минимум роль **читателя** в контроле доступа к учетным записям (IAM). Кроме того, предоставьте разрешение по крайней мере на **чтение и выполнение** в корневом каталоге Data Lake Store (/) и его дочерних элементах. Иначе может появиться сообщение "Указаны недопустимые учетные данные".

**Пример. Использование проверки подлинности на основе учетных данных пользователя**
```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "sessionId": "<session ID>",
            "authorization": "<authorization URL>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

#### <a name="token-expiration"></a>Срок действия маркера
Срок действия кода авторизации, созданного с помощью кнопки **Авторизовать**, через некоторое время истекает. Следующее сообщение означает, что срок действия маркера проверки подлинности истек.

"Произошла ошибка при операции с учетными данными: invalid_grant — AADSTS70002. Ошибка при проверке учетных данных. AADSTS70008: срок действия предоставленных прав доступа истек или они были отозваны. Идентификатор отслеживания: d18629e8-af88-43c5-88e3-d8419eb1fca1 Идентификатор корреляции: fac30a0c-6be6-4e02-8d69-a776d2ffefd7 Метка времени: 2015-12-15 21-09-31Z".

Сроки действия для различных типов учетных записей пользователей см. в следующей таблице.

| Тип пользователя | Срок действия |
|:--- |:--- |
| Учетные записи пользователей, которые *не* управляются с помощью Azure Active Directory (например, @hotmail.com или @live.com). |12 часов |
| Учетные записи пользователей, которые управляются с помощью Azure Active Directory |14 дней после последнего выполнения среза. <br/><br/>90 дней, если срез, основанный на связанной службе на основе OAuth, выполняется по крайней мере раз в 14 дней. |

Если пароль изменяется прежде, чем срок действия маркера истечет, маркер мгновенно устаревает. Появится сообщение, упомянутое ранее в этом разделе.

Когда срок действия маркера истечет, вы можете повторно авторизовать учетную запись с помощью кнопки **Авторизовать** и повторно развернуть связанную службу. Значения свойств **sessionId** и **authorization** также можно задавать программно с помощью следующего кода:


```csharp
if (linkedService.Properties.TypeProperties is AzureDataLakeStoreLinkedService ||
    linkedService.Properties.TypeProperties is AzureDataLakeAnalyticsLinkedService)
{
    AuthorizationSessionGetResponse authorizationSession = this.Client.OAuth.Get(this.ResourceGroupName, this.DataFactoryName, linkedService.Properties.Type);

    WindowsFormsWebAuthenticationDialog authenticationDialog = new WindowsFormsWebAuthenticationDialog(null);
    string authorization = authenticationDialog.AuthenticateAAD(authorizationSession.AuthorizationSession.Endpoint, new Uri("urn:ietf:wg:oauth:2.0:oob"));

    AzureDataLakeStoreLinkedService azureDataLakeStoreProperties = linkedService.Properties.TypeProperties as AzureDataLakeStoreLinkedService;
    if (azureDataLakeStoreProperties != null)
    {
        azureDataLakeStoreProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeStoreProperties.Authorization = authorization;
    }

    AzureDataLakeAnalyticsLinkedService azureDataLakeAnalyticsProperties = linkedService.Properties.TypeProperties as AzureDataLakeAnalyticsLinkedService;
    if (azureDataLakeAnalyticsProperties != null)
    {
        azureDataLakeAnalyticsProperties.SessionId = authorizationSession.AuthorizationSession.SessionId;
        azureDataLakeAnalyticsProperties.Authorization = authorization;
    }
}
```
Подробные сведения о классах фабрики данных, используемых в коде, см. в статьях, посвященных классам [AzureDataLakeStoreLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakestorelinkedservice), [AzureDataLakeAnalyticsLinkedService](/dotnet/api/microsoft.azure.management.datafactories.models.azuredatalakeanalyticslinkedservice) и [AuthorizationSessionGetResponse](/dotnet/api/microsoft.azure.management.datafactories.models.authorizationsessiongetresponse). Добавьте ссылку на версию `2.9.10826.1824``Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll` для класса `WindowsFormsWebAuthenticationDialog`, используемого в коде.

## <a name="troubleshooting-tips"></a>Советы по устранению неполадок

**Симптом.** При копировании данных **в** Azure Data Lake Store действие копирования завершается следующей ошибкой:

  ```
  Failed to detect the region for Azure Data Lake account {your account name}. Please make sure that the Resource Group name: {resource group name} and subscription ID: {subscription ID} of this Azure Data Lake Store resource are correct.
  ```

**Первопричина.** Есть две возможные причины:

1. В связанной службе Azure Data Lake Store значения `resourceGroupName` и (или) `subscriptionId` указаны неправильно.
2. Пользователю или субъекту-службе не предоставлены необходимые разрешения.

**Решение:**

1. Убедитесь, что для параметров `subscriptionId` и `resourceGroupName`, указанных в связанной службе `typeProperties`, заданы та группа ресурсов и подписка, к которым принадлежит ваша учетная запись Data Lake.

2. Убедитесь, что пользователю или субъекту-службе предоставлена по крайней мере роль **читателя** в учетной записи Data Lake. Вот как это можно сделать:

    1. Перейдите к портал Azure-> учетной записи Data Lake Store
    2. В колонке Data Lake Store щелкните **Управление доступом (IAM)**.
    3. Нажмите кнопку **добавить назначение ролей** .
    4. Чтобы предоставить доступ, задайте **роли** значение **Читатель** и выберите пользователя или субъект-службу, используемые для копирования.

3. Если вы не хотите предоставлять роль **читателя** пользователю или субъекту-службе, альтернативным вариантом является [явное указание расположения выполнения](data-factory-data-movement-activities.md#global) в действии копирования с расположением Data Lake Store. Пример

    ```json
    {
      "name": "CopyToADLS",
      "type": "Copy",
      ......
      "typeProperties": {
        "source": {
          "type": "<source type>"
        },
        "sink": {
          "type": "AzureDataLakeStoreSink"
        },
        "exeuctionLocation": "West US"
      }
    }
    ```

## <a name="dataset-properties"></a>Свойства набора данных
Чтобы указать набор данных, представляющий входные данные в Data Lake Store, присвойте свойству **типа** набора данных значение **AzureDataLakeStore**. Для свойства **linkedServiceName** набора данных задайте имя связанной службы Data Lake Store. Полный список разделов и свойств JSON, используемых для определения наборов данных, см. в статье [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md). Разделы набора данных в JSON, такие как **structure**, **availability** и **policy**, одинаковы для всех типов наборов данных (таких как базы данных SQL, Blob-объекты Azure и таблицы Azure). Раздел **typeProperties** отличается для разных типов наборов данных. Он содержит такие сведения, как расположение данных в хранилище данных и формат данных.

Раздел **typeProperties** набора данных типа **AzureDataLakeStore** содержит следующие свойства.

| Свойство. | Описание | Обязательно |
|:--- |:--- |:--- |
| **folderPath** |Путь к контейнеру и папке в Data Lake Store. |Да |
| **fileName** |Имя файла в Azure Data Lake Store. Свойство **fileName** является необязательным и в нем учитывается регистр символов. <br/><br/>Если указать значение **fileName**, то действие (включая копирование) работает с определенным файлом.<br/><br/>Если значение **fileName** не указано, то копируются все файлы в **folderPath** для входного набора данных.<br/><br/>Если параметр **filename** не указан для выходного набора данных и **preserveHierarchy** не указан в приемнике действия, имя созданного файла имеет формат `Data._Guid_.txt` . Например: Data.0a405f8a-93ff-4c6f-b3be-f69616f1df7a.txt. |Нет |
| **partitionedBy** |Свойство **partitionedBy** является необязательным. Его можно использовать, чтобы указать динамический путь к папке и имя файла для данных временного ряда. Например, путь к папке (**folderPath**) каждый час может быть другим. Дополнительные сведения и примеры см. в разделе "Свойство partitionedBy". |Нет |
| **format** | Поддерживаются следующие типы форматов: **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** и **ParquetFormat**. Свойству **type** в разделе **format** необходимо присвоить одно из этих значений. Дополнительные сведения см. в разделах [Текстовый формат](data-factory-supported-file-and-compression-formats.md#text-format), [Формат JSON](data-factory-supported-file-and-compression-formats.md#json-format), [Формат Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Формат ORC](data-factory-supported-file-and-compression-formats.md#orc-format) и [Формат Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format) статьи [Форматы файлов и сжатия данных, поддерживаемые фабрикой данных Azure](data-factory-supported-file-and-compression-formats.md). <br><br> Если требуется скопировать файлы между файловыми хранилищами как есть (двоичное копирование), можно пропустить раздел `format` в определениях входного и выходного наборов данных. |Нет |
| **compression** | Укажите тип и уровень сжатия данных. Поддерживаемые типы: **GZip**, **Deflate**, **BZip2** и **ZipDeflate**. Поддерживаемые уровни: **Optimal** и **Fastest**. Дополнительные сведения см. в статье [Форматы файлов и сжатия данных, поддерживаемые фабрикой данных Azure](data-factory-supported-file-and-compression-formats.md#compression-support). |Нет |

### <a name="the-partitionedby-property"></a>Свойство partitionedBy
Для данных временных рядов путь к папке (**folderPath**) и имя файла (**fileName**) можно указывать динамически. Это делается с помощью свойства **partitionedBy**, функций фабрики данных и системных переменных. Подробные сведения см. в статье [Фабрика данных Azure — функции и системные переменные](data-factory-functions-variables.md).


В следующем примере `{Slice}` заменяется значением `SliceStart` (системная переменная фабрики данных) в указанном формате (`yyyyMMddHH`). `SliceStart` в имени указывает время начала среза. Свойство `folderPath` отличается для каждого среза, как и в `wikidatagateway/wikisampledataout/2014100103` или `wikidatagateway/wikisampledataout/2014100104`.

```JSON
"folderPath": "wikidatagateway/wikisampledataout/{Slice}",
"partitionedBy":
[
    { "name": "Slice", "value": { "type": "DateTime", "date": "SliceStart", "format": "yyyyMMddHH" } },
],
```

В следующем примере год, месяц, день и время `SliceStart` извлекаются в отдельные переменные, используемые в свойствах `folderPath` и `fileName`.
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
Дополнительные сведения о наборах данных временных рядов, планировании и срезах см. в статьях [Наборы данных в фабрике данных Azure](data-factory-create-datasets.md) и [Планирование и исполнение с использованием фабрики данных](data-factory-scheduling-and-execution.md).


## <a name="copy-activity-properties"></a>Свойства действия копирования
Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](data-factory-create-pipelines.md). Свойства (включая имя, описание, входные и выходные таблицы, политику и т. д.) доступны для всех типов действий.

Свойства, доступные в разделе **typeProperties** действия, зависят от конкретного типа действия. Для действия копирования они различаются в зависимости от типов источников и приемников.

**AzureDataLakeStoreSource** поддерживает следующее свойство в разделе **typeProperties**:

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| **рекурсивные** |Указывает, следует ли читать данные рекурсивно из вложенных папок или только из указанной папки. |True (значение по умолчанию), False |Нет |

**AzureDataLakeStoreSink** поддерживает следующие свойства в разделе **typeProperties**:

| Свойство. | Описание | Допустимые значения | Обязательно |
| --- | --- | --- | --- |
| **copyBehavior** |Определяет поведение копирования. |<b>PreserveHierarchy</b>: сохраняет иерархию файлов в целевой папке. Относительный путь исходного файла в исходной папке идентичен относительному пути целевого файла в целевой папке.<br/><br/><b>FlattenHierarchy</b>: все файлы из исходной папки создаются на первом уровне в целевой папке. Целевые файлы создаются с автоматически сформированными именами.<br/><br/><b>MergeFiles</b>: объединяет все файлы из исходной папки в один файл. Если указано имя Blob-объекта или имя файла, то оно присваивается объединенному файлу. В противном случае имя файла создается автоматически. |нет |

### <a name="recursive-and-copybehavior-examples"></a>Примеры recursive и copyBehavior
В данном разделе описываются результаты выполнения операции копирования при использовании различных сочетаний значений recursive и copyBehavior.

| recursive | copyBehavior | Результаты выполнения операции |
| --- | --- | --- |
| Да |preserveHierarchy |Если у исходной папки "Папка1" такая структура:  <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается с такой же структурой, как и исходная папка.<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5 |
| Да |flattenHierarchy |Если у исходной папки "Папка1" такая структура:  <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой: <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл1"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл2"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл3"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл4"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл5" |
| Да |mergeFiles |Если у исходной папки "Папка1" такая структура:  <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой: <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Содержимое файлов "Файл1", "Файл2", "Файл3", "Файл4" и "Файл5" объединяется в один файл с автоматически созданным именем. |
| false |preserveHierarchy |Если у исходной папки "Папка1" такая структура:  <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/><br/><br/>Папка "Вложенная_папка1" с файлами "Файл3", "Файл4" и "Файл5" не будет включена в эту папку. |
| false |flattenHierarchy |Если у исходной папки "Папка1" такая структура: <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл1"<br/>&nbsp;&nbsp;&nbsp;&nbsp;Автоматически созданное имя для "Файл2"<br/><br/><br/>Папка "Вложенная_папка1" с файлами "Файл3", "Файл4" и "Файл5" не будет включена в эту папку. |
| false |mergeFiles |Если у исходной папки "Папка1" такая структура: <br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Файл2<br/>&nbsp;&nbsp;&nbsp;&nbsp;Вложенная_папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл3<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл4<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Файл5<br/><br/>Целевая папка "Папка1" создается со следующей структурой:<br/><br/>Папка1<br/>&nbsp;&nbsp;&nbsp;&nbsp;Содержимое файлов "Файл1" и "Файл2" объединяется в один файл с автоматически созданным именем. Автоматически созданное имя для "Файл1"<br/><br/>Папка "Вложенная_папка1" с файлами "Файл3", "Файл4" и "Файл5" не будет включена в эту папку. |

## <a name="supported-file-and-compression-formats"></a>Поддерживаемые форматы файлов и сжатия
Дополнительные сведения см. в статье [Форматы файлов и сжатия данных, поддерживаемые фабрикой данных Azure](data-factory-supported-file-and-compression-formats.md).

## <a name="json-examples-for-copying-data-to-and-from-data-lake-store"></a>Примеры JSON для копирования данных в Data Lake Store и обратно
Ниже приведены примеры определений JSON. Эти образцы определений можно использовать для создания конвейера с помощью [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) или [Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). В них показано, как копировать данные в Data Lake Store и хранилище BLOB-объектов Azure и обратно. Однако данные можно скопировать данные _непосредственно_ из любых источников на любой из поддерживаемых приемников. Дополнительные сведения см. в разделе "Поддерживаемые хранилища данных и форматы" статьи [Перемещение данных с помощью действия копирования](data-factory-data-movement-activities.md).

### <a name="example-copy-data-from-azure-blob-storage-to-azure-data-lake-store"></a>Пример. Копирование данных из хранилища BLOB-объектов Azure в Azure Data Lake Store
В примере кода в этом разделе показано следующее:

* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Связанная служба типа [AzureDataLakeStore](#linked-service-properties).
* Входной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureDataLakeStore](#dataset-properties).
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, который использует [BlobSource](data-factory-azure-blob-connector.md#copy-activity-properties) и [AzureDataLakeStoreSink](#copy-activity-properties).

В этом примере показано копирование данных временного ряда из хранилища BLOB-объектов Azure в Data Lake Store каждый час.

**Связанная служба хранения Azure**

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

**Связанная служба Azure Data Lake Store**

```JSON
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>",
            "subscriptionId": "<subscription of ADLS>",
            "resourceGroupName": "<resource group of ADLS>"
        }
    }
}
```

> [!NOTE]
> Сведения о настройке см. в разделе [Свойства связанной службы](#linked-service-properties).
>

**Входной набор данных большого двоичного объекта Azure**

В следующем примере данные берутся из нового BLOB-объекта каждый час (`"frequency": "Hour", "interval": 1`). Путь к папке с BLOB-объектом и имя файла вычисляются динамически на основе времени начала обрабатываемого среза. В пути к папке используется год, месяц, день и час времени начала. В имени файла используется часть времени начала, обозначающая час. Параметр `"external": true` указывает фабрике данных, что эта таблица является внешней по отношению к фабрике данных, а не созданной в результате какого-либо действия в фабрике данных.

```JSON
{
  "name": "AzureBlobInput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}",
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

**Выходной набор данных Azure Data Lake Store:**

Следующий пример кода копирует данные в Data Lake Store. Новые данные копируются в Data Lake Store каждый час.

```JSON
{
    "name": "AzureDataLakeStoreOutput",
    "properties": {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/output/"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

**Действие копирования в конвейере с BLOB-объектом в качестве источника и Data Lake Store в качестве приемника**

В следующем примере конвейер содержит действие копирования, которое использует входные и выходные наборы данных. Действие копирования выполняется по расписанию каждый час. В определении JSON конвейера для типа `source` задано значение `BlobSource`, а для типа `sink` — значение `AzureDataLakeStoreSink`.

```json
{
    "name":"SamplePipeline",
    "properties":
    {
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline with copy activity",
        "activities":
        [
            {
                "name": "AzureBlobtoDataLake",
                "description": "Copy Activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureBlobInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureDataLakeStoreOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "BlobSource"
                    },
                    "sink": {
                        "type": "AzureDataLakeStoreSink"
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

### <a name="example-copy-data-from-azure-data-lake-store-to-an-azure-blob"></a>Пример. Копирование данных из Azure Data Lake Store в BLOB-объект Azure
В примере кода в этом разделе показано следующее:

* Связанная служба типа [AzureDataLakeStore](#linked-service-properties).
* Связанная служба типа [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
* Входной [набор данных](data-factory-create-datasets.md) типа [AzureDataLakeStore](#dataset-properties).
* Выходной [набор данных](data-factory-create-datasets.md) типа [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
* [Конвейер](data-factory-create-pipelines.md) с действием копирования, который использует [AzureDataLakeStoreSource](#copy-activity-properties) и [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

В этом коде данные временного ряда каждый час копируются из Data Lake Store в Blob-объект Azure.

**Связанная служба Azure Data Lake Store**

```json
{
    "name": "AzureDataLakeStoreLinkedService",
    "properties": {
        "type": "AzureDataLakeStore",
        "typeProperties": {
            "dataLakeStoreUri": "https://<accountname>.azuredatalakestore.net/webhdfs/v1",
            "servicePrincipalId": "<service principal id>",
            "servicePrincipalKey": "<service principal key>",
            "tenant": "<tenant info, e.g. microsoft.onmicrosoft.com>"
        }
    }
}
```

> [!NOTE]
> Сведения о настройке см. в разделе [Свойства связанной службы](#linked-service-properties).
>

**Связанная служба хранения Azure**

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
**Входной набор данных Azure Data Lake**

В этом примере значение `true` для параметра `"external"` указывает фабрике данных, что эта таблица является внешней по отношению к фабрике данных, а не созданной в результате какого-либо действия в фабрике данных.

```json
{
    "name": "AzureDataLakeStoreInput",
    "properties":
    {
        "type": "AzureDataLakeStore",
        "linkedServiceName": "AzureDataLakeStoreLinkedService",
        "typeProperties": {
            "folderPath": "datalake/input/",
            "fileName": "SearchLog.tsv",
            "format": {
                "type": "TextFormat",
                "rowDelimiter": "\n",
                "columnDelimiter": "\t"
            }
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
**Выходной набор данных большого двоичного объекта Azure**

В следующем примере данные записываются в новый BLOB-объект каждый час (`"frequency": "Hour", "interval": 1`). Путь к папке BLOB-объекта вычисляется динамически на основе времени начала обрабатываемого среза. В пути к папке используются год, месяц, день и час времени начала.

```JSON
{
  "name": "AzureBlobOutput",
  "properties": {
    "type": "AzureBlob",
    "linkedServiceName": "StorageLinkedService",
    "typeProperties": {
      "folderPath": "mycontainer/myfolder/yearno={Year}/monthno={Month}/dayno={Day}/hourno={Hour}",
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

**Действие копирования в конвейере с Azure Data Lake Store в качестве источника и BLOB-объектом в качестве приемника:**

В следующем примере конвейер содержит действие копирования, которое использует входные и выходные наборы данных. Действие копирования выполняется по расписанию каждый час. В определении JSON конвейера для типа `source` задано значение `AzureDataLakeStoreSource`, а для типа `sink` — значение `BlobSink`.

```json
{
    "name":"SamplePipeline",
    "properties":{
        "start":"2014-06-01T18:00:00",
        "end":"2014-06-01T19:00:00",
        "description":"pipeline for copy activity",
        "activities":[
            {
                "name": "AzureDakeLaketoBlob",
                "description": "copy activity",
                "type": "Copy",
                "inputs": [
                    {
                        "name": "AzureDataLakeStoreInput"
                    }
                ],
                "outputs": [
                    {
                        "name": "AzureBlobOutput"
                    }
                ],
                "typeProperties": {
                    "source": {
                        "type": "AzureDataLakeStoreSource",
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

В определении действия копирования также можно сопоставить столбцы из набора данных, используемого в качестве источника, со столбцами из приемника. Дополнительные сведения см. в статье [Сопоставление столбцов исходного набора данных со столбцами целевого набора данных](data-factory-map-columns.md).

## <a name="performance-and-tuning"></a>Производительность и настройка
Сведения о факторах, влияющих на производительность действия копирования, и способах оптимизации этого процесса см. в статье [Руководство по настройке производительности действия копирования](data-factory-copy-activity-performance.md).