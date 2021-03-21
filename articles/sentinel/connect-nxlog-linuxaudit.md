---
title: Подключение данных Нкслог Линуксаудит к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных Нкслог Линуксаудит для извлечения журналов Линуксаудит в Azure Sentinel. Просмотр данных Линуксаудит в книгах, создание оповещений и улучшение расследования.
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
ms.date: 03/02/2021
ms.author: yelevin
ms.openlocfilehash: 2010b21a3cdb81f2b2aa4180f87857862cd02bf5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101700939"
---
# <a name="connect-your-nxlog-linuxaudit-to-azure-sentinel"></a>Подключение Нкслог Линуксаудит к Azure Sentinel

> [!IMPORTANT]
> Соединитель Нкслог Линуксаудит в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

Соединитель [Нкслог линуксаудит](https://nxlog.co/documentation/nxlog-user-guide/im_linuxaudit.html) позволяет легко экспортировать события безопасности Linux в метку Azure в режиме реального времени, что позволяет получить представление об активности сервера доменных имен в Организации. Модуль Нкслог Линуксаудит поддерживает пользовательские правила аудита и собирает журналы без аудита или другого программного обеспечения, используемого для пользователя. IP-адреса и идентификаторы групп и пользователей разрешаются в соответствующие им имена, что делает журналы [аудита Linux](https://nxlog.co/documentation/nxlog-user-guide/linux-audit.html) более читаемыми для аналитиков безопасности. Интеграция между Нкслог Линуксаудит и Sentinel Azure упрощается с помощью REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-nxlog-linuxaudit"></a>Настройка и подключение Нкслог Линуксаудит

Нкслог можно настроить для отправки событий в формате JSON непосредственно в Azure Sentinel.

1. На портале Sentinel Azure щелкните **соединители данных** и выберите **нкслог линуксаудит** Connector.

1. Выберите **открыть страницу соединителя**.

1. Следуйте пошаговым инструкциям в разделе Интеграция с *руководством пользователя нкслог* [Microsoft Azure Sentinel](https://nxlog.co/documentation/nxlog-user-guide/sentinel.html) для настройки переадресации с помощью REST API.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе  **пользовательские журналы** в таблице *LinuxAudit_CL* .

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как использовать Нкслог Линуксаудит для приема событий безопасности Linux в Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
