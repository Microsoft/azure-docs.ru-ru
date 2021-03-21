---
title: Управление журналами — портал Azure — база данных Azure для PostgreSQL — один сервер
description: В этой статье описывается, как настроить журналы сервера (файлы журнала) и получить доступ к ним в базе данных Azure для PostgreSQL-Single Server из портал Azure.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: how-to
ms.date: 5/6/2019
ms.openlocfilehash: 3b52cea1d440506caf5b8244c9643719edd8755c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95999252"
---
# <a name="configure-and-access-azure-database-for-postgresql---single-server-logs-from-the-azure-portal"></a>Настройка журналов базы данных Azure для PostgreSQL и доступ к ним из портал Azure

Вы можете настроить, перечислить и скачать [журналы базы данных Azure для PostgreSQL](concepts-server-logs.md) из портал Azure.

## <a name="prerequisites"></a>Предварительные условия
Для выполнения действий, описанных в этой статье, требуется [сервер базы данных Azure для PostgreSQL](quickstart-create-server-database-portal.md).

## <a name="configure-logging"></a>Настройка журнала
Настройте доступ к журналам запросов и журналам ошибок. 

1. Войдите на [портал Azure](https://portal.azure.com/).

2. Выберите сервер базы данных Azure для PostgreSQL.

3. В разделе **мониторинг** на боковой панели выберите **журналы сервера**. 

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/1-select-server-logs-configure.png" alt-text="Снимок экрана параметров журналов сервера":::

4. Чтобы просмотреть параметры сервера, выберите **щелкните здесь, чтобы включить журналы и настроить параметры журнала**.

5. Измените параметры, которые необходимо настроить. Все изменения, вносимые в этом сеансе, выделены фиолетовым цветом.

   После изменения параметров нажмите кнопку **сохранить**. Вы также можете отменить изменения. 

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/3-save-discard.png" alt-text="Снимок экрана параметров параметров сервера":::

На странице **Параметры сервера** можно вернуться к списку журналов, закрыв страницу.

## <a name="view-list-and-download-logs"></a>Просмотр списка журналов и их скачивание
После начала ведения журнала можно просмотреть список доступных журналов и загрузить отдельные файлы журналов. 

1. Перейдите на портал Azure.

2. Выберите сервер базы данных Azure для PostgreSQL.

3. В разделе **мониторинг** на боковой панели выберите **журналы сервера**. На странице отображается список файлов журнала.

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/4-server-logs-list.png" alt-text="Снимок экрана: страница &quot;журналы сервера&quot; с выделенным списком журналов":::

   > [!TIP]
   > Действует следующее соглашение об именовании журналов: **postgresql-гггг-мм-дд_чч0000.log**. Дата и время, используемые в имени файла, — это время, когда был выдан журнал. Файлы журналов поворачиваются каждый час или 100 МБ, в зависимости от того, что происходит первым.

4. При необходимости используйте поле поиска, чтобы быстро сократить до определенного журнала на основе даты и времени. Поиск осуществляется по имени журнала.

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/5-search.png" alt-text="Снимок экрана: страница &quot;журналы сервера&quot; с выделенным полем поиска и результатами":::

5. Чтобы загрузить отдельные файлы журналов, щелкните значок со стрелкой вниз рядом с каждым файлом журнала в строке таблицы.

   :::image type="content" source="./media/howto-configure-server-logs-in-portal/6-download.png" alt-text="Снимок экрана: страница &quot;журналы сервера&quot; с выделенным значком &quot;стрелка вниз&quot;":::

## <a name="next-steps"></a>Дальнейшие действия
- Сведения о загрузке журналов программным способом см. в разделе [Журналы доступа к серверу в интерфейсе командной строки](howto-configure-server-logs-using-cli.md) .
- Дополнительные сведения о [журналах сервера](concepts-server-logs.md) в базе данных Azure для PostgreSQL. 
- Дополнительные сведения об определениях параметров и ведении журнала PostgreSQL см. в документации PostgreSQL по [отчетам об ошибках и ведению журнала](https://www.postgresql.org/docs/current/static/runtime-config-logging.html).

