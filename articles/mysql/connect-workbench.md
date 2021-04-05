---
title: Краткое руководство. Подключение к Базе данных Azure для MySQL с помощью MySQL Workbench
description: В этом кратком руководстве описывается, как использовать MySQL Workbench для подключения к базе данных Azure для MySQL и запроса данных из нее.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 5/26/2020
ms.openlocfilehash: 5e27cfec0a3f0a58c1e94a822e0c831f4efa1b32
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94535543"
---
# <a name="quickstart-use-mysql-workbench-to-connect-and-query-data-in-azure-database-for-mysql"></a>Краткое руководство. Подключение к Базе данных Azure для MySQL и запрос данных с помощью MySQL Workbench

В этом кратком руководстве показано, как подключиться к базе данных Azure для MySQL с помощью приложения MySQL Workbench.

## <a name="prerequisites"></a>Предварительные требования

В качестве отправной точки в этом кратком руководстве используются ресурсы, созданные в соответствии со следующими материалами:
- [Create an Azure Database for MySQL server using Azure portal](./quickstart-create-mysql-server-database-using-azure-portal.md) (Создание базы данных Azure для сервера MySQL с помощью портала Azure)
- [Краткое руководство. Создание сервера базы данных Azure для MySQL с помощью Azure CLI](./quickstart-create-mysql-server-database-using-azure-cli.md)

> [!IMPORTANT] 
> Убедитесь, что IP-адрес, с которого вы подключаетесь, добавлен в правила брандмауэра на сервере через [портал Azure](./howto-manage-firewall-using-portal.md) или [Azure CLI](./howto-manage-firewall-using-cli.md).

