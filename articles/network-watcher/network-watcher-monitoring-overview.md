---
title: Наблюдатель за сетями Azure | Документация Майкрософт
description: Сведения о функциях Наблюдателя за сетями Azure (мониторинге, диагностике, метриках и ведении журнала) для ресурсов в виртуальной сети.
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: 14bc2266-99e3-42a2-8d19-bd7257fec35e
ms.service: network-watcher
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/04/2021
ms.author: damendo
ms.custom: mvc
ms.openlocfilehash: 5e032eaa5138f63122ed019441d29ad7eafca2a2
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106055898"
---
# <a name="what-is-azure-network-watcher"></a>Наблюдатель за сетями Azure

Наблюдатель за сетями Azure предоставляет инструменты для мониторинга, диагностики, просмотра метрик и включения или отключения журналов для ресурсов в виртуальной сети Azure. Служба "Наблюдатель за сетями" предназначена для мониторинга и восстановления работоспособности сети для продуктов IaaS (инфраструктура как услуга), в том числе виртуальных машин, виртуальных сетей, шлюзов приложений, подсистем балансировки нагрузки и т. д. Примечание. Она не предназначена для мониторинга продуктов PaaS или веб-аналитики и не будет работать с ними. 

## <a name="monitoring"></a>Мониторинг

### <a name="monitor-communication-between-a-virtual-machine-and-an-endpoint"></a><a name = "connection-monitor"></a>Мониторинг взаимодействия между виртуальной машиной и конечной точкой

Конечной точкой может быть другая виртуальная машина, полное доменное имя (FQDN), универсальный код ресурса (URI) или адрес IPv4. Функция *Монитор подключения* отслеживает взаимодействие с регулярной периодичностью и предоставляет информацию об изменениях доступности, задержки и топологии сети между виртуальной машиной и конечной точкой. Например, у вас может быть виртуальная машина веб-сервера, которая взаимодействует с виртуальной машиной сервера базы данных. Кто-то из вашей организации может без вашего ведома применить к виртуальной машине веб-сервера, сервера базы данных или подсети пользовательский маршрут или правило безопасности сети.

