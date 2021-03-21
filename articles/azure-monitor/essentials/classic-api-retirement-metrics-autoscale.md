---
title: Снятие API развертывания для метрик Azure Monitor и автомасштабирования
description: Прекращается поддержка классических API метрик и автомасштабирования, именуемых также моделью развертывания RDFE или Azure Service Management (ASM)
ms.topic: conceptual
ms.date: 11/19/2018
ms.openlocfilehash: 0169de93b92de179c0ae9a36ff8dba3b7b1dc0fb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102049038"
---
# <a name="azure-monitor-retirement-of-classic-deployment-model-apis-for-metrics-and-autoscale"></a>Прекращается поддержка API классической модели развертывания для метрик и автомасштабирования в Azure Monitor

Платформа Azure Monitor (при первом выпуске звалась Azure Insights) в настоящее время позволяет создавать параметры автомасштабирования и управлять ими для классических виртуальных машин и классических облачных служб, а также использовать метрики таких машин и служб. Поддержка первоначального набора, включающего API классической модели развертывания, будет **прекращена после 30 июня 2019 г.** во всех общедоступных и частных облаках Azure во всех регионах.   

Те же операции поддерживались благодаря набору API на основе Azure Resource Manager в течение более года. На портале Azure используются новые REST API для автомасштабирования и метрик. Доступны также новый пакет SDK, PowerShell и интерфейс командной строки, основанные на этих API Resource Manager. Наши партнерские службы для мониторинга используют новые REST API на основе Resource Manager в Azure Monitor.  

## <a name="who-is-not-affected"></a>На кого изменения не повлияют

Если вы управляете автомасштабированием с помощью портала Azure, [нового пакета SDK для Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/), PowerShell, интерфейса командной строки или шаблонов Resource Manager, никакие действия не требуются.  

При использовании метрик через портал Azure или с помощью различных [партнерских служб мониторинга](../partners.md) никакие действия не требуются. Корпорация Майкрософт вместе с партнерами по мониторингу работают над миграцией на новые API.

## <a name="who-is-affected"></a>На кого изменения повлияют

Эта статья написана для вас, если вы используете следующие компоненты:

- **Классический пакет SDK для Azure Insights.** Если вы используете [классический пакет SDK для Azure Insights](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/), переходите на использование нового пакета SDK Azure Monitor для [.NET](https://github.com/azure/azure-libraries-for-net#download) или [Java](https://github.com/azure/azure-libraries-for-java#download). Скачайте [пакет NuGet SDK для Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/).

- **Классическое автомасштабирование.** Если вы вызываете [классические API параметров автомасштабирования](/previous-versions/azure/reference/mt348562(v=azure.100)) из пользовательских средств или с помощью [классического пакета SDK для Azure Insights](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/), следует перейти к использованию [REST API для Azure Monitor Resource Manager](/rest/api/monitor/autoscalesettings).

- **Классические метрики.** Если вы используете метрики с помощью [классических REST API](/previous-versions/azure/reference/dn510374(v=azure.100)) или [классического пакета SDK для Azure Insights](https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/) из пользовательских средств, следует перейти к использованию [REST API для Azure Monitor Resource Manager](/rest/api/monitor/autoscalesettings). 

Если вы не уверены, происходит ли вызов классических API со стороны вашего кода или пользовательских средств, проверьте следующее:

- Просмотрите универсальный код ресурса (URI), на который ссылается код или средство. Классические API используют URI https://management.core.windows.net. Следует использовать более новые URI для интерфейсов API на основе Resource Manager, начинающиеся с `https://management.azure.com/`.

- Сравните имя сборки на вашем компьютере. Более старая классическая сборка находится здесь: https://www.nuget.org/packages/Microsoft.WindowsAzure.Management.Monitoring/.

- Если вы используете проверку подлинности на основе сертификата для доступа к API автомасштабирования или метрик, вы используете классическую конечную точку и библиотеку. Для более новых API Resource Manager требуется проверка подлинности Azure Active Directory с помощью субъекта-службы или субъекта-пользователя.

- Если вы используете вызовы, упомянутые в документации по любой из указанных ниже ссылок, то вы используете более старые классические API.

  - [Библиотека классов Windows.Azure.Management.Monitoring](/previous-versions/azure/dn510414(v=azure.100))

  - [.NET для мониторинга (классический вариант)](/previous-versions/azure/reference/mt348562(v%3dazure.100))

  - [Интерфейс IMetricOperations](/previous-versions/azure/reference/dn802395(v%3dazure.100))

## <a name="why-you-should-switch"></a>Почему нужно перейти

Все существующие функциональные возможности автомасштабирования и метрик останутся благодаря новым интерфейсам API.  

Миграция на более новые API-интерфейсы поступает с возможностями на основе диспетчер ресурсов, например с поддержкой согласованного управления доступом на основе ролей Azure (Azure RBAC) во всех службах мониторинга. Вы также получите такие дополнительные функции для метрик: 

- поддержка измерений;
- унифицированная 1-минутная степень детализации метрик для служб; 
- усовершенствованные запросы;
- более высокий срок хранения данных (93 дней метрик по сравнению с 30 днями) 

Как и другие службы в Azure, API для Azure Monitor на основе Resource Manager обеспечивают лучшую производительность, масштабируемость и надежность. 

## <a name="what-happens-if-you-do-not-migrate"></a>Что произойдет, если не выполнить миграцию

### <a name="before-retirement"></a>До прекращения поддержки

Не будет никакого прямого влияния на ваши службы Azure или их рабочие нагрузки.  

### <a name="after-retirement"></a>После прекращения поддержки

Вызовы классических API, перечисленных ранее, завершатся ошибкой. Будут возвращаться сообщения об ошибках примерно следующего вида:

Для автомасштабирования: *Этот API является устаревшим. Используйте шаблоны портал Azure, Azure Monitor SDK, PowerShell, интерфейса командной строки или диспетчер ресурсов для управления параметрами автомасштабирования*.  

Для метрик: *Этот API является устаревшим. Используйте портал Azure, Azure Monitor SDK, PowerShell, CLI для запроса метрик*.

## <a name="email-notifications"></a>уведомления по электронной почте.

Было отправлено уведомление о прекращении поддержки на электронные адреса для следующих ролей учетной записи: 

- администраторы служб и учетной записи;
- соадминистраторы.  

Если у вас возникли какие-либо вопросы, пишите нам на адрес MonitorClassicAPIhelp@microsoft.com.  

## <a name="references"></a>Ссылки

- [Более новые REST API для Azure Monitor](/rest/api/monitor/) 
- [Более новый пакет SDK для Azure Monitor](https://www.nuget.org/packages/Microsoft.Azure.Management.Monitor/)

