---
title: Сведения о структуре определения инициативы
description: Описание использования определений инициативы политики для определения групповых политик для развертывания в ресурсах Azure в Организации.
ms.date: 03/16/2021
ms.topic: conceptual
ms.openlocfilehash: edd3f25dd528d1a718c9287c9f30988b87fb73e2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104587225"
---
# <a name="azure-policy-initiative-definition-structure"></a>Структура определения инициативы политики Azure

Инициативы дают возможность сгруппировать несколько связанных определений политик, чтобы упростить их назначение и управление ими, так как это позволяет работать с группой как с единым элементом. Например, можно сгруппировать связанные определения политик тегов в одной инициативе. Вместо назначения каждой политики по отдельности вы сможете применить инициативу.

Для создания определения инициативы политики используется JSON. Определение инициативы политики содержит элементы для:

- display name
- description
- метаданные
- parameters
- определения политик
- группы политик (это свойство является частью [функции соответствия нормативным требованиям (Предварительная версия)](./regulatory-compliance.md))

В следующем примере показано, как создать инициативу для обработки двух тегов, `costCenter` и `productName`. Для применения значения тега по умолчанию используются две встроенные политики.

```json
{
    "properties": {
        "displayName": "Billing Tags Policy",
        "policyType": "Custom",
        "description": "Specify cost Center tag and product name tag",
        "metadata": {
            "version": "1.0.0",
            "category": "Tags"
        },
        "parameters": {
            "costCenterValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for Cost Center tag"
                },
                "defaultValue": "DefaultCostCenter"
            },
            "productNameValue": {
                "type": "String",
                "metadata": {
                    "description": "required value for product Name tag"
                },
                "defaultValue": "DefaultProduct"
            }
        },
        "policyDefinitions": [{
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "costCenter"
                    },
                    "tagValue": {
                        "value": "[parameters('costCenterValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            },
            {
                "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/2a0e14a6-b0a6-4fab-991a-187a4f81c498",
                "parameters": {
                    "tagName": {
                        "value": "productName"
                    },
                    "tagValue": {
                        "value": "[parameters('productNameValue')]"
                    }
                }
            }
        ]
    }
}
```

Встроенные и шаблоны политики Azure приведены в [примерах политики Azure](../samples/index.md).

## <a name="metadata"></a>Метаданные

Необязательное `metadata` свойство сохраняет сведения об определении инициативы политики.
Клиенты могут определять любые свойства и значения, полезные для Организации в `metadata` . Однако существуют некоторые _Общие_ свойства, используемые политикой Azure и встроенными модулями.

### <a name="common-metadata-properties"></a>Общие свойства метаданных

- `version` (строка): отслеживает сведения о версии содержимого определения инициативы политики.
- `category` (строка): определяет, в какой категории в портал Azure отображается определение политики.

  > [!NOTE]
  > Для инициативы по соответствию [нормативным требованиям](./regulatory-compliance.md) `category` должно быть **соблюдение нормативных требований**.

- `preview` (Boolean): флаг true или false для, если определение инициативы политики является _предварительной версией_.
- `deprecated` (Boolean): флаг true или false для, если определение инициативы политики помечено как _нерекомендуемое_.

