---
title: Подключение данных Google Workspace (G Suite) к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель данных Google Workspace (G Suite) для приема событий действий Google Workspace в Azure Sentinel. Просмотр данных Google Workspace в книгах, создание оповещений и улучшенное исследование.
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
ms.date: 02/28/2021
ms.author: yelevin
ms.openlocfilehash: 4ada570502d913283ba9ee4cc4c65b7bdd853935
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746287"
---
# <a name="connect-your-google-workspace-to-azure-sentinel"></a>Подключение Google Workspace к Azure Sentinel

> [!IMPORTANT]
> Соединитель Google Workspace сейчас находится на **этапе предварительной версии**. Ознакомьтесь с дополнительными [условиями использования Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) предварительных версий для дополнительных юридических условий, которые относятся к функциям Azure, которые доступны в бета-версии, предварительном просмотре или еще не выпущены в общедоступную версию.

Соединитель данных [Google Workspace (прежнее название — G Suite)](https://workspace.google.com/) предоставляет возможность приема событий действий Google Workspace в Azure Sentinel с помощью REST API. Соединитель предоставляет эти [события](https://developers.google.com/admin-sdk/reports/reference/rest/v1/activities) в приложении SoC, которое помогает исследовать потенциальные риски безопасности, анализировать использование совместной работы, диагностировать проблемы с конфигурацией, отслеживать, кто входит в систему и когда, анализировать действия администратора, понимать, как пользователи создают и совместно используют содержимое, а также просматривают больше событий в Организации.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="prerequisites"></a>Предварительные условия

Чтобы получить данные Google Workspace, необходимы следующие разрешения и учетные данные:

- Разрешения на чтение и запись в рабочей области Sentinel Azure.

- Разрешения на чтение общих ключей для рабочей области. Дополнительные [сведения о ключах рабочих областей](../azure-monitor/agents/log-analytics-agent.md#workspace-id-and-key).

- Разрешения на чтение и запись в функции Azure для создания приложение-функция. Дополнительные [сведения о функциях Azure](../azure-functions/index.yml).

- Для REST API требуются учетные данные **гуглепикклестринг** . Дополнительные [сведения о Google REST API](https://developers.google.com/admin-sdk/reports/v1/reference/activities). [Узнайте, как получить учетные данные](https://developers.google.com/admin-sdk/reports/v1/quickstart/python).

## <a name="configure-and-connect-google-workspace"></a>Настройка и подключение Google Workspace

Google Workspace может интегрировать и экспортировать журналы непосредственно в Azure Sentinel с помощью приложение-функция Azure.

> [!NOTE]
> Этот соединитель использует функции Azure для подключения к API Google Reports для получения событий действий в Azure Sentinel. Это может привести к дополнительным затратам на прием данных. Дополнительные сведения см. на странице с [ценами на функции Azure](https://azure.microsoft.com/pricing/details/functions/) .

1. На портале Sentinel Azure щелкните **соединители данных**. 

1. Выберите **Google Workspace (G Suite) (Предварительная версия** ) в галерее соединителей и щелкните **открыть страницу соединителя**.

1. Выполните действия, описанные в разделе **Настройка** на странице соединителя.

## <a name="validate-connectivity-and-find-your-data"></a>Проверка подключения и поиск данных

Пока вы не сможете увидеть полученные данные в Azure Sentinel, может потребоваться до 20 минут.

После установки успешного подключения данные отображаются в **журналах** в разделе **пользовательские журналы** в следующих таблицах:
- `GWorkspace_ReportsAPI_admin_CL`
- `GWorkspace_ReportsAPI_calendar_CL`
- `GWorkspace_ReportsAPI_drive_CL`
- `GWorkspace_ReportsAPI_login_CL`
- `GWorkspace_ReportsAPI_mobile_CL`
- `GWorkspace_ReportsAPI_token_CL`
- `GWorkspace_ReportsAPI_user_accounts_CL`

Этот соединитель данных зависит от средства синтаксического анализа, основанного на функции Kusto, чтобы работать должным образом. Чтобы создать псевдоним функций **гворкспацеактивитирепортс** Kusto, [выполните следующие действия](https://aka.ms/sentinel-GWorkspaceReportsAPI-parser) .

Полезные примеры запросов см. на вкладке **дальнейшие действия** на странице соединителя.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить Google Workspace к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [получить представление о данных и потенциальных угрозах](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
