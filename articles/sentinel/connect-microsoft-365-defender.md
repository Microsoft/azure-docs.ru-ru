---
title: Подключение Microsoft 365 данных защитника к Azure Sentinel | Документация Майкрософт
description: Узнайте, как принимать инциденты, оповещения и необработанные данные о событиях из Microsoft 365 защитника в Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/13/2019
ms.author: yelevin
ms.openlocfilehash: 6500805a4dc7e26f5e1bc601df9ea78279ae17e9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101709348"
---
# <a name="connect-data-from-microsoft-365-defender-to-azure-sentinel"></a>Подключение данных из защитника Microsoft 365 к Azure Sentinel

> [!IMPORTANT]
>
> **Защитник Microsoft 365** ранее назывался **Microsoft Threat protection** или **MTP**.
>
> **Защитник Майкрософт для конечной точки** ранее назывался **Microsoft Defender Advanced Threat protection** или **мдатп**.
>
> Вы можете увидеть, что старые имена по-прежнему используются в течение определенного периода времени.

## <a name="background"></a>Историческая справка

Соединитель [Microsoft 365 защитника Azure (M365D)](/microsoft-365/security/mtp/microsoft-threat-protection) с интеграцией инцидентов позволяет выполнять потоковую передачу всех инцидентов и оповещений M365D в Azure Sentinel, а также синхронизировать инциденты между обоими порталами. M365D инциденты включают все свои оповещения, сущности и другие важные сведения, а также их дополненные и группируют предупреждения от службы компонентов M365D's, защитник Майкрософт **для конечной точки**, **защитник Майкрософт для идентификации**, **Microsoft defender для Office 365** и **Microsoft Cloud App Security**.

Соединитель также позволяет выполнять потоковую передачу **дополнительных** событий подсчета от защитника Майкрософт для конечной точки в Azure Sentinel, позволяя копировать расширенные запросы на поиск в формате MDE в Azure Sentinel, расширять метки с помощью данных о событиях MDE, чтобы предоставить дополнительные сведения, а также хранить журналы с увеличением срока хранения в log Analytics.

Дополнительные сведения об интеграции инцидентов и сборе событий расширенной сборки см. [в статье интеграция защитника Microsoft 365 с Azure Sentinel](microsoft-365-defender-sentinel-integration.md).

> [!IMPORTANT]
>
> Соединитель защитника Microsoft 365 в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

## <a name="prerequisites"></a>Предварительные условия

- Для защитника Microsoft 365 необходимо иметь действительную лицензию, как описано в разделе [Предварительные требования для защитника Microsoft 365](/microsoft-365/security/mtp/prerequisites). 

- Вы должны быть **глобальным администратором** или **администратором безопасности** в Azure Active Directory.

## <a name="connect-to-microsoft-365-defender"></a>Подключение к защитнику Microsoft 365

1. В качестве метки Azure выберите **соединители данных**, **Microsoft 365 защитник (Предварительная версия)** из коллекции и щелкните **открыть страницу соединителя**.

1. В разделе **Конфигурация** статьи **подключение инцидентов & предупреждения** нажмите кнопку **подключить инциденты & предупреждения** .

1. Чтобы избежать дублирования инцидентов, рекомендуется пометить флажок **Отключить все правила создания инцидентов Майкрософт для этих продуктов.**

    > [!NOTE]
    > При включении соединителя защитника Microsoft 365 все соединители компонентов M365D (упомянутые в начале этой статьи) автоматически подключаются в фоновом режиме. Чтобы отключить один из соединителей компонентов, сначала необходимо отключить соединитель защитника Microsoft 365.

1. Чтобы запросить данные об инциденте защитника M365, используйте следующую инструкцию в окне запроса:
    ```kusto
    SecurityIncident
    | where ProviderName == "Microsoft 365 Defender"
    ```

1. Если вы хотите собирать дополнительные события с помощью защитника Майкрософт для конечной точки, можно собирать следующие типы событий из соответствующих расширенных таблиц.

    1. Установите флажки для таблиц с типами событий, которые вы хотите получать:

       | Имя таблицы | Тип событий |
       |-|-|
       | DeviceInfo | Сведения о компьютере (включая сведения о ОС) |
       | DeviceNetworkInfo | Свойства сети компьютеров |
       | DeviceProcessEvents | Создание процессов и связанные события |
       | DeviceNetworkEvents | Сетевое подключение и связанные события |
       | DeviceFileEvents | Создание, изменение и другие события файловой системы |
       | DeviceRegistryEvents | Создание и изменение записей реестра |
       | DeviceLogonEvents | Входы и другие события проверки подлинности |
       | DeviceImageLoadEvents | События загрузки DLL |
       | DeviceEvents | Дополнительные типы событий |
       | DeviceFileCertificateInfo | Сведения о сертификате подписанных файлов |
       |

    1. Щелкните **Применить изменения**.

    1. Чтобы запросить дополнительные таблицы поиска в Log Analytics, введите имя таблицы из списка выше в окне запроса.

## <a name="verify-data-ingestion"></a>Проверка приема данных

Диаграмма данных на странице соединитель указывает на то, что вы принимаете данные. Вы заметите, что каждая строка для инцидентов, предупреждений и событий отображается по одной строке, а строка событий — это совокупность объема событий во всех включенных таблицах. После включения соединителя можно использовать следующие запросы ККЛ для создания более конкретных графов.

Используйте следующий запрос ККЛ для графа входящих инцидентов M365 Defender:

```kusto
let Now = now(); 
(range TimeGenerated from ago(14d) to Now-1d step 1d 
| extend Count = 0 
| union isfuzzy=true ( 
    SecurityIncident
    | where ProviderName == "Microsoft 365 Defender"
    | summarize Count = count() by bin_at(TimeGenerated, 1d, Now) 
) 
| summarize Count=max(Count) by bin_at(TimeGenerated, 1d, Now) 
| sort by TimeGenerated 
| project Value = iff(isnull(Count), 0, Count), Time = TimeGenerated, Legend = "Events") 
| render timechart 
```

Используйте следующий запрос ККЛ для создания графа объема событий для одной таблицы (измените таблицу *девицеевентс* на нужную таблицу по своему усмотрению):

```kusto
let Now = now();
(range TimeGenerated from ago(14d) to Now-1d step 1d
| extend Count = 0
| union isfuzzy=true (
    DeviceEvents
    | summarize Count = count() by bin_at(TimeGenerated, 1d, Now)
)
| summarize Count=max(Count) by bin_at(TimeGenerated, 1d, Now)
| sort by TimeGenerated
| project Value = iff(isnull(Count), 0, Count), Time = TimeGenerated, Legend = "Events")
| render timechart
```

На вкладке **дальнейшие действия** вы найдете некоторые полезные книги, примеры запросов и шаблоны правил аналитики, которые были добавлены. Их можно запустить в месте или изменить и сохранить.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как интегрировать инциденты защитника Microsoft 365 и Расширенные данные о событиях из защитника Майкрософт для конечных точек в Azure Sentinel с помощью соединителя Microsoft 365 Defender. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](./tutorial-detect-threats-built-in.md).