Если конечная точка становится недоступной, функция устранения неполадок с подключением проинформирует вас о причине. Потенциальными причинами являются проблема разрешения имен DNS, ЦП, память или брандмауэр в операционной системе виртуальной машины, тип прыжка пользовательского маршрута или правило безопасности для виртуальной машины или же подсети исходящего соединения. См. дополнительные сведения о [правилах безопасности](../virtual-network/network-security-groups-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#security-rules) и [типах прыжков](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

Монитор подключения также предоставляет сведения о минимальной, средней и максимальной задержках за период времени. Изучив задержку подключения, вы обнаружите, что можете уменьшить задержку, перемещая ресурсы Azure в разные регионы Azure. См. дополнительные сведения [об определении относительных задержек между регионами Azure и поставщиками услуг Интернета](#determine-relative-latencies-between-azure-regions-and-internet-service-providers) и о [мониторинге взаимодействия между виртуальной машиной и конечной точкой с помощью монитора подключения](connection-monitor.md). Если вы предпочитаете проверять подключение в определенный момент времени, вместо отслеживания с течением времени, как, например, с помощью монитора подключения, используйте функцию [Устранение неполадок с подключением](#connection-troubleshoot).

Монитор производительности сети — это облачное гибридное решение для сетевого мониторинга, которое позволяет отслеживать производительность сети между различными точками в сетевой инфраструктуре. Это решение также помогает отслеживать сетевое подключение к конечным точкам служб и приложений, а также производительность Azure ExpressRoute. Монитор производительности сети обнаруживает проблемы сети, такие как появление "черных дыр" в трафике, ошибки маршрутизации и проблемы, которые невозможно обнаружить с помощью традиционных методов мониторинга сети. Решение создает предупреждения и уведомляет о превышении порогового значения для сетевого соединения. Кроте того, оно обеспечивает своевременное обнаружение проблем с производительностью сети и определяет конкретный сегмент сети или сетевое устройство как источник проблемы. См. дополнительные сведения о [Мониторе производительности сети](../azure-monitor/insights/network-performance-monitor.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

### <a name="view-resources-in-a-virtual-network-and-their-relationships"></a>Просмотр ресурсов в виртуальной сети и их связей

По мере добавления ресурсов в виртуальную сеть может быть трудно определить, какие ресурсы находятся в виртуальной сети и как они связаны друг с другом. Функция *топологии* позволяет создавать визуальную диаграмму ресурсов в виртуальной сети и связей между ними. На следующем рисунке представлен пример диаграммы топологии для виртуальной сети с тремя подсетями, двумя виртуальными машинами, сетевыми интерфейсами, общедоступными IP-адресами, группами безопасности сети, таблицами маршрутов и связями между ресурсами.

![Представление топологии](./media/network-watcher-monitoring-overview/topology.png)

Вы можете загрузить изменяемую версию изображения в формате SVG. См. дополнительные сведения о [представлении топологии](view-network-topology.md).

## <a name="diagnostics"></a>Диагностика

### <a name="diagnose-network-traffic-filtering-problems-to-or-from-a-vm"></a>Диагностика проблем с фильтрацией входящего или исходящего трафика виртуальной машины

При развертывании виртуальной машины Azure применяет к ней несколько правил безопасности по умолчанию, которые разрешают или запрещают входящий или исходящий трафик виртуальной машины. Вы можете переопределить правила по умолчанию Azure или создать дополнительные правила. В какой-то момент виртуальная машина может перестать взаимодействовать с другими ресурсами из-за правила безопасности. Функция *проверки IP-потока* позволяет указать IPv4-адрес источника и получателя, порт, протокол (TCP или UDP) и направление трафика (входящий или исходящий). Затем эта функция проверяет взаимодействие и информирует вас о том, установлено ли подключение. Если происходит сбой подключения, проверка IP-потока сообщает вам, какое из правил безопасности разрешило или запретило взаимодействие. Таким образом вы сможете решить проблему. Получите дополнительные сведения о проверке IP-потока, выполнив задачи в руководстве по [диагностике проблем фильтрации для трафика виртуальных машин](diagnose-vm-network-traffic-filtering-problem.md).

### <a name="diagnose-network-routing-problems-from-a-vm"></a>Диагностика проблем сетевой маршрутизации на виртуальной машине

Когда вы создаете виртуальную сеть, Azure создает несколько исходящих маршрутов по умолчанию для трафика. Исходящий трафик из всех ресурсов, таких как виртуальные машины, развернутые в виртуальной сети, направляется на основе маршрутов Azure по умолчанию. Вы можете переопределить маршруты Azure по умолчанию или создать дополнительные маршруты. Вы можете обнаружить, что виртуальная машина больше не может взаимодействовать с другими ресурсами из-за определенного маршрута. Функция *Следующий прыжок* позволяет указать IPv4-адрес источника и получателя. Затем следующий прыжок проверяет взаимодействие и сообщает вам, какой тип следующего прыжка используется для маршрутизации трафика. Затем вы можете удалить, изменить или добавить маршрут, чтобы решить проблему маршрутизации. См. дополнительные сведения о [функции следующего прыжка](diagnose-vm-network-routing-problem.md).

### <a name="diagnose-outbound-connections-from-a-vm"></a><a name="connection-troubleshoot"></a>Диагностика исходящих подключений виртуальной машины

Функция *устранения неполадок с подключением* позволяет проверить подключение между виртуальной машиной и другой виртуальной машиной, полным доменным именем, URI или IPv4-адресом. Тест возвращает информацию аналогичную возвращаемой при использовании функции [Монитор подключения](#connection-monitor), но проверяет подключение в определенный момент времени, а не контролирует его с течением времени, как это делает монитор подключения. См. дополнительные сведения об использовании [функции устранения проблемы с подключениями](network-watcher-connectivity-overview.md).

### <a name="capture-packets-to-and-from-a-vm"></a>Запись пакетов, направляющихся в виртуальную машину и из нее

Дополнительные параметры фильтрации и элементы управления, такие как возможность установить ограничения времени и размера, обеспечивают гибкость в работе. Запись можно сохранить в службе хранилища Azure, на диске виртуальной машины или же в обоих расположениях. Затем вы можете проанализировать файл записи с помощью нескольких стандартных инструментов анализа записи сетевого трафика. См. дополнительные сведения о [записи пакетов](network-watcher-packet-capture-overview.md).

### <a name="diagnose-problems-with-an-azure-virtual-network-gateway-and-connections"></a>Диагностика проблем со шлюзом и подключениями виртуальной сети Azure

Шлюзы виртуальной сети обеспечивают связь между локальными ресурсами и виртуальными сетями Azure. Мониторинг шлюзов и их подключений крайне важен для обеспечения бесперебойной связи. Функция *Диагностика VPN* позволяет выполнять диагностику шлюзов и подключений. Диагностика VPN проверяет работоспособность шлюза или подключения шлюза и информирует вас о том, доступны ли шлюзы и их подключения. Если шлюз или подключения недоступны, диагностика VPN сообщает вам причину, чтобы вы могли решить проблему. Получите дополнительные сведения о диагностике VPN, выполнив задачи в [этом руководстве](diagnose-communication-problem-between-networks.md).

### <a name="determine-relative-latencies-between-azure-regions-and-internet-service-providers"></a>Определение относительных задержек между регионами Azure и поставщиками услуг Интернета

Вы можете отправить в Наблюдатель за сетями запрос на получение информации о задержках между регионами Azure и поставщиками услуг Интернета. Если вам известны задержки между регионами Azure и поставщиками услуг Интернета, вы можете развернуть ресурсы Azure таким образом, чтобы оптимизировать время отклика сети. См. дополнительные сведения об [относительных задержках](view-relative-latencies.md).

### <a name="view-security-rules-for-a-network-interface"></a>Просмотр правил безопасности для сетевого интерфейса

Действующие правила безопасности для сетевого интерфейса представляют собой комбинацию всех правил безопасности, применяемых к сетевому интерфейсу и подсети, в которой он находится.  В *представлении группы безопасности* отображаются все правила безопасности, применяемые к сетевому интерфейсу и подсети, в которой он находится. Зная, какие правила применяются к сетевому интерфейсу, вы можете добавлять, удалять или изменять правила, если они разрешают или запрещают трафик, который вы хотите изменить. См. дополнительные сведения о [представлении группы безопасности](network-watcher-security-group-view-overview.md).

## <a name="metrics"></a>Метрики

На количество сетевых ресурсов, которые вы можете создать в подписке и регионе Azure, существуют [ограничения](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#azure-resource-manager-virtual-networking-limits). Достигнув предела, вы не сможете создавать больше ресурсов в рамках подписки или региона. Функция *Ограничение на сетевую подписку* предоставляет сводку количества сетевых ресурсов, развернутых в подписке и регионе, а также сведения об ограничении для ресурса. На следующем рисунке показаны частичные выходные данные сетевых ресурсов, развернутых в регионе "Восточная часть США", для примера подписки.

![Ограничения подписки](./media/network-watcher-monitoring-overview/subscription-limit.png)

Эта информация полезна при планировании будущих развертываний ресурсов.

## <a name="logs"></a>Журналы

### <a name="analyze-traffic-to-or-from-a-network-security-group"></a>Анализ трафика, направляющегося в группу безопасности сети или из нее

Группы безопасности сети (NSG) разрешают или запрещают входящий или исходящий трафик для сетевого интерфейса в виртуальной машине. Функция *Журнал потоков NSG* позволяет вам регистрировать исходный и целевой IP-адрес, порт, протокол и сведения о том, был ли трафик разрешен или запрещен группой безопасности сети. Вы можете анализировать журналы с помощью различных инструментов, таких как Power BI и функция *анализа трафика*. Аналитика трафика обеспечивает расширенную визуализацию данных, записанных в журналы потоков NSG. На следующем рисунке содержатся сведения и визуализации, которые функция анализа трафика представляет из данных журнала потоков NSG.

![Аналитика трафика](./media/network-watcher-monitoring-overview/traffic-analytics.png)

Получите дополнительные сведения о журналах потока NSG, выполнив задачи в руководстве по [ведению журнала входящего и исходящего трафика виртуальной машины](network-watcher-nsg-flow-logging-portal.md) и узнайте, как реализовать [анализ трафика](traffic-analytics.md).

### <a name="view-diagnostic-logs-for-network-resources"></a>Просмотр журналов диагностики для сетевых ресурсов

Вы можете включить ведение журнала диагностики для сетевых ресурсов Azure, таких как группы безопасности сети, общедоступные IP-адреса, подсистемы балансировки нагрузки, шлюзы виртуальной сети и шлюзы приложений. Функция *Журналы диагностики* предоставляет единый интерфейс для включения и отключения журналов диагностики сетевых ресурсов для любого имеющегося сетевого ресурса, который создает журнал диагностики. Вы можете просматривать журналы диагностики с помощью таких инструментов, как Microsoft Power BI и журналы Azure Monitor. Дополнительные сведения об анализе журналов диагностики сети Azure см. в статье [Решения для мониторинга сетей Azure в Log Analytics](../azure-monitor/insights/azure-networking-analytics.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json).

## <a name="network-watcher-automatic-enablement"></a>Автоматическое включение Наблюдателя за сетями
При создании или обновлении виртуальной сети в подписке Наблюдатель за сетями включается автоматически в регионе вашей виртуальной сети. Автоматическое включение Наблюдателя за сетями не влияет на ваши ресурсы, и за него не взимается дополнительная плата. Дополнительные сведения см. в статье [Создание экземпляра Наблюдателя за сетями Azure](network-watcher-create.md).

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы получили представление о Наблюдателе за сетями. Чтобы приступить к использованию Наблюдателя за сетями, выполните диагностику распространенных проблем исходящего и входящего обмена данными для виртуальной машины с помощью проверки IP-потока. Инструкции см. в кратком руководстве по [диагностике проблемы с фильтрацией трафика виртуальной машины](diagnose-vm-network-traffic-filtering-problem.md).