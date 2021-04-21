---
title: Руководство по созданию приложения PHP (Laravel) с помощью Базы данных Azure для MySQL (Гибкий сервер)
description: В этом руководстве объясняется, как создать приложение PHP с использованием гибкого сервера.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: tutorial
ms.devlang: php
ms.date: 9/21/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: bb38b72af6e7c649c0904c41d3052b15a4c36955
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770063"
---
# <a name="tutorial-build-a-php-laravel-and-mysql-flexible-server-preview-app-in-azure-app-service"></a>Руководство по созданию приложения PHP (Laravel) и гибкого сервера MySQL (предварительная версия) в Службе приложений Azure


:::image type="content" source="media/tutorial-php-database-app/complete-checkbox-published.png" alt-text="Веб-приложение PHP в Azure с гибким сервером":::

[Служба приложений Azure](../../app-service/overview.md) — это высокомасштабируемая служба размещения с самостоятельной установкой исправлений на основе операционной системы Linux. В этом руководстве показано, как создать приложение PHP в Azure и подключить его к базе данных MySQL. По завершении вы получите приложение [Laravel](https://laravel.com/), работающее в Службе приложений Azure в Linux.

В этом руководстве вы узнаете, как:
> [!div class="checklist"]
> * Настройка приложения PHP (Laravel) локальной службы MySQL.
> * Создание гибкого сервера MySQL (предварительная версия).
> * Подключение приложения PHP к гибкому серверу MySQL (предварительная версия)
> * Развертывание приложения в Службе приложений Azure.
> * Обновление модели данных и повторное развертывание приложения.
> * Управление приложением на портале Azure.

Если у вас еще нет [подписки Azure](../../guides/developer/azure-developer-guide.md#understanding-accounts-subscriptions-and-billing), создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio), прежде чем начать работу.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

1. [установите Git](https://git-scm.com/);
2. [PHP 5.6.4 или более поздней версии](https://php.net/downloads.php);
3. [Composer](https://getcomposer.org/doc/00-intro.md);
4. Включите следующие расширения PHP, необходимые для Laravel: OpenSSL, PDO-MySQL, Mbstring, Tokenizer и XML.
5. [MySQL](https://dev.mysql.com/doc/refman/5.7/en/installing.html) (этот компонент потребуется запустить).

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

В окне терминала перейдите к пустому каталогу, куда можно клонировать пример приложения.  Выполните команду ниже, чтобы клонировать репозиторий с примером.

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

:::image type="content" source="media/tutorial-php-database-app/mysql-connect-success.png" alt-text="Приложение PHP успешно подключается к базе данных MySQL":::

Чтобы остановить приложение PHP, введите `Ctrl + C` в окне терминала.

## <a name="create-a-mysql-flexible-server-preview"></a>Создание гибкого сервера MySQL (предварительная версия).
На этом шаге вы создадите базу данных MySQL в режиме развертывания [База данных Azure для MySQL (Гибкий сервер)](../index.yml), который предоставляется в общедоступной предварительной версии. Позже вы настроите приложение PHP для подключения к этой базе данных. Запустите [Azure Cloud Shell](../../cloud-shell/overview.md) и создайте сервер, выполнив команду [`az flexible-server create`](/cli/azure/mysql/server#az_mysql_flexible_server_create).

```azurecli-interactive
az mysql flexible-server create  --resource-group myResourceGroup --public-access <IP-Address>
```

> [!IMPORTANT]
> - Запишите значения **имени сервера** и **строки подключения**, чтобы применить их на следующем шаге для подключения к Laravel и запуска переноса данных.
> - Для аргумента **IP-Address** укажите IP-адрес клиентского компьютера. Созданный сервер сразу блокируется, и для локального управления этим сервером необходимо разрешить доступ клиентскому компьютеру.

### <a name="configure-server-firewall-to-allow-web-app-to-connect-to-the-server"></a>Настройка брандмауэра сервера для подключения веб-приложения к серверу

В Cloud Shell создайте правило брандмауэра для сервера MySQL, чтобы разрешить подключения клиентов, выполнив команду az mysql server firewall-rule create. Если для начального и конечного IP-адресов задано значение ```0.0.0.0```, брандмауэр открывается только для тех служб Azure, которые не используют статические IP-адреса для подключения к серверу.

```azurecli
az mysql flexible-server firewall-rule create --name allanyAzureIPs --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

### <a name="connect-to-production-mysql-server-locally"></a>Локальное подключение к серверу рабочей базы данных MySQL

В окне терминала на локальном компьютере подключитесь к серверу MySQL в Azure. Используйте значение, указанное ранее для ```<admin-user>``` и ```<mysql-server-name>```. При появлении запроса на ввод пароля используйте пароль, указанный во время создания базы данных в Azure.

```bash
mysql -u <admin-user> -h <mysql-server-name>.mysql.database.azure.com -P 3306 -p
```

### <a name="create-a-production-database"></a>Создание рабочей базы данных

В командной строке `mysql` создайте базу данных.

```sql
CREATE DATABASE sampledb;
```

### <a name="create-a-user-with-permissions"></a>Создание пользователя с разрешениями

Создайте пользователя базы данных с именем _phpappuser_ и предоставьте ему все привилегии в базе данных `sampledb`. Для простоты в этом руководстве используется пароль _MySQLAzure2020_.

```sql
CREATE USER 'phpappuser' IDENTIFIED BY 'MySQLAzure2020';
GRANT ALL PRIVILEGES ON sampledb.* TO 'phpappuser';
```

Завершите подключение к серверу, введя команду `quit`.

```sql
quit
```

## <a name="connect-app-to-mysql-flexible-server"></a>Подключение приложения к гибкому серверу MySQL

На этом шаге вы подключите приложение PHP к базе данных MySQL, созданной в Базе данных Azure для MySQL.

<a name="devconfig"></a>

### <a name="configure-the-database-connection"></a>Настройка подключения к базе данных

В корневой папке репозитория создайте файл _.env.production_ и скопируйте в него следующие переменные. Замените заполнитель _&lt;mysql-server-name>_ в значениях *DB_HOST* и *DB_USERNAME*.

```
APP_ENV=production
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=<mysql-server-name>.mysql.database.azure.com
DB_DATABASE=sampledb
DB_USERNAME=phpappuser
DB_PASSWORD=MySQLAzure2017
MYSQL_SSL=true
```

Сохраните изменения.

> [!TIP]
> Чтобы защитить сведения о подключении к MySQL, этот файл извлекается из репозитория Git (см. файл _.gitignore_ в корне репозитория). Позже вы узнаете, как настроить переменные среды в службе приложений для подключения к базе данных в Базе данных Azure для MySQL. Благодаря этим переменным вам не нужен файл *.env* в службе приложений.
>

### <a name="configure-tlsssl-certificate"></a>Настройка TLS/SSL-сертификата

По умолчанию гибкий сервер MySQL требует, чтобы клиенты использовали подключения по протоколу TLS. Чтобы подключиться к базе данных MySQL в Azure, вам потребуется сертификат с расширением [_PEM_, предоставленный службой "База данных Azure для MySQL (Гибкий сервер)"](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem). Скачайте [этот сертификат](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) и поместите его в папку **SSL** в локальной копии репозитория с примером приложения.

Откройте файл _config/database.php_ и добавьте в `connections.mysql` параметры `sslmode` и `options`, как показано в следующем коде.

```php
'mysql' => [
    ...
    'sslmode' => env('DB_SSLMODE', 'prefer'),
    'options' => (env('MYSQL_SSL') && extension_loaded('pdo_mysql')) ? [
        PDO::MYSQL_ATTR_SSL_KEY    => '/ssl/DigiCertGlobalRootCA.crt.pem',
    ] : []
],
```

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

:::image type="content" source="media/tutorial-php-database-app/mysql-connect-success.png" alt-text="Подключение приложения PHP к Базе данных Azure для MySQL":::

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

Для развертывания в веб-приложение Azure из FTP и локального репозитория Git можно использовать пользователя развертывания. Настроив один раз пользователя развертывания, вы сможете использовать его для всех последующих развертываний в Azure. Имя пользователя и пароль учетной записи развертывания отличаются от учетных данных подписки Azure.

Чтобы настроить пользователя развертывания, выполните в Azure Cloud Shell команду [az webapp deployment user set](/cli/azure/webapp/deployment/user#az_webapp_deployment_user_set). Вместо _&lt;username>_ и _&lt;password>_ укажите имя пользователя и пароль для вашего развертывания.

Имя пользователя должно быть уникальным в Azure. Кроме того, чтобы отправка в локальный репозиторий Git работала, имя пользователя не должно содержать символ @.
Пароль должен содержать не менее восьми символов и включать два из трех следующих элементов: буквы, цифры и символы.

```bash
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku F1 --is-linux
```

В выходных данных JSON пароль отображается как NULL. Если вы видите сообщение "Conflict. Сведения. Ошибка 409", измените имя пользователя. Если вы видите сообщение об ошибке "Bad Request. Сведения. Ошибка 400", используйте более надежный пароль. **Запишите имя пользователя и пароль и используйте их для развертывания веб-приложений.**

### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

В Cloud Shell создайте план службы приложений в группе ресурсов, выполнив команду [az appservice plan create](/cli/azure/appservice/plan#az_appservice_plan_create). В следующем примере создается план Службы приложений с именем myAppServicePlan в ценовой категории "Бесплатный" (--sku F1) в контейнере Linux (--is-linux).

az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku F1 --is-linux

<a name="create"></a>

### <a name="create-a-web-app"></a>Создание веб-приложения

Создайте [веб-приложение](../../app-service/overview.md#app-service-on-linux) в плане Службы приложений myAppServicePlan.

В Cloud Shell для этого можно использовать команду [az webapp create](/cli/azure/webapp#az_webapp_create). В следующем примере замените _&lt;app-name>_ глобальным уникальным именем приложения (допустимые символы: `a-z`, `0-9` и `-`). Для среды выполнения установлено значение `PHP|7.0`. Чтобы просмотреть все поддерживаемые среды выполнения, выполните команду [az webapp list-runtimes --linux](/cli/azure/webapp#az_webapp_list_runtimes).

```bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "PHP|7.3" --deployment-local-git
```

Когда веб-приложение будет создано, в Azure CLI отобразится примерно следующее:

```
Local git is configured with url of 'https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app-name>.azurewebsites.net",
  "deploymentLocalGitUrl": "https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

Вы создали пустое веб-приложение с включенным развертыванием Git.

> [!NOTE]
> URL-адрес удаленного репозитория Git отображается в свойстве deploymentLocalGitUrl в таком формате: https://<username>@<имя_приложения>.scm.azurewebsites.net/<имя_приложения>.git. Сохраните этот URL-адрес для дальнейшего использования.

### <a name="configure-database-settings"></a>Настройка параметров базы данных

В службе приложений переменные среды задаются в качестве _параметров приложения_ с помощью команды [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set).

Команда ниже позволяет настроить параметры приложения `DB_HOST`, `DB_DATABASE`, `DB_USERNAME` и `DB_PASSWORD`. Замените заполнители _&lt;app-name>_ и _&lt;mysql-server-name>_ .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings DB_HOST="<mysql-server-name>.mysql.database.azure.com" DB_DATABASE="sampledb" DB_USERNAME="phpappuser" DB_PASSWORD="MySQLAzure2017" MYSQL_SSL="true"
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

[Жизненный цикл приложения Laravel](https://laravel.com/docs/5.4/lifecycle) начинается в _общем_ каталоге, а не в корневом каталоге приложения. Образ PHP Docker по умолчанию для службы приложений использует Apache и не позволяет настраивать `DocumentRoot` для Laravel. Тем не менее вы можете использовать `.htaccess` для повторного создания всех запросов к точке в каталоге _/public_ вместо корневого каталога. В корне репозитория для этой цели уже добавлен `.htaccess`. С ним приложение Laravel будет готово к развертыванию.

См. подробнее об [изменении корневого каталога сайта](../../app-service/configure-language-php.md?pivots=platform-linux#change-site-root).

### <a name="push-to-azure-from-git"></a>Публикация в Azure из Git

Вернитесь к окну терминала (в локальном расположении) и добавьте удаленное приложение Azure в локальный репозиторий Git. Замените _&lt;deploymentLocalGitUrl-from-create-step>_ URL-адресом удаленного репозитория Git, который вы сохранили при [создании веб-приложения](#create-a-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Отправьте код в удаленное приложение Azure, чтобы развернуть приложение. При появлении запроса на ввод учетных данных в диспетчере учетных данных Git введите учетные данные, созданные на шаге **настройки пользователя развертывания** (а не те, которые используются для входа на портал Azure).

```bash
git push azure main
```

Выполнение этой команды может занять несколько минут. При выполнении эта команда выводит приблизительно следующие сведения:

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

### <a name="browse-to-the-azure-app"></a>Переход к приложению Azure

Перейдите по адресу `http://<app-name>.azurewebsites.net` и добавьте несколько задач в список.

:::image type="content" source="media/tutorial-php-database-app/php-mysql-in-azure.png" alt-text="Веб-приложение PHP в Azure":::

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

:::image type="content" source="media/tutorial-php-database-app/complete-checkbox.png" alt-text="Добавлен флажок для задачи":::

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

:::image type="content" source="media/tutorial-php-database-app/complete-checkbox-published.png" alt-text="Изменения модели и базы данных, опубликованные в Azure":::

Добавленные задачи сохраняются в базе данных. Изменения в схеме данных не влияют на имеющиеся данные.

## <a name="clean-up-resources"></a>Очистка ресурсов
На предыдущем шаге вы создали ресурсы Azure в группе ресурсов. Если эти ресурсы вам не понадобятся в будущем, вы можете удалить группу ресурсов, выполнив следующую команду в Cloud Shell:

```bash
az group delete --name myResourceGroup
```

<a name="next"></a>

## <a name="next-steps"></a>Следующие шаги

> [!div class="nextstepaction"]
> [Управление ресурсами с помощью портала Azure](../../azure-resource-manager/management/manage-resources-portal.md) <br/>
> [!div class="nextstepaction"]
> [Управление сервером](how-to-manage-server-cli.md)