> [!NOTE]
> Служба Политика Azure использует свойства `version`, `preview` и `deprecated` для передачи уровня изменений встроенному определению политики или инициативе и состоянию. Формат `version` — `{Major}.{Minor}.{Patch}`. Конкретные состояния, такие как _устаревший_ или _предварительная версия_, добавляются к свойству `version` или в другом свойстве как **логическое значение**. Дополнительные сведения о способах встроенных версий политик Azure см. в разделе [Встроенная поддержка версий](https://github.com/Azure/azure-policy/blob/master/built-in-policies/README.md).

## <a name="parameters"></a>Параметры

Параметры помогают упростить управление политиками за счет сокращения числа определений политик. Параметры можно рассматривать как поля в форме: `name`, `address`, `city`, `state`. Эти параметры никогда не меняются, однако их значения изменяются в зависимости от того, как пользователь заполняет форму.
Параметры работают так же, как при создании инициатив по политике. Включив параметры в определение инициативы политики, можно повторно использовать этот параметр в включенных политиках.

> [!NOTE]
> После назначения инициативы невозможно изменить параметры уровня инитативе. Из-за этого рекомендуется устанавливать **defaultValue** при определении параметра.

### <a name="parameter-properties"></a>Свойства параметра

Параметр имеет следующие свойства, которые используются в определении инициативы политики:

- `name`: Имя параметра. Используется функцией развертывания `parameters` в правиле политики. Дополнительные сведения см. в разделе [об использовании значения параметра](#passing-a-parameter-value-to-a-policy-definition).
- `type`: Определяет, является ли параметр **строкой**, **массивом**, **объектом**, **логическим значением**, **целым числом**, **числом с плавающей запятой** или **значением даты/времени**.
- `metadata`: Определяет вложенное свойство, которое в основном используется для отображения понятной пользователям информации на портале Azure:
  - `description`: Объясняет назначение параметра. Можно использовать для примеров допустимых значений.
  - `displayName`: Понятное имя параметра, которое отображается на портале.
  - `strongType`: (Необязательно.) Используется при назначении определения политики через портал. Предоставляет список с учетом контекста. См. дополнительные сведения о [вложенном свойстве strongType](#strongtype).
- `defaultValue`: (Необязательно.) Устанавливает значение параметра в назначении, если значение не указано.
- `allowedValues`: (Необязательно.) Предоставляет массив значений, которые параметр принимает во время назначения.

Например, определение инициативы политики можно определить для ограничения расположения ресурсов в различных включаемых определениях политик. Параметр для этого определения инициативы политики может быть **allowedLocations**. Затем параметр доступен для каждого включенного определения политики и определяется во время назначения инициативы политики.

```json
"parameters": {
    "init_allowedLocations": {
        "type": "array",
        "metadata": {
            "description": "The list of allowed locations for resources.",
            "displayName": "Allowed locations",
            "strongType": "location"
        },
        "defaultValue": [ "westus2" ],
        "allowedValues": [
            "eastus2",
            "westus2",
            "westus"
        ]
    }
}
```

### <a name="passing-a-parameter-value-to-a-policy-definition"></a>Передача значения параметра в определение политики

Вы определяете, какие из параметров инициативы были переданы, включая определения политик в массиве [PolicyDefinitions](#policy-definitions) определения инициативы. Хотя имя параметра может быть одинаковым, использование различных имен в инициативах, чем в определениях политик, упрощает удобочитаемость кода.

Например, параметр **init_allowedLocations** инициативы, определенный ранее, можно передать в несколько включаемых определений политик и их параметров, **sql_locations** и **vm_locations** следующим образом:

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0ec8fc28-d5b7-4603-8fec-39044f00a92b",
        "policyDefinitionReferenceId": "allowedLocationsSQL",
        "parameters": {
            "sql_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/aa09bd0f-aa5f-4343-b6ab-a33a6a6304f3",
        "policyDefinitionReferenceId": "allowedLocationsVMs",
        "parameters": {
            "vm_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    }
]
```

Этот образец ссылается на параметр **init_allowedLocations** , который был показан в [свойствах параметров](#parameter-properties).

### <a name="strongtype"></a>strongType

В свойстве `metadata` можно использовать вложенное свойство **strongType**, чтобы предоставить список для выбора параметров на портале Azure. **strongType** может быть поддерживаемым _типом ресурса_ или допустимым значением. Чтобы определить, допустим ли _тип ресурса_ для **strongType**, используйте [Get-AzResourceProvider](/powershell/module/az.resources/get-azresourceprovider).

Некоторые поддерживаемые типы ресурсов **Get-AzResourceProvider** не возвращает. Эти типы ресурсов:

- `Microsoft.RecoveryServices/vaults/backupPolicies`

Для **стронгтипе** допустимы следующие значения типа, не относящегося к ресурсу:

- `location`
- `resourceTypes`
- `storageSkus`
- `vmSKUs`
- `existingResourceGroups`

## <a name="policy-definitions"></a>Определения политик

`policyDefinitions`Часть определения инициативы — это _массив_ , из которого в инициативу включены существующие определения политик. Как упоминалось при [передаче значения параметра в определение политики](#passing-a-parameter-value-to-a-policy-definition), это свойство позволяет передавать [Параметры инициативы](#parameters) в определение политики.

### <a name="policy-definition-properties"></a>Свойства определения политики

Каждый элемент _массива_ , представляющий определение политики, имеет следующие свойства.

- `policyDefinitionId` (строка): идентификатор пользовательского или встроенного определения политики для включения.
- `policyDefinitionReferenceId` (строка): краткое имя для определения включаемой политики.
- `parameters`(Необязательно.) пары "имя-значение" для передачи параметра инициативы в включаемое определение политики в качестве свойства в определении политики. Дополнительные сведения см. в разделе [Параметры](#parameters).
- `groupNames` (массив строк): (необязательно) группа, членом которой является определение политики. Дополнительные сведения см. в разделе [группы политик](#policy-definition-groups).

Ниже приведен пример с `policyDefinitions` двумя включенными определениями политик, в каждом из которых передан один и тот же параметр инициативы:

```json
"policyDefinitions": [
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/0ec8fc28-d5b7-4603-8fec-39044f00a92b",
        "policyDefinitionReferenceId": "allowedLocationsSQL",
        "parameters": {
            "sql_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    },
    {
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/aa09bd0f-aa5f-4343-b6ab-a33a6a6304f3",
        "policyDefinitionReferenceId": "allowedLocationsVMs",
        "parameters": {
            "vm_locations": {
                "value": "[parameters('init_allowedLocations')]"
            }
        }
    }
]
```

## <a name="policy-definition-groups"></a>Группы определений политик

Определения политик в определении инициативы можно группировать и классифицировать. Функция [соответствия нормативным требованиям](./regulatory-compliance.md) (Предварительная версия) политики Azure использует это свойство для группирования определений в **элементы управления** и **домены соответствия**. Эти сведения определяются в `policyDefinitionGroups` свойстве _Array_ . Дополнительные сведения о группировании можно найти в объекте **полициметадата** , созданном корпорацией Майкрософт. Дополнительные сведения см. в разделе [Metadata Objects](#metadata-objects).

### <a name="policy-definition-groups-parameters"></a>Параметры групп определений политик

Каждый элемент _массива_ в `policyDefinitionGroups` должен иметь оба следующих свойства:

- `name` (строка) \[ обязательное \] — короткое имя **группы**. В соответствии с нормативным соответствием **элементу управления**. Значение этого свойства используется `groupNames` в `policyDefinitions` .
- `category` (строка): иерархия, к которой принадлежит группа. В соответствии с нормативными требованиями — **домен соответствия** элемента управления.
- `displayName` (строка): понятное имя **группы** или **элемента управления**. Используется порталом.
- `description` (строка): описание того, что охватывает **Группа** или **элемент управления** .
- `additionalMetadataId` (строка): расположение объекта [полициметадата](#metadata-objects) , в котором содержатся дополнительные сведения об **элементе управления** и **домене соответствия**.

  > [!NOTE]
  > Клиенты могут указывать на существующий объект [полициметадата](#metadata-objects) . Однако эти объекты доступны _только для чтения_ и создаются только корпорацией Майкрософт.

Пример `policyDefinitionGroups` свойства из встроенного определения инициативы NIST выглядит следующим образом:

```json
"policyDefinitionGroups": [
    {
        "name": "NIST_SP_800-53_R4_AC-1",
        "additionalMetadataId": "/providers/Microsoft.PolicyInsights/policyMetadata/NIST_SP_800-53_R4_AC-1"
    }
]
```

### <a name="metadata-objects"></a>Объекты метаданных

Встроенные системы соответствия нормативным требованиям, созданные корпорацией Майкрософт, содержат дополнительные сведения о каждом элементе управления.
Эти сведения:

- Отображается в портал Azure в обзоре **элемента управления** в рамках инициативы по соответствию нормативным требованиям.
- Доступно через REST API. См. сведения о `Microsoft.PolicyInsights` поставщике ресурсов и [группе операций полициметадата](/rest/api/policy/policymetadata/getresource).
- Доступно через Azure CLI. См. команду [AZ Policy Metadata](/cli/azure/policy/metadata) .

> [!IMPORTANT]
> Объекты метаданных для обеспечения соответствия нормативным требованиям доступны _только для чтения_ и не могут быть созданы клиентами.

Метаданные для группирования политики имеют следующие сведения в `properties` узле:

- `metadataId`: **Идентификатор элемента управления** , к которому относится группирование.
- `category` (обязательно): **домен соответствия** , к которому относится **элемент управления** .
- `title` (обязательно): понятное имя **идентификатора элемента управления**.
- `owner` (обязательно): определяет, кто несет ответственность за элемент управления в Azure: _Customer_, _Microsoft_, _Shared_.
- `description`: Дополнительные сведения об элементе управления.
- `requirements`: Сведения о ответственности реализации элемента управления.
- `additionalContentUrl`: Ссылка на дополнительные сведения об элементе управления. Это свойство обычно является ссылкой на раздел документации, посвященный этому элементу управления в стандарте соответствия.

Ниже приведен пример объекта **полициметадата** . Этот пример метаданных относится к элементу управления _AC-1 в NIST SP 800-53_ .

```json
{
  "properties": {
    "metadataId": "NIST SP 800-53 R4 AC-1",
    "category": "Access Control",
    "title": "Access Control Policy and Procedures",
    "owner": "Shared",
    "description": "**The organization:**    \na. Develops, documents, and disseminates to [Assignment: organization-defined personnel or roles]:  \n1. An access control policy that addresses purpose, scope, roles, responsibilities, management commitment, coordination among organizational entities, and compliance; and  \n2. Procedures to facilitate the implementation of the access control policy and associated access controls; and  \n  
\nb. Reviews and updates the current:  \n1. Access control policy [Assignment: organization-defined frequency]; and  \n2. Access control procedures [Assignment: organization-defined frequency].",
    "requirements": "**a.**  The customer is responsible for developing, documenting, and disseminating access control policies and procedures. The customer access control policies and procedures address access to all customer-deployed resources and customer system access (e.g., access to customer-deployed virtual machines, access to customer-built applications).  \n**b.**  The customer is responsible for reviewing and updating access control policies and procedures in accordance with FedRAMP requirements.",
    "additionalContentUrl": "https://nvd.nist.gov/800-53/Rev4/control/AC-1"
  },
  "id": "/providers/Microsoft.PolicyInsights/policyMetadata/NIST_SP_800-53_R4_AC-1",
  "name": "NIST_SP_800-53_R4_AC-1",
  "type": "Microsoft.PolicyInsights/policyMetadata"
}
```

## <a name="next-steps"></a>Дальнейшие действия

- См. [структуру определения](./definition-structure.md)
- См. другие [примеры шаблонов для службы Политика Azure](../samples/index.md).
- Изучите [сведения о действии политик](effects.md).
- Узнайте о [программном создании политик](../how-to/programmatically-create.md).
- Узнайте, как [получать сведения о соответствии](../how-to/get-compliance-data.md).
- Узнайте, как [исправлять несоответствующие ресурсы](../how-to/remediate-resources.md).
- Дополнительные сведения о группе управления см. в статье [Упорядочивание ресурсов с помощью групп управления Azure](../../management-groups/overview.md).
