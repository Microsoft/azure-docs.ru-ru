---
title: Определяемые пользователем функции в шаблонах
description: Описывает, как определять и использовать определяемые пользователем функции в шаблоне Azure Resource Manager (шаблон ARM).
ms.topic: conceptual
ms.date: 02/11/2021
ms.openlocfilehash: 9c7480958e6315c8aea1fd8d12613bcf9d606723
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100379630"
---
# <a name="user-defined-functions-in-arm-template"></a>Определяемые пользователем функции в шаблоне ARM

В шаблоне можно создать свои собственные функции. Эти функции доступны для использования в шаблоне. Определяемые пользователем функции отделены от [стандартных функций шаблонов](template-functions.md) , которые автоматически доступны в шаблоне. Создавайте собственные функции при наличии сложных выражений, которые многократно используются в шаблоне.

В этой статье описывается добавление определяемых пользователем функций в шаблон Azure Resource Manager (шаблон ARM).

## <a name="define-the-function"></a>Определение функции

Для функции требуется значение пространства имен, чтобы избежать конфликтов именования с функциями шаблона. В следующем примере показана функция, возвращающая уникальное имя:

```json
"functions": [
  {
    "namespace": "contoso",
    "members": {
      "uniqueName": {
        "parameters": [
          {
            "name": "namePrefix",
            "type": "string"
          }
        ],
        "output": {
          "type": "string",
          "value": "[concat(toLower(parameters('namePrefix')), uniqueString(resourceGroup().id))]"
        }
      }
    }
  }
],
```

## <a name="use-the-function"></a>Использование функции

В следующем примере показан шаблон, который содержит определяемую пользователем функцию для получения уникального имени для учетной записи хранения. Шаблон имеет параметр с именем `storageNamePrefix` , который передается в качестве параметра функции.

```json
{
 "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
 "contentVersion": "1.0.0.0",
 "parameters": {
   "storageNamePrefix": {
     "type": "string",
     "maxLength": 11
   }
 },
 "functions": [
  {
    "namespace": "contoso",
    "members": {
      "uniqueName": {
        "parameters": [
          {
            "name": "namePrefix",
            "type": "string"
          }
        ],
        "output": {
          "type": "string",
          "value": "[concat(toLower(parameters('namePrefix')), uniqueString(resourceGroup().id))]"
        }
      }
    }
  }
],
 "resources": [
   {
     "type": "Microsoft.Storage/storageAccounts",
     "apiVersion": "2019-04-01",
     "name": "[contoso.uniqueName(parameters('storageNamePrefix'))]",
     "location": "South Central US",
     "sku": {
       "name": "Standard_LRS"
     },
     "kind": "StorageV2",
     "properties": {
       "supportsHttpsTrafficOnly": true
     }
   }
 ]
}
```

Во время развертывания `storageNamePrefix` параметр передается в функцию:

* Шаблон определяет параметр с именем `storageNamePrefix` .
* Функция использует, `namePrefix` поскольку можно использовать только параметры, определенные в функции. Дополнительные сведения см. в разделе [ограничения](#limitations).
* В `resources` разделе шаблона `name` элемент использует функцию и передает `storageNamePrefix` значение в функцию `namePrefix` .

## <a name="limitations"></a>Ограничения

При определении пользовательской функции есть несколько ограничений:

* Функция не может обращаться к переменным.
* Функция может использовать только параметры, определенные в самой функции. При использовании функции [Parameters](template-functions-deployment.md#parameters) в определяемой пользователем функции вы можете использовать только параметры этой функции.
* Функция не может вызывать другие функции, определяемые пользователем.
* Функция не может использовать функцию [Reference](template-functions-resource.md#reference) или любую из функций [списка](template-functions-resource.md#list) .
* Для параметров этой функции нельзя задавать значения по умолчанию.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о доступных свойствах определяемых пользователем функций см. в разделе Общие сведения о [структуре и синтаксисе шаблонов ARM](template-syntax.md).
* Список доступных функций шаблонов см. в разделе [функции шаблонов ARM](template-functions.md).
