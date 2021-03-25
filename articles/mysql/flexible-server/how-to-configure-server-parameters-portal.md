---
title: Настройка параметров сервера — портал Azure — гибкий сервер базы данных Azure для MySQL
description: В этой статье описывается, как настроить параметры сервера MySQL в базе данных Azure для гибкого сервера MySQL с помощью портал Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 11/10/2020
ms.openlocfilehash: 618862e12bd62fbe37ef5e621c89babd7942c04b
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105106961"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql---flexible-server-using-the-azure-portal"></a>Настройка параметров сервера в базе данных Azure для MySQL — гибкого сервера с помощью портал Azure

> [!IMPORTANT] 
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

Вы можете управлять конфигурацией гибкого сервера базы данных Azure для MySQL с помощью параметров сервера. При создании сервера для параметров сервера настраиваются значения по умолчанию и Рекомендуемые.  

В этой статье описывается, как просматривать и настраивать параметры сервера с помощью портал Azure. Колонка параметров сервера на портал Azure показывает как изменяемый, так и неизменяемый параметр сервера. Неизменяемые параметры сервера выделяются серым цветом.

>[!Note]
> Параметры сервера можно обновлять глобально на уровне сервера, использовать [Azure CLI](./how-to-configure-server-parameters-cli.md) или [портал Azure](./how-to-configure-server-parameters-portal.md).

## <a name="configure-server-parameters"></a>Настройка параметров сервера

1. Войдите в [портал Azure](https://portal.azure.com), а затем найдите гибкий сервер базы данных Azure для MySQL.
2. В разделе **Параметры** щелкните **Параметры сервера** , чтобы открыть страницу Параметры сервера для гибкого сервера базы данных Azure для MySQL.
[:::image type="content" source="./media/how-to-server-parameters/azure-portal-server-parameters.png" alt-text="Страница параметров сервера портал Azure":::](./media/how-to-server-parameters/azure-portal-server-parameters.png#lightbox)
3. Нахождение любого параметра сервера, который необходимо настроить. Просмотрите столбец **Описание**, чтобы понять назначение и допустимые значения.
[:::image type="content" source="./media/how-to-server-parameters/3-toggle-parameter.png" alt-text="Перечисление раскрывающегося списка":::](./media/how-to-server-parameters/3-toggle-parameter.png#lightbox)
4. Нажмите кнопку  **сохранить** , чтобы сохранить изменения.
[:::image type="content" source="./media/how-to-server-parameters/4-save-parameters.png" alt-text="Сохранить или отменить изменения":::](./media/how-to-server-parameters/4-save-parameters.png#lightbox)
5. Статические параметры, для которых требуется перезагрузка сервера, вступают в силу. Если вы изменяете статический параметр, вам будет предложено **перезагрузить** компьютер или **перезагрузить компьютер позже**.
[:::image type="content" source="./media/how-to-server-parameters/5-save-parameter.png" alt-text="Перезапустить при сохранении статических параметров":::](./media/how-to-server-parameters/5-save-parameter.png#lightbox)
6. Если вы сохранили новые значения параметров, всегда можно восстановить значения по умолчанию, выбрав **Сбросить все к значениям по умолчанию**.
[:::image type="content" source="./media/how-to-server-parameters/6-reset-parameters.png" alt-text="Сбросить все до значений по умолчанию":::](./media/how-to-server-parameters/6-reset-parameters.png#lightbox)

## <a name="setting-non-modifiable-server-parameters"></a>Задание неизменяемых параметров сервера

Если параметр сервера, который требуется обновить, не является изменяемым, можно при необходимости задать параметр на уровне соединения с помощью `init_connect` . Это задает параметры сервера для каждого клиента, подключающегося к серверу. 

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
> Необходимо перезапустить сервер, чтобы убедиться, что таблицы часовых поясов заполнены правильно.<!-- FIX ME To restart the server, use the [Azure portal](how-to-restart-server-portal.md) or [CLI](how-to-restart-server-cli.md).-->

Чтобы просмотреть доступные значения часового пояса, выполните следующую команду.

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>Установка часового пояса глобального уровня

Часовой пояс глобального уровня можно задать на странице **Параметры сервера** на портале Azure. Ниже приведен пример, который задает глобальный часовой пояс "US/Pacific" (США, Тихоокеанский регион).

[:::image type="content" source="./media/how-to-server-parameters/timezone.png" alt-text="Настройка параметра часового пояса":::](./media/how-to-server-parameters/timezone.png#lightbox)

### <a name="setting-the-session-level-time-zone"></a>Настройка часового пояса уровня сеанса

Часовой пояс уровня сеанса можно задать, выполнив команду `SET time_zone` в командной строке MySQL или MySQL Workbench. В приведенном ниже примере задается часовой пояс **US/Pacific** (США, Тихоокеанский регион).

```sql
SET time_zone = 'US/Pacific';
```

Описание [функций даты и времени](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz) можно прочитать в документации по MySQL.

## <a name="next-steps"></a>Следующие шаги

- Настройка [параметров сервера в Azure CLI](./how-to-configure-server-parameters-cli.md)
