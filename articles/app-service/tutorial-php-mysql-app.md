---
title: Руководство по приложению PHP с MySQL
description: Узнайте, как создать приложение PHP, работающее в Azure, с подключением к базе данных MySQL в Azure. В этом руководстве используется Laravel.
ms.assetid: 14feb4f3-5095-496e-9a40-690e1414bd73
ms.devlang: php
ms.topic: tutorial
ms.date: 06/15/2020
ms.custom: mvc, cli-validate, seodec18, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: 0810f023f4e2e192f2cb0d83f2a028cdded9e275
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107779476"
---
# <a name="tutorial-build-a-php-and-mysql-app-in-azure-app-service"></a>Руководство по Разработка приложения на основе PHP и MySQL в Службе приложений Azure

::: zone pivot="platform-windows"  

[Служба приложений Azure](overview.md) — это высокомасштабируемая служба размещения с самостоятельной установкой исправлений на основе операционной системы Windows. В этом руководстве показано, как создать приложение PHP в Azure и подключить его к базе данных MySQL. По завершении вы получите приложение [Laravel](https://laravel.com/), работающее в Службе приложений Azure в Windows.

::: zone-end

::: zone pivot="platform-linux"

[Служба приложений Azure](overview.md) — это высокомасштабируемая служба размещения с самостоятельной установкой исправлений на основе операционной системы Linux. В этом руководстве показано, как создать приложение PHP в Azure и подключить его к базе данных MySQL. По завершении вы получите приложение [Laravel](https://laravel.com/), работающее в Службе приложений Azure в Linux.

::: zone-end

:::image type="content" source="./media/tutorial-php-mysql-app/complete-checkbox-published.png" alt-text="Снимок экрана: пример приложения PHP с названием Task List.":::

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание базы данных MySQL в Azure.
> * Подключение приложения PHP к MySQL.
> * Развертывание приложения в Azure
> * Обновление модели данных и повторное развертывание приложения.
> * Потоковая передача журналов диагностики из Azure.
> * Управление приложением на портале Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

- [установите Git](https://git-scm.com/);
- [PHP 5.6.4 или более поздней версии](https://php.net/downloads.php);
- [Composer](https://getcomposer.org/doc/00-intro.md);
- Включите следующие расширения PHP, необходимые для Laravel: OpenSSL, PDO-MySQL, Mbstring, Tokenizer и XML.
- [Установка и запуск MySQL](https://dev.mysql.com/doc/refman/5.7/en/installing.html)
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)] 

## <a name="prepare-local-mysql"></a>Подготовка локальной базы данных MySQL

На этом шаге вы создадите базу данных на локальном сервере MySQL для использования в этом руководстве.

### <a name="connect-to-local-mysql-server"></a>Подключение к локальному серверу MySQL

В окне терминала подключитесь к локальному серверу MySQL. Используйте это окно терминала для выполнения всех команд в рамках этого руководства.

```bash
mysql -u root -p
```

Если появится предложение ввести пароль, введите пароль для учетной записи `root`. Если вы не помните пароль учетной записи привилегированного пользователя, ознакомьтесь с разделом [MySQL: How to Reset the Root Password](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html) (MySQL: как сбросить пароль привилегированного пользователя).

Если команда выполнена успешно, значит, сервер MySQL работает. В противном случае обеспечьте запуск локального сервера MySQL, выполнив соответствующие [действия после установки MySQL](https://dev.mysql.com/doc/refman/5.7/en/postinstallation.html).

### <a name="create-a-database-locally"></a>Локальное создание базы данных

В командной строке `mysql` создайте базу данных.

```sql 
CREATE DATABASE sampledb;
```

Завершите подключение к серверу, введя команду `quit`.

```sql
quit
```

<a name="step2"></a>

## <a name="create-a-php-app-locally"></a>Локальное создание приложения PHP
На этом шаге вы создадите пример приложения Laravel, настроите его подключение к базе данных и запустите это приложение в локальной среде. 

### <a name="clone-the-sample"></a>Клонирования репозитория

Откройте окно терминала и c помощью команды `cd` перейдите в рабочий каталог.

Выполните команду ниже, чтобы клонировать репозиторий с примером.

```bash
git clone https://github.com/Azure-Samples/laravel-tasks
```

С помощью команды `cd` перейдите в клонированный каталог.
Установите необходимые пакеты.

```bash
cd laravel-tasks
composer install
```

### <a name="configure-mysql-connection"></a>Настройка подключения к MySQL

В корне репозитория создайте файл *.env*. Скопируйте в файл *.env* приведенные ниже переменные. Замените заполнитель _&lt;root_password >_ паролем привилегированного пользователя MySQL.

```txt
APP_ENV=local
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=sampledb
DB_USERNAME=root
DB_PASSWORD=<root_password>
```

Сведения о том, как Laravel использует файл _.env_, см. в разделе [Environment Configuration](https://laravel.com/docs/5.4/configuration#environment-configuration) (Конфигурация среды).

### <a name="run-the-sample-locally"></a>Локальный запуск примера

Выполните [перенос базы данных Laravel](https://laravel.com/docs/5.4/migrations), чтобы создать таблицы, необходимые для приложения. Чтобы узнать, какие таблицы создаются при переносе, просмотрите каталог _database/migrations_ в этом репозитории Git.

```bash
php artisan migrate
```

Создайте ключ приложения Laravel.

```bash
php artisan key:generate
```

Запустите приложение.

```bash
php artisan serve
```

Откройте браузер и перейдите по адресу `http://localhost:8000`. Добавьте несколько задач на странице.

![Приложение PHP успешно подключается к базе данных MySQL](./media/tutorial-php-mysql-app/mysql-connect-success.png)

Чтобы остановить приложение PHP, введите `Ctrl + C` в окне терминала.

## <a name="create-mysql-in-azure"></a>Создание базы данных MySQL в Azure

На этом шаге вы создадите базу данных MySQL в [Базе данных Azure для MySQL](../mysql/index.yml). Позже вы настроите приложение PHP для подключения к этой базе данных.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)] 

### <a name="create-a-mysql-server"></a>Создание сервера MySQL

В Cloud Shell создайте сервер в службе "База данных Azure для MySQL", выполнив команду [`az mysql server create`](/cli/azure/mysql/server#az_mysql_server_create).

В следующей команде заполнитель *\<mysql-server-name>* замените уникальным именем сервера, а заполнители *\<admin-user>* и *\<admin-password>*  — именем пользователя и паролем. Это имя используется как часть конечной точки MySQL (`https://<mysql-server-name>.mysql.database.azure.com`), поэтому оно должно быть уникальным на всех серверах в Azure. Дополнительные сведения о выборе номера SKU базы данных MySQL см. в разделе о [создании сервера Базы данных Azure для MySQL](../mysql/quickstart-create-mysql-server-database-using-azure-cli.md#create-an-azure-database-for-mysql-server).

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql-server-name> --location "West Europe" --admin-user <admin-user> --admin-password <admin-password> --sku-name B_Gen5_1
```

После создания сервера MySQL в Azure CLI отображаются следующие сведения.

<pre>
{
  "administratorLogin": "&lt;admin-user&gt;",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "&lt;mysql-server-name&gt;.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/&lt;mysql-server-name&gt;",
  "location": "westeurope",
  "name": "&lt;mysql-server-name&gt;",
  "resourceGroup": "myResourceGroup",
  ...
  -  &lt; Output has been truncated for readability &gt;
}
</pre>

### <a name="configure-server-firewall"></a>Настройка брандмауэра сервера

В Cloud Shell создайте правило брандмауэра для сервера MySQL, чтобы разрешить подключения клиентов, выполнив команду [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule#az_mysql_server_firewall_rule_create). Если для начального и конечного IP-адресов задано значение 0.0.0.0, брандмауэр открыт только для других ресурсов Azure. 

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP] 
> Вы можете применить еще более строгие ограничения в правиле брандмауэра, [разрешив только исходящие IP-адреса, используемые приложением](overview-inbound-outbound-ips.md#find-outbound-ips).
>

Чтобы разрешить доступ с локального компьютера, повторно выполните команду в Cloud Shell, заменив *\<your-ip-address>* [соответствующим локальным IPv4-адресом](https://www.whatsmyip.org/).

```azurecli-interactive
az mysql server firewall-rule create --name AllowLocalClient --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address=<your-ip-address> --end-ip-address=<your-ip-address>
```

### <a name="connect-to-production-mysql-server-locally"></a>Локальное подключение к серверу рабочей базы данных MySQL

В окне терминала на локальном компьютере подключитесь к серверу MySQL в Azure. Используйте значение, указанное ранее для _&lt;admin-user>_ и _&lt;mysql-server-name>_ . При появлении запроса на ввод пароля используйте пароль, указанный во время создания базы данных в Azure.

```bash
mysql -u <admin-user>@<mysql-server-name> -h <mysql-server-name>.mysql.database.azure.com -P 3306 -p
```

### <a name="create-a-production-database"></a>Создание рабочей базы данных

В командной строке `mysql` создайте базу данных.

```sql
CREATE DATABASE sampledb;
```

### <a name="create-a-user-with-permissions"></a>Создание пользователя с разрешениями

Создайте пользователя базы данных с именем _phpappuser_ и предоставьте ему все привилегии в базе данных `sampledb`. Для простоты в этом руководстве используется пароль _MySQLAzure2017_.

```sql
CREATE USER 'phpappuser' IDENTIFIED BY 'MySQLAzure2017'; 
GRANT ALL PRIVILEGES ON sampledb.* TO 'phpappuser';
```

Завершите подключение к серверу, введя команду `quit`.

```sql
quit
```

## <a name="connect-app-to-azure-mysql"></a>Подключение приложения к базе данных Azure MySQL

На этом шаге вы подключите приложение PHP к базе данных MySQL, созданной в Базе данных Azure для MySQL.

<a name="devconfig"></a>

### <a name="configure-the-database-connection"></a>Настройка подключения к базе данных

В корневой папке репозитория создайте файл _.env.production_ и скопируйте в него следующие переменные. Замените заполнитель &lt;mysql-server-name> в *DB_HOST* и *DB_USERNAME*.

```
APP_ENV=production
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=<mysql-server-name>.mysql.database.azure.com
DB_DATABASE=sampledb
DB_USERNAME=phpappuser@<mysql-server-name>
DB_PASSWORD=MySQLAzure2017
MYSQL_SSL=true
```

Сохраните изменения.

> [!TIP]
> Чтобы защитить сведения о подключении к MySQL, этот файл извлекается из репозитория Git (см. файл _.gitignore_ в корне репозитория). Позже вы узнаете, как настроить переменные среды в службе приложений для подключения к базе данных в Базе данных Azure для MySQL. Благодаря этим переменным вам не нужен файл *.env* в службе приложений.
>

### <a name="configure-tlsssl-certificate"></a>Настройка TLS/SSL-сертификата

По умолчанию База данных Azure для MySQL требует использования TLS для подключений из клиентов. Чтобы подключиться к базе данных MySQL в Azure, вам потребуется SSL-сертификат с расширением [_PEM_, предоставляемый службой "База данных Azure для MySQL"](../mysql/howto-configure-ssl.md).

Откройте файл _config/database.php_ и добавьте в `connections.mysql` параметры `sslmode` и `options`, как показано в следующем коде.

::: zone pivot="platform-windows"  

```php
'mysql' => [
    ...
    'sslmode' => env('DB_SSLMODE', 'prefer'),
    'options' => (env('MYSQL_SSL')) ? [
        PDO::MYSQL_ATTR_SSL_KEY    => '/ssl/BaltimoreCyberTrustRoot.crt.pem', 
    ] : []
],
```

::: zone-end

::: zone pivot="platform-linux"

```php
'mysql' => [
    ...
    'sslmode' => env('DB_SSLMODE', 'prefer'),
    'options' => (env('MYSQL_SSL') && extension_loaded('pdo_mysql')) ? [
        PDO::MYSQL_ATTR_SSL_KEY    => '/ssl/BaltimoreCyberTrustRoot.crt.pem',
    ] : []
],
```

::: zone-end

В рамках этого руководства сертификат `BaltimoreCyberTrustRoot.crt.pem` предоставляется в репозитории для удобства. 

### <a name="test-the-application-locally"></a>Локальное тестирование приложения

Выполните перенос базы данных Laravel с помощью файла _.env.production_, указав его как файл среды, чтобы создать таблицы в базе данных MySQL, размещенной в Базе данных Azure для MySQL. Помните, что файл _.env.production_ содержит сведения о подключении к базе данных MySQL в Azure.

```bash
php artisan migrate --env=production --force
```

Файл _.env.production_ еще не содержит действительный ключ приложения. Создайте для него новый ключ в окне терминала.

```bash
php artisan key:generate --env=production --force
```

Запустите пример приложения, указав файл _.env.production_ как файл среды.

```bash
php artisan serve --env=production
```

Перейдите на страницу `http://localhost:8000`. Если страница загрузилась без ошибок, значит, приложение PHP подключается к базе данных MySQL в Azure.

Добавьте несколько задач на странице.

![Подключение приложения PHP к Базе данных Azure для MySQL](./media/tutorial-php-mysql-app/mysql-connect-success.png)

Чтобы остановить приложение PHP, введите `Ctrl + C` в окне терминала.

### <a name="commit-your-changes"></a>Фиксация изменений

Чтобы зафиксировать изменения, выполните следующие команды Git:

```bash
git add .
git commit -m "database.php updates"
```

Ваше приложение готово к развертыванию.

## <a name="deploy-to-azure"></a>Развертывание в Azure

На этом шаге вы развернете приложение PHP, подключенное к базе данных MySQL, в службе приложений Azure.

### <a name="configure-a-deployment-user"></a>Настойка пользователя развертывания

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

::: zone pivot="platform-windows"  

[!INCLUDE [Create app service plan no h](../../includes/app-service-web-create-app-service-plan-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Create app service plan no h](../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]

::: zone-end

<a name="create"></a>
### <a name="create-a-web-app"></a>Создание веб-приложения

::: zone pivot="platform-windows"  

[!INCLUDE [Create web app no h](../../includes/app-service-web-create-web-app-php-no-h.md)] 

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-php-linux-no-h.md)] 

::: zone-end

### <a name="configure-database-settings"></a>Настройка параметров базы данных

В службе приложений переменные среды задаются в качестве _параметров приложения_ с помощью команды [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set).

Команда ниже позволяет настроить параметры приложения `DB_HOST`, `DB_DATABASE`, `DB_USERNAME` и `DB_PASSWORD`. Замените заполнители _&lt;app-name>_ и _&lt;mysql-server-name>_ .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings DB_HOST="<mysql-server-name>.mysql.database.azure.com" DB_DATABASE="sampledb" DB_USERNAME="phpappuser@<mysql-server-name>" DB_PASSWORD="MySQLAzure2017" MYSQL_SSL="true"
```

Для доступа к параметрам можно использовать метод PHP [getenv](https://www.php.net/manual/en/function.getenv.php). В коде Laravel используется программа-оболочка [env](https://laravel.com/docs/5.4/helpers#method-env) для PHP `getenv`. Например, конфигурация MySQL в файле _config/database.php_ выглядит следующим образом:

```php
'mysql' => [
    'driver'    => 'mysql',
    'host'      => env('DB_HOST', 'localhost'),
    'database'  => env('DB_DATABASE', 'forge'),
    'username'  => env('DB_USERNAME', 'forge'),
    'password'  => env('DB_PASSWORD', ''),
    ...
],
```

### <a name="configure-laravel-environment-variables"></a>Настройка переменных среды Laravel

В среде Laravel требуется ключ приложения из службы приложений. Его можно настроить с помощью параметров приложения.

В окне терминала на локальном компьютере используйте `php artisan` для создания ключа приложения без сохранения в файле _.env_.

```bash
php artisan key:generate --show
```

В Cloud Shell задайте ключ приложения в приложении Службы приложений с помощью команды [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set). Замените заполнители _&lt;app-name>_ и _&lt;outputofphpartisankey:generate>_ .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings APP_KEY="<output_of_php_artisan_key:generate>" APP_DEBUG="true"
```

Значение `APP_DEBUG="true"` указывает Laravel возвращать отладочную информацию в случае ошибок в работе развернутого приложения. При выполнении рабочего приложения задайте для него более безопасное значение `false`.

### <a name="set-the-virtual-application-path"></a>Задание пути виртуального приложения

::: zone pivot="platform-windows"  

Задайте для приложения путь виртуального приложения. Этот шаг требуется из-за того, что [жизненный цикл приложения Laravel](https://laravel.com/docs/5.4/lifecycle) начинается в _общем_ каталоге, а не в корневом каталоге приложения. Другие платформы PHP, жизненный цикл которых начинается в корневом каталоге, могут работать без ручной настройки пути виртуального приложения.

В Cloud Shell задайте путь виртуального приложения с помощью команды [`az resource update`](/cli/azure/resource#az_resource_update). Замените заполнитель _&lt;app-name>_ .

```azurecli-interactive
az resource update --name web --resource-group myResourceGroup --namespace Microsoft.Web --resource-type config --parent sites/<app_name> --set properties.virtualApplications[0].physicalPath="site\wwwroot\public" --api-version 2015-06-01
```

По умолчанию служба приложений Azure указывает корневой путь виртуального приложения ( _/_ ) к корневому каталогу файлов развернутого приложения (_sites\wwwroot_).

::: zone-end

::: zone pivot="platform-linux"

[Жизненный цикл приложения Laravel](https://laravel.com/docs/5.4/lifecycle) начинается в _общем_ каталоге, а не в корневом каталоге приложения. Образ PHP Docker по умолчанию для службы приложений использует Apache и не позволяет настраивать `DocumentRoot` для Laravel. Тем не менее вы можете использовать `.htaccess` для повторного создания всех запросов к точке в каталоге _/public_ вместо корневого каталога. В корне репозитория для этой цели уже добавлен `.htaccess`. С ним приложение Laravel будет готово к развертыванию.

См. подробнее об [изменении корневого каталога сайта](configure-language-php.md#change-site-root).

::: zone-end

### <a name="push-to-azure-from-git"></a>Публикация в Azure из Git

::: zone pivot="platform-windows"  

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 291 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'a5e076db9c'.
remote: Running custom deployment command...
remote: Running deployment command...
...
&lt; Output has been truncated for readability &gt;
</pre>

> [!NOTE]
> Вы можете заметить, что в конце процесс развертывания устанавливает пакеты [Composer](https://getcomposer.org/). Служба приложений не запускает эти сценарии автоматизации во время развертывания по умолчанию. Для этого в данном примере в корневом каталоге репозитория расположены три дополнительных файла.
>
> - `.deployment` — этот файл означает, что служба приложений должна запустить `bash deploy.sh` как пользовательский скрипт развертывания.
> - `deploy.sh` — пользовательский скрипт развертывания. Если открыть этот файл, вы увидите, что он запускает `php composer.phar install` после `npm install`.
> - `composer.phar` — диспетчер пакетов Composer.
>
> Точно так же можно добавить любые действия развертывания в службе приложений на основе Git. Дополнительные сведения см. в статье [Custom Deployment Script](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script) (Пользовательский сценарий развертывания).
>

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 291 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'a5e076db9c'.
remote: Running custom deployment command...
remote: Running deployment command...
...
&lt; Output has been truncated for readability &gt;
</pre>

::: zone-end

### <a name="browse-to-the-azure-app"></a>Переход к приложению Azure

Перейдите по адресу `http://<app-name>.azurewebsites.net` и добавьте несколько задач в список.

:::image type="content" source="./media/tutorial-php-mysql-app/php-mysql-in-azure.png" alt-text="Снимок экрана: пример приложения Azure с названием Task List, в котором показаны добавленные новые задачи.":::

Вы запустили управляемое данными приложение PHP в службе приложений Azure.

## <a name="update-model-locally-and-redeploy"></a>Локальное обновление и повторное развертывание модели

На этом шаге вы внесете некоторые изменения в модель данных `task` и веб-приложение, а затем опубликуете обновления в Azure.

Для сценария с задачами необходимо изменить приложение, чтобы можно было пометить задачу как выполненную.

### <a name="add-a-column"></a>Добавление столбца

В окне терминала на локальном компьютере перейдите к корневой папке репозитория Git.

Создайте перенос базы данных для таблицы `tasks`:

```bash
php artisan make:migration add_complete_column --table=tasks
```

Эта команда отображает имя файла переноса, который создается. Найдите этот файл в каталоге _database/migrations_ и откройте его.

Замените метод `up` следующим кодом:

```php
public function up()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->boolean('complete')->default(False);
    });
}
```

Предыдущий код добавляет столбец логических значений `complete` в таблицу `tasks`.

Замените метод `down` следующим кодом действия отката:

```php
public function down()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropColumn('complete');
    });
}
```

В окне терминала на локальном компьютере выполните перенос базы данных Laravel, чтобы внести изменения в локальную базу данных.

```bash
php artisan migrate
```

В соответствии с условиями [соглашения об именовании Laravel](https://laravel.com/docs/5.4/eloquent#defining-models) модель `Task` (см. _app/Task.php_) сопоставляется с таблицей `tasks` по умолчанию.

### <a name="update-application-logic"></a>Обновление логики приложения

Откройте файл *routes/web.php*. В нем приложение определяет свои маршруты и бизнес-логику.

В конце файла добавьте маршрут с помощью следующего кода.

```php
/**
 * Toggle Task completeness
 */
Route::post('/task/{id}', function ($id) {
    error_log('INFO: post /task/'.$id);
    $task = Task::findOrFail($id);

    $task->complete = !$task->complete;
    $task->save();

    return redirect('/');
});
```

Предыдущий код выполняет простое обновление модели данных, переключая значение `complete`.

### <a name="update-the-view"></a>Обновление представления

Откройте файл *resources/views/tasks.blade.php*. Найдите открывающий тег `<tr>` и замените его приведенным ниже значением.

```html
<tr class="{{ $task->complete ? 'success' : 'active' }}" >
```

Предыдущий код изменяет цвет строк в зависимости от того, завершена ли задача.

Следующая строка содержит приведенный ниже код.

```html
<td class="table-text"><div>{{ $task->name }}</div></td>
```

Замените всю эту строку следующим кодом.

```html
<td>
    <form action="{{ url('task/'.$task->id) }}" method="POST">
        {{ csrf_field() }}

        <button type="submit" class="btn btn-xs">
            <i class="fa {{$task->complete ? 'fa-check-square-o' : 'fa-square-o'}}"></i>
        </button>
        {{ $task->name }}
    </form>
</td>
```

Предыдущий код добавляет кнопку "Submit" (Отправить), которая ссылается на маршрут, определенный ранее.

### <a name="test-the-changes-locally"></a>Локальная проверка изменений

В окне терминала на локальном компьютере запустите сервер разработки из корневого каталога репозитория Git.

```bash
php artisan serve
```

Чтобы увидеть, как изменится состояние задачи, перейдите по адресу `http://localhost:8000` и установите флажок.

![Добавлен флажок для задачи](./media/tutorial-php-mysql-app/complete-checkbox.png)

Чтобы остановить приложение PHP, введите `Ctrl + C` в окне терминала.

### <a name="publish-changes-to-azure"></a>Публикация изменений в Azure

В окне терминала на локальном компьютере выполните перенос базы данных Laravel со строкой подключения рабочей базы данных, чтобы внести изменения в базу данных в Azure.

```bash
php artisan migrate --env=production --force
```

Зафиксируйте все изменения в Git, а затем отправьте изменения кода в Azure.

```bash
git add .
git commit -m "added complete checkbox"
git push azure main
```

После выполнения команды `git push` перейдите в приложение Azure и проверьте новые функции.

![Изменения модели и базы данных, опубликованные в Azure](media/tutorial-php-mysql-app/complete-checkbox-published.png)

Добавленные задачи сохраняются в базе данных. Изменения в схеме данных не влияют на имеющиеся данные.

## <a name="stream-diagnostic-logs"></a>Потоковая передача журналов диагностики

::: zone pivot="platform-windows"  

При запуске приложения PHP в службе приложений Azure можно передавать журналы консоли в свой терминал. Таким образом, вы будете получать те же диагностические сообщения, которые помогут устранить ошибки приложения.

Чтобы настроить потоки для журналов, выполните команду [`az webapp log tail`](/cli/azure/webapp/log#az_webapp_log_tail) в Cloud Shell.

```azurecli-interactive
az webapp log tail --name <app_name> --resource-group myResourceGroup
```

После настройки потоков обновите приложение Azure в браузере, чтобы получить немного трафика. Вы должны увидеть, что журналы консоли теперь направляются в терминал. Если журналы консоли не отображаются, проверьте еще раз через 30 секунд.

Чтобы остановить потоковую передачу журналов, нажмите клавиши `Ctrl`+`C`.

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end

> [!TIP]
> Приложение PHP может использовать стандартный метод [error_log()](https://php.net/manual/function.error-log.php) для вывода в консоль. Пример приложения использует этот подход в файле конфигурации _app/Http/routes.php_.
>
> Являясь веб-платформой, [Laravel использует Monolog](https://laravel.com/docs/5.4/errors) в качестве поставщика службы ведения журнала. Дополнительные сведения см. в статье [PHP: How to use monolog to log to console (php://out)?](https://stackoverflow.com/questions/25787258/php-how-to-use-monolog-to-log-to-console-php-out) (PHP: как выполнять запись в консоль с помощью Monolog (php://out)).
>
>

## <a name="manage-the-azure-app"></a>Управление приложением Azure

Перейдите на [портал Azure](https://portal.azure.com), чтобы управлять созданным приложением.

В меню слева щелкните **Службы приложений**, а затем — имя своего приложения Azure.

![Переход к приложению Azure на портале](./media/tutorial-php-mysql-app/access-portal.png)

Отобразится страница обзора вашего приложения. Вы можете выполнять базовые задачи управления: обзор, завершение, запуск, перезагрузку и удаление.

В меню слева доступны страницы для настройки приложения.

![Страница службы приложений на портале Azure](./media/tutorial-php-mysql-app/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание базы данных MySQL в Azure.
> * Подключение приложения PHP к MySQL.
> * Развертывание приложения в Azure
> * Обновление модели данных и повторное развертывание приложения.
> * Потоковая передача журналов диагностики из Azure.
> * Управление приложением на портале Azure.

Перейдите к следующему руководству, чтобы научиться сопоставлять пользовательские DNS-имена с приложением.

> [!div class="nextstepaction"]
> [Руководство. Сопоставление настраиваемого DNS-имени с приложением](app-service-web-tutorial-custom-domain.md)

Также ознакомьтесь с другими ресурсами:

> [!div class="nextstepaction"]
> [Настройка приложения PHP](configure-language-php.md)
