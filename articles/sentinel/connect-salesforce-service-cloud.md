---
title: Подключение облачных данных службы Salesforce к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель облачных данных службы salesforce для извлечения журналов Salesforce в Azure Sentinel. Просмотр данных Salesforce в книгах, создание оповещений и улучшение расследования.
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
ms.date: 01/17/2021
ms.author: yelevin
ms.openlocfilehash: 1efd91d92bac1bc1f39d82aaa0cc71daa0275f8e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100570529"
---
# <a name="connect-your-salesforce-service-cloud-to-azure-sentinel"></a>Подключение облака службы Salesforce к Azure Sentinel

> [!IMPORTANT]
> Соединитель облака службы Salesforce в настоящее время находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить облачное решение службы Salesforce к Azure Sentinel. Соединитель облачных данных Salesforce Service позволяет легко подключать данные Salesforce к Azure Sentinel, чтобы их можно было просматривать в книгах, использовать для создания пользовательских оповещений и внедрять их для улучшения расследования. При интеграции между облаком службы Salesforce и Sentinel в Azure используется REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должно быть разрешение на чтение и запись в рабочей области "Sentinel" Azure.

- Необходимо иметь разрешения на чтение общих ключей для рабочей области. Дополнительные [сведения о ключах рабочих областей](../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key).

- Чтобы создать приложение-функция, необходимо иметь разрешения на чтение и запись для функций Azure. Дополнительные [сведения о функциях Azure](../azure-functions/index.yml).

- Необходимо иметь следующие учетные данные Salesforce REST API: **имя пользователя API Salesforce**, **пароль API Salesforce**, **токен безопасности Salesforce**, **ключ потребителя** Salesforce, **секрет потребителя Salesforce**. Дополнительные [сведения о Salesforce REST API](https://developer.salesforce.com/docs/atlas.en-us.api_rest.meta/api_rest/quickstart.htm).

## <a name="configure-and-connect-salesforce-service-cloud"></a>Настройка и подключение облака службы Salesforce

Облако службы Salesforce может интегрировать и экспортировать журналы непосредственно в Azure Sentinel.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите **облако службы Salesforce (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Выполните действия, описанные в разделе **Настройка** на странице соединителя.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе **CustomLogs** в `SalesforceServiceCloud_CL` таблице.

Полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить облако службы Salesforce к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.