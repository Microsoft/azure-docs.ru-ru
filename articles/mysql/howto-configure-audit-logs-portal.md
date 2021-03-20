---
title: Доступ к журналам аудита — портал Azure — база данных Azure для MySQL
description: В этой статье описывается, как настроить и получить доступ к журналам аудита в базе данных Azure для MySQL из портал Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 9/29/2020
ms.openlocfilehash: a73a3c77310c0e19792758c0586975e14cfaebf8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94541663"
---
# <a name="configure-and-access-audit-logs-for-azure-database-for-mysql-in-the-azure-portal"></a>Настройка и доступ к журналам аудита для базы данных Azure для MySQL в портал Azure

Вы можете настроить [журналы аудита базы данных Azure для MySQL](concepts-audit-logs.md) и параметры диагностики из портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к выполнению этого руководства, необходимы следующие компоненты:

- [Сервер базы данных Azure для MySQL](quickstart-create-mysql-server-database-using-azure-portal.md)

## <a name="configure-audit-logging"></a>Настройка ведения журнала аудита

>[!IMPORTANT]
> Рекомендуется регистрировать только те типы событий и пользователей, которые необходимы для целей аудита, чтобы гарантировать, что производительность сервера не окажется сильной.

Включение и Настройка ведения журнала аудита.

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Выберите сервер базы данных Azure для MySQL.

1. В разделе **Параметры** на боковой панели выберите **Параметры сервера**.
    :::image type="content" source="./media/howto-configure-audit-logs-portal/server-parameters.png" alt-text="Параметры сервера":::

1. Обновите параметр **audit_log_enabled** в значение ON.
    :::image type="content" source="./media/howto-configure-audit-logs-portal/audit-log-enabled.png" alt-text="Включить журналы аудита":::

1. Выберите [типы событий](concepts-audit-logs.md#configure-audit-logging) для записи, обновив параметр **audit_log_events** .
    :::image type="content" source="./media/howto-configure-audit-logs-portal/audit-log-events.png" alt-text="События журнала аудита":::

1. Добавьте пользователей MySQL, которые будут включены или исключены из ведения журнала, обновив параметры **audit_log_exclude_users** и **audit_log_include_users** . Укажите пользователей, указав имя пользователя MySQL.
    :::image type="content" source="./media/howto-configure-audit-logs-portal/audit-log-exclude-users.png" alt-text="Исключить пользователей из журнала аудита":::

1. После изменения параметров нажмите кнопку **Сохранить**. Также вы можете нажать кнопку **Отменить**, чтобы отменить изменения.
    :::image type="content" source="./media/howto-configure-audit-logs-portal/save-parameters.png" alt-text="Сохранить":::

## <a name="set-up-diagnostic-logs"></a>Настройка журналов диагностики

1. В разделе **мониторинг** на боковой панели выберите **параметры диагностики**.

1. Щелкните "+ добавить параметр диагностики" :::image type="content" source="./media/howto-configure-audit-logs-portal/add-diagnostic-setting.png" alt-text="Добавить параметр диагностики":::

1. Укажите имя параметра диагностики.

1. Укажите приемники данных для отправки журналов аудита (учетную запись хранения, концентратор событий и (или) рабочую область Log Analytics).

1. Выберите "Мисклаудитлогс" в качестве типа журнала.
:::image type="content" source="./media/howto-configure-audit-logs-portal/configure-diagnostic-setting.png" alt-text="Настройка параметра диагностики":::

1. После настройки приемников данных для передачи журналов аудита в можно нажать кнопку **сохранить**.
:::image type="content" source="./media/howto-configure-audit-logs-portal/save-diagnostic-setting.png" alt-text="Сохранить параметр диагностики":::

1. Получите доступ к журналам аудита, просмотрев их в настроенных приемниках данных. Для отображения журналов может потребоваться до 10 минут.

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [журналах аудита](concepts-audit-logs.md) в базе данных Azure для MySQL
- Узнайте, как настроить журналы аудита в [Azure CLI](howto-configure-audit-logs-cli.md)