---
title: Создание ссылок на функции определения пользовательского интерфейса
description: Описывает функции, используемые при создании определений пользовательского интерфейса для портал Azure, ссылающихся на другие объекты.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: ad21c5b34a58c35b2cef5e430be7cb8cd1296402
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87098173"
---
# <a name="createuidefinition-referencing-functions"></a>CreateUiDefinition, ссылающиеся на функции

Функции, используемые при ссылке на выходные данные из свойств или контекста файла CreateUiDefinition.

## <a name="basics"></a>basics

Возвращает выходные значения элемента, определенного на этапе [основ](create-uidefinition-overview.md#basics) . Передайте имя элемента в качестве параметра этой функции.

Чтобы получить выходные значения элементов в других шагах, используйте функцию [шаги ()](#steps) .

В следующем примере возвращаются выходные данные элемента с именем `clusterName` на шаге basics.

```json
"[basics('clusterName')]"
```

Возвращаемые значения зависят от типа извлекаемого элемента.

## <a name="location"></a>location

Возвращает расположение, выбранное на шаге basics или в текущем контексте.

В следующем примере возвращается значение, например `"westus"` :

```json
"[location()]"
```

## <a name="resourcegroup"></a>resourceGroup

Возвращает сведения о параметре resourceGroup, выбранном на шаге основные или в текущем контексте.

В следующем примере происходит следующее:

```json
"[resourceGroup()]"
```

Возвращает следующие свойства:

```json
{
    "mode": "New" or "Existing",
    "name": "{resourceGroupName}",
    "location": "{resourceGroupLocation}"
}
```

Любое конкретное значение можно получить с помощью точечной нотации.

```json
"[resourceGroup().name]"
```

## <a name="steps"></a>steps

Возвращает элементы на указанном шаге. Передайте имя шага в качестве параметра этой функции. Из возвращенных элементов можно получить определенные значения свойств.

Чтобы получить выходные значения элементов на этапе основ, используйте функцию [основные ()](#basics) .

В следующем примере возвращается шаг с именем `vmParameters` . На этом шаге — это элемент с именем `adminUsername` .

```json
"[steps('vmParameters').adminUsername]"
```

## <a name="subscription"></a>Подписка

Возвращает свойства для подписки, выбранной на этапе базовых сведений или в текущем контексте.

В следующем примере происходит следующее:

```json
"[subscription()]"
```

Возвращает следующие свойства:

```json
{
    "id": "/subscriptions/{subscription-id}",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}",
    "displayName": "{name-of-subscription}"
}
```

## <a name="next-steps"></a>Дальнейшие действия

* Общие сведения о разработке интерфейса портала см. в статье [CreateUiDefinition.jsо создании управляемого приложения Azure](create-uidefinition-overview.md).
