---
title: Подключение данных безопасности электронной почты Proofpoint On Demand по запросу к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных безопасности электронной почты Proofpoint On Demand по запросу для извлечения журналов безопасности электронной почты POD в Azure Sentinel. Просмотр данных безопасности электронной почты POD в книгах, создание оповещений и улучшенное исследование.
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
ms.openlocfilehash: 73a9d9c7ab321aebd615922e5d4395c0318e809c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100580453"
---
# <a name="connect-your-proofpoint-on-demand-email-security-pod-solution-to-azure-sentinel"></a>Подключение решения для защиты электронной почты Proofpoint On Demand по запросу (POD) к Azure Sentinel

> [!IMPORTANT]
> Соединитель безопасности электронной почты Proofpoint On Demand по запросу сейчас находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

В этой статье объясняется, как подключить устройство безопасности электронной почты Proofpoint On Demand по запросу к Azure Sentinel. Соединитель данных POD позволяет легко подключать журналы POD с помощью Sentinel Azure, позволяя просматривать данные в книгах, использовать их для создания пользовательских оповещений и внедрять их для улучшения расследования.  Интеграция между Proofpoint On Demand по электронной почте и в Azure Sentinel использует API WebSocket.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

- У вас должно быть разрешение на чтение и запись в рабочей области "Sentinel" Azure.

- Необходимо иметь разрешения на чтение общих ключей для рабочей области. Дополнительные [сведения о ключах рабочих областей](../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key).

- Чтобы создать приложение-функция, необходимо иметь разрешения на чтение и запись для функций Azure. Дополнительные [сведения о функциях Azure](../azure-functions/index.yml).

- Необходимо иметь следующие учетные данные API WebSocket: Пруфпоинтклустерид, Пруфпоинттокен. Дополнительные [сведения о API WebSocket](https://proofpointcommunities.force.com/community/s/article/Proofpoint-on-Demand-Pod-Log-API).

## <a name="configure-and-connect-proofpoint-on-demand-email-security"></a>Настройка и подключение безопасности электронной почты Proofpoint On Demand по запросу

Безопасность электронной почты proofpoint On Demand по запросу может интегрировать и экспортировать журналы непосредственно в Azure Sentinel.

1. В меню навигации меток Azure выберите **соединители данных**.

1. В коллекции **соединителей данных** выберите **Proofpoint On Demand по запросу безопасность электронной почты (Предварительная версия)** , а затем **откройте страницу соединителя**.

1. Выполните действия, описанные в разделе **Настройка** на странице соединителя.

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные отображаются в **журналах** в разделе *пользовательские журналы* в следующих таблицах:
- `ProofpointPOD_message_CL`
- `ProofpointPOD_maillog_CL`

Полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 60 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Proofpoint On Demand по запросу к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.