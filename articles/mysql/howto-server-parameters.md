---
title: Настройка параметров сервера — портал Azure — база данных Azure для MySQL
description: В этой статье описывается, как настроить параметры MySQL сервера в базе данных Azure для MySQL с помощью портала Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 10/1/2020
ms.openlocfilehash: 363be8b34f230b812bc24276e1f3925faf0cdc1c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94540847"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-using-the-azure-portal"></a>Настройка параметров сервера в базе данных Azure для MySQL с помощью портал Azure

База данных Azure для MySQL поддерживает настройку некоторых параметров сервера. В этой статье описывается настройка этих параметров с помощью портала Azure. Не все параметры сервера можно настроить.

>[!Note]
> Параметры сервера можно обновлять глобально на уровне сервера с помощью [Azure CLI](./howto-configure-server-parameters-using-cli.md), [PowerShell](./howto-configure-server-parameters-using-powershell.md)или [портала Azure](./howto-server-parameters.md).

## <a name="configure-server-parameters"></a>Настройка параметров сервера

1. Войдите в [портал Azure](https://portal.azure.com), а затем найдите сервер базы данных Azure для MySQL.
2. В разделе **Параметры** щелкните **Параметры сервера**, чтобы открыть страницу параметров сервера для сервера Базы данных Azure для MySQL.
:::image type="content" source="./media/howto-server-parameters/auzre-portal-server-parameters.png" alt-text="Страница параметров сервера на портале Azure":::
3. Найдите все параметры, которые необходимо настроить. Просмотрите столбец **Описание**, чтобы понять назначение и допустимые значения.
:::image type="content" source="./media/howto-server-parameters/3-toggle_parameter.png" alt-text="Раскрывающийся список для перечисляемого типа":::
4. Нажмите кнопку  **сохранить** , чтобы сохранить изменения.
:::image type="content" source="./media/howto-server-parameters/4-save_parameters.png" alt-text="Сохранение или отмена изменений":::
5. Если вы сохранили новые значения параметров, всегда можно восстановить значения по умолчанию, выбрав **Сбросить все к значениям по умолчанию**.
:::image type="content" source="./media/howto-server-parameters/5-reset_parameters.png" alt-text="Сбросить все к значениям по умолчанию":::

## <a name="setting-parameters-not-listed"></a>Параметры настройки не указаны

Если параметр сервера, который требуется обновить, не указан в портал Azure, можно при необходимости задать параметр на уровне соединения с помощью `init_connect` . Это задает параметры сервера для каждого клиента, подключающегося к серверу. 

1. В разделе **Параметры** щелкните **Параметры сервера**, чтобы открыть страницу параметров сервера для сервера Базы данных Azure для MySQL.
2. Поиск `init_connect`
3. Добавьте параметры сервера в формате. `SET parameter_name=YOUR_DESIRED_VALUE` значение в столбце значение.

    Например, можно изменить кодировку сервера, задав `init_connect` для значение `SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;`
4. Нажмите кнопку **Сохранить**, чтобы сохранить изменения.

>[!Note]
> `init_connect` можно использовать для изменения параметров, которым не нужны привилегии SUPER на уровне сеанса. Чтобы проверить, можно ли задать параметр с помощью `init_connect`, выполните команду `set session parameter_name=YOUR_DESIRED_VALUE;`. Если отображается сообщение об ошибке **Access denied; you need SUPER privileges(s)** (Отказано в доступе. Для выполнения этого действия необходимо иметь привилегии SUPER.), то параметр нельзя задать с помощью init_connect.

## <a name="working-with-the-time-zone-parameter"></a>Работа с параметром часового пояса

### <a name="populating-the-time-zone-tables"></a>Заполнение таблиц часовых поясов

Таблицы часовых поясов на сервере можно заполнить, вызвав хранимую процедуру `mysql.az_load_timezone` с помощью такого инструмента, как командная строка MySQL или MySQL Workbench.

> [!NOTE]
> Если вы используете команду `mysql.az_load_timezone` в MySQL Workbench, может потребоваться предварительно отключить режим безопасного обновления с помощью `SET SQL_SAFE_UPDATES=0;`.

```sql
CALL mysql.az_load_timezone();
```

> [!IMPORTANT]
> Необходимо перезапустить сервер, чтобы убедиться, что таблицы часовых поясов заполнены правильно. Чтобы перезапустить сервер, используйте [портал Azure](howto-restart-server-portal.md) или [CLI](howto-restart-server-cli.md).

Чтобы просмотреть доступные значения часового пояса, выполните следующую команду.

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>Установка часового пояса глобального уровня

Часовой пояс глобального уровня можно задать на странице **Параметры сервера** на портале Azure. Ниже приведен пример, который задает глобальный часовой пояс "US/Pacific" (США, Тихоокеанский регион).

:::image type="content" source="./media/howto-server-parameters/timezone.png" alt-text="Настройка параметра часового пояса":::

### <a name="setting-the-session-level-time-zone"></a>Настройка часового пояса уровня сеанса

Часовой пояс уровня сеанса можно задать, выполнив команду `SET time_zone` в командной строке MySQL или MySQL Workbench. В приведенном ниже примере задается часовой пояс **US/Pacific** (США, Тихоокеанский регион).

```sql
SET time_zone = 'US/Pacific';
```

Описание [функций даты и времени](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz) можно прочитать в документации по MySQL.

## <a name="next-steps"></a>Дальнейшие действия

- [Библиотеки подключений для базы данных Azure для MySQL](concepts-connection-libraries.md).
