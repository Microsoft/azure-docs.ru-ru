---
title: Настройка приложений PHP
description: Узнайте, как настроить приложение PHP в собственных экземплярах Windows или в предварительно созданном контейнере PHP в службе приложений Azure. В этой статье показаны наиболее распространенные задачи настройки.
ms.devlang: php
ms.topic: article
ms.date: 06/02/2020
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: afac8273b5729bcf5470be471145214426dc7dab
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90055305"
---
# <a name="configure-a-php-app-for-azure-app-service"></a>Настройка приложения PHP для службы приложений Azure

В этом руководство показано, как настроить веб-приложения PHP, мобильные серверные интерфейсы и приложения API в службе приложений Azure.

Это краткое описание содержит основные понятия и инструкции для разработчиков PHP, которые развертывают приложения в службе приложений. Если вы раньше не использовали Службу приложений Azure, сначала ознакомьтесь с [кратким руководством по PHP](quickstart-php.md) и [руководством по использованию PHP с MySQL](tutorial-php-mysql-app.md).

## <a name="show-php-version"></a>Отображение версии PHP

::: zone pivot="platform-windows"  

Чтобы отобразить сведения о текущей версии PHP, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query phpVersion
```

Чтобы отобразить сведения обо всех поддерживаемых версиях PHP, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes | grep php
```

::: zone-end

::: zone pivot="platform-linux"

Чтобы отобразить сведения о текущей версии PHP, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

Чтобы отобразить сведения обо всех поддерживаемых версиях PHP, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes --linux | grep PHP
```

::: zone-end

## <a name="set-php-version"></a>как установить версию PHP;

::: zone pivot="platform-windows"  

Выполните следующую команду в [Cloud Shell](https://shell.azure.com) , чтобы установить версию PHP 7,4:

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --php-version 7.4
```

::: zone-end

::: zone pivot="platform-linux"

Чтобы настроить для PHP версию 7.2, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --linux-fx-version "PHP|7.2"
```

::: zone-end

::: zone pivot="platform-windows"  

## <a name="run-composer"></a>Запуск Composer

Если вы хотите, чтобы служба приложений выполняла [Composer](https://getcomposer.org/) во время развертывания, проще всего включить компоновщик в репозиторий.

В окне терминала перейдите в корневой каталог репозитория и следуйте инструкциям по [загрузке Composer](https://getcomposer.org/download/) , чтобы скачать файл *Composer. Phar* в корневой каталог.

Выполните следующие команды (необходимо установить [NPM](https://www.npmjs.com/get-npm) ):

```bash
npm install kuduscript -g
kuduscript --node --scriptType bash --suppressPrompt
```

Корневой каталог репозитория теперь содержит два дополнительных файла: *. Deployment* и *deploy.sh*.

Откройте *deploy.sh* и найдите `Deployment` раздел, который выглядит следующим образом:

```bash
##################################################################################################################################
# Deployment
# ----------
```

Добавьте раздел кода, в котором необходимо запустить требуемое средство *в конце* `Deployment` раздела:

```bash
# 4. Use composer
echo "$DEPLOYMENT_TARGET"
if [ -e "$DEPLOYMENT_TARGET/composer.json" ]; then
  echo "Found composer.json"
  pushd "$DEPLOYMENT_TARGET"
  php composer.phar install $COMPOSER_ARGS
  exitWithMessageOnError "Composer install failed"
  popd
