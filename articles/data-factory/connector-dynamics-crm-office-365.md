---
title: Копирование данных в Dynamics (Common Data Service)
description: Узнайте, как копировать данные из Microsoft Dynamics CRM или Microsoft Dynamics 365 (Common Data Service или Microsoft инверсия) в поддерживаемые хранилища данных-приемники или из поддерживаемых исходных хранилищ данных в Dynamics CRM или Dynamics 365 с помощью действия копирования в конвейере фабрики данных.
ms.service: data-factory
ms.topic: conceptual
ms.author: jingwang
author: linda33wj
ms.custom: seo-lt-2019
ms.date: 03/17/2021
ms.openlocfilehash: f2db75fdcd4519b5ba0869bf4ef89c8323435539
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105565985"
---
# <a name="copy-data-from-and-to-dynamics-365-common-data-servicemicrosoft-dataverse-or-dynamics-crm-by-using-azure-data-factory"></a>Копирование данных из Dynamics 365 (Common Data Service/Microsoft инверсия) или Dynamics CRM с помощью фабрики данных Azure

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

В этой статье описано, как с помощью действия копирования в фабрике данных Azure копировать данные в Microsoft Dynamics 365 и Microsoft Dynamics CRM и обратно. Она основана на статье [Обзор действия копирования](copy-activity-overview.md) , в которой представлен общий обзор действия копирования.

## <a name="supported-capabilities"></a>Поддерживаемые возможности

Этот соединитель поддерживается для следующих действий:

- [Действие копирования](copy-activity-overview.md) с [поддерживаемой матрицей источника и приемника](copy-activity-overview.md)
- [Действие поиска](control-flow-lookup-activity.md)

Данные из Dynamics 365 (Common Data Service, Microsoft DataStore) или Dynamics CRM можно скопировать в любое поддерживаемое хранилище данных, которое является приемником. В свою очередь, данные из любого хранилища данных, поддерживаемого в качестве источника, можно скопировать в Dynamics 365 (Common Data Service) или Dynamics CRM. Список хранилищ данных, поддерживаемых действием копирования в качестве источников и приемников, см. в таблице [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats) .

Этот соединитель Dynamics поддерживает версии Dynamics 7 – 9 для Интернет-и локальной среды. В частности:

- Версия 7 сопоставляется с Dynamics CRM 2015.
- Версия 8 сопоставляется с Dynamics CRM 2016 и ранней версией Dynamics 365.
- Версия 9 сопоставляется с более поздней версией Dynamics 365.

См. приведенную ниже таблицу поддерживаемых типов проверки подлинности и конфигураций для версий Dynamics и продуктов.

