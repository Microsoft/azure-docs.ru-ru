---
title: Подключение данных журналов Нкслог Windows DNS к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных журналов DNS Нкслог для извлечения событий DNS Windows в Azure Sentinel. Просмотр данных DNS Windows в книгах, создание оповещений и улучшение расследования.
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
ms.openlocfilehash: 880aad438d98605d11e5a2a7c314d89bd8beb5c5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101747081"
---
# <a name="connect-your-nxlog-windows-dns-logs-to-azure-sentinel"></a>Подключение журналов DNS Нкслог Windows к Sentinel Azure

> [!IMPORTANT]
> Соединитель журналов DNS Нкслог в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

Соединитель [журналов DNS нкслог](https://nxlog.co/documentation/nxlog-user-guide/windows-dns-server.html) позволяет легко экспортировать все события DNS-сервера Windows в метку Azure в режиме реального времени, что позволяет получить представление об активности сервера доменных имен в Организации. Он использует [трассировку событий высокой производительности для Windows (ETW)](https://nxlog.co/documentation/nxlog-user-guide/windows-dns-server.html#dns_windows_etw) для сбора событий аудита и аналитического события DNS-сервера в режиме реального времени и поддерживает дополнение событий с помощью настраиваемых полей. Интеграция между Нкслог журналами DNS и Azure Sentinel упрощается с помощью REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-nxlog-dns-logs"></a>Настройка и подключение журналов DNS Нкслог

Нкслог можно настроить для отправки событий в формате JSON непосредственно в Azure Sentinel.

1. На портале Sentinel Azure щелкните **Data Connectors (соединители данных** ) и выберите **Нкслог DNS-журналы** .

1. Выберите **открыть страницу соединителя**.

1. Следуйте пошаговым инструкциям в разделе Интеграция с *руководством пользователя нкслог* [Microsoft Azure Sentinel](https://nxlog.co/documentation/nxlog-user-guide/sentinel.html) для настройки переадресации с помощью REST API.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе  **пользовательские журналы** в таблице *DNS_Logs_CL* .

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как использовать Нкслог для приема журналов DNS Windows в Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
