---
title: Руководство по созданию многоконтейнерного приложения
description: Узнайте, как создать в Службе приложений Azure многоконтейнерное приложение, содержащее приложение WordPress и контейнер MySQL, и настроить приложение WordPress.
keywords: служба приложений azure, веб-приложение, linux, docker, compose, многоконтейнерное, несколько контейнеров, веб-приложение для контейнеров, много контейнеров, контейнер, wordpress, azure db для mysql, рабочая база данных с контейнерами
author: msangapu-msft
ms.topic: tutorial
ms.date: 10/31/2020
ms.author: msangapu
ms.custom: cli-validate, devx-track-azurecli
ms.openlocfilehash: dee00c6f733cfbebf68276ee4b54f91b8e2cb35b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765545"
---
# <a name="tutorial-create-a-multi-container-preview-app-in-web-app-for-containers"></a>Руководство по Создание многоконтейнерного приложения (предварительная версия) на платформе Azure "Веб-приложение для контейнеров"

> [!NOTE]
> Многоконтейнерное приложение поддерживается в предварительной версии.

Платформа [Веб-приложение для контейнеров](overview.md#app-service-on-linux) предоставляет гибкие возможности для использования образов Docker. В этом руководстве объясняется, как создать многоконтейнерное приложение с помощью WordPress и MySQL. Примеры из этого руководства вы будете выполнять в Cloud Shell, но эти команды также можно выполнить на локальном компьютере с помощью программы командной строки [Azure CLI](/cli/azure/install-azure-cli) (версии 2.0.32 или более поздней).

В этом руководстве описано следующее:

> [!div class="checklist"]
> * преобразование конфигурации Docker Compose для работы на платформе "Веб-приложение для контейнеров";
> * развертывание многоконтейнерного приложения в Azure;
> * добавление параметров приложения;
> * использование постоянного хранилища для контейнеров;
> * Подключение к базе данных Azure для MySQL
> * Устранение неполадок

[!INCLUDE [Free trial note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для изучения этого учебника требуется опыт работы с [Docker Compose](https://docs.docker.com/compose/).

## <a name="download-the-sample"></a>Скачивание примера приложения

При работе с этим руководством вы будете использовать файл из [Docker Compose](https://docs.docker.com/compose/wordpress/#define-the-project), но измените его, добавив сведения о Базе данных Azure для MySQL, постоянном хранилище и Redis. Файл конфигурации можно найти в [репозитории примеров для Azure](https://github.com/Azure-Samples/multicontainerwordpress). Поддерживаемые параметры конфигурации см. в разделе [Docker Compose options](configure-custom-container.md#docker-compose-options) (Параметры Docker Compose).

[!code-yml[Main](../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

В Cloud Shell создайте каталог tutorial и перейдите в него.

```bash
mkdir tutorial

cd tutorial
```

Затем выполните следующую команду, чтобы клонировать репозиторий с примером приложения в локальный каталог tutorial. Затем перейдите в каталог `multicontainerwordpress`.

```bash
git clone https://github.com/Azure-Samples/multicontainerwordpress

cd multicontainerwordpress
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [resource group intro text](../../includes/resource-group.md)]

В Cloud Shell создайте группу ресурсов с помощью команды [`az group create`](/cli/azure/group#az_group_create). В следующем примере создается группа ресурсов с именем *myResourceGroup* в расположении *Центрально-южная часть США*. Чтобы просмотреть все поддерживаемые расположения для службы приложений в Linux на уровне **Стандартный**, выполните команду [`az appservice list-locations --sku S1 --linux-workers-enabled`](/cli/azure/appservice#az_appservice_list_locations).

```azurecli-interactive
az group create --name myResourceGroup --location "South Central US"
```

Обычно группа ресурсов и ресурсы создаются в ближайших регионах.

По завершении команды в выходных данных JSON будут отображаться свойства группы ресурсов.

## <a name="create-an-azure-app-service-plan"></a>Создание плана службы приложений Azure

В Cloud Shell создайте план службы приложений в группе ресурсов с помощью команды [`az appservice plan create`](/cli/azure/appservice/plan#az_appservice_plan_create).

<!-- [!INCLUDE [app-service-plan](app-service-plan-linux.md)] -->

В следующем примере создается план службы приложений с именем `myAppServicePlan` в ценовой категории **Стандартный** (`--sku S1`) в контейнере Linux (`--is-linux`).

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

После создания плана службы приложений в Cloud Shell отображается информация следующего вида:

<pre>
{
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "South Central US",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "linux",
  "location": "South Central US",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
}
</pre>

### <a name="docker-compose-with-wordpress-and-mysql-containers"></a>Docker Compose с контейнерами WordPress и MySQL

## <a name="create-a-docker-compose-app"></a>Создание приложения Docker Compose

В Cloud Shell создайте многоконтейнерное [веб-приложение](overview.md) в рамках плана службы приложений `myAppServicePlan` с помощью команды [az webapp create](/cli/azure/webapp#az_webapp_create). Не забудьте указать уникальное имя приложения вместо _\<app-name>_ .

```azurecli-interactive
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

Когда веб-приложение будет создано, в Cloud Shell отобразится примерно следующее:

<pre>
{
  "additionalProperties": {},
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;app-name&gt;.azurewebsites.net",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>

### <a name="browse-to-the-app"></a>Переход в приложение

Перейдите в развернутое приложение по адресу `http://<app-name>.azurewebsites.net`. Для загрузки приложения может потребоваться несколько минут. Если произойдет ошибка, подождите несколько минут, а затем обновите страницу в браузере. Если возникнет проблема, которую нужно будет устранить, просмотрите [журналы контейнера](#find-docker-container-logs).

![Пример многоконтейнерного приложения на платформе "Веб-приложение для контейнеров"][1]

**Поздравляем!** Вы создали многоконтейнерное приложение на платформе "Веб-приложение для контейнеров". Далее вам предстоит настроить приложение для использования базы данных Azure для MySQL. Не устанавливайте WordPress в этом этапе.

## <a name="connect-to-production-database"></a>Подключение к рабочей базе данных

Не рекомендуется использовать контейнеры базы данных в рабочей среде. Локальные контейнеры не являются масштабируемыми. Вместо этого вы будете использовать базу данных Azure для MySQL, которую можно масштабировать.

### <a name="create-an-azure-database-for-mysql-server"></a>Создайте сервер базы данных Azure для MySQL.

Создайте сервер базы данных Azure для MySQL с помощью команды [`az mysql server create`](/cli/azure/mysql/server#az_mysql_server_create).

В следующей команде замените _&lt;mysql-server-name>_ именем своего сервера MySQL везде, где встречается этот заполнитель. Допустимые символы: `a-z`, `0-9` и `-`. Это имя является частью имени узла сервера MySQL (`<mysql-server-name>.database.windows.net`). Оно должно быть глобально уникальным.

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql-server-name>  --location "South Central US" --admin-user adminuser --admin-password My5up3rStr0ngPaSw0rd! --sku-name B_Gen5_1 --version 5.7
```

Создание сервера может занять несколько минут. После создания сервера MySQL в Cloud Shell отображаются следующие сведения:

<pre>
{
  "administratorLogin": "adminuser",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "&lt;mysql-server-name&gt;.database.windows.net",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/&lt;mysql-server-name&gt;",
  "location": "southcentralus",
  "name": "&lt;mysql-server-name&gt;",
  "resourceGroup": "myResourceGroup",
  ...
}
</pre>

### <a name="configure-server-firewall"></a>Настройка брандмауэра сервера

Создайте правило брандмауэра для сервера MySQL, чтобы разрешить подключения клиентов, выполнив команду [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule#az_mysql_server_firewall_rule_create). Если для начального и конечного IP-адресов задано значение 0.0.0.0, брандмауэр открыт только для других ресурсов Azure.

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP]
> Вы можете применить еще более строгие ограничения в правиле брандмауэра, [разрешив только исходящие IP-адреса, используемые приложением](overview-inbound-outbound-ips.md#find-outbound-ips).
>

### <a name="create-the-wordpress-database"></a>Создание базы данных WordPress

```azurecli-interactive
az mysql db create --resource-group myResourceGroup --server-name <mysql-server-name> --name wordpress
```

После создания базы данных в Cloud Shell отображаются следующие сведения:

<pre>
{
  "additionalProperties": {},
  "charset": "latin1",
  "collation": "latin1_swedish_ci",
  "id": "/subscriptions/12db1644-4b12-4cab-ba54-8ba2f2822c1f/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/&lt;mysql-server-name&gt;/databases/wordpress",
  "name": "wordpress",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DBforMySQL/servers/databases"
}
</pre>

### <a name="configure-database-variables-in-wordpress"></a>Настройка переменных базы данных в WordPress

Чтобы подключить приложение WordPress к новому серверу MySQL, задайте несколько переменных среды для WordPress, включая путь к выданному центром сертификации SSL-сертификату, который определяется с помощью `MYSQL_SSL_CA`. Сведения о сертификате [Baltimore CyberTrust Root](https://www.digicert.com/digicert-root-certificates.htm) от [DigiCert](https://www.digicert.com/) см. в разделе о [пользовательском образе](#use-a-custom-image-for-mysql-ssl-and-other-configurations) ниже.

Чтобы внести эти изменения, выполните команду [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) в Cloud Shell. Параметры приложения чувствительны к регистру и используются с разделителями-пробелами.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WORDPRESS_DB_HOST="<mysql-server-name>.mysql.database.azure.com" WORDPRESS_DB_USER="adminuser@<mysql-server-name>" WORDPRESS_DB_PASSWORD="My5up3rStr0ngPaSw0rd!" WORDPRESS_DB_NAME="wordpress" MYSQL_SSL_CA="BaltimoreCyberTrustroot.crt.pem"
```

После создания параметров приложения в Cloud Shell отображается информация следующего вида:

<pre>
[
  {
    "name": "WORDPRESS_DB_HOST",
    "slotSetting": false,
    "value": "&lt;mysql-server-name&gt;.mysql.database.azure.com"
  },
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@&lt;mysql-server-name&gt;"
  },
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WORDPRESS_DB_PASSWORD",
    "slotSetting": false,
    "value": "My5up3rStr0ngPaSw0rd!"
  },
  {
    "name": "MYSQL_SSL_CA",
    "slotSetting": false,
    "value": "BaltimoreCyberTrustroot.crt.pem"
  }
]
</pre>

Подробные сведения о переменных среды см. в разделе [Configure environment variables](configure-custom-container.md#configure-environment-variables) (Настройка переменных среды).

### <a name="use-a-custom-image-for-mysql-ssl-and-other-configurations"></a>Использование пользовательского образа для SSL-соединения с MySQL и других конфигураций

По умолчанию SSL-соединение используется в базе данных Azure для MySQL. Но чтобы использовать это соединение в WordPress, нужно задать дополнительную конфигурацию. "Официальный образ" WordPress не содержит таких настроек, но для удобства мы подготовили [пользовательский образ](https://github.com/Azure-Samples/multicontainerwordpress). На практике требуемые изменения нужно добавлять в свой образ.

Пользовательский образ создан на основе официального образа [WordPress из Docker Hub](https://hub.docker.com/_/wordpress/). В этот пользовательский образ внесены следующие изменения для базы данных Azure для MySQL:

* [добавлен файл сертификата Baltimore CyberTrust Root для SSL-соединения с MySQL;](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L61)
* [задан параметр приложения для SSL-сертификата MySQL, предоставленного центром сертификации, в файле wp-config.php WordPress;](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L163)
* [добавлено определение WordPress для MYSQL_CLIENT_FLAGS, необходимое для SSL-соединения с MySQL.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L164)

Для Redis (используется в следующем разделе) внесены следующие изменения:

* [добавлено расширение PHP для Redis версии 4.0.2;](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [добавлена команда распаковки, необходимая для извлечения файла;](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [добавлен подключаемый модуль Redis Object Cache 1.3.8 для WordPress.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [задан параметр приложения для имени узла Redis в файле wp-config.php WordPress.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

Чтобы применить пользовательский образ, нужно обновить файл docker-compose-wordpress.yml. В Cloud Shell введите `nano docker-compose-wordpress.yml`, чтобы открыть текстовый редактор Nano. Измените `image: wordpress` на `image: mcr.microsoft.com/azuredocs/multicontainerwordpress`. Вам больше не понадобится контейнер базы данных. Удалите разделы `db`, `environment`, `depends_on` и `volumes` из файла конфигурации. Файл должен выглядеть примерно так, как в следующем коде:

```yaml
version: '3.3'

services:
   wordpress:
     image: mcr.microsoft.com/azuredocs/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always
```

Сохраните изменения и выйдите из редактора Nano. Выполните команду `^O`, чтобы сохранить файл, и `^X` — чтобы выйти.

### <a name="update-app-with-new-configuration"></a>Обновление приложения с указанием новой конфигурации

В Cloud Shell измените конфигурацию многоконтейнерного [веб-приложения](overview.md) с помощью команды [az webapp config container set](/cli/azure/webapp/config/container#az_webapp_config_container_set). Не забудьте заменить _\<app-name>_ именем веб-приложения, созданного ранее.

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

После изменения настроек приложения в Cloud Shell отображаются следующие сведения:

<pre>
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICB3b3JkcHJlc3M6CiAgICAgaW1hZ2U6IG1zYW5nYXB1L3dvcmRwcmVzcwogICAgIHBvcnRzOgogICAgICAgLSAiODAwMDo4MCIKICAgICByZXN0YXJ0OiBhbHdheXM="
  }
]
</pre>

### <a name="browse-to-the-app"></a>Переход в приложение

Перейдите в развернутое приложение по адресу `http://<app-name>.azurewebsites.net`. Теперь в приложении используется база данных Azure для MySQL.

![Пример многоконтейнерного приложения на платформе "Веб-приложение для контейнеров"][1]

## <a name="add-persistent-storage"></a>Добавление постоянного хранилища

Ваше многоконтейнерное приложение теперь работает на платформе "Веб-приложение для контейнеров". Но если установить WordPress сейчас и после этого перезапустить приложение, вы обнаружите, что данные установки WordPress удалены. Это происходит потому, что сейчас в конфигурации Docker Compose указано расположение для хранения в контейнере. Файлы, установленные в контейнере, исчезают после перезапуска приложения. Из этого раздела вы узнаете, как [добавить постоянное хранилище](configure-custom-container.md#use-persistent-shared-storage) в контейнер WordPress.

### <a name="configure-environment-variables"></a>Настройка переменных среды

Чтобы использовать постоянное хранилище, вам нужно включить этот параметр в Службе приложений. Чтобы внести это изменение, выполните команду [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) в Cloud Shell. Параметры приложения чувствительны к регистру и используются с разделителями-пробелами.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

После создания параметров приложения в Cloud Shell отображается информация следующего вида:

<pre>
[
  &lt; JSON data removed for brevity. &gt;
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  }
]
</pre>

### <a name="modify-configuration-file"></a>Изменение файла конфигурации

В Cloud Shell введите `nano docker-compose-wordpress.yml`, чтобы открыть текстовый редактор Nano.

Параметр `volumes` сопоставляет файловую систему с каталогом в контейнере. `${WEBAPP_STORAGE_HOME}` — это переменная среды в Службе приложений, которая сопоставляется с постоянным хранилищем для вашего приложения. Используйте эту переменную среды в параметре volumes, чтобы файлы WordPress устанавливались в постоянное хранилище, а не в контейнер. Внесите в файл следующие изменения:

Добавьте в раздел `wordpress` параметр `volumes`, как показано в следующем коде:

```yaml
version: '3.3'

services:
   wordpress:
     image: mcr.microsoft.com/azuredocs/multicontainerwordpress
     volumes:
      - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
     ports:
       - "8000:80"
     restart: always
```

### <a name="update-app-with-new-configuration"></a>Обновление приложения с указанием новой конфигурации

В Cloud Shell измените конфигурацию многоконтейнерного [веб-приложения](overview.md) с помощью команды [az webapp config container set](/cli/azure/webapp/config/container#az_webapp_config_container_set). Не забудьте указать уникальное имя приложения вместо _\<app-name>_ .

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

После выполнения команда вернет результат, аналогичный следующему примеру:

<pre>
[
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  },
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
</pre>

### <a name="browse-to-the-app"></a>Переход в приложение

Перейдите в развернутое приложение по адресу `http://<app-name>.azurewebsites.net`.

Теперь контейнер WordPress использует базу данных Azure для MySQL и постоянное хранилище.

## <a name="add-redis-container"></a>Добавление контейнера Redis

 Официальный образ WordPress не содержит зависимостей для Redis. Эти зависимости и дополнительные конфигурации, необходимые для использования Redis с WordPress, подготовлены для вас в этом [пользовательском образе](https://github.com/Azure-Samples/multicontainerwordpress). На практике требуемые изменения нужно добавлять в свой образ.

Пользовательский образ создан на основе официального образа [WordPress из Docker Hub](https://hub.docker.com/_/wordpress/). В пользовательский образ для Redis внесены следующие изменения:

* [добавлено расширение PHP для Redis версии 4.0.2;](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [добавлена команда распаковки, необходимая для извлечения файла;](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [добавлен подключаемый модуль Redis Object Cache 1.3.8 для WordPress.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [задан параметр приложения для имени узла Redis в файле wp-config.php WordPress.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

Добавьте контейнер Redis в конец файла конфигурации, как показано в следующем примере:

```yaml
version: '3.3'

services:
   wordpress:
     image: mcr.microsoft.com/azuredocs/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always

   redis:
     image: mcr.microsoft.com/oss/bitnami/redis:6.0.8
     environment: 
      - ALLOW_EMPTY_PASSWORD=yes
     restart: always
```

### <a name="configure-environment-variables"></a>Настройка переменных среды

Чтобы использовать Redis, включите параметр `WP_REDIS_HOST` в Службе приложений. Это *обязательный параметр* для взаимодействия WordPress с узлом Redis. Чтобы внести это изменение, выполните команду [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) в Cloud Shell. Параметры приложения чувствительны к регистру и используются с разделителями-пробелами.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WP_REDIS_HOST="redis"
```

После создания параметров приложения в Cloud Shell отображается информация следующего вида:

<pre>
[
  &lt; JSON data removed for brevity. &gt;
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@&lt;mysql-server-name&gt;"
  },
  {
    "name": "WP_REDIS_HOST",
    "slotSetting": false,
    "value": "redis"
  }
]
</pre>

### <a name="update-app-with-new-configuration"></a>Обновление приложения с указанием новой конфигурации

В Cloud Shell измените конфигурацию многоконтейнерного [веб-приложения](overview.md) с помощью команды [az webapp config container set](/cli/azure/webapp/config/container#az_webapp_config_container_set). Не забудьте указать уникальное имя приложения вместо _\<app-name>_ .

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

После выполнения команда вернет результат, аналогичный следующему примеру:

<pre>
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
</pre>

### <a name="browse-to-the-app"></a>Переход в приложение

Перейдите в развернутое приложение по адресу `http://<app-name>.azurewebsites.net`.

Выполните шаги по установке WordPress.

### <a name="connect-wordpress-to-redis"></a>Подключение WordPress к Redis

Войдите в WordPress, используя учетную запись администратора. В области навигации слева последовательно выберите **Plugins** (Подключаемые модули) и **Installed Plugins** (Установленные подключаемые модули).

![Выбор подключаемых модулей WordPress][2]

Отображение всех подключаемых модулей

На странице подключаемых модулей найдите **Redis Object Cache** (Кэш объектов Redis) и щелкните **Activate** (Активировать).

![Активация Redis][3]

Щелкните **Параметры**.

![Щелкните Settings (Параметры).][4]

Нажмите кнопку **Enable Object Cache** (Включить Object Cache).

![Нажатие кнопки "Enable Object Cache" (Включить Object Cache)][5]

WordPress подключится к серверу Redis. **Состояние** подключения отобразится на той же странице.

![WordPress подключится к серверу Redis. **Состояние** подключения, отображаемое на той же странице][6]

**Поздравляем!** Вы подключили WordPress к Redis. Сейчас в готовом для рабочей среды приложении используется **база данных Azure для MySQL, постоянное хранилище и Redis**. Теперь вы можете горизонтального увеличивать масштаб плана службы приложений до нескольких экземпляров.

## <a name="find-docker-container-logs"></a>Поиск журналов контейнера Docker

Если при использовании нескольких контейнеров возникли проблемы, вы можете перейти к журналам контейнера по адресу `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.

Результат будет выглядеть примерно так:

<pre>
[
   {
      "machineName":"RD00XYZYZE567A",
      "lastUpdated":"2018-05-10T04:11:45Z",
      "size":25125,
      "href":"https://&lt;app-name&gt;.scm.azurewebsites.net/api/vfs/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log",
      "path":"/home/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log"
   }
]
</pre>

Вы увидите журнал для каждого контейнера и дополнительный журнал для родительского процесса. Скопируйте соответствующее значение `href` в браузер, чтобы просмотреть журнал.

[!INCLUDE [Clean-up section](../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:
> [!div class="checklist"]
> * преобразование конфигурации Docker Compose для работы на платформе "Веб-приложение для контейнеров";
> * развертывание многоконтейнерного приложения в Azure;
> * добавление параметров приложения;
> * использование постоянного хранилища для контейнеров;
> * Подключение к базе данных Azure для MySQL
> * Устранение неполадок

Перейдите к следующему руководству, чтобы научиться сопоставлять пользовательские DNS-имена с приложением.

> [!div class="nextstepaction"]
> [Руководство. Сопоставление настраиваемого DNS-имени с приложением](app-service-web-tutorial-custom-domain.md)

Также ознакомьтесь с другими ресурсами:

> [!div class="nextstepaction"]
> [Настройка пользовательского контейнера](configure-custom-container.md)

<!--Image references-->
[1]: ./media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png
[2]: ./media/tutorial-multi-container-app/wordpress-plugins.png
[3]: ./media/tutorial-multi-container-app/activate-redis.png
[4]: ./media/tutorial-multi-container-app/redis-settings.png
[5]: ./media/tutorial-multi-container-app/enable-object-cache.png
[6]: ./media/tutorial-multi-container-app/redis-connected.png
