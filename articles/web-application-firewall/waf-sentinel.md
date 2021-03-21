---
title: Использование Sentinel Azure с брандмауэром веб-приложения Azure
description: В этой статье показано, как использовать Sentinel Azure с брандмауэром веб-приложения Azure (WAF).
services: web-application-firewall
author: TreMansdoerfer
ms.service: web-application-firewall
ms.date: 10/12/2020
ms.author: victorh
ms.topic: how-to
ms.openlocfilehash: 3d905dd1e6acab8f9f6d3885c882dd9c32133cb4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100596425"
---
# <a name="using-azure-sentinel-with-azure-web-application-firewall"></a>Использование Sentinel Azure с брандмауэром веб-приложения Azure

Брандмауэр веб-приложения Azure (WAF) в сочетании с Sentinel Azure может предоставлять сведения о безопасности для ресурсов WAF. Azure Sentinel обеспечивает анализ безопасности с помощью Log Analytics, что позволяет легко разбивать и просматривать данные WAF. С помощью Sentinel Azure можно получить доступ к предварительно созданным книгам и изменить их в соответствии с потребностями вашей организации. Книга может показывать аналитику для WAF в сети доставки содержимого (CDN) Azure, WAF на передней дверце Azure и WAF в шлюзе приложений для нескольких подписок и рабочих областей.

## <a name="waf-log-analytics-categories"></a>Категории log Analytics WAF

WAF log Analytics разделяется на следующие категории:  

- Все выполненные действия WAF 
- Первые 40 адресов URI заблокированных запросов 
- Первые 50 триггеров событий,  
- Сообщения с течением времени 
- Полные сведения о сообщении 
- События атаки по сообщениям  
- События атак с течением времени 
- Фильтр отслеживания ИДЕНТИФИКАТОРов 
- Сообщения отслеживания ИДЕНТИФИКАТОРов 
- 10 основных IP-адресов для атак 
- Сообщения об атаках IP-адресов 

## <a name="waf-workbook-examples"></a>Примеры книг WAF

В следующих примерах книги WAF показаны образцы данных:

:::image type="content" source="media//waf-sentinel/waf-actions-filter.png" alt-text="Фильтр действий WAF":::

:::image type="content" source="media//waf-sentinel/top-50-event-trigger.png" alt-text="Топ 50 событий":::

:::image type="content" source="media//waf-sentinel/attack-events.png" alt-text="События атак":::

:::image type="content" source="media//waf-sentinel/top-10-attacking-ip-address.png" alt-text="10 основных IP-адресов для атак":::

## <a name="launch-a-waf-workbook"></a>Запуск книги WAF

Книга WAF работает для всех интерфейсов передней дверцы Azure, шлюза приложений и CDN WAF. Перед подключением данных из этих ресурсов необходимо включить в ресурсе log Analytics. 

Чтобы включить log Analytics для каждого ресурса, перейдите к отдельной передней дверце Azure, шлюзу приложений или ресурсу CDN:

1. Выберите **Параметры диагностики**.
2. Щелкните команду **Добавить параметр диагностики**. 
3. На странице параметры диагностики:
   1. Введите имя. 
   1. Установите флажок **Отправить в Log Analytics**. 
   1. Выберите рабочую область назначения журнала. 
   1. Выберите типы журналов, которые необходимо проанализировать:
      1. Шлюз приложений: "ApplicationGatewayAccessLog" и "ApplicationGatewayFirewallLog"
      1. Передняя дверца Azure: "Фронтдуракцесслог" и "Фронтдурфиревалллог"
      1. CDN: "Азурекднакцесслог"
   1. Щелкните **Сохранить**.

   :::image type="content" source="media//waf-sentinel/diagnostics-setting.png" alt-text="Параметр диагностики":::

4. На домашней странице Azure в строке поиска введите **Azure Sentinel** и выберите ресурс **Sentinel Azure** . 
2. Выберите уже активную рабочую область или создайте новую рабочую область. 
3. На левой боковой панели в разделе **Конфигурация** выберите **соединители данных**.
4. Найдите **брандмауэр веб-приложения Майкрософт** и выберите **брандмауэр веб-приложения Майкрософт (WAF)**. Выберите **открыть страницу соединителя** в правом нижнем углу.

   :::image type="content" source="media//waf-sentinel/data-connectors.png" alt-text="Соединители данных":::

8. Следуйте инструкциям в разделе **Конфигурация** для каждого ресурса WAF, для которого требуется получить данные аналитики журнала, если это не было сделано ранее.
6. После завершения настройки отдельных ресурсов WAF перейдите на вкладку **дальнейшие действия** . Выберите одну из рекомендуемых книг. В этой книге будут использоваться все данные журнала аналитики, которые были включены ранее. Теперь для ресурсов WAF должна существовать Рабочая книга WAF.

   :::image type="content" source="media//waf-sentinel/waf-workbooks.png" alt-text="Книги WAF":::


## <a name="next-steps"></a>Дальнейшие действия

- [Дополнительные сведения об Azure Sentinel](../sentinel/overview.md)
- [Дополнительные сведения о Azure Monitor книгах](../azure-monitor/visualize/workbooks-overview.md)