| Версии Dynamics | Типы проверки подлинности | Примеры связанной службы |
|:--- |:--- |:--- |
| Common Data Service <br/><br/> Dynamics 365 Online <br/><br/> Dynamics CRM Online | Субъект-служба Azure Active Directory (Azure AD) <br/><br/> Office 365 | [Служба Dynamics Online и Azure AD — проверка подлинности субъекта или Office 365](#dynamics-365-and-dynamics-crm-online) |
| Локальная среда Dynamics 365 с развертыванием с выходом в Интернет (IFD) <br/><br/> Dynamics CRM 2016 (локальная версия) с IFD <br/><br/> Dynamics CRM 2015 (локальная версия) с IFD | IFD | [Локальная среда Dynamics с IFD и IFD Authentication](#dynamics-365-and-dynamics-crm-on-premises-with-ifd) |

В частности, для Dynamics 365 поддерживаются следующие типы приложений:

- Dynamics 365 for Sales;
- Dynamics 365 for Customer Service;
- Dynamics 365 for Field Service;
- Dynamics 365 for Project Service Automation;
- Dynamics 365 for Marketing.

Этот соединитель не поддерживает другие типы приложений, такие как финансы, операции и прочее.

>[!TIP]
>Чтобы скопировать данные из Dynamics 365 Финансы и операции, можно использовать [соединитель Dynamics AX](connector-dynamics-ax.md).

Этот соединитель Dynamics построен на основе [инструментов Dynamics XRM](/dynamics365/customer-engagement/developer/build-windows-client-applications-xrm-tools).

## <a name="prerequisites"></a>Предварительные условия

Чтобы использовать этот соединитель с проверкой подлинности субъекта-службы Azure AD, необходимо настроить проверку подлинности "сервер-сервер" (S2S) в Common Data Service или Dynamics. Подробные инструкции см. в [этой статье](/powerapps/developer/common-data-service/build-web-applications-server-server-s2s-authentication) .

## <a name="get-started"></a>Начало работы

[!INCLUDE [data-factory-v2-connector-get-started](../../includes/data-factory-v2-connector-get-started.md)]

Следующие разделы содержат сведения о свойствах, которые используются для определения сущностей фабрики данных, характерных для Dynamics.

## <a name="linked-service-properties"></a>Свойства связанной службы

Для связанной службы Dynamics поддерживаются следующие свойства:

### <a name="dynamics-365-and-dynamics-crm-online"></a>Dynamics 365 и Dynamics CRM Online

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type должно иметь значение "Dynamics", "Динамикскрм" или "Коммондатасервицефораппс". | Да |
| deploymentType | Тип развертывания для экземпляра Dynamics. Для Dynamics Online значение должно быть "Online". | Да |
| serviceUri | URL-адрес службы экземпляра Dynamics, к которому вы обращаетесь из браузера. Пример: "https:// \<organization-name> . CRM [x]. Dynamics. com". | Да |
| authenticationType | Тип проверки подлинности для подключения к серверу Dynamics. Допустимые значения: "АадсервицепринЦипал" и "Office 365". | Да |
| servicePrincipalId | Идентификатор клиента приложения Azure AD. | Да, если проверка подлинности — "АадсервицепринЦипал" |
| сервицепринЦипалкредентиалтипе | Тип учетных данных, используемый для проверки подлинности субъекта-службы. Допустимые значения: "СервицепринЦипалкэй" и "СервицепринЦипалцерт". | Да, если проверка подлинности — "АадсервицепринЦипал" |
| сервицепринЦипалкредентиал | Учетные данные субъекта-службы. <br/><br/>При использовании "СервицепринЦипалкэй" в качестве типа учетных данных `servicePrincipalCredential` может быть строкой, которая шифруется фабрикой данных Azure при развертывании связанной службы. Или это может быть ссылка на секрет в Azure Key Vault. <br/><br/>При использовании "СервицепринЦипалцерт" в качестве учетных данных `servicePrincipalCredential` должен быть ссылкой на сертификат в Azure Key Vault. | Да, если проверка подлинности — "АадсервицепринЦипал" |
| username | Имя пользователя для подключения к Dynamics. | Да, если проверка подлинности — "Office 365" |
| password | Пароль для учетной записи пользователя, указанной в качестве имени пользователя. Пометьте это поле "SecureString", чтобы безопасно хранить его в фабрике данных, или [сослаться на секрет, хранящийся в Azure Key Vault](store-credentials-in-key-vault.md). | Да, если проверка подлинности — "Office 365" |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Если значение не указано, свойство использует среду выполнения интеграции Azure по умолчанию. | Нет |

>[!NOTE]
>Соединитель Dynamics ранее использовал необязательное свойство **название_организации** для задания экземпляра Dynamics CRM или Dynamics 365 Online. Хотя это свойство по-прежнему работает, мы рекомендуем указать новое свойство **ServiceUri** , чтобы повысить производительность при обнаружении экземпляра.

#### <a name="example-dynamics-online-using-azure-ad-service-principal-and-key-authentication"></a>Пример: Dynamics Online с использованием службы Azure AD — субъект и проверка подлинности ключа

```json
{  
    "name": "DynamicsLinkedService",  
    "properties": {  
        "type": "Dynamics",  
        "typeProperties": {  
            "deploymentType": "Online",  
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com",  
            "authenticationType": "AADServicePrincipal",  
            "servicePrincipalId": "<service principal id>",  
            "servicePrincipalCredentialType": "ServicePrincipalKey",  
            "servicePrincipalCredential": "<service principal key>"
        },  
        "connectVia": {  
            "referenceName": "<name of Integration Runtime>",  
            "type": "IntegrationRuntimeReference"  
        }  
    }  
}  
```
#### <a name="example-dynamics-online-using-azure-ad-service-principal-and-certificate-authentication"></a>Пример: Dynamics Online с использованием службы Azure AD — субъект и проверка подлинности на основе сертификата

```json
{ 
    "name": "DynamicsLinkedService", 
    "properties": { 
        "type": "Dynamics", 
        "typeProperties": { 
            "deploymentType": "Online", 
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com", 
            "authenticationType": "AADServicePrincipal", 
            "servicePrincipalId": "<service principal id>", 
            "servicePrincipalCredentialType": "ServicePrincipalCert", 
            "servicePrincipalCredential": { 
                "type": "AzureKeyVaultSecret", 
                "store": { 
                    "referenceName": "<AKV reference>", 
                    "type": "LinkedServiceReference" 
                }, 
                "secretName": "<certificate name in AKV>" 
            } 
        }, 
        "connectVia": { 
            "referenceName": "<name of Integration Runtime>", 
            "type": "IntegrationRuntimeReference" 
        } 
    } 
} 
```

#### <a name="example-dynamics-online-using-office-365-authentication"></a>Пример: Dynamics Online с использованием проверки подлинности Office 365

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "typeProperties": {
            "deploymentType": "Online",
            "serviceUri": "https://<organization-name>.crm[x].dynamics.com",
            "authenticationType": "Office365",
            "username": "test@contoso.onmicrosoft.com",
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

### <a name="dynamics-365-and-dynamics-crm-on-premises-with-ifd"></a>Dynamics 365 и Dynamics CRM (локальная версия) с IFD

Дополнительными свойствами, которые сравниваются с Dynamics Online, являются **имя узла** и **порт**.

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type должно иметь значение "Dynamics", "Динамикскрм" или "Коммондатасервицефораппс". | Да. |
| deploymentType | Тип развертывания для экземпляра Dynamics. Значение должно быть "OnPremisesWithIfd" для Dynamics в локальной среде с IFD.| Да. |
| hostName | Имя узла локального сервера Dynamics. | Да. |
| порт | Порт локального сервера Dynamics. | Нет. Значение по умолчанию — 443. |
| оrganizationName | Имя организации экземпляра Dynamics. | Да. |
| authenticationType | Тип проверки подлинности для подключения к серверу Dynamics. Укажите Ifd для Dynamics (локальная версия) с IFD. | Да. |
| username | Имя пользователя для подключения к Dynamics. | Да. |
| password | Пароль для учетной записи пользователя, указанной для имени пользователя. Вы можете пометить это поле "SecureString", чтобы безопасно хранить его в фабрике данных. Кроме того, можно сохранить пароль в Key Vault и разрешить извлечение действия копирования при копировании данных. Подробнее о [хранении учетных данных в Key Vault](store-credentials-in-key-vault.md). | Да. |
| connectVia | [Среда выполнения интеграции](concepts-integration-runtime.md), используемая для подключения к хранилищу данных. Если значение не указано, свойство использует среду выполнения интеграции Azure по умолчанию. | Нет |

#### <a name="example-dynamics-on-premises-with-ifd-using-ifd-authentication"></a>Пример. Dynamics (локальная версия) с IFD с использованием проверки подлинности

```json
{
    "name": "DynamicsLinkedService",
    "properties": {
        "type": "Dynamics",
        "description": "Dynamics on-premises with IFD linked service using IFD authentication",
        "typeProperties": {
            "deploymentType": "OnPremisesWithIFD",
            "hostName": "contosodynamicsserver.contoso.com",
            "port": 443,
            "organizationName": "admsDynamicsTest",
            "authenticationType": "Ifd",
            "username": "test@contoso.onmicrosoft.com",
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

Полный список разделов и свойств, доступных для определения наборов данных, см. в статье о [наборах данных](concepts-datasets-linked-services.md). В этом разделе содержится список свойств, поддерживаемых набором данных Dynamics.

Для копирования данных из и в Dynamics поддерживаются следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type набора данных должно иметь значение "DynamicsEntity", "Динамикскрментити" или "Коммондатасервицефораппсентити". |Да |
| entityName | Логическое имя сущности, которое требуется получить. | Нет для источника, если для источника действия задано значение "запрос" и Да для приемника |

#### <a name="example"></a>Пример

```json
{
    "name": "DynamicsDataset",
    "properties": {
        "type": "DynamicsEntity",
        "schema": [],
        "typeProperties": {
            "entityName": "account"
        },
        "linkedServiceName": {
            "referenceName": "<Dynamics linked service name>",
            "type": "linkedservicereference"
        }
    }
}
```

## <a name="copy-activity-properties"></a>Свойства действия копирования

Полный список разделов и свойств, используемых для определения действий, см. в статье [Конвейеры и действия в фабрике данных Azure](concepts-pipelines-activities.md). Этот раздел содержит список свойств, поддерживаемых типами источника и приемника Dynamics.

### <a name="dynamics-as-a-source-type"></a>Dynamics в качестве источника данных

Чтобы скопировать данные из Dynamics, раздел **источника** действия копирования поддерживает следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type источника действия копирования должно иметь значение "DynamicsSource", "Динамикскрмсаурце" или "Коммондатасервицефораппссаурце". | Да |
| query | FetchXML — это собственный язык запросов, используемый в Dynamics Online и локальной среде. См. следующий пример. Дополнительные сведения см. в статье [Создание запросов с помощью FetchXML](/previous-versions/dynamicscrm-2016/developers-guide/gg328332(v=crm.8)). | No `entityName` , если в наборе данных указан |

>[!NOTE]
>Столбец первичного ключа будет копироваться всегда, даже если он отсутствует в проекции столбца, настроенной в запросе FetchXML.

> [!IMPORTANT]
>- При копировании данных из Dynamics явное сопоставление столбцов из Dynamics с приемником является необязательным. Но мы настоятельно рекомендуем использовать сопоставление для обеспечения детерминированного результата копирования.
>- Когда фабрика данных импортирует схему в пользовательском интерфейсе разработки, она выводит схему. Это достигается путем выборки верхних строк из результата запроса Dynamics для инициализации списка исходных столбцов. В этом случае столбцы без значений в верхних строках опущены. Такое же поведение применяется к выполнению копирования, если нет явного сопоставления. Вы можете просматривать и добавлять в сопоставление дополнительные столбцы, которые учитываются во время выполнения копирования.

#### <a name="example"></a>Например, .

```json
"activities":[
    {
        "name": "CopyFromDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<Dynamics input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "DynamicsSource",
                "query": "<FetchXML Query>"
            },
            "sink": {
                "type": "<sink type>"
            }
        }
    }
]
```

### <a name="sample-fetchxml-query"></a>Образец запроса FetchXML

```xml
<fetch>
  <entity name="account">
    <attribute name="accountid" />
    <attribute name="name" />
    <attribute name="marketingonly" />
    <attribute name="modifiedon" />
    <order attribute="modifiedon" descending="false" />
    <filter type="and">
      <condition attribute ="modifiedon" operator="between">
        <value>2017-03-10 18:40:00z</value>
        <value>2017-03-12 20:40:00z</value>
      </condition>
    </filter>
  </entity>
</fetch>
```

### <a name="dynamics-as-a-sink-type"></a>Dynamics в качестве типа приемника

Чтобы скопировать данные в Dynamics, раздел **приемника** действия копирования поддерживает следующие свойства:

| Свойство | Описание | Обязательно |
|:--- |:--- |:--- |
| type | Свойство Type приемника действия копирования должно иметь значение "DynamicsSink", "Динамикскрмсинк" или "Коммондатасервицефораппссинк". | Да. |
| writeBehavior | Поведение операции при записи. Значение должно быть "Upsert". | Да |
| алтернатекэйнаме | Альтернативное имя ключа, определенное в сущности для Upsert. | Нет. |
| writeBatchSize | Количество строк данных, записываемых в Dynamics в каждом пакете. | Нет. Значение по умолчанию — 10. |
| ignoreNullValues | Следует ли пропускать значения NULL из входных данных, отличных от ключевых полей во время операции записи.<br/><br/>Допустимые значения: **true** и **false**.<ul><li>**True**: при выполнении операции Upsert или обновления оставляет данные в целевом объекте без изменений. При выполнении операции вставки (insert) вставьте определенное значение по умолчанию.</li><li>**False**: при выполнении операции Upsert или Update обновите данные в целевом объекте до значения NULL. При выполнении операции вставки вставляется значение null.</li></ul> | Нет. Значение по умолчанию — **false**. |
| maxConcurrentConnections |Верхний предел одновременных подключений, установленных для хранилища данных во время выполнения действия. Укажите значение, только если требуется ограничить количество одновременных подключений.| Нет |

>[!NOTE]
>Значение по умолчанию для приемника **writeBatchSize** и **[parallelCopies](copy-activity-performance-features.md#parallel-copy)** действия копирования для приемника Dynamics — 10. Таким образом, 100 по умолчанию в Dynamics отправляются записи.

Для Dynamics 365 Online существует ограничение в [двух одновременных пакетных вызовах для каждой организации](/previous-versions/dynamicscrm-2016/developers-guide/jj863631(v=crm.8)#Run-time%20limitations). При превышении этого ограничения выдается исключение «сервер занят», прежде чем первый запрос запустится. Старайтесь не использовать **writeBatchSize** в 10 или менее, чтобы избежать такого регулирования одновременных вызовов.

Оптимальное сочетание **writeBatchSize** и **parallelCopies** зависит от схемы сущности. Элементы схемы включают число столбцов, размер строк и число подключаемых модулей, рабочих процессов или действий рабочего процесса, привязанных к этим вызовам. По умолчанию параметр **writeBatchSize** (10) &times; **parallelCopies** (10) является рекомендацией в соответствии со службой Dynamics. Это значение подходит для большинства сущностей Dynamics, хотя оно может не дать наилучшей производительности. Вы можете настроить производительность путем корректировки комбинации в настройках активности копирования.

#### <a name="example"></a>Например, .

```json
"activities":[
    {
        "name": "CopyToDynamics",
        "type": "Copy",
        "inputs": [
            {
                "referenceName": "<input dataset>",
                "type": "DatasetReference"
            }
        ],
        "outputs": [
            {
                "referenceName": "<Dynamics output dataset>",
                "type": "DatasetReference"
            }
        ],
        "typeProperties": {
            "source": {
                "type": "<source type>"
            },
            "sink": {
                "type": "DynamicsSink",
                "writeBehavior": "Upsert",
                "writeBatchSize": 10,
                "ignoreNullValues": true
            }
        }
    }
]
```

## <a name="retrieving-data-from-views"></a>Получение данных из представлений

Чтобы получить данные из представлений Dynamics, необходимо получить сохраненный запрос представления и использовать запрос для получения данных.

Существуют две сущности, которые хранят различные типы представлений: "сохраненный запрос" хранит системное представление, а "пользовательский запрос" сохраняет пользовательское представление. Чтобы получить сведения о представлениях, обратитесь к следующему FetchXML запросу и замените "ТАРЖЕТЕНТИТИ" на `savedquery` или `userquery` . Каждый тип сущности имеет больше доступных атрибутов, которые можно добавить в запрос в зависимости от ваших нужд. Дополнительные сведения о сущности [саведкуери](/dynamics365/customer-engagement/web-api/savedquery) и [сущности усеркуери](/dynamics365/customer-engagement/web-api/userquery).

```xml
<fetch top="5000" >
  <entity name="<TARGETENTITY>">
    <attribute name="name" />
    <attribute name="fetchxml" />
    <attribute name="returnedtypecode" />
    <attribute name="querytype" />
  </entity>
</fetch>
```

Можно также добавить фильтры для фильтрации представлений. Например, добавьте следующий фильтр, чтобы получить представление с именем "Мои активные учетные записи" в сущности учетной записи.

```xml
<filter type="and" >
    <condition attribute="returnedtypecode" operator="eq" value="1" />
    <condition attribute="name" operator="eq" value="My Active Accounts" />
</filter>
```

## <a name="data-type-mapping-for-dynamics"></a>Сопоставление типов данных для Dynamics

При копировании данных из Dynamics в следующей таблице показаны сопоставления типов данных Dynamics с промежуточными типами данных фабрики данных. Сведения о сопоставлении действия копирования с исходной схемой и типе данных, сопоставленном с приемником, см. в разделе [схемы и сопоставления типов данных](copy-activity-schema-and-type-mapping.md).

Настройте соответствующий тип данных фабрики данных в структуре набора данных, основанной на типе данных источника Dynamics, используя следующую таблицу сопоставления:

| Тип данных Dynamics | Тип промежуточных данных фабрики данных | Поддерживается в качестве источника | Поддерживается в качестве приемника |
|:--- |:--- |:--- |:--- |
| AttributeTypeCode.BigInt | Long | ✓ | ✓ |
| AttributeTypeCode.Boolean | Логическое | ✓ | ✓ |
| AttributeType.Customer | Код GUID | ✓ | ✓ (См. [руководство](#writing-data-to-a-lookup-field)) |
| AttributeType.DateTime | Datetime | ✓ | ✓ |
| AttributeType.Decimal | Decimal | ✓ | ✓ |
| AttributeType.Double | Double | ✓ | ✓ |
| AttributeType.EntityName | Строка | ✓ | ✓ |
| AttributeType.Integer | Int32 | ✓ | ✓ |
| AttributeType.Lookup | Код GUID | ✓ | ✓ (См. [руководство](#writing-data-to-a-lookup-field)) |
| AttributeType.ManagedProperty | Логическое | ✓ | |
| AttributeType.Memo | Строка | ✓ | ✓ |
| AttributeType.Money | Decimal | ✓ | ✓ |
| AttributeType.Owner | Код GUID | ✓ | ✓ (См. [руководство](#writing-data-to-a-lookup-field)) |
| AttributeType.Picklist | Int32 | ✓ | ✓ |
| AttributeType.Uniqueidentifier | Код GUID | ✓ | ✓ |
| AttributeType.String | Строка | ✓ | ✓ |
| AttributeType.State | Int32 | ✓ | ✓ |
| AttributeType.Status | Int32 | ✓ | ✓ |

> [!NOTE]
> Типы данных Dynamics **attributeType. CalendarRules**, **attributeType. мултиселектпикклист** и **attributeType. PartyList** не поддерживаются.

## <a name="writing-data-to-a-lookup-field"></a>Запись данных в поле подстановки

Чтобы записать данные в поле подстановки с несколькими целевыми объектами, такими как Customer и Owner, следуйте указаниям и примерам.

1. Убедитесь, что источник содержит как значение поля, так и соответствующее имя целевой сущности.
   - Если все записи сопоставляются с одной и той же целевой сущностью, убедитесь, что одно из следующих условий:
      - Исходные данные содержат столбец, в котором хранится имя целевой сущности.
      - Вы добавили дополнительный столбец в источнике действия копирования, чтобы определить целевую сущность.
   - Если разные записи сопоставляются с разными целевыми сущностями, убедитесь, что в источнике данных есть столбец, в котором хранится соответствующее имя целевой сущности.

1. Сопоставьте значения и ссылочные столбцы сущностей из источника в приемник. Столбец ссылки на сущность должен быть сопоставлен с виртуальным столбцом с использованием специального шаблона именования `{lookup_field_name}@EntityReference` . Этот столбец фактически не существует в Dynamics. Он используется для указания, что этот столбец является столбцом метаданных заданного поля подстановки с нацеливанием.

Например, предположим, что источник содержит следующие два столбца:

- **Кустомерфиелд** столбец типа **GUID**, который является значением первичного ключа целевой сущности в Dynamics.
- **Целевой** столбец типа **String**, который является логическим именем целевой сущности.

Также предположим, что необходимо скопировать такие данные в поле приемника сущности Dynamics **кустомерфиелд** типа **Customer**.

В сопоставлении столбцов копирования и действий сопоставьте два столбца следующим образом:

- **Кустомерфиелд** **кустомерфиелд**. Это сопоставление является нормальным сопоставлением полей.
- **Целевой объект** для **кустомерфиелд \@ EntityReference**. Столбец приемника представляет собой виртуальный столбец, представляющий ссылку на сущность. Введите такие имена полей в сопоставлении, так как они не будут отображаться при импорте схем.

![Сопоставление столбцов полей в Dynamics](./media/connector-dynamics-crm-office-365/connector-dynamics-lookup-field-column-mapping.png)

Если все исходные записи сопоставляются с одной и той же целевой сущностью, а исходные данные не содержат имя целевой сущности, ниже приводится ярлык: в источнике действия копирования добавьте дополнительный столбец. Назовите новый столбец с помощью шаблона `{lookup_field_name}@EntityReference` , установите значение в поле Имя целевой сущности, а затем выберите сопоставление столбцов, как обычно. Если имена столбцов источника и приемника идентичны, можно также пропустить явное сопоставление столбцов, так как действие копирования по умолчанию сопоставляет столбцы по имени.

![Подстановка Dynamics-поле для добавления столбца ссылки на сущность](./media/connector-dynamics-crm-office-365/connector-dynamics-add-entity-reference-column.png)

## <a name="lookup-activity-properties"></a>Свойства действия поиска

Дополнительные сведения о свойствах см. в разделе [действие поиска](control-flow-lookup-activity.md).

## <a name="next-steps"></a>Дальнейшие действия
Список хранилищ данных, которые действие копирования в фабрике данных поддерживает как источники и приемники, см. в разделе [Поддерживаемые хранилища данных](copy-activity-overview.md#supported-data-stores-and-formats).