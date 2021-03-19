---
title: Подключение ресурсов
description: Сведения о выполнении адаптации ресурсов с помощью настраиваемых поставщиков Azure для применения управления или настройки к другим типам ресурсов Azure.
author: jjbfour
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: jobreen
ms.openlocfilehash: 1846b036f12fe7e691021ec0248782cad946d9b7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "75650413"
---
# <a name="azure-custom-providers-resource-onboarding-overview"></a>Обзор адаптации ресурсов настраиваемых поставщиков Azure

Адаптация ресурсов настраиваемых поставщиков Azure — это модель расширяемости для типов ресурсов Azure. Она позволяет применять операции или управление в существующих ресурсах Azure в нужном масштабе. Дополнительные сведения см. в статье [как настраиваемые поставщики Azure могут расширить Azure](overview.md). В этой статье рассматриваются следующие вопросы:

- Возможности адаптации ресурсов.
- Основные сведения о адаптации ресурсов и способах ее использования.
- Где найти руководства и примеры кода, чтобы приступить к работе.

> [!IMPORTANT]
> В настоящее время настраиваемые поставщики находятся в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок в рабочей среде. Некоторые функции могут быть не поддерживаются или могут иметь ограниченные возможности.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="what-can-resource-onboarding-do"></a>Что может делать ресурс при адаптации?

Как и настраиваемые [ресурсы настраиваемых поставщиков Azure](./custom-providers-resources-endpoint-how-to.md), адаптации ресурсов определяет контракт, который будет выполнять запросы прокси-сервера к конечной точке. В отличие от настраиваемых ресурсов, при адаптации ресурсов не создается новый тип ресурса. Вместо этого он разрешает расширение существующих типов ресурсов. И подключение ресурсов работает с политикой Azure, поэтому управление и Настройка ресурсов можно выполнять в масштабе. Некоторые примеры рабочих процессов адаптации ресурсов:

- Установка и управление расширениями виртуальной машины.
- Отправка и Настройка значений по умолчанию для учетных записей хранения Azure.
- Включение базовых параметров диагностики в масштабе.

## <a name="resource-onboarding-basics"></a>Основные сведения о адаптации ресурсов

Вы настраиваете подключение ресурсов с помощью настраиваемых поставщиков Azure с помощью типов ресурсов Microsoft. Кустомпровидерс/Ресаурцепровидерс и Microsoft. Кустомпровидерс/Associations. Чтобы включить подключение ресурсов для настраиваемого поставщика, в процессе настройки создайте **ResourceType** с именем "Associations" и **раутингтипе** , включающий "Extension". Microsoft. Кустомпровидерс/Associations и Microsoft. Кустомпровидерс/Ресаурцепровидерс не должны принадлежать к одной группе ресурсов.

