---
title: Подключение данных Sophos Cloud Оптикс к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель Sophos Cloud Оптикс для извлечения <PRODUCT NAME> журналов в Azure Sentinel. Просмотр <PRODUCT NAME> данных в книгах, создание оповещений и улучшенное исследование.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/26/2021
ms.author: yelevin
ms.openlocfilehash: c66205ffd9bd5a742d645cbf2f9251a44329a16e
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101700934"
---
# <a name="connect-your-sophos-cloud-optix-to-azure-sentinel"></a>Подключение Sophos Cloud Оптикс к Azure Sentinel

> [!IMPORTANT]
> Соединитель Sophos Cloud Оптикс сейчас находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

Соединитель Sophos Cloud Оптикс позволяет легко подключать все журналы решений безопасности Sophos Cloud Оптикс с помощью вашего Sentinel Azure, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование.  Эта возможность позволяет получить более подробные сведения о безопасности в облаке и обеспечении соответствия требованиям вашей организации, а также улучшает возможности облачных операций безопасности. При интеграции между Sophos Cloud Оптикс и Sentinel Azure используется REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-sophos-cloud-optix"></a>Настройка и подключение Sophos Cloud Оптикс

Sophos Cloud Оптикс может интегрировать и экспортировать журналы непосредственно в Azure Sentinel.

1. На портале Sentinel Azure щелкните **Data Connectors (соединители данных** ) и выберите **Sophos Cloud Оптикс (Предварительная версия)**.

1. Выберите **открыть страницу соединителя**.

1. Скопируйте и сохраните **идентификатор рабочей области** и **первичный ключ** на странице соединителя.

1. Следуйте инструкциям из Sophos для [интеграции с Microsoft Azure Sentinel](https://docs.sophos.com/pcg/optix/help/en-us/pcg/optix/tasks/IntegrateAzureSentinel.html) (начиная с третьего шага).

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе **пользовательские журналы** в таблице *SophosCloudOptix_CL* .

Чтобы запросить данные Sophos Cloud Оптикс, введите `SophosCloudOptix_CL` в окне запроса. Некоторые полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя и в [документации по Sophos](https://docs.sophos.com/pcg/optix/help/en-us/pcg/optix/concepts/ExampleAzureSentinelQueries.html).

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут. 

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Sophos Cloud Оптикс к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
