---
title: Настройка журналов запросов с высокой производительностью. портал Azure — гибкий сервер базы данных Azure для MySQL
description: В этой статье описывается, как настроить и получить доступ к журналам запросов для гибкого сервера базы данных Azure для MySQL с портал Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: a61c8e3451d661dae2e5ad56a0d4a947252ec873
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94540069"
---
# <a name="configure-and-access-slow-query-logs-for-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Настройка и доступ к журналам запросов для базы данных Azure для MySQL — гибкого сервера с помощью портал Azure

> [!IMPORTANT]
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

Вы можете настроить, перечислить и скачать базу данных Azure для гибких серверов [запросов](concepts-slow-query-logs.md) MySQL из портал Azure.

## <a name="prerequisites"></a>Предварительные условия
Для выполнения действий, описанных в этой статье, требуется [гибкий сервер](quickstart-create-server-portal.md).

## <a name="configure-logging"></a>Настройка журнала
Настройте доступ к журналу медленных запросов MySQL. 

1. Войдите на [портал Azure](https://portal.azure.com/).

1. Выберите гибкий сервер.

1. В разделе **Параметры** на боковой панели выберите **Параметры сервера**.
   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/server-parameters.png" alt-text="Страница параметров сервера.":::

1. Обновите параметр **slow_query_log** в значение **On**.
   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/slow-query-log-enable.png" alt-text="Включите журналы запросов с высокой скоростью.":::

1. Измените необходимые параметры (например, `long_query_time`, `log_slow_admin_statements`). Дополнительные параметры см. в документации по [журналам медленных запросов](./concepts-slow-query-logs.md#configure-slow-query-logging) .  
   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/long-query-time.png" alt-text="Обновите параметры, связанные с журналом запросов с задержкой.":::

1. Щелкните **Сохранить**. 
   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/save-parameters.png" alt-text="Сохранение параметров журнала запросов с высокой скоростью.":::

На странице **Параметры сервера** можно вернуться к списку журналов, закрыв страницу.

## <a name="set-up-diagnostics"></a>Настройка диагностики

Журналы слишком долго интегрируются с Azure Monitor параметрами диагностики, что позволяет передавать журналы в Azure Monitor журналы, концентраторы событий или службу хранилища Azure.

1. В разделе **мониторинг** на боковой панели выберите **параметры диагностики**  >  **Добавить параметры диагностики**.

   :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/add-diagnostic-setting.png" alt-text="Снимок экрана параметров параметров диагностики":::

1. Укажите имя параметра диагностики.

1. Укажите назначения для отправки журналов запросов с задержкой (учетную запись хранения, концентратор событий или рабочую область Log Analytics).

1. В качестве типа журнала выберите **мисклсловлогс** .
    :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/configure-diagnostic-setting.png" alt-text="Снимок экрана параметров конфигурации параметров диагностики":::

1. После настройки приемников данных для передачи журналов запросов с задержкой нажмите кнопку **сохранить**.
    :::image type="content" source="./media/how-to-configure-slow-query-logs-portal/save-diagnostic-setting.png" alt-text="Снимок экрана параметров конфигурации параметров диагностики с выделенным сохранением":::

1. Получите доступ к журналам запросов с задержкой, изучив их в настроенных приемниках данных. Для отображения журналов может потребоваться до 10 минут.

Если вы передаете журналы в Azure Monitor журналы (Log Analytics), ознакомьтесь с [примерами запросов](concepts-slow-query-logs.md#analyze-logs-in-azure-monitor-logs) , которые можно использовать для анализа. 

## <a name="next-steps"></a>Дальнейшие действия
<!-- - See [Access slow query Logs in CLI](howto-configure-server-logs-in-cli.md) to learn how to download slow query logs programmatically.-->
- Дополнительные сведения о [журналах медленных запросов](concepts-slow-query-logs.md)
- Дополнительные сведения об определениях параметров и ведении журнала MySQL см. в документации MySQL по [журналам](https://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html).
- Дополнительные сведения о [журналах аудита](concepts-audit-logs.md)
