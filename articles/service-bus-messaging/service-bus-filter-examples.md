---
title: Настройка фильтров подписок в служебной шине Azure | Документация Майкрософт
description: В этой статье приведены примеры определения фильтров и действий в подписках на разделы служебной шины Azure.
ms.topic: how-to
ms.date: 02/17/2021
ms.openlocfilehash: bcbb72901ed8e2dfe0932163ee18683e0011ce70
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100654578"
---
# <a name="set-subscription-filters-azure-service-bus"></a>Установка фильтров подписок (служебная шина Azure)
В этой статье представлено несколько примеров настройки фильтров для подписок на разделы служебной шины. Основные сведения о фильтрах см. в разделе [фильтры](topic-filters.md).

## <a name="filter-on-system-properties"></a>Фильтрация по свойствам системы
Для ссылки на системное свойство в фильтре используйте следующий формат: `sys.<system-property-name>` . 

```csharp
sys.label LIKE '%bus%'`
sys.messageid = 'xxxx'
sys.correlationid like 'abc-%'
```

## <a name="filter-on-message-properties"></a>Фильтрация по свойствам сообщения
Ниже приведены примеры использования свойств сообщений в фильтре. Доступ к свойствам сообщений можно получить с помощью `user.property-name` или просто `property-name` .

```csharp
MessageProperty = 'A'
SuperHero like 'SuperMan%'
```

## <a name="filter-on-message-properties-with-special-characters"></a>Фильтрация по свойствам сообщения с помощью специальных символов
Если имя свойства сообщения содержит специальные символы, используйте двойные кавычки ( `"` ), чтобы заключить имя свойства. Например, если имя свойства — `"http://schemas.microsoft.com/xrm/2011/Claims/EntityLogicalName"` , используйте следующий синтаксис в фильтре. 

```csharp
"http://schemas.microsoft.com/xrm/2011/Claims/EntityLogicalName" = 'account'
```

## <a name="filter-on-message-properties-with-numeric-values"></a>Фильтрация по свойствам сообщения с помощью числовых значений
В следующих примерах показано, как можно использовать свойства с числовыми значениями в фильтрах. 

```csharp
MessageProperty = 1
MessageProperty > 1
MessageProperty > 2.08
MessageProperty = 1 AND MessageProperty2 = 3
MessageProperty = 1 OR MessageProperty2 = 3
```

## <a name="parameter-based-filters"></a>Фильтры на основе параметров
Ниже приведено несколько примеров использования фильтров на основе параметров. В этих примерах `DataTimeMp` свойство является свойством Message типа `DateTime` и `@dtParam` является параметром, передаваемым в фильтр в качестве `DateTime` объекта.

```csharp
DateTimeMp < @dtParam
DateTimeMp > @dtParam

(DateTimeMp2-DateTimeMp1) <= @timespan //@timespan is a parameter of type TimeSpan
DateTimeMp2-DateTimeMp1 <= @timespan
```

## <a name="using-in-and-not-in"></a>Использование IN и NOT IN

```csharp
StoreId IN('Store1', 'Store2', 'Store3')"

sys.To IN ('Store5','Store6','Store7') OR StoreId = 'Store8'

sys.To NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8') OR StoreId NOT IN ('Store1','Store2','Store3','Store4','Store5','Store6','Store7','Store8')
```

Пример для C# см. [в разделе Пример фильтров разделов на сайте GitHub](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Azure.Messaging.ServiceBus/BasicSendReceiveTutorialwithFilters).


## <a name="correlation-filter-using-correlationid"></a>Фильтр корреляции с использованием CorrelationID

```csharp
new CorrelationFilter("Contoso");
```

Он фильтрует сообщения с параметром, имеющим `CorrelationID` значение `Contoso` . 

## <a name="correlation-filter-using-system-and-user-properties"></a>Фильтр корреляции с использованием свойств системы и пользователя

```csharp
var filter = new CorrelationFilter();
filter.Label = "Important";
filter.ReplyTo = "johndoe@contoso.com";
filter.Properties["color"] = "Red";
```

Это эквивалентно следующему: `sys.ReplyTo = 'johndoe@contoso.com' AND sys.Label = 'Important' AND color = 'Red'`

## <a name="next-steps"></a>Дальнейшие действия
См. следующие примеры: 

- [.NET — руководство по отправке и получению с фильтрами](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/BasicSendReceiveTutorialwithFilters/BasicSendReceiveTutorialWithFilters)
- [Фильтры разделов .NET](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/TopicFilters)
- [Шаблон Azure Resource Manager](/azure/templates/microsoft.servicebus/2017-04-01/namespaces/topics/subscriptions/rules)