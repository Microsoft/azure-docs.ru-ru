---
title: Справочник по метаданным конфигурации Azure Блокчейн Workbench
description: Обзор метаданных конфигурации приложения Azure Блокчейн Workbench Preview.
ms.date: 12/09/2019
ms.topic: article
ms.reviewer: brendal
ms.openlocfilehash: f0ba19bf1d7fdf05014ac199fae9392b5c3249d1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87073075"
---
# <a name="azure-blockchain-workbench-configuration-reference"></a>Справочник по конфигурации Azure Blockchain Workbench

Приложения Azure Blockchain Workbench — это рабочие процессы с несколькими участниками, определяемые метаданными конфигурации и кодом смарт-контракта. Метаданные конфигурации определяют высокоуровневые рабочие процессы и модель взаимодействия блокчейн-приложения. Интеллектуальные контракты определяют бизнес-логику блокчейн-приложения. Workbench создает возможности блокчейн-приложения с помощью конфигурации и кода смарт-контракта.

Метаданные конфигурации указывают для каждого блокчейн-приложения следующие сведения:

* Имя и описание блокчейн-приложения.
* Уникальные роли для пользователей, которые могут выполнять действия или быть участниками в блокчейн-приложении.
* Один или несколько рабочих процессов. Каждый рабочий процесс выступает в качестве конечного автомата для управления потоком бизнес-логики. Рабочие процессы могут быть независимыми друг от друга или взаимодействовать друг с другом.

В каждом определенном рабочем процессе указывается следующее:

* Имя и описание рабочего процесса.
* Состояния рабочего процесса.  Каждое состояние — это ступень в потоке управления бизнес-логики. 
* Действия для перехода в следующее состояние.
* Роли пользователей, у которых есть разрешения инициировать каждое действие.
* Интеллектуальные контракты, которые представляют бизнес-логику в файлах кода.

## <a name="application"></a>Приложение

Блокчейн-приложение содержит метаданные конфигурации, рабочие процессы и роли пользователей, которые могут выполнять действия или быть участниками в приложении.

