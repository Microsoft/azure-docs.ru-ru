---
title: Общие сведения о настраиваемых поставщиках
description: Узнайте о поставщиках настраиваемых ресурсов Azure и о том, как расширить плоскость API Azure в соответствии с рабочими процессами.
author: jjbfour
ms.topic: conceptual
ms.date: 06/19/2019
ms.author: jobreen
ms.openlocfilehash: 68b8bd187d58cd71778b8a922684cc3817a0715d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "80398473"
---
# <a name="azure-custom-resource-providers-overview"></a>Общие сведения о поставщиках настраиваемых ресурсов Azure

Поставщики настраиваемых ресурсов Azure — это платформа расширяемости для Azure. Он позволяет определять пользовательские интерфейсы API, которые можно использовать для расширения возможностей Azure по умолчанию. В этой документации описывается следующее:

- Как создать и развернуть поставщик настраиваемых ресурсов Azure.
- Использование поставщиков настраиваемых ресурсов Azure для расширения существующих рабочих процессов.
- Где найти руководства и примеры кода, чтобы приступить к работе.

![Обзор пользовательского поставщика](./media/overview/overview.png)

> [!IMPORTANT]
> В настоящее время настраиваемые поставщики находятся в общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="what-can-custom-resource-providers-do"></a>Что могут делать пользовательские поставщики ресурсов

Ниже приведены некоторые примеры того, что можно достичь с помощью настраиваемых поставщиков ресурсов Azure.

- Расширение Azure Resource Manager REST API для включения внутренних и внешних служб.
- Включите пользовательские сценарии на основе существующих рабочих процессов Azure.
- Настройка управления и влияния шаблонов Azure Resource Manager.

## <a name="what-is-a-custom-resource-provider"></a>Что такое пользовательский поставщик ресурсов

Поставщики настраиваемых ресурсов Azure создаются путем создания контракта между Azure и конечной точкой. Этот контракт определяет список новых ресурсов и действий с помощью нового ресурса, **Microsoft. кустомпровидерс/ресаурцепровидерс**. Затем настраиваемый поставщик ресурсов предоставит эти новые API в Azure. Поставщики настраиваемых ресурсов Azure состоят из трех частей: поставщика настраиваемых ресурсов, **конечных точек** и настраиваемых ресурсов.

## <a name="how-to-build-custom-resource-providers"></a>Создание пользовательских поставщиков ресурсов

Настраиваемые поставщики ресурсов представляют собой список контрактов между Azure и конечными точками. Этот контракт описывает, как Azure должен взаимодействовать с конечной точкой. Поставщик ресурсов выступает в качестве прокси-сервера и пересылает запросы и ответы из указанной **конечной точки** и отменяет их. Поставщик ресурсов может указывать два типа контрактов: [**ресаурцетипес**](./custom-providers-resources-endpoint-how-to.md) и [**Actions**](./custom-providers-action-endpoint-how-to.md). Они включаются с помощью определений конечных точек. Определение конечной точки состоит из трех полей: **Name**, **раутингтипе** и **Endpoint**.

Пример конечной точки:

```JSON
{
  "name": "{endpointDefinitionName}",
  "routingType": "Proxy",
  "endpoint": "https://{endpointURL}/"
}
```

Свойство | Обязательно | Описание
---|---|---
name | *Да* | Имя определения конечной точки. Azure предоставит это имя через свой API-интерфейс в папке "/subscriptions/{ИД_подписки}/resourceGroups/{Имя_группы_ресурсов}/providers/Microsoft.CustomProviders/<br>resourceProviders/{Имя_поставщика ресурсов}/{Имя_определения_конечной_точки}"
routingType | *Нет* | Определяет тип контракта с **конечной точкой**. Если не указан, по умолчанию будет «Прокси».
endpoint | *Да* | Конечная точка для направления запросов. Ответ будет обработан так же, как и любые побочные эффекты запроса.

### <a name="building-custom-resources"></a>Создание пользовательских ресурсов

