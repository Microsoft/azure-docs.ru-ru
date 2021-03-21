---
title: Подключение с перенаправлением — база данных Azure для MariaDB
description: В этой статье описывается, как настроить приложение для подключения к базе данных Azure для MariaDB с перенаправлением.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 6/8/2020
ms.openlocfilehash: 3f26de72839fcaa39bff4d827aba757721736934
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98664908"
---
# <a name="connect-to-azure-database-for-mariadb-with-redirection"></a>Подключение к базе данных Azure для MariaDB с перенаправлением

В этом разделе объясняется, как подключить приложение к серверу базы данных Azure для MariaDB с режимом перенаправления. Перенаправление нацелено на сокращение задержки в сети между клиентскими приложениями и серверами MariaDB, позволяя приложениям подключаться непосредственно к внутренним серверам.

## <a name="before-you-begin"></a>Перед началом
Войдите на [портал Azure](https://portal.azure.com). Создайте сервер базы данных Azure для MariaDB с ядром версии 10,2 или 10,3. 

Дополнительные сведения см. в статье Создание сервера базы данных Azure для MariaDB с помощью [портал Azure](quickstart-create-mariadb-server-database-using-azure-portal.md) или [Azure CLI](quickstart-create-mariadb-server-database-using-azure-cli.md).

## <a name="enable-redirection"></a>Включить перенаправление

На сервере базы данных Azure для MariaDB задайте для `redirect_enabled` параметра значение, `ON` чтобы разрешить подключения с помощью режима перенаправления. Чтобы обновить этот параметр сервера, используйте [портал Azure](howto-server-parameters.md) или [Azure CLI](howto-configure-server-parameters-cli.md).

## <a name="php"></a>PHP

Поддержка перенаправления в приложениях PHP доступна через расширение [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure), разработанное корпорацией Майкрософт. 

Расширение mysqlnd_azure можно добавить в приложения PHP через PECL. Мы настоятельно рекомендуем устанавливать и настраивать это расширение с помощью официально опубликованного [пакета PECL](https://pecl.php.net/package/mysqlnd_azure).

> [!IMPORTANT]
> Поддержка перенаправления в расширении PHP [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure) в настоящее время доступна в предварительной версии.

### <a name="redirection-logic"></a>Логика перенаправления

>[!IMPORTANT]
> Логика и поведение перенаправления изменились начиная с версии 1.1.0, и мы **рекомендуем использовать версию 1.1.0 или выше**.

Поведение перенаправления определяется значением `mysqlnd_azure.enableRedirect`. В следующей таблице описано поведение перенаправления в зависимости от этого значения, как оно реализовано **в версии 1.1.0 и более поздних**.

Если вы используете более старую версию расширения mysqlnd_azure (с 1.0.0 по 1.0.3), поведение перенаправления определяется значением `mysqlnd_azure.enabled`. Допустимые значения: `off` (аналогично описанному в таблице ниже) и `on` (аналогично значению `preferred` в таблице ниже).  

|**значение mysqlnd_azure.enableRedirect**| **Поведение**|
|----------------------------------------|-------------|
|`off` либо `0`|Перенаправление не будет использоваться. |
|`on` либо `1`|– Если для подключения не используется протокол SSL на стороне драйвера, оно не будет установлено. Будет возвращена следующая ошибка: *"mysqlnd_azure.enableRedirect is on, but SSL option is not set in connection string. Redirection is only possible with SSL."* (Расширение mysqlnd_azure.enableRedirect включено, но в строке подключения не указано использование SSL. Перенаправление возможно только при использовании SSL.)<br>— Если на стороне драйвера используется протокол SSL, но перенаправление не поддерживается на сервере, первое подключение прерывается и возвращается следующая ошибка: *"соединение прервано, так как перенаправление не включено на сервере MariaDB, или сетевой пакет не соответствует протоколу перенаправления".*<br>— Если сервер MariaDB поддерживает перенаправление, но по какой-либо причине перенаправленное соединение не удалось, также прерывается первое подключение прокси. При этом возвращается ошибка о перенаправленном подключении.|
|`preferred` либо `2`<br> (значение по умолчанию)|– mysqlnd_azure будет использовать перенаправление, если возможно.<br>– Если при подключении не используется протокол SSL на стороне драйвера, сервер не поддерживает перенаправление или перенаправленное подключение невозможно установить по любой устранимой причине, но при этом первое прокси-подключение остается допустимым, оно будет использоваться как резервное.|

В последующих разделах документа описано, как установить расширение `mysqlnd_azure` с помощью PECL и задать значение этого параметра.

### <a name="ubuntu-linux"></a>Ubuntu Linux

#### <a name="prerequisites"></a>Предварительные требования 
- Версия PHP 7.2.15 и выше или 7.3.2 и выше
- PHP PEAR 
- php-mysql
- Сервер базы данных Azure для MariaDB

1. Установите [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure) с помощью [PECL](https://pecl.php.net/package/mysqlnd_azure). Мы рекомендуем использовать версию 1.1.0 или выше.

    ```bash
    sudo pecl install mysqlnd_azure
    ```

2. Найдите каталог расширений (`extension_dir`), выполнив следующую команду:

    ```bash
    php -i | grep "extension_dir"
    ```

3. Перейдите в папку, путь к которой только что получили, и проверьте наличие в ней файла `mysqlnd_azure.so`. 

4. Найдите папку для INI-файлов, выполнив следующую команду: 

    ```bash
    php -i | grep "dir for additional .ini files"
    ```

5. Перейдите в эту папку. 

6. Создайте новый INI-файл для `mysqlnd_azure`. Убедитесь, что имя этого файла в отсортированном по алфавиту списке располагается следом за файлом для mysqnld, так как модули загружаются в порядке имен INI-файлов. Например, если INI-файл для `mysqlnd` имеет имя `10-mysqlnd.ini`, переименуйте mysqlnd ini в `20-mysqlnd-azure.ini`.

7. В новый INI-файл добавьте следующие строки, чтобы включить перенаправление.

    ```bash
    extension=mysqlnd_azure
    mysqlnd_azure.enableRedirect = on/off/preferred
    ```

### <a name="windows"></a>Windows

#### <a name="prerequisites"></a>Предварительные требования 
- Версия PHP 7.2.15 и выше или 7.3.2 и выше
- php-mysql
- Сервер базы данных Azure для MariaDB

1. Чтобы узнать используемую версию PHP (x64 или x86), выполните следующую команду:

    ```cmd
    php -i | findstr "Thread"
    ```

2. Скачайте версию библиотеки DLL [mysqlnd_azure](https://github.com/microsoft/mysqlnd_azure) для платформы x64 или x86 из [PECL](https://pecl.php.net/package/mysqlnd_azure) в соответствии с используемой версией PHP. Мы рекомендуем использовать версию 1.1.0 или выше.

3. Извлеките файлы из ZIP-архива и найдите библиотеку DLL с именем `php_mysqlnd_azure.dll`.

4. Найдите каталог расширений (`extension_dir`), выполнив следующую команду:

    ```cmd
    php -i | find "extension_dir"
    ```

5. Скопируйте файл `php_mysqlnd_azure.dll` в каталог, полученный на шаге 4. 

6. Найдите папку PHP, которая содержит файл `php.ini`, выполнив следующую команду:

    ```cmd
    php -i | find "Loaded Configuration File"
    ```

7. Измените файл `php.ini`, добавив в него следующие строки, чтобы включить перенаправление. 

    В разделе Dynamic Extensions: 
    ```cmd
    extension=mysqlnd_azure
    ```
    
    В разделе Module Settings:     
    ```cmd 
    [mysqlnd_azure]
    mysqlnd_azure.enableRedirect = on/off/preferred
    ```

### <a name="confirm-redirection"></a>Проверка перенаправления

Проверить правильность настройки перенаправления можно с помощью приведенного ниже примера кода PHP. Создайте PHP-файл с именем `mysqlConnect.php` и вставьте в него приведенный ниже код. Замените имя сервера, имя пользователя и пароль собственными значениями. 
 
 ```php
<?php
$host = '<yourservername>.mariadb.database.azure.com';
$username = '<yourusername>@<yourservername>';
$password = '<yourpassword>';
$db_name = 'testdb';
  echo "mysqlnd_azure.enableRedirect: ", ini_get("mysqlnd_azure.enableRedirect"), "\n";
  $db = mysqli_init();
  //The connection must be configured with SSL for redirection test
  $link = mysqli_real_connect ($db, $host, $username, $password, $db_name, 3306, NULL, MYSQLI_CLIENT_SSL);
  if (!$link) {
     die ('Connect error (' . mysqli_connect_errno() . '): ' . mysqli_connect_error() . "\n");
  }
  else {
    echo $db->host_info, "\n"; //if redirection succeeds, the host_info will differ from the hostname you used used to connect
    $res = $db->query('SHOW TABLES;'); //test query with the connection
    print_r ($res);
    $db->close();
  }
?>
 ```

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о строках подключения см. в [этой статье](howto-connection-string.md).