| Поле | Описание | Обязательно |
|-------|-------------|:--------:|
| ApplicationName | Уникальное имя приложения. Для соответствующего смарт-контракта должно использоваться то же самое имя **ApplicationName**, что и для применимого класса контракта.  | Да |
| DisplayName | Понятное отображаемое имя приложения. | Да |
| Описание | Описание приложения. | Нет |
| ApplicationRoles | Коллекция ролей [ApplicationRoles](#application-roles). Роли пользователей, которые могут выполнять действия или быть участниками в приложении.  | Да |
| Рабочие процессы | Коллекция рабочих процессов [Workflows](#workflows). Каждый рабочий процесс выступает в качестве конечного автомата для управления потоком бизнес-логики. | Да |

См. [пример файла конфигурации](#configuration-file-example).

## <a name="workflows"></a>Рабочие процессы

Бизнес-логику приложения можно смоделировать в качестве конечного автомата, когда при выполнении действия поток бизнес-логики переходит из одного состояния в другое. Рабочий процесс — это совокупность таких состояний и действий. Каждый рабочий процесс состоит из одного или нескольких интеллектуальных контрактов, которые представляют бизнес-логику в файлах кода. Исполняемый контракт — это экземпляр рабочего процесса.

| Поле | Описание | Обязательно | Максимальная длина |
|-------|-------------|:--------:|-----------:|
| Имя | Уникальное имя рабочего процесса. Для соответствующего смарт-контракта должно использоваться то же самое имя **Name**, что и для применимого класса контракта. | Да | 50 |
| DisplayName | Понятное отображаемое имя рабочего процесса. | Да | 255 |
| Описание | Описание рабочего процесса. | Нет | 255 |
| Инициаторы | Коллекция ролей [ApplicationRoles](#application-roles). Роли, назначаемые пользователям, которым разрешено создавать контракты в рабочем процессе. | Да | |
| StartState | Имя первоначального состояния рабочего процесса. | Да | |
| Свойства | Коллекция [идентификаторов](#identifiers). Представляет данные, которые могут считываться вне сети или визуализируются в средстве пользователя. | Да | |
| Конструктор | Определяет входные параметры для создания экземпляра рабочего процесса. | Да | |
| Функции | Коллекция [функций](#functions), которые могут выполняться в рабочем процессе. | Да | |
| Состояния | Коллекция [состояний](#states) рабочего процесса. | Да | |

См. [пример файла конфигурации](#configuration-file-example).

## <a name="type"></a>Type

Поддерживаемые типы данных.

| Type | Описание |
|-------|-------------|
| address  | Тип адреса блокчейна, такой как *контракты* или *пользователи* |
| array    | Одномерный массив типа integer, bool, money или time. Массивы могут быть статическими или динамическими. Используйте **ElementType**, чтобы указать тип данных элементов в массиве. Ознакомьтесь с [примером конфигурации](#example-configuration-of-type-array). |
| bool     | Логический тип данных. |
| contract | Адрес типа контракта. |
| enum     | Перечисляемый набор именованных значений. При использовании типа enum следует также указать список значений EnumValues. Каждое значение ограничено 255 символами. Допустимые символы в значениях — прописные и строчные буквы латинского алфавита (A–Z, a–z) и цифры (0–9). Ознакомьтесь с [примером конфигурации и использованием в Solidity](#example-configuration-of-type-enum). |
| INT      | Целочисленный тип данных. |
| money    | Денежный тип данных. |
| Состояние    | Состояние рабочего процесса. |
| строка  | Строковый тип данных. Не более 4000 знаков. Ознакомьтесь с [примером конфигурации](#example-configuration-of-type-string). |
| пользователь     | Адрес типа пользователя. |
| time     | Временной тип данных. |
|`[ Application Role Name ]`| Любое имя, указанное в роли приложения. Ограничивает пользователей этим типом роли. |

### <a name="example-configuration-of-type-array"></a>Пример конфигурации типа array

```json
{
  "Name": "Quotes",
  "Description": "Market quotes",
  "DisplayName": "Quotes",
  "Type": {
    "Name": "array",
    "ElementType": {
        "Name": "int"
    }
  }
}
```

#### <a name="using-a-property-of-type-array"></a>Использование свойства типа array

Если вы определяете в конфигурации свойство как тип array, необходимо добавить явно определяемую функцию get для возвращения общедоступного свойства типа array в Solidity. Пример:

```
function GetQuotes() public constant returns (int[]) {
     return Quotes;
}
```

### <a name="example-configuration-of-type-string"></a>Пример конфигурации типа string

``` json
{
  "Name": "description",
  "Description": "Descriptive text",
  "DisplayName": "Description",
  "Type": {
    "Name": "string"
  }
}
```

### <a name="example-configuration-of-type-enum"></a>Пример конфигурации типа enum

``` json
{
  "Name": "PropertyType",
  "DisplayName": "Property Type",
  "Description": "The type of the property",
  "Type": {
    "Name": "enum",
    "EnumValues": ["House", "Townhouse", "Condo", "Land"]
  }
}
```

#### <a name="using-enumeration-type-in-solidity"></a>Использование типа перечисления на Solidity

После определения enum в конфигурации можно использовать типы перечисления на Solidity. Например, можно определить перечисление с именем PropertyTypeEnum.

```
enum PropertyTypeEnum {House, Townhouse, Condo, Land} PropertyTypeEnum public PropertyType; 
```

Списки строк в конфигурации и смарт-контракте должны совпадать, чтобы в Blockchain Workbench формировались допустимые и согласованные объявления.

Пример назначения:

```
PropertyType = PropertyTypeEnum.Townhouse;
```

Пример параметра функции: 

``` 
function AssetTransfer(string description, uint256 price, PropertyTypeEnum propertyType) public
{
    InstanceOwner = msg.sender;
    AskingPrice = price;
    Description = description;
    PropertyType = propertyType;
    State = StateType.Active;
    ContractCreated();
}

```

## <a name="constructor"></a>Конструктор

Определяет входные параметры для экземпляра рабочего процесса.

| Поле | Описание | Обязательно |
|-------|-------------|:--------:|
| Параметры | Коллекция [идентификаторов](#identifiers), необходимых для инициирования смарт-контракта. | Да |

### <a name="constructor-example"></a>Пример конструктора

``` json
{
  "Parameters": [
    {
      "Name": "description",
      "Description": "The description of this asset",
      "DisplayName": "Description",
      "Type": {
        "Name": "string"
      }
    },
    {
      "Name": "price",
      "Description": "The price of this asset",
      "DisplayName": "Price",
      "Type": {
        "Name": "money"
      }
    }
  ]
}
```

## <a name="functions"></a>Функции

Определяет функции, которые могут выполняться в рабочем процессе.

| Поле | Описание | Обязательно | Максимальная длина |
|-------|-------------|:--------:|-----------:|
| Имя | Уникальное имя функции. Для соответствующего смарт-контракта должно использоваться то же самое имя **Name**, что и для применимой функции. | Да | 50 |
| DisplayName | Понятное отображаемое имя функции. | Да | 255 |
| Описание | Описание функции. | Нет | 255 |
| Параметры | Коллекция [идентификаторов](#identifiers), соответствующих параметрам функции. | Да | |

### <a name="functions-example"></a>Пример функций

``` json
"Functions": [
  {
    "Name": "Modify",
    "DisplayName": "Modify",
    "Description": "Modify the description/price attributes of this asset transfer instance",
    "Parameters": [
      {
        "Name": "description",
        "Description": "The new description of the asset",
        "DisplayName": "Description",
        "Type": {
          "Name": "string"
        }
      },
      {
        "Name": "price",
        "Description": "The new price of the asset",
        "DisplayName": "Price",
        "Type": {
          "Name": "money"
        }
      }
    ]
  },
  {
    "Name": "Terminate",
    "DisplayName": "Terminate",
    "Description": "Used to cancel this particular instance of asset transfer",
    "Parameters": []
  }
]

```

## <a name="states"></a>Состояния

Коллекция уникальных состояний рабочего процесса. Каждое состояние охватывает ступень в потоке управления бизнес-логики. 

| Поле | Описание | Обязательно | Максимальная длина |
|-------|-------------|:--------:|-----------:|
| Имя | Уникальное имя состояния. Для соответствующего смарт-контракта должно использоваться то же самое имя **Name**, что и для применимого состояния. | Да | 50 |
| DisplayName | Понятное отображаемое имя состояния. | Да | 255 |
| Описание | Описание состояния. | Нет | 255 |
| PercentComplete | Целочисленное значение в пользовательском интерфейсе Blockchain Workbench, которое отображает ход выполнения в потоке управления бизнес-логики. | Да | |
| Стиль | Визуальная подсказка, указывающая, каким является состояние: успешным или неуспешным. Есть два допустимых значения: `Success` или `Failure`. | Да | |
| Transitions | Коллекция доступных [переходов](#transitions) из текущего состояния в следующий набор состояний. | Нет | |

### <a name="states-example"></a>Пример раздела с состояниями

``` json
"States": [
    {
      "Name": "Active",
      "DisplayName": "Active",
      "Description": "The initial state of the asset transfer workflow",
      "PercentComplete": 20,
      "Style": "Success",
      "Transitions": [
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Cancels this instance of asset transfer",
          "Function": "Terminate",
          "NextStates": [ "Terminated" ],
          "DisplayName": "Terminate Offer"
        },
        {
          "AllowedRoles": [ "Buyer" ],
          "AllowedInstanceRoles": [],
          "Description": "Make an offer for this asset",
          "Function": "MakeOffer",
          "NextStates": [ "OfferPlaced" ],
          "DisplayName": "Make Offer"
        },
        {
          "AllowedRoles": [],
          "AllowedInstanceRoles": [ "InstanceOwner" ],
          "Description": "Modify attributes of this asset transfer instance",
          "Function": "Modify",
          "NextStates": [ "Active" ],
          "DisplayName": "Modify"
        }
      ]
    },
    {
      "Name": "Accepted",
      "DisplayName": "Accepted",
      "Description": "Asset transfer process is complete",
      "PercentComplete": 100,
      "Style": "Success",
      "Transitions": []
    },
    {
      "Name": "Terminated",
      "DisplayName": "Terminated",
      "Description": "Asset transfer has been canceled",
      "PercentComplete": 100,
      "Style": "Failure",
      "Transitions": []
    }
  ]
```

## <a name="transitions"></a>Transitions

Доступные действия для перехода в следующее состояние. Одна или несколько ролей пользователей могут выполнять действие в каждом состоянии, при этом действие может переходить из одного состояния в другое в рабочем процессе. 

| Поле | Описание | Обязательно |
|-------|-------------|:--------:|
| AllowedRoles | Список ролей приложения, которым разрешено инициировать переход. Все пользователи в указанной роли могут выполнять действия. | Нет |
| AllowedInstanceRoles | Список ролей пользователей, участвующих или указанных в смарт-контракте, которым разрешено инициировать переход. Роли экземпляра определяются в **свойствах** рабочих процессов. Поле AllowedInstanceRoles представляет пользователя, участвующего в смарт-контракте. AllowedInstanceRoles дает возможность ограничить действия для роли пользователя в экземпляре контракта.  Например, вы можете разрешить завершить работу пользователю, который создал контракт (InstanceOwner), а не всем пользователям в типе роли (Владелец), если вы указали роль в AllowedRoles. | Нет |
| DisplayName | Понятное отображаемое имя перехода. | Да |
| Описание | Описание перехода. | Нет |
| Компонент | Имя функции, которая инициирует переход. | Да |
| NextStates | Коллекция возможных следующих состояний после успешного перехода. | Да |

### <a name="transitions-example"></a>Пример раздела с переходами

``` json
"Transitions": [
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Cancels this instance of asset transfer",
    "Function": "Terminate",
    "NextStates": [ "Terminated" ],
    "DisplayName": "Terminate Offer"
  },
  {
    "AllowedRoles": [ "Buyer" ],
    "AllowedInstanceRoles": [],
    "Description": "Make an offer for this asset",
    "Function": "MakeOffer",
    "NextStates": [ "OfferPlaced" ],
    "DisplayName": "Make Offer"
  },
  {
    "AllowedRoles": [],
    "AllowedInstanceRoles": [ "InstanceOwner" ],
    "Description": "Modify attributes of this asset transfer instance",
    "Function": "Modify",
    "NextStates": [ "Active" ],
    "DisplayName": "Modify"
  }
]

```

## <a name="application-roles"></a>Роли приложений

Роли приложения определяют набор ролей, которые могут назначаться пользователям, которые хотят выполнять действия или быть участниками в приложении. Роли приложений можно использовать для ограничения действий и участия в блокчейн-приложении и соответствующих рабочих процессах. 

| Поле | Описание | Обязательно | Максимальная длина |
|-------|-------------|:--------:|-----------:|
| Имя | Уникальное имя роли приложения. Для соответствующего смарт-контракта должно использоваться то же самое имя **Name**, что и для применимой роли. Имена базового типа зарезервированы. Роли приложения нельзя присвоить имя, которое уже используется для типа [Type](#type)| Да | 50 |
| Описание | Описание роли приложения. | Нет | 255 |

### <a name="application-roles-example"></a>Пример раздела с ролями приложения

``` json
"ApplicationRoles": [
  {
    "Name": "Appraiser",
    "Description": "User that signs off on the asset price"
  },
  {
    "Name": "Buyer",
    "Description": "User that places an offer on an asset"
  }
]
```
## <a name="identifiers"></a>Идентификаторы

Идентификаторы представляют собой коллекцию сведений, используемых для описания свойств рабочего процесса, конструктора и параметров функции. 

| Поле | Описание | Обязательно | Максимальная длина |
|-------|-------------|:--------:|-----------:|
| Имя | Уникальное имя свойства или параметра. Для соответствующего смарт-контракта должно использоваться то же самое имя **Name**, что и для применимого свойства или параметра. | Да | 50 |
| DisplayName | Понятное отображаемое имя свойства или параметра. | Да | 255 |
| Описание | Описание свойства или параметра. | Нет | 255 |
| Type | [Тип данных](#type)свойства. | Да |

### <a name="identifiers-example"></a>Пример раздела с идентификаторами

``` json
"Properties": [
  {
    "Name": "State",
    "DisplayName": "State",
    "Description": "Holds the state of the contract",
    "Type": {
      "Name": "state"
    }
  },
  {
    "Name": "Description",
    "DisplayName": "Description",
    "Description": "Describes the asset being sold",
    "Type": {
      "Name": "string"
    }
  }
]
```

## <a name="configuration-file-example"></a>Пример файла конфигурации

Перемещение актива — это сценарий смарт-контракта для покупки и продажи высокоценных активов, для которого требуется инспектор и оценщик. Продавцы могут перечислять свои активы, создавая смарт-контракт перемещения активов. Покупатели могут делать предложения, выполняя действия в соответствии со смарт-контрактом, а другие стороны могут выполнять проверку или оценку актива. После того как актив будет отмечен как проверенный и оцененный, покупатель и продавец подтвердят продажу снова до завершения контракта. На каждом этапе процесса все участники могут просматривать состояние контракта по мере его обновления. 

Дополнительные сведения, включая файлы кода, см. в статье [Asset Transfer Sample for Azure Blockchain Workbench](https://github.com/Azure-Samples/blockchain/tree/master/blockchain-workbench/application-and-smart-contract-samples/asset-transfer) (Пример перемещения активов для Azure Blockchain Workbench).

Следующий файл конфигурации предназначен для примера перемещения активов:

``` json
{
  "ApplicationName": "AssetTransfer",
  "DisplayName": "Asset Transfer",
  "Description": "Allows transfer of assets between a buyer and a seller, with appraisal/inspection functionality",
  "ApplicationRoles": [
    {
      "Name": "Appraiser",
      "Description": "User that signs off on the asset price"
    },
    {
      "Name": "Buyer",
      "Description": "User that places an offer on an asset"
    },
    {
      "Name": "Inspector",
      "Description": "User that inspects the asset and signs off on inspection"
    },
    {
      "Name": "Owner",
      "Description": "User that signs off on the asset price"
    }
  ],
  "Workflows": [
    {
      "Name": "AssetTransfer",
      "DisplayName": "Asset Transfer",
      "Description": "Handles the business logic for the asset transfer scenario",
      "Initiators": [ "Owner" ],
      "StartState":  "Active",
      "Properties": [
        {
          "Name": "State",
          "DisplayName": "State",
          "Description": "Holds the state of the contract",
          "Type": {
            "Name": "state"
          }
        },
        {
          "Name": "Description",
          "DisplayName": "Description",
          "Description": "Describes the asset being sold",
          "Type": {
            "Name": "string"
          }
        },
        {
          "Name": "AskingPrice",
          "DisplayName": "Asking Price",
          "Description": "The asking price for the asset",
          "Type": {
            "Name": "money"
          }
        },
        {
          "Name": "OfferPrice",
          "DisplayName": "Offer Price",
          "Description": "The price being offered for the asset",
          "Type": {
            "Name": "money"
          }
        },
        {
          "Name": "InstanceAppraiser",
          "DisplayName": "Instance Appraiser",
          "Description": "The user that appraises the asset",
          "Type": {
            "Name": "Appraiser"
          }
        },
        {
          "Name": "InstanceBuyer",
          "DisplayName": "Instance Buyer",
          "Description": "The user that places an offer for this asset",
          "Type": {
            "Name": "Buyer"
          }
        },
        {
          "Name": "InstanceInspector",
          "DisplayName": "Instance Inspector",
          "Description": "The user that inspects this asset",
          "Type": {
            "Name": "Inspector"
          }
        },
        {
          "Name": "InstanceOwner",
          "DisplayName": "Instance Owner",
          "Description": "The seller of this particular asset",
          "Type": {
            "Name": "Owner"
          }
        }
      ],
      "Constructor": {
        "Parameters": [
          {
            "Name": "description",
            "Description": "The description of this asset",
            "DisplayName": "Description",
            "Type": {
              "Name": "string"
            }
          },
          {
            "Name": "price",
            "Description": "The price of this asset",
            "DisplayName": "Price",
            "Type": {
              "Name": "money"
            }
          }
        ]
      },
      "Functions": [
        {
          "Name": "Modify",
          "DisplayName": "Modify",
          "Description": "Modify the description/price attributes of this asset transfer instance",
          "Parameters": [
            {
              "Name": "description",
              "Description": "The new description of the asset",
              "DisplayName": "Description",
              "Type": {
                "Name": "string"
              }
            },
            {
              "Name": "price",
              "Description": "The new price of the asset",
              "DisplayName": "Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Terminate",
          "DisplayName": "Terminate",
          "Description": "Used to cancel this particular instance of asset transfer",
          "Parameters": []
        },
        {
          "Name": "MakeOffer",
          "DisplayName": "Make Offer",
          "Description": "Place an offer for this asset",
          "Parameters": [
            {
              "Name": "inspector",
              "Description": "Specify a user to inspect this asset",
              "DisplayName": "Inspector",
              "Type": {
                "Name": "Inspector"
              }
            },
            {
              "Name": "appraiser",
              "Description": "Specify a user to appraise this asset",
              "DisplayName": "Appraiser",
              "Type": {
                "Name": "Appraiser"
              }
            },
            {
              "Name": "offerPrice",
              "Description": "Specify your offer price for this asset",
              "DisplayName": "Offer Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Reject",
          "DisplayName": "Reject",
          "Description": "Reject the user's offer",
          "Parameters": []
        },
        {
          "Name": "AcceptOffer",
          "DisplayName": "Accept Offer",
          "Description": "Accept the user's offer",
          "Parameters": []
        },
        {
          "Name": "RescindOffer",
          "DisplayName": "Rescind Offer",
          "Description": "Rescind your placed offer",
          "Parameters": []
        },
        {
          "Name": "ModifyOffer",
          "DisplayName": "Modify Offer",
          "Description": "Modify the price of your placed offer",
          "Parameters": [
            {
              "Name": "offerPrice",
              "DisplayName": "Price",
              "Type": {
                "Name": "money"
              }
            }
          ]
        },
        {
          "Name": "Accept",
          "DisplayName": "Accept",
          "Description": "Accept the inspection/appraisal results",
          "Parameters": []
        },
        {
          "Name": "MarkInspected",
          "DisplayName": "Mark Inspected",
          "Description": "Mark the asset as inspected",
          "Parameters": []
        },
        {
          "Name": "MarkAppraised",
          "DisplayName": "Mark Appraised",
          "Description": "Mark the asset as appraised",
          "Parameters": []
        }
      ],
      "States": [
        {
          "Name": "Active",
          "DisplayName": "Active",
          "Description": "The initial state of the asset transfer workflow",
          "PercentComplete": 20,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancels this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate Offer"
            },
            {
              "AllowedRoles": [ "Buyer" ],
              "AllowedInstanceRoles": [],
              "Description": "Make an offer for this asset",
              "Function": "MakeOffer",
              "NextStates": [ "OfferPlaced" ],
              "DisplayName": "Make Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Modify attributes of this asset transfer instance",
              "Function": "Modify",
              "NextStates": [ "Active" ],
              "DisplayName": "Modify"
            }
          ]
        },
        {
          "Name": "OfferPlaced",
          "DisplayName": "Offer Placed",
          "Description": "Offer has been placed for the asset",
          "PercentComplete": 30,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Accept the proposed offer for the asset",
              "Function": "AcceptOffer",
              "NextStates": [ "PendingInspection" ],
              "DisplayName": "Accept Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you previously placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Modify the price that you specified for your offer",
              "Function": "ModifyOffer",
              "NextStates": [ "OfferPlaced" ],
              "DisplayName": "Modify Offer"
            }
          ]
        },
        {
          "Name": "PendingInspection",
          "DisplayName": "Pending Inspection",
          "Description": "Asset is pending inspection",
          "PercentComplete": 40,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceInspector" ],
              "Description": "Mark this asset as inspected",
              "Function": "MarkInspected",
              "NextStates": [ "Inspected" ],
              "DisplayName": "Mark Inspected"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceAppraiser" ],
              "Description": "Mark this asset as appraised",
              "Function": "MarkAppraised",
              "NextStates": [ "Appraised" ],
              "DisplayName": "Mark Appraised"
            }
          ]
        },
        {
          "Name": "Inspected",
          "DisplayName": "Inspected",
          "PercentComplete": 45,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceAppraiser" ],
              "Description": "Mark this asset as appraised",
              "Function": "MarkAppraised",
              "NextStates": [ "NotionalAcceptance" ],
              "DisplayName": "Mark Appraised"
            }
          ]
        },
        {
          "Name": "Appraised",
          "DisplayName": "Appraised",
          "Description": "Asset has been appraised, now awaiting inspection",
          "PercentComplete": 45,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the offer",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel the offer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceInspector" ],
              "Description": "Mark the asset as inspected",
              "Function": "MarkInspected",
              "NextStates": [ "NotionalAcceptance" ],
              "DisplayName": "Mark Inspected"
            }
          ]
        },
        {
          "Name": "NotionalAcceptance",
          "DisplayName": "Notional Acceptance",
          "Description": "Asset has been inspected and appraised, awaiting final sign-off from buyer and seller",
          "PercentComplete": 50,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "SellerAccepted" ],
              "DisplayName": "SellerAccept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "BuyerAccepted" ],
              "DisplayName": "BuyerAccept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            }
          ]
        },
        {
          "Name": "BuyerAccepted",
          "DisplayName": "Buyer Accepted",
          "Description": "Buyer has signed-off on inspection and appraisal",
          "PercentComplete": 75,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "SellerAccepted" ],
              "DisplayName": "Accept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Reject the proposed offer for the asset",
              "Function": "Reject",
              "NextStates": [ "Active" ],
              "DisplayName": "Reject"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceOwner" ],
              "Description": "Cancel this instance of asset transfer",
              "Function": "Terminate",
              "NextStates": [ "Terminated" ],
              "DisplayName": "Terminate"
            }
          ]
        },
        {
          "Name": "SellerAccepted",
          "DisplayName": "Seller Accepted",
          "Description": "Seller has signed-off on inspection and appraisal",
          "PercentComplete": 75,
          "Style": "Success",
          "Transitions": [
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Sign-off on inspection and appraisal",
              "Function": "Accept",
              "NextStates": [ "Accepted" ],
              "DisplayName": "Accept"
            },
            {
              "AllowedRoles": [],
              "AllowedInstanceRoles": [ "InstanceBuyer" ],
              "Description": "Rescind the offer you placed for this asset",
              "Function": "RescindOffer",
              "NextStates": [ "Active" ],
              "DisplayName": "Rescind Offer"
            }
          ]
        },
        {
          "Name": "Accepted",
          "DisplayName": "Accepted",
          "Description": "Asset transfer process is complete",
          "PercentComplete": 100,
          "Style": "Success",
          "Transitions": []
        },
        {
          "Name": "Terminated",
          "DisplayName": "Terminated",
          "Description": "Asset transfer has been canceled",
          "PercentComplete": 100,
          "Style": "Failure",
          "Transitions": []
        }
      ]
    }
  ]
}
```
## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Azure Blockchain Workbench REST API](/rest/api/azure-blockchain-workbench) (REST API Azure Blockchain Workbench)