fi
```

Зафиксируйте все изменения и разверните код с помощью Git или ZIP deploy с включенной автоматизацией сборки. Теперь компоновщик должен работать как часть автоматизации развертывания.

## <a name="run-gruntbowergulp"></a>Запуск grunt/Bower/gulp

Если вы хотите, чтобы служба приложений выполняла популярные средства автоматизации во время развертывания, например grunt, Bower или gulp, необходимо предоставить [Пользовательский скрипт развертывания](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script). Служба приложений запускает этот скрипт при развертывании с помощью Git или при [развертывании ZIP](deploy-zip.md) с включенной автоматизацией сборок. 

Чтобы разрешить репозиторию запускать эти средства, необходимо добавить их в зависимости в *package.js.* Пример:

```json
"dependencies": {
  "bower": "^1.7.9",
  "grunt": "^1.0.1",
  "gulp": "^3.9.1",
  ...
}
```

В окне терминала перейдите в каталог корня репозитория и выполните следующие команды (необходимо установить [NPM](https://www.npmjs.com/get-npm) ):

```bash
npm install kuduscript -g
kuduscript --node --scriptType bash --suppressPrompt
```

Корневой каталог репозитория теперь содержит два дополнительных файла: *. Deployment* и *deploy.sh*.

Откройте *deploy.sh* и найдите `Deployment` раздел, который выглядит следующим образом:

```bash
##################################################################################################################################
# Deployment
# ----------
```

Этот раздел завершается выполнением `npm install --production` . Добавьте раздел кода, в котором необходимо запустить требуемое средство *в конце* `Deployment` раздела:

- [Bower](#bower)
- [Gulp](#gulp)
- [Grunt](#grunt)

См. [пример в примере MEAN.js](https://github.com/Azure-Samples/meanjs/blob/master/deploy.sh#L112-L135), где скрипт развертывания также выполняет пользовательскую `npm install` команду.

### <a name="bower"></a>Bower

Этот фрагмент кода выполняется `bower install` .

```bash
if [ -e "$DEPLOYMENT_TARGET/bower.json" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/bower install
  exitWithMessageOnError "bower failed"
  cd - > /dev/null
fi
```

### <a name="gulp"></a>Gulp

Этот фрагмент кода выполняется `gulp imagemin` .

```bash
if [ -e "$DEPLOYMENT_TARGET/gulpfile.js" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/gulp imagemin
  exitWithMessageOnError "gulp failed"
  cd - > /dev/null
fi
```

### <a name="grunt"></a>Grunt

Этот фрагмент кода выполняется `grunt` .

```bash
if [ -e "$DEPLOYMENT_TARGET/Gruntfile.js" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/grunt
  exitWithMessageOnError "Grunt failed"
  cd - > /dev/null
fi
```

::: zone-end

::: zone pivot="platform-linux"

## <a name="customize-build-automation"></a>Настройка автоматизации сборки

Если приложение развертывается с использованием Git или ZIP-пакетов с включенной автоматизацией сборки, то автоматизация сборки cлужбы приложений проходит в такой последовательности:

1. Запустите пользовательский скрипт, если он указан `PRE_BUILD_SCRIPT_PATH`.
1. Выполните `php composer.phar install`.
1. Запустите пользовательский скрипт, если он указан `POST_BUILD_SCRIPT_PATH`.

`PRE_BUILD_COMMAND` и `POST_BUILD_COMMAND` являются переменными среды, которые по умолчанию пустые. Чтобы выполнить команды перед сборкой, определите `PRE_BUILD_COMMAND`. Чтобы выполнить команды после сборки, определите `POST_BUILD_COMMAND`.

В следующем примере указываются две переменные для ряда команд, разделенных запятыми.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PRE_BUILD_COMMAND="echo foo, scripts/prebuild.sh"
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings POST_BUILD_COMMAND="echo foo, scripts/postbuild.sh"
```

Дополнительные переменные среды для настройки автоматизации сборки см. в статье [Конфигурация Oryx](https://github.com/microsoft/Oryx/blob/master/doc/configuration.md).

Дополнительные сведения о том, как Служба приложений выполняет и создает приложения PHP в Linux, см. в документации по [Oryx (сведения об обнаружении и создании приложений PHP)](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/php.md).

## <a name="customize-start-up"></a>Настройка запуска

По умолчанию во встроенном контейнере PHP выполняется сервер Apache. При запуске он выполняет команду `apache2ctl -D FOREGROUND"`. При необходимости вы можете выполнять другую команду при запуске. Для этого выполните в [Cloud Shell](https://shell.azure.com) следующую команду:

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<custom-command>"
```

::: zone-end

## <a name="access-environment-variables"></a>Доступ к переменным среды

В Службе приложений можно [задать параметры приложения](configure-common.md#configure-app-settings) вне кода приложения. Затем вы сможете получать к ним доступ, используя стандартный шаблон [getenv()](https://secure.php.net/manual/function.getenv.php). Например, для доступа к параметру приложения с именем `DB_HOST` используйте следующий код:

```php
getenv("DB_HOST")
```

## <a name="change-site-root"></a>Изменение корня сайта

::: zone pivot="platform-windows"  

Выбранная вами веб-платформа может использовать подкаталог в качестве корня сайта. Например, [Laravel](https://laravel.com/)использует *открытый/* подкаталог в качестве корня сайта.

Чтобы настроить корневой каталог сайта, задайте путь к виртуальному приложению для приложения с помощью [`az resource update`](/cli/azure/resource#az-resource-update) команды. В следующем примере для корня сайта задается *общедоступный или* подкаталог в репозитории. 

```azurecli-interactive
az resource update --name web --resource-group <group-name> --namespace Microsoft.Web --resource-type config --parent sites/<app-name> --set properties.virtualApplications[0].physicalPath="site\wwwroot\public" --api-version 2015-06-01
```

По умолчанию служба приложений Azure указывает корневой путь виртуального приложения ( _/_ ) к корневому каталогу файлов развернутого приложения (_sites\wwwroot_).

::: zone-end

::: zone pivot="platform-linux"

Выбранная вами веб-платформа может использовать подкаталог в качестве корня сайта. Например, [Laravel](https://laravel.com/) использует для этого подкаталог `public/`.

Образ PHP по умолчанию для Службы приложений использует Apache и не позволяет настраивать корень сайта для приложения. Чтобы обойти это ограничение, добавьте файл *.htaccess* в корень репозитория со следующим содержимым:

```
<IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteCond %{REQUEST_URI} ^(.*)
    RewriteRule ^(.*)$ /public/$1 [NC,L,QSA]
</IfModule>
```

Если вы предпочитаете не использовать повторное создание с помощью *.htaccess*, вместо него приложение Laravel можно развернуть из [пользовательского образа Docker](quickstart-custom-container.md).

::: zone-end

## <a name="detect-https-session"></a>Обнаружение сеанса HTTPS

В Службе приложений [завершение SSL-запросов](https://wikipedia.org/wiki/TLS_termination_proxy) происходит в подсистеме балансировки нагрузки сети, поэтому все HTTPS-запросы достигают вашего приложения в виде незашифрованных HTTP-запросов. Если логика вашего приложения проверяет, зашифрованы ли пользовательские запросы, проверяйте заголовок `X-Forwarded-Proto`.

```php
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
// Do something when HTTPS is used
}
```

Популярные веб-платформы позволяют получить доступ к информации `X-Forwarded-*` в стандартном шаблоне приложения. В [CodeIgniter](https://codeigniter.com/) функция [is_https()](https://github.com/bcit-ci/CodeIgniter/blob/master/system/core/Common.php#L338-L365) по умолчанию проверяет значение `X_FORWARDED_PROTO`.

## <a name="customize-phpini-settings"></a>Настройка параметров php.ini

Если вам необходимо внести изменения в установку PHP, вы можете изменить любые [директивы php.ini](https://www.php.net/manual/ini.list.php), выполнив следующие инструкции.

> [!NOTE]
> Оптимальный способ для просмотра версии PHP и текущей конфигурации *php.ini* — это вызов [phpinfo()](https://php.net/manual/function.phpinfo.php) в приложении.
>

### <a name="customize-non-php_ini_system-directives"></a><a name="Customize-non-PHP_INI_SYSTEM directives"></a>Настройка не относящихся к PHP_INI_SYSTEM директив

::: zone pivot="platform-windows"  

Чтобы настроить директивы PHP_INI_USER, PHP_INI_PERDIR и PHP_INI_ALL (см. [ директивыphp.ini](https://www.php.net/manual/ini.list.php)), добавьте `.user.ini` файл в корневой каталог приложения.

Добавьте параметры конфигурации в файл `.user.ini`, используя тот же синтаксис, что и для файла `php.ini`. Например, чтобы включить параметр `display_errors` и установить для параметра `upload_max_filesize` значение 10 МБ, в файле `.user.ini` будет указан следующий текст.

```
 ; Example Settings
 display_errors=On
 upload_max_filesize=10M

 ; Write errors to d:\home\LogFiles\php_errors.log
 ; log_errors=On
```

Повторно разверните приложение с внесенными изменениями и перезапустите его.

В качестве альтернативы использованию `.user.ini` файла можно использовать [ini_set ()](https://www.php.net/manual/function.ini-set.php) в приложении, чтобы настроить эти директивы, не относящиеся к PHP_INI_SYSTEM.

::: zone-end

::: zone pivot="platform-linux"

Чтобы настроить директивы PHP_INI_USER, PHP_INI_PERDIR и PHP_INI_ALL (см. дополнительные сведения о [директивах php.ini](https://www.php.net/manual/ini.list.php)), добавьте файл *.htaccess* в корневой каталог приложения.

В файле *.htaccess* добавьте директивы, используя синтаксис `php_value <directive-name> <value>`. Пример:

```
php_value upload_max_filesize 1000M
php_value post_max_size 2000M
php_value memory_limit 3000M
php_value max_execution_time 180
php_value max_input_time 180
php_value display_errors On
php_value upload_max_filesize 10M
```

Повторно разверните приложение с внесенными изменениями и перезапустите его. При развертывании с помощью Kudu (например, с использованием [Git](deploy-local-git.md)) приложение будет автоматически перезапущено после развертывания.

В качестве альтернативы *.htaccess* вы можете использовать [ini_set()](https://www.php.net/manual/function.ini-set.php) в приложении для настройки таких не относящихся к PHP_INI_SYSTEM директив.

::: zone-end

### <a name="customize-php_ini_system-directives"></a><a name="customize-php_ini_system-directives"></a>Настройка директив PHP_INI_SYSTEM

::: zone pivot="platform-windows"  

При настройке директив PHP_INI_SYSTEM (см. подробные сведения о [директивах php.ini](https://www.php.net/manual/ini.list.php)) нельзя использовать подход с *.htaccess*. Служба приложений предоставляет отдельный механизм с использованием параметра приложения `PHP_INI_SCAN_DIR`.

Сначала выполните следующую команду в [Cloud Shell](https://shell.azure.com), чтобы добавить параметр приложения с именем `PHP_INI_SCAN_DIR`:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PHP_INI_SCAN_DIR="d:\home\site\ini"
```

Перейдите в консоль KUDU ( `https://<app-name>.scm.azurewebsites.net/DebugConsole` ) и перейдите к `d:\home\site` .

Создайте каталог по пути `d:\home\site` с именем `ini`, а затем создайте файл *.ini* в каталоге `d:\home\site\ini` (например, *settings.ini)* с нужными директивами. Используйте тот же синтаксис, который используется в файле *php.ini*. 

Например, чтобы изменить значение [expose_php](https://php.net/manual/ini.core.php#ini.expose-php), выполните следующие команды:

```bash
cd /home/site
mkdir ini
echo "expose_php = Off" >> ini/setting.ini
```

Чтобы изменения вступили в силу, перезапустите приложение.

::: zone-end

::: zone pivot="platform-linux"

При настройке директив PHP_INI_SYSTEM (см. подробные сведения о [директивах php.ini](https://www.php.net/manual/ini.list.php)) нельзя использовать подход с *.htaccess*. Служба приложений предоставляет отдельный механизм с использованием параметра приложения `PHP_INI_SCAN_DIR`.

Сначала выполните следующую команду в [Cloud Shell](https://shell.azure.com), чтобы добавить параметр приложения с именем `PHP_INI_SCAN_DIR`:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PHP_INI_SCAN_DIR="/usr/local/etc/php/conf.d:/home/site/ini"
```

`/usr/local/etc/php/conf.d` — это каталог по умолчанию, в котором располагается файл *php.ini*. `/home/site/ini` — это пользовательский каталог, в который вы добавите пользовательский файл *.ini*. Значения в нем нужно разделять с помощью символа `:`.

Перейдите к веб-сеансу SSH с контейнером Linux (`https://<app-name>.scm.azurewebsites.net/webssh/host`).

Создайте каталог по пути `/home/site` с именем `ini`, а затем создайте файл *.ini* в каталоге `/home/site/ini` (например, *settings.ini)* с нужными директивами. Используйте тот же синтаксис, который используется в файле *php.ini*. 

> [!TIP]
> Во встроенных контейнерах Linux в Службе приложений в качестве постоянного общего хранилища используется каталог */home*. 
>

Например, чтобы изменить значение [expose_php](https://php.net/manual/ini.core.php#ini.expose-php), выполните следующие команды:

```bash
cd /home/site
mkdir ini
echo "expose_php = Off" >> ini/setting.ini
```

Чтобы изменения вступили в силу, перезапустите приложение.

::: zone-end

## <a name="enable-php-extensions"></a>Включение расширений PHP

::: zone pivot="platform-windows"  

Встроенные установки PHP содержат наиболее часто используемые расширения. Для включения дополнительных расширений выполните те же действия, что и для [настройки директив php.ini](#customize-php_ini_system-directives).

> [!NOTE]
> Оптимальный способ для просмотра версии PHP и текущей конфигурации *php.ini* — это вызов [phpinfo()](https://php.net/manual/function.phpinfo.php) в приложении.
>

Чтобы включить дополнительные расширения, выполните следующие действия.

Добавьте `bin` каталог в корневой каталог приложения и вставьте `.dll` в него файлы расширения (например, *mongodb.dll*). Убедитесь, что расширения совместимы с версией PHP в Azure, а также с VC9 и непотокобезопасного кода (nts).

Примените изменения.

Выполните действия, описанные в разделе [Настройка директив PHP_INI_SYSTEM](#customize-php_ini_system-directives), добавьте расширения в пользовательский файл *.ini* с директивами [extension](https://www.php.net/manual/ini.core.php#ini.extension) или [zend_extension](https://www.php.net/manual/ini.core.php#ini.zend-extension).

```
extension=d:\home\site\wwwroot\bin\mongodb.dll
zend_extension=d:\home\site\wwwroot\bin\xdebug.dll
```

Чтобы изменения вступили в силу, перезапустите приложение.

::: zone-end

::: zone pivot="platform-linux"

Встроенные установки PHP содержат наиболее часто используемые расширения. Для включения дополнительных расширений выполните те же действия, что и для [настройки директив php.ini](#customize-php_ini_system-directives).

> [!NOTE]
> Оптимальный способ для просмотра версии PHP и текущей конфигурации *php.ini* — это вызов [phpinfo()](https://php.net/manual/function.phpinfo.php) в приложении.
>

Чтобы включить дополнительные расширения, выполните следующие действия.

Добавьте каталог `bin` в корневой каталог приложения и поместите в него файлы с расширением `.so` (например, *mongodb.so*). Убедитесь, что расширения совместимы с версией PHP в Azure, а также с VC9 и непотокобезопасного кода (nts).

Примените изменения.

Выполните действия, описанные в разделе [Настройка директив PHP_INI_SYSTEM](#customize-php_ini_system-directives), добавьте расширения в пользовательский файл *.ini* с директивами [extension](https://www.php.net/manual/ini.core.php#ini.extension) или [zend_extension](https://www.php.net/manual/ini.core.php#ini.zend-extension).

```ini
extension=/home/site/wwwroot/bin/mongodb.so
zend_extension=/home/site/wwwroot/bin/xdebug.so
```

Чтобы изменения вступили в силу, перезапустите приложение.

::: zone-end

## <a name="access-diagnostic-logs"></a>Доступ к журналам диагностики

::: zone pivot="platform-windows"  

Используйте стандартную служебную программу [error_log ()](https://php.net/manual/function.error-log.php) , чтобы убедиться, что журналы диагностики отображаются в службе приложений Azure.

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

::: zone-end

## <a name="troubleshooting"></a>Устранение неполадок

Если рабочее приложение PHP в Службе приложений ведет себя иначе или сообщает об ошибках, попробуйте сделать следующее:

- [Получите доступ к потоку журнала](#access-diagnostic-logs).
- Протестируйте приложение локально в рабочем режиме. Служба приложений запускает приложение в рабочем режиме, поэтому необходимо убедиться, что проект работает в режиме рабочей среды локально. Пример:
    - В зависимости от содержимого файла *composer.json* для рабочего режима могут быть установлены разные пакеты (`require` или `require-dev`).
    - Некоторые веб-платформы в рабочем режиме могут выполнять другие операции при развертывании статических файлов.
    - Некоторые веб-платформы могут использовать специальные скрипты запуска в рабочем режиме.
- Выполните свое приложение в Службе приложений в режиме отладки. Например, в [Laravel](https://meanjs.org/) вы можете настроить для приложения вывод сообщений отладки в производственной среде, [задав для параметра приложения `APP_DEBUG` значение `true`](configure-common.md#configure-app-settings).

::: zone pivot="platform-linux"

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. по приложению PHP с MySQL](tutorial-php-mysql-app.md)

::: zone pivot="platform-linux"

> [!div class="nextstepaction"]
> [Служба приложений под управлением Linux: вопросы и ответы](faq-app-service-linux.md)

::: zone-end