**Ресаурцетипес** описывают новые пользовательские ресурсы, которые добавляются в Azure. Они предоставляют базовые методы CRUD для RESTFUL. См. [Дополнительные сведения о создании настраиваемых ресурсов](./custom-providers-resources-endpoint-how-to.md) .

Пример пользовательского поставщика ресурсов с **ресаурцетипес**:

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "myCustomResources",
        "routingType": "Proxy",
        "endpoint": "https://{endpointURL}/"
      }
    ]
  },
  "location": "eastus"
}
```

API, добавленные в Azure для приведенного выше примера:

HttpMethod | Образец URI | Описание
---|---|---
PUT | /subscriptions/{subscriptionId}/resourceGroups/{имя_группы_ресурсов}/<br>providers/Microsoft.CustomProviders/resourceProviders/{имя поставщика ресурсов}/<br>Микустомресаурцес/{Кустомресаурценаме}? API-Version = 2018-09 01-Preview | Вызов REST API Azure для создания нового ресурса.
DELETE | /subscriptions/{subscriptionId}/resourceGroups/{имя_группы_ресурсов}/<br>providers/Microsoft.CustomProviders/resourceProviders/{имя поставщика ресурсов}/<br>Микустомресаурцес/{Кустомресаурценаме}? API-Version = 2018-09 01-Preview | Вызов REST API Azure для удаления существующего ресурса.
GET | /subscriptions/{subscriptionId}/resourceGroups/{имя_группы_ресурсов}/<br>providers/Microsoft.CustomProviders/resourceProviders/{имя поставщика ресурсов}/<br>Микустомресаурцес/{Кустомресаурценаме}? API-Version = 2018-09 01-Preview | Вызов REST API Azure для получения существующего ресурса.
GET | /subscriptions/{subscriptionId}/resourceGroups/{имя_группы_ресурсов}/<br>providers/Microsoft.CustomProviders/resourceProviders/{имя поставщика ресурсов}/<br>Микустомресаурцес? API-Version = 2018-9 01-Preview | Вызов REST API Azure для получения списка существующих ресурсов.

### <a name="building-custom-actions"></a>Создание настраиваемых действий

**Действия** описывают новые действия, которые добавляются в Azure. Они могут быть представлены поверх поставщика ресурсов или вложены в **ResourceType**. См. [Дополнительные сведения о создании настраиваемых действий](./custom-providers-action-endpoint-how-to.md) .

Пример пользовательского поставщика ресурсов с **действиями**:

```JSON
{
  "properties": {
    "actions": [
      {
        "name": "myCustomAction",
        "routingType": "Proxy",
        "endpoint": "https://{endpointURL}/"
      }
    ]
  },
  "location": "eastus"
}
```

API, добавленные в Azure для приведенного выше примера:

HttpMethod | Образец URI | Описание
---|---|---
POST | /subscriptions/{subscriptionId}/resourceGroups/{имя_группы_ресурсов}/<br>providers/Microsoft.CustomProviders/resourceProviders/{имя поставщика ресурсов}/<br>Микустомактион? API-Version = 2018-9 01-Preview | Вызов REST API Azure для активации действия.

## <a name="looking-for-help"></a>Требуется помощь?

Если у вас возникли вопросы по разработке поставщика настраиваемых ресурсов Azure, попробуйте запрашивать [Stack overflow](https://stackoverflow.com/questions/tagged/azure-custom-providers). Подобный вопрос, возможно, уже был задан, поэтому перед его публикацией сначала проверьте наличие ответа. Чтобы быстрее получить ответ, добавьте к вопросу тег ```azure-custom-providers```!

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали о настраиваемых поставщиках. Перейдите к следующей статье, чтобы создать настраиваемый поставщик.

- [Краткое руководство. Создание настраиваемого поставщика ресурсов Azure и развертывание настраиваемых ресурсов](./create-custom-provider.md)
- [Руководство. Создание настраиваемых действий и ресурсов в Azure](./tutorial-get-started-with-custom-providers.md)
- [Как добавлять настраиваемые действия в Azure REST API](./custom-providers-action-endpoint-how-to.md)
- [Как добавлять пользовательские ресурсы в Azure REST API](./custom-providers-resources-endpoint-how-to.md)