## <a name="install-mysql-workbench"></a>Установка MySQL Workbench
Скачайте и установите MySQL Workbench на свой компьютер с [веб-сайта MySQL](https://dev.mysql.com/downloads/workbench/).

## <a name="get-connection-information"></a>Получение сведений о подключении
Получите сведения о подключении, необходимые для подключения к базе данных Azure.для MySQL. Вам потребуется полное имя сервера и учетные данные для входа.

1. Войдите на [портал Azure](https://portal.azure.com/).

2. В меню слева на портале Azure щелкните **Все ресурсы** и выполните поиск по имени созданного сервера (например, **mydemoserver**).

3. Щелкните имя сервера.

4. Запишите **имя сервера** и **имя для входа администратора сервера** с панели сервера **Обзор**. Если вы забыли свой пароль, можно также сбросить пароль с помощью этой панели.
 :::image type="content" source="./media/connect-php/1_server-overview-name-login.png" alt-text="Имя сервера базы данных Azure для MySQL":::

## <a name="connect-to-the-server-by-using-mysql-workbench"></a>Подключение к серверу с помощью MySQL Workbench 
Чтобы подключиться к серверу MySQL Azure с помощью инструмента графического интерфейса MySQL Workbench, сделайте следующее.

1.    Запустите приложение MySQL Workbench на своем компьютере. 

2.    В диалоговом окне **настройки нового подключения** на вкладке **Параметры** введите следующие сведения:

:::image type="content" source="./media/connect-workbench/2-setup-new-connection.png" alt-text="Настройка нового подключения":::

| **Параметр** | **Рекомендуемое значение** | **Описание поля** |
|---|---|---|
|    Имя подключения | Пример подключения | Укажите метку для этого подключения. |
| Способ подключения | Стандартный способ (по протоколу TCP/IP) | Стандартный способ (по протоколу TCP/IP) соответствует требованиям. |
| Имя узла | *имя сервера* | Укажите значение имени сервера, которое вы использовали ранее при создании базы данных Azure для MySQL. В нашем примере используется такое имя сервера: mydemoserver.mysql.database.azure.com. Используйте полное доменное имя (\*.mysql.database.azure.com), как показано в примере. Выполните действия из предыдущего раздела, чтобы найти сведения для подключения, если вы не помните имя своего сервера.  |
| Порт | 3306 | Всегда используйте порт 3306 при подключении к базе данных Azure для MySQL. |
| Имя пользователя |  *имя для входа администратора сервера* | Введите имя администратора сервера, которое вы использовали ранее при создании базы данных Azure для MySQL. Пример нашего имени пользователя — myadmin@mydemoserver. Выполните действия из предыдущего раздела, чтобы найти сведения для подключения, если вы не помните имя пользователя. Формат *имя пользователя\@имя сервера*.
| Пароль | Ваш пароль. | Нажмите кнопку **Store in Vault...** (Сохранить в хранилище), чтобы сохранить пароль. |

3.   Щелкните **Проверить подключение**, чтобы проверить, все ли параметры верно настроены. 

4.   Нажмите кнопку **ОК**, чтобы сохранить подключение. 

5.   В списке **подключений MySQL** щелкните элемент, соответствующий вашему серверу, и ожидайте установления подключения.

        Откроется новая вкладка SQL с пустым окном редактора, в котором можно вводить запросы.
    
        > [!NOTE]
        > По умолчанию защита SSL-подключения является обязательной и применяется к базе данных Azure для сервера MySQL. Обычно для подключения MySQL Workbench к вашему серверу не требуется никаких дополнительных настроек с использованием сертификатов SSL. Но мы рекомендуем привязать SSL-сертификат ЦС к MySQL Workbench. Дополнительные сведения о скачивании и привязке сертификатов см. в статье [Настройка SSL-подключений в приложении для безопасного подключения к базе данных Azure для MySQL](./howto-configure-ssl.md).  Если необходимо отключить протокол SSL, войдите на портал Azure и перейдите на страницу "Безопасность подключения", чтобы отключить выключатель "Принудительно использовать SSL-соединение".

## <a name="create-a-table-insert-data-read-data-update-data-delete-data"></a>Создание таблицы, добавление, считывание, обновление и удаление данных
1. Скопируйте и вставьте пример кода SQL в пустую вкладку SQL для иллюстрации примера данных.

    Этот код создает пустую базу данных с именем quickstartdb, а затем создает пример таблицы с именем inventory. Он добавляет несколько строк, а затем считывает эти строки. Он изменяет данные с помощью инструкции update, а затем еще раз считывает строки. Наконец он удаляет одну строку и еще раз считывает строки.
    
    ```sql
    -- Create a database
    -- DROP DATABASE IF EXISTS quickstartdb;
    CREATE DATABASE quickstartdb;
    USE quickstartdb;
    
    -- Create a table and insert rows
    DROP TABLE IF EXISTS inventory;
    CREATE TABLE inventory (id serial PRIMARY KEY, name VARCHAR(50), quantity INTEGER);
    INSERT INTO inventory (name, quantity) VALUES ('banana', 150);
    INSERT INTO inventory (name, quantity) VALUES ('orange', 154);
    INSERT INTO inventory (name, quantity) VALUES ('apple', 100);
    
    -- Read
    SELECT * FROM inventory;
    
    -- Update
    UPDATE inventory SET quantity = 200 WHERE id = 1;
    SELECT * FROM inventory;
    
    -- Delete
    DELETE FROM inventory WHERE id = 2;
    SELECT * FROM inventory;
    ```

    На снимке экрана показан пример кода SQL в SQL Workbench и выходные данные после его выполнения.
    
    :::image type="content" source="media/connect-workbench/3-workbench-sql-tab.png" alt-text="MySQL Workbench: вкладка SQL для выполнения примера кода SQL":::

2. Чтобы выполнить пример кода SQL, щелкните значок молнии на панели инструментов вкладки **SQL File** (Файл SQL).
3. Обратите внимание на три вкладки с результатами, которые отображаются в разделе **Сетка результатов** посередине страницы. 
4. Обратите внимание на список **Output** (Выходные данные) внизу страницы. Показано состояние каждой команды. 

Вы подключились к базе данных Azure для MySQL с помощью MySQL Workbench и запросили данные с помощью языка SQL.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы очистить все ресурсы, используемые во время этого краткого руководства, удалите группу ресурсов с помощью следующей команды:

```azurecli
az group delete \
    --name $AZ_RESOURCE_GROUP \
    --yes
```

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Перенос базы данных с помощью экспорта и импорта](./concepts-migrate-import-export.md)