Ниже приведен пример настраиваемого поставщика Azure:

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "associations",
        "routingType": "Proxy,Cache,Extension",
        "endpoint": "https://microsoft.com/"
      }
    ]
  },
  "location": "eastus"
}
```

Свойство | Необходим? | Описание
---|---|---
name | Да | Имя определения конечной точки. Для адаптации ресурсов имя должно иметь значение "Associations".
routingType | Да | Определяет тип контракта с конечной точкой. Для адаптации ресурсов допустимыми **раутингтипес** являются "прокси-сервер, кэш, расширение" и "веб-перехватчик, кэш, расширение".
endpoint | Да | Конечная точка для направления запросов. Это приведет к обработке отклика и любых побочных эффектов запроса.

После создания настраиваемого поставщика с типом ресурса Associations можно ориентироваться с помощью Microsoft. Кустомпровидерс/Associations. Microsoft. Кустомпровидерс/Associations — это ресурс расширения, который может расширить любой другой ресурс Azure. При создании экземпляра Microsoft. Кустомпровидерс/Associations будет использоваться свойство **targetResourceId**, которое должно быть ДОПУСТИМЫм идентификатором ресурса Microsoft. Кустомпровидерс/Ресаурцепровидерс или Microsoft. Solutions/Applications. В таких случаях запрос будет перенаправлен к типу ресурса Associations в созданном вами экземпляре Microsoft. Кустомпровидерс/Ресаурцепровидерс.

> [!NOTE]
> Если идентификатор ресурса Microsoft. Solutions/Applications указан в качестве **targetResourceId**, в управляемой группе ресурсов должно быть развернуто приложение Microsoft. Кустомпровидерс/ресаурцепровидерс с именем "Public".

Пример сопоставления настраиваемых поставщиков Azure:

```JSON
{
  "properties": {
    "targetResourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}",
    ...
  }
}
```

Свойство | Необходим? | Описание
---|---|---
targetResourceId | Да | Идентификатор ресурса Microsoft. Кустомпровидерс/Ресаурцепровидерс или Microsoft. Solutions/Applications.

## <a name="how-to-use-resource-onboarding"></a>Использование адаптации ресурсов

Адаптация ресурсов работает путем расширения других ресурсов с помощью ресурса расширения Microsoft. Кустомпровидерс/Associations. В следующем примере запрос выполняется для виртуальной машины, но любой ресурс можно расширить.

Сначала необходимо создать настраиваемый ресурс поставщика с типом ресурса Associations. При этом будет объявлен URL-адрес обратного вызова, который будет использоваться при создании соответствующего ресурса Microsoft. Кустомпровидерс/Associations, который предназначен для пользовательского поставщика.

Образец запроса на создание для Microsoft. Кустомпровидерс/Ресаурцепровидерс:

``` HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json

{
  "properties": {
    "resourceTypes": [
      {
        "name": "associations",
        "routingType": "Proxy,Cache,Extension",
        "endpoint": "https://{myCustomEndpoint}/"
      }
    ]
  },
  "location": "{location}"
}
```

После создания настраиваемого поставщика можно ориентироваться на другие ресурсы и применить к ним побочные эффекты пользовательского поставщика.

Образец запроса на создание для Microsoft. Кустомпровидерс/Associations:

``` HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{virtualMachineName}/providers/Microsoft.CustomProviders/associations/{associationName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json

{
  "properties": {
    "targetResourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}",
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3" : "myPropertyValue3"
    }
  }
}
```

Этот запрос будет перенаправлен в конечную точку, указанную в созданном вами пользовательском поставщике, на который ссылается **targetResourceId** в этой форме:

``` HTTP
PUT https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/associations/{associationName}
X-MS-CustomProviders-ExtensionPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{virtualMachineName}/providers/Microsoft.CustomProviders/associations/{associationName}
X-MS-CustomProviders-ExtendedResource: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{virtualMachineName}

{
  "properties": {
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3" : "myPropertyValue3"
    }
  }
}
```

Конечная точка должна отвечать с помощью приложения/JSON `Content-Type` и допустимого текста ответа JSON. Поля, возвращаемые в объекте **свойств** JSON, будут добавлены в ответ на возвращаемый запрос ассоциации.

## <a name="getting-help"></a>Получение справки

Если у вас возникли вопросы по разработке поставщиков настраиваемых ресурсов Azure, попробуйте запрашивать их на [Stack overflow](https://stackoverflow.com/questions/tagged/azure-custom-providers). Подобный вопрос, возможно, уже был задан, поэтому перед его публикацией сначала проверьте наличие ответа. Чтобы быстрее получить ответ, добавьте к вопросу тег ```azure-custom-providers```!

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали о настраиваемых поставщиках. Дополнительные сведения см. в статьях ниже:

- [Учебник. Адаптация ресурсов с помощью настраиваемых поставщиков](./tutorial-resource-onboarding.md)
- [Руководство. Создание настраиваемых действий и ресурсов в Azure](./tutorial-get-started-with-custom-providers.md)
- [Краткое руководство. Создание настраиваемого поставщика ресурсов и развертывание настраиваемых ресурсов](./create-custom-provider.md)
- [Как добавлять настраиваемые действия в REST API Azure](./custom-providers-action-endpoint-how-to.md)
- [Как добавлять пользовательские ресурсы в REST API Azure](./custom-providers-resources-endpoint-how-to.md)
