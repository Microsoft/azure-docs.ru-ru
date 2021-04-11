---
title: Краткое руководство. Создание сервера — портал Azure — База данных Azure для PostgreSQL — отдельный сервер
description: В этом кратком руководстве описывается создание сервера Базы данных Azure для PostgreSQL и управление им с помощью портала Azure.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.custom: mvc
ms.topic: quickstart
ms.date: 10/18/2020
ms.openlocfilehash: 6b5b145e84a3844b646ec9adfb46f8a221c6d0a6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105606510"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-server-by-using-the-azure-portal"></a>Краткое руководство. Создание сервера Базы данных Azure для PostgreSQL с помощью портала Azure

База данных Azure для PostgreSQL — это управляемая служба, с помощью которой можно запускать и масштабировать базы данных PostgreSQL высокой доступности, а также управлять ими в облаке. В этом кратком руководстве описывается создание базы данных Azure PostgreSQL с отдельным сервером и подключение к ней.

## <a name="prerequisites"></a>Предварительные требования
Требуется подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/) Azure, прежде чем начинать работу.

## <a name="create-an-azure-database-for-postgresql-server"></a>Создание сервера Базы данных Azure для PostgreSQL
Чтобы создать базу данных Azure для PostgreSQL с отдельным сервером, перейдите на [портал Azure](https://portal.azure.com/). Найдите и выберите *Серверы Базы данных Azure для PostgreSQL*.

>[!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstart-create-database-portal/search-postgres.png" alt-text="Поиск пункта &quot;Серверы Базы данных Azure для PostgreSQL&quot;":::

1. Выберите **Добавить**.

2. На странице Create a Azure Database for PostgreSQL (Создание базы данных Azure для PostgreSQL) выберите **Отдельный сервер**.

    >[!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-database-portal/select-single-server.png" alt-text="Выбор пункта &quot;Отдельный сервер&quot;":::

3. Заполните форму **Основные данные** , указав следующую информацию:

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-database-portal/create-basics.png" alt-text="Снимок экрана, на котором показана вкладка &quot;Основные сведения&quot; для создания одного сервера.":::

   |Параметр|Рекомендуемое значение|Описание|
   |:---|:---|:---|
   |Подписка|Имя вашей подписки|Выберите нужную подписку Azure.|
   |Группа ресурсов|*myresourcegroup*| Имя новой или уже существующей группы ресурсов из подписки.|
   |Имя сервера |*mydemoserver*|Уникальное имя, идентифицирующее базу данных Azure для сервера PostgreSQL. Имя домена *postgres.database.azure.com* добавляется к указанному имени сервера. Сервер может содержать только строчные буквы, цифры и знак дефиса (-). Длина должна составлять от 3 до 63 символов.|
   |Источник данных | None | Чтобы создать сервер с нуля, выберите **Нет**. Выберите **Backup** (Резервная копия), только если восстановление выполняется из геоизбыточной резервной копии существующего сервера.|
   |Имя администратора |*myadmin*| Укажите имя администратора сервера. Оно не должно начинаться с **pg_** . Кроме того, нельзя использовать такие значения: **azure_superuser**, **azure_pg_admin**, **admin**, **administrator**, **root**, **guest** или **public**.|
   |Пароль |Ваш пароль.| Новый пароль для администратора сервера. Должен содержать от 8 до 128 знаков из таких трех категорий: прописные латинские буквы, строчные латинские буквы, цифры (0–9) и небуквенно-цифровые символы (например, !, $, #, %).|
   |Расположение|Нужное расположение| Выберите расположение из раскрывающегося списка.|
   |Версия|Последний основной номер версии| Последний основной номер версии PostgreSQL, если нет особых требований.|
   |Вычисления и хранилище | *Используйте значения по умолчанию*| Ценовая категория по умолчанию — **Общего назначения** с **4 виртуальными ядрами** и хранилищем объемом **100 ГБ**. Для срока хранения резервных копий задано значение **7 дней** с **геоизбыточным** резервным копированием.<br/>Ознакомьтесь со [сведениями о ценах](https://azure.microsoft.com/pricing/details/postgresql/server/) и при необходимости обновите значения по умолчанию.|


   > [!NOTE]
   > Используйте ценовую категорию "Базовый", если для вашей рабочей нагрузки не требуется большое количество вычислительных ресурсов и операций ввода-вывода. Обратите внимание, что серверы, созданные в ценовой категории "Базовый", нельзя масштабировать до ценовых категорий "Общего назначения" или "Оптимизированная для операций в памяти".

5. Выберите **Просмотр и создание** , чтобы просмотреть выбранные элементы. Щелкните **Создать**, чтобы подготовить сервер. Эта операция может занять несколько минут.
    > [!NOTE]
    > Создается пустая база данных **postgres**. Кроме того, будет создана база данных **azure_maintenance**, которая используется для отделения процессов управляемой службы от действий пользователя. Не удается получить доступ к базе данных **azure_maintenance**.

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstart-create-database-portal/deployment-success.png" alt-text="Развертывание успешно завершено":::

[Возникли проблемы? Сообщите нам!](https://aka.ms/postgres-doc-feedback)

## <a name="configure-a-firewall-rule"></a>Настройка правила брандмауэра
По умолчанию создаваемый сервер не является общедоступным. Поэтому для доступа к IP-адресу необходимо предоставить разрешения. Перейдите к ресурсу сервера на портале Azure и выберите **Безопасность подключения** в левой части меню для этого ресурса сервера. Если вы не знаете, как найти ресурс, ознакомьтесь с разделом [Открытие ресурсов](../azure-resource-manager/management/manage-resources-portal.md#open-resources).

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/quickstart-create-database-portal/add-current-ip-firewall.png" alt-text="Снимок экрана, на котором показаны правила брандмауэра для обеспечения безопасности подключения.":::

Выберите **Добавить текущий IP-адрес клиента** и щелкните **Сохранить**. Вы можете добавить дополнительные IP-адреса или указать диапазон адресов, чтобы подключаться к серверу с любого из этих IP-адресов. Дополнительные сведения см. в статье [Правила брандмауэра в Базе данных Azure для PostgreSQL](./concepts-firewall-rules.md).

> [!NOTE]
> Чтобы избежать проблем с подключением, проверьте, разрешена ли в вашей сети передача исходящего трафика через порт 5432. База данных Azure для PostgreSQL использует этот порт.

[Возникли проблемы? Сообщите нам!](https://aka.ms/postgres-doc-feedback)

## <a name="connect-to-the-server-with-psql"></a>Подключение к серверу с помощью psql

Вы можете использовать популярные клиенты для PostgreSQL: [psql](http://postgresguide.com/utilities/psql.html) или [pgAdmin](https://www.pgadmin.org/docs/pgadmin4/latest/connecting.html). В этом кратком руководстве предполагается, что для подключения используется psql в [Azure Cloud Shell](../cloud-shell/overview.md) на портале Azure.

1. Запишите имя нового сервера, имя администратора этого сервера, пароль и идентификатор подписки из раздела **Обзор**.
    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/quickstart-create-database-portal/overview-new.png" alt-text="Получение сведений о подключении":::


2. Откройте Azure Cloud Shell на портале, щелкнув значок в верхней левой части.

   > [!NOTE]
   > Если вы открываете Cloud Shell первый раз, отобразится запрос на создание группы ресурсов и учетной записи хранения. Это одноразовое действие, которое автоматически применяется для всех сеансов.

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="media/quickstart-create-database-portal/use-in-cloud-shell.png" alt-text="Снимок экрана, на котором показаны сведения о сервере и значок для открытия Azure Cloud Shell.":::

3. Воспользуйтесь следующей командой в терминале Azure Cloud Shell. Замените предложенные значения реальными именами сервера и администратора. Используйте пустую базу данных **postgres** от имени администратора в следующем формате: `<admin-username>@<servername>`.

   ```azurecli-interactive
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

   Ниже показано, как выглядит этот процесс в терминале Cloud Shell.

   ```bash
    Requesting a Cloud Shell.Succeeded.
    Connecting terminal...

    Welcome to Azure Cloud Shell

    Type "az" to use Azure CLI
    Type "help" to learn about Cloud Shell

    user@Azure:~$psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
    Password for user myadmin@mydemoserver.postgres.database.azure.com:
    psql (12.2 (Ubuntu 12.2-2.pgdg16.04+1), server 11.6)
    SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
    Type "help" for help.

    postgres=>
    ```
4. В том же терминале Azure Cloud Shell создайте базу данных с именем **guest**.

   ```bash
   postgres=> CREATE DATABASE guest;
   ```

5. Переключите текущее подключение на новую базу данных **guest**.

   ```bash
   \c guest
   ```
6. Введите `\q` и нажмите клавишу ВВОД, чтобы закрыть psql.

[Возникли проблемы? Сообщите нам!](https://aka.ms/postgres-doc-feedback)

## <a name="clean-up-resources"></a>Очистка ресурсов
Вы успешно создали сервер Базы данных Azure для PostgreSQL в группе ресурсов. Если вы ожидаете, что в будущем эти ресурсы вам не понадобятся, их можно удалить, удалив группу ресурсов или сам сервер PostgreSQL.

Чтобы удалить группу ресурсов, сделайте следующее:

1. Войдите на портал Azure; найдите в поиске и выберите **Группы ресурсов**.
2. В списке групп ресурсов выберите имя нужной группы ресурсов.
3. На странице **Обзор** для группы ресурсов выберите **Удалить группу ресурсов**.
4. В диалоговом окне подтверждения введите имя группы ресурсов и щелкните **Удалить**.

Чтобы удалить сервер, нажмите кнопку **Удалить** на его странице **Обзор**.

> [!div class="mx-imgBorder"]
> :::image type="content" source="media/quickstart-create-database-portal/12-delete.png" alt-text="Снимок экрана, на котором показана кнопка для удаления сервера.":::

[Возникли проблемы? Сообщите нам!](https://aka.ms/postgres-doc-feedback)

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Перенос базы данных с помощью экспорта и импорта](./howto-migrate-using-export-and-import.md) <br/>

> [!div class="nextstepaction"]
> [Проектирование базы данных](./tutorial-design-database-using-azure-portal.md#create-tables-in-the-database)

[Не можете найти нужную информацию? Сообщите нам!](https://aka.ms/postgres-doc-feedback)