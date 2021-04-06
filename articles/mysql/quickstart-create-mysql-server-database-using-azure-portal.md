---
title: Краткое руководство. Создание сервера Базы данных Azure для MySQL с помощью портала Azure
description: В этой статье описывается создание примера базы данных Azure для сервера MySQL с помощью портала Azure за 5 минут.
author: savjani
ms.author: pariks
ms.service: mysql
ms.custom: mvc
ms.topic: quickstart
ms.date: 11/04/2020
ms.openlocfilehash: b8f3a8e06bb27051417205c4dc3141948960bb00
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98567800"
---
# <a name="quickstart-create-an-azure-database-for-mysql-server-by-using-the-azure-portal"></a>Краткое руководство. Создание базы данных Azure для сервера MySQL с помощью портала Azure

База данных Azure для MySQL — это управляемая служба, которая позволяет запускать, администрировать и масштабировать в облаке базы данных MySQL с высоким уровнем доступности. В этом кратком руководстве описывается создание отдельного сервера Базы данных Azure для MySQL на портале Azure. Здесь также приведены сведения о подключении к серверу.

## <a name="prerequisites"></a>Предварительные требования
Требуется подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/) Azure, прежде чем начинать работу.

## <a name="create-an-azure-database-for-mysql-single-server"></a>Создание отдельного сервера Базы данных Azure для MySQL
1. Чтобы создать отдельный сервер Базы данных Azure для MySQL, перейдите на [портал Azure](https://portal.azure.com/). Найдите и выберите **База данных Azure для MySQL**:

   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/find-azure-mysql-in-portal.png" alt-text="Поиск Базы данных Azure для MySQL":::

1. Выберите **Добавить**.

2. На странице **Select Azure Database for MySQL deployment option** (Выберите вариант развертывания Базы данных Azure для MySQL) выберите **Отдельный сервер**:
   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/choose-singleserver.png" alt-text="Снимок экрана: параметр &quot;Отдельный сервер&quot;.":::

3. Введите основные параметры для нового отдельного сервера:

   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/4-create-form.png" alt-text="Снимок экрана: страница создания сервера MySQL.":::

   **Параметр** | **Рекомендуемое значение** | **Описание**
   ---|---|---
   Подписка | Ваша подписка | Выберите нужную подписку Azure.
   Группа ресурсов | **myresourcegroup** | Введите новую группу ресурсов или уже имеющуюся из подписки.
   Имя сервера | **mydemoserver** | Укажите уникальное имя. Имя сервера может содержать только строчные буквы, цифры и знак дефиса (-). Длина должна составлять от 3 до 63 символов.
   Источник данных |**None** | Чтобы создать сервер с нуля, выберите **Нет**. Выберите **Резервная копия**, только если восстановление выполняется из геоизбыточной резервной копии существующего сервера.
   Расположение |Нужное расположение | Выберите расположение из списка.
   Версия | Последний основной номер версии| Используйте последний основной номер версии. См. [все поддерживаемые версии](concepts-supported-versions.md).
   Вычисления и хранилище | Используются значения по умолчанию| Ценовая категория по умолчанию — **Общего назначения** с **4 виртуальными ядрами** и хранилищем объемом **100 ГБ**. Для срока хранения резервных копий задано значение **7 дней** с **геоизбыточным** резервным копированием.<br/>Просмотрите страницу [цен](https://azure.microsoft.com/pricing/details/mysql/) и при необходимости обновите значения по умолчанию.
   Имя администратора | **mydemoadmin** | Укажите имя администратора сервера. Не используйте для имени учетной записи администратора такие варианты: **azure_superuser**, **admin**, **administrator**, **root**, **guest** или **public**.
   Пароль | Пароль. | Новый пароль для администратора сервера. Длина пароля должна составлять 8–128 символов с комбинацией прописных или строчных букв, цифр и символов, отличных от буквенно-цифровых (!, $, #, % и т. д.).
  

   > [!NOTE]
   > Используйте ценовую категорию "Базовый", если для вашей рабочей нагрузки не требуется большое количество вычислительных ресурсов и операций ввода-вывода. Обратите внимание, что серверы, созданные в ценовой категории "Базовый", нельзя масштабировать до ценовых категорий "Общего назначения" или "Оптимизированная для операций в памяти".

4. Щелкните **Просмотр и создание**, чтобы подготовить сервер.

5. Дождитесь, пока на странице портала появится сообщение **Развертывание выполнено**. Выберите **Перейти к ресурсу**, чтобы открыть созданную страницу сервера:

   > [!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/deployment-complete.png" alt-text="Снимок экрана с сообщением &quot;Развертывание выполнено&quot;.":::

[Возникли проблемы? Сообщите нам!](https://aka.ms/mysql-doc-feedback)

## <a name="configure-a-server-level-firewall-rule"></a>Настройка правила брандмауэра на уровне сервера

По умолчанию новый сервер защищен брандмауэром. Чтобы выполнить подключение, необходимо предоставить доступ к IP-адресу, выполнив следующие действия:

1. Перейдите в раздел **Безопасность подключения** в левой области ресурса сервера. Если вы не знаете, как найти ресурс, см. раздел об [открытии ресурса](../azure-resource-manager/management/manage-resources-portal.md#open-resources).

   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/add-current-ip-firewall.png" alt-text="Снимок экрана: раздел &quot;Безопасность подключения&quot; на странице &quot;Правила брандмауэра&quot;.":::

2. Выберите **Добавить текущий IP-адрес клиента** и щелкните **Сохранить**.

   > [!NOTE]
   > Чтобы избежать проблем с подключением, проверьте, разрешает ли сеть исходящий трафик на порт 3306, который использует служба "База данных Azure для MySQL". 

Вы можете добавить дополнительные IP-адреса или указать диапазон адресов, чтобы подключаться к серверу с любого из этих IP-адресов. Дополнительные сведения см. в статье об [управлении правилами брандмауэра для сервера Базы данных Azure для MySQL](./concepts-firewall-rules.md).


[Возникли проблемы? Сообщите нам об этом](https://aka.ms/mysql-doc-feedback)

## <a name="connect-to-the-server-by-using-mysqlexe"></a>Подключение к серверу с помощью mysql.exe
Вы можете использовать [mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) или [MySQL Workbench](./connect-workbench.md), чтобы подключиться к серверу из локальной среды. В этом кратком руководстве показано, как использовать mysql.exe в [Azure Cloud Shell](../cloud-shell/overview.md) для подключения к серверу.


1. Откройте Azure Cloud Shell на портале, нажав первую кнопку на панели инструментов, как показано на следующем снимке экрана. Запишите имя нового сервера, имя администратора этого сервера и подписку для сервера из раздела **Обзор**, как показано на снимке экрана.

    > [!NOTE]
    > Если вы открываете Cloud Shell первый раз, отобразится запрос на создание группы ресурсов и учетной записи хранения. Это одноразовое действие, которое автоматически применяется для всех сеансов.

   >[!div class="mx-imgBorder"]
   > :::image type="content" source="./media/quickstart-create-mysql-server-database-using-azure-portal/use-in-cloud-shell.png" alt-text="Снимок экрана: Cloud Shell на портале Azure.":::
2. Воспользуйтесь следующей командой в терминале Azure Cloud Shell. Замените показанные здесь значения реальными именами сервера и администратора. Для службы "База данных Azure для MySQL" к имени администратора требуется добавить `@\<servername>`, как показано ниже: 

      ```azurecli-interactive
      mysql --host=mydemoserver.mysql.database.azure.com --user=myadmin@mydemoserver -p
      ```

      Вот так выглядит этот процесс в терминале Cloud Shell:

      ```
      Requesting a Cloud Shell.Succeeded.
      Connecting terminal...

      Welcome to Azure Cloud Shell

      Type "az" to use Azure CLI
      Type "help" to learn about Cloud Shell

      user@Azure:~$mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
      Enter password:
      Welcome to the MySQL monitor.  Commands end with ; or \g.
      Your MySQL connection id is 64796
      Server version: 5.6.42.0 Source distribution

      Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.
      Oracle is a registered trademark of Oracle Corporation and/or its
      affiliates. Other names may be trademarks of their respective
      owners.

      Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
      mysql>
      ```
3. В том же терминале Azure Cloud Shell создайте базу данных с именем `guest`:
      ```
      mysql> CREATE DATABASE guest;
      Query OK, 1 row affected (0.27 sec)
      ```
4. Переключитесь на базу данных `guest`:
      ```
      mysql> USE guest;
      Database changed
      ```
5. Введите `quit` и нажмите клавишу **ВВОД**, чтобы выйти из mysql.

[Возникли проблемы? Сообщите нам!](https://aka.ms/mysql-doc-feedback)

## <a name="clean-up-resources"></a>Очистка ресурсов
Вы создали сервер службы "База данных Azure для MySQL" в группе ресурсов.  Если вы ожидаете, что в будущем эти ресурсы вам не понадобятся, их можно удалить, удалив группу ресурсов или сам сервер MySQL. Чтобы удалить группу ресурсов, выполните следующие шаги.
1. Войдите на портал Azure; найдите в поиске и выберите **Группы ресурсов**.
2. В списке групп ресурсов выберите имя группы ресурсов.
3. На странице **Обзор** для группы ресурсов выберите **Удалить группу ресурсов**.
4. В диалоговом окне подтверждения введите имя группы ресурсов и щелкните **Удалить**.

Чтобы удалить сервер, нажмите кнопку **Удалить** на странице **Обзор** сервера, как показано ниже:
> [!div class="mx-imgBorder"]
> :::image type="content" source="media/quickstart-create-mysql-server-database-using-azure-portal/delete-server.png" alt-text="Снимок экрана: кнопка &quot;Удалить&quot; на странице обзора сервера.":::

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
>[Создание приложения PHP с подключением к MySQL (Windows)](../app-service/tutorial-php-mysql-app.md) <br/>

> [!div class="nextstepaction"]
>[Создание приложение PHP в Linux с MySQL](../app-service/tutorial-php-mysql-app.md?pivots=platform-linux%3fpivots%3dplatform-linux)<br/><br/>

[Не можете найти нужную информацию? Сообщите нам!](https://aka.ms/mysql-doc-feedback)
