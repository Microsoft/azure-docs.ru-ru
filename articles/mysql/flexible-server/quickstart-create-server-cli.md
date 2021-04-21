---
title: Краткое руководство. Создание сервера Базы данных Azure для MySQL (Гибкий сервер) с помощью Azure CLI
description: В этом кратком руководстве описывается, как создать Базу данных Azure для MySQL (Гибкий сервер) в группе ресурсов Azure с помощью Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 9/21/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 7addfc3a0d91b85c4d63afa4ee6a55b5202c3855
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107770243"
---
# <a name="quickstart-create-an-azure-database-for-mysql-flexible-server-using-azure-cli"></a>Краткое руководство. Создание Базы данных Azure для MySQL (Гибкий сервер) с помощью Azure CLI

В этом кратком руководстве описывается, как с помощью команд [Azure CLI](/cli/azure/get-started-with-azure-cli) в [Azure Cloud Shell](https://shell.azure.com) за 5 минут создать гибкий сервер Базы данных Azure для MySQL. Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.

> [!IMPORTANT] 
> Сейчас предоставляется общедоступная предварительная версия Гибкого сервера Базы данных Azure для MySQL.

## <a name="launch-azure-cloud-shell"></a>Запуск Azure Cloud Shell

[Azure Cloud Shell](../../cloud-shell/overview.md) — это бесплатная интерактивная оболочка, с помощью которой можно выполнять действия, описанные в этой статье. Она включает предварительно установленные общие инструменты Azure и настроена для использования с вашей учетной записью.

Чтобы открыть Cloud Shell, просто выберите **Попробовать** в правом верхнем углу блока кода. Кроме того, Cloud Shell можно открыть в отдельной вкладке браузера. Для этого перейдите на страницу [https://shell.azure.com/bash](https://shell.azure.com/bash). Нажмите кнопку **Копировать**, чтобы скопировать блоки кода. Вставьте код в Cloud Shell и нажмите клавишу **ВВОД**, чтобы выполнить его.

Если вы решили установить и использовать CLI локально, для выполнения инструкций, приведенных в этом кратком руководстве, вам потребуется Azure CLI 2.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="prerequisites"></a>Предварительные требования

Вам потребуется выполнить вход в учетную запись с помощью команды [az login](/cli/azure/reference-index#az_login). Обратите внимание на свойство **идентификатора**, которое ссылается на **идентификатор подписки** вашей учетной записи Azure.

```azurecli-interactive
az login
```

Выберите конкретную подписку вашей учетной записи, выполнив команду [az account set](/cli/azure/account#az_account_set). Запишите значение **идентификатора** из выходных данных команды **az login**, чтобы использовать его в команде в качестве значения аргумента **подписки**. Если вы используете несколько подписок, выберите соответствующую, в которой за ресурс будет взиматься плата. Чтобы отобразить все ваши подписки, выполните команду [az account list](/cli/azure/account#az_account_list).

```azurecli-interactive
az account set --subscription <subscription id>
```

## <a name="create-a-flexible-server"></a>Создание гибкого сервера

Создайте [группу ресурсов Azure](../../azure-resource-manager/management/overview.md) с помощью команды `az group create`, а затем создайте гибкий сервер MySQL в этой группе ресурсов. Необходимо указать уникальное имя. В следующем примере создается группа ресурсов с именем `myresourcegroup` в расположении именем `eastus2`.

```azurecli-interactive
az group create --name myresourcegroup --location eastus2
```

Создайте гибкий сервер с помощью команды `az mysql flexible-server create`. Сервер может управлять несколькими базами данных. Следующая команда создает сервер, используя параметры и значения по умолчанию для [локального контекста](/cli/azure/local-context) Azure CLI: 

```azurecli-interactive
az mysql flexible-server create
```

Сервер создается со следующими атрибутами: 
- Автоматически созданное имя сервера, имя пользователя и пароль администратора, имя группы ресурсов (если не указано в локальном контексте) и расположение, которое выбрано для группы ресурсов. 
- Службы по умолчанию для остальных конфигураций сервера: уровень вычислений (с накапливаемыми ресурсами), размер (уровень) вычислительных ресурсов или номер SKU (B1MS), срок хранения резервных копий (7 дней) и версия MySQL (5.7).
- Для метода подключения по умолчанию назначается вариант "Закрытый доступ (интеграция с виртуальной сетью)" с автоматически созданными виртуальной сетью и подсетью.

> [!NOTE] 
> После создания сервера нельзя изменить метод подключения. Например, если во время создания вы выбрали *Закрытый доступ (интеграция с виртуальной сетью)* , то позднее не сможете назначить *Открытый доступ (разрешенные IP-адреса)* . Мы настоятельно рекомендуем создать сервер с закрытым доступом, чтобы безопасно обращаться к нему через интеграцию с виртуальной сетью. Дополнительные сведения о закрытом доступе см. в статье с [основными понятиями](./concepts-networking.md).

Если вы хотите изменить значения по умолчанию, воспользуйтесь [справочной документацией](/cli/azure/mysql/flexible-server) по Azure CLI, где описаны все настраиваемые параметры интерфейса командной строки. 

Ниже приведен пример выходных данных: 

```json
Command group 'mysql flexible-server' is in preview. It may be changed/removed in a future release.
Creating Resource Group 'groupXXXXXXXXXX'...
Creating new vnet "serverXXXXXXXXXVNET" in resource group "groupXXXXXXXXXX"...
Creating new subnet "serverXXXXXXXXXSubnet" in resource group "groupXXXXXXXXXX" and delegating it to "Microsoft.DBforMySQL/flexibleServers"...
Creating MySQL Server 'serverXXXXXXXXX' in group 'groupXXXXXXXXXX'...
Your server 'serverXXXXXXXXX' is using sku 'Standard_B1ms' (Paid Tier). Please refer to https://aka.ms/mysql-pricing for pricing details
Creating MySQL database 'flexibleserverdb'...
Make a note of your password. If you forget, you would have to reset your password with 'az mysql flexible-server update -n serverXXXXXXXXX -g groupXXXXXXXXXX -p <new-password>'.
{
  "connectionString": "server=serverXXXXXXXXX.mysql.database.azure.com;database=flexibleserverdb;uid=secureusername;pwd=securepasswordstring",
  "databaseName": "flexibleserverdb",
  "host": "serverXXXXXXXXX.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/groupXXXXXXXXXX/providers/Microsoft.DBforMySQL/flexibleServers/serverXXXXXXXXX",
  "location": "East US 2",
  "password": "securepasswordstring",
  "resourceGroup": "groupXXXXXXXXXX",
  "skuname": "Standard_B1ms",
  "subnetId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/groupXXXXXXXXXX/providers/Microsoft.Network/virtualNetworks/serverXXXXXXXXXVNET/subnets/serverXXXXXXXXXSubnet",
  "username": "secureusername",
  "version": "5.7"
}
```

Если вы хотите изменить значения по умолчанию, воспользуйтесь [справочной документацией](/cli/azure/mysql/flexible-server) по Azure CLI, где описаны все настраиваемые параметры интерфейса командной строки. 

## <a name="create-a-database"></a>Создание базы данных
Выполните следующую команду, чтобы создать базу данных **newdatabase**, если она еще не создана.

```azurecli-interactive
az mysql flexible-server db create -d newdatabase
```

> [!NOTE]
> Подключитесь к базе данных Azure для MySQL через порт 3306. Если вы пытаетесь подключиться из корпоративной сети, исходящий трафик через порт 3306 может быть запрещен. В таком случае вы не сможете подключиться к серверу. Для этого ваш ИТ-отдел должен открыть порт 3306.

## <a name="get-the-connection-information"></a>Получение сведений о подключении

Чтобы подключиться к серверу, необходимо указать сведения об узле и учетные данные для доступа.

```azurecli-interactive
az mysql flexible-server show --resource-group myresourcegroup --name mydemoserver
```

Результаты выводятся в формате JSON. Запишите значения **fullyQualifiedDomainName** и **administratorLogin**. Ниже приведен пример выходных данных JSON: 

```json
{
  "administratorLogin": "myadminusername",
  "administratorLoginPassword": null,
  "delegatedSubnetArguments": {
    "subnetArmResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/mydemoserverVNET/subnets/mydemoserverSubnet"
  },
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/flexibleServers/mydemoserver",
  "location": "East US 2",
  "name": "mydemoserver",
  "publicNetworkAccess": "Disabled",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 0,
    "name": "Standard_B1ms",
    "tier": "Burstable"
  },
  "storageProfile": {
    "backupRetentionDays": 7,
    "fileStorageSkuName": "Premium_LRS",
    "storageAutogrow": "Disabled",
    "storageIops": 0,
    "storageMb": 10240
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/flexibleServers",
  "version": "5.7"
}
```

## <a name="connect-and-test-the-connection-using-azure-cli"></a>Подключение и проверка подключения с помощью Azure CLI

Гибкий сервер базы данных Azure для MySQL позволяет подключаться к серверу MySQL с помощью команды Azure CLI ```az mysql flexible-server connect```. Эта команда позволяет проверить возможность подключения к серверу базы данных, быстро создать базу данных для начала работы и выполнить запросы непосредственно к серверу без необходимости установки mysql.exe или MySQL Workbench.  Вы можете также запустить команду в интерактивном режиме, чтобы выполнить несколько запросов.

Выполните следующий скрипт, чтобы протестировать и проверить подключение к базе данных из окружения разработки.

```azurecli-interactive
az mysql flexible-server connect -n <servername> -u <username> -p <password> -d <databasename>
```
**Пример**.
```azurecli-interactive
az mysql flexible-server connect -n mysqldemoserver1 -u dbuser -p "dbpassword" -d newdatabase
```
Если подключение выполнено успешно, отобразятся следующие выходные данные:

```output
Command group 'mysql flexible-server' is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus
Connecting to newdatabase database.
Successfully connected to mysqldemoserver1.
```
Если установить подключение не удалось, воспользуйтесь приведенными ниже решениями:
- Проверьте, открыт ли порт 3306 на клиентском компьютере.
- Проверьте, правильно ли указаны имя пользователя и пароль администратора сервера.
- Проверьте, настроено ли правило брандмауэра для клиентского компьютера.
- Если вы настроили сервер с закрытым доступом в виртуальной сети, убедитесь, что клиентский компьютер находится в той же виртуальной сети.

Используйте следующую команду, чтобы выполнить один запрос с помощью аргумента ```--querytext```, ```-q```.

```azurecli-interactive
az mysql flexible-server connect -n <server-name> -u <username> -p "<password>" -d <database-name> --querytext "<query text>"
```

**Пример**.
```azurecli-interactive
az mysql flexible-server connect -n mysqldemoserver1 -u dbuser -p "dbpassword" -d newdatabase -q "select * from table1;" --output table
```
Дополнительные сведения об использовании команды ```az mysql flexible-server connect``` см. в документации по [подключению и запросам](connect-azure-cli.md).

## <a name="connect-using-mysql-command-line-client"></a>Подключение с помощью клиента командной строки mysql

Если вы создали гибкий сервер с закрытым доступом (интеграция с виртуальной сетью), к этому серверу нужно будет подключаться с другого ресурса в той же виртуальной сети. Например, можно создать виртуальную машину и добавить ее в виртуальную сеть, созданную для гибкого сервера. Дополнительные сведения см. в [документации по настройке частного доступа](how-to-manage-virtual-network-portal.md).

Если вы создали гибкий сервер с открытым доступом (разрешенные IP-адреса), вы можете добавить локальный IP-адрес в список правил брандмауэра на этом сервере. Пошаговые инструкции см. в [документации по созданию и администрированию правил брандмауэра](how-to-manage-firewall-portal.md).

Вы можете использовать [mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) или [MySQL Workbench](./connect-workbench.md), чтобы подключиться к серверу из локальной среды. Гибкий сервер базы данных Azure для MySQL поддерживает подключение службы MySQL к клиентским приложениям с помощью протокола TLS, ранее известного как SSL. TLS — это стандартный протокол, обеспечивающий шифрование сетевых подключений между сервером базы данных и клиентскими приложениями, что позволяет обеспечивать соответствие требованиям. Для подключения к гибкому серверу MySQL потребуется скачать [общедоступный SSL-сертификат](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) для проверки центра сертификации. Дополнительные сведения о зашифрованных подключениях или отключении SSL см. в статье [Подключение к базе данных Azure для MySQL — гибкому серверу с зашифрованными подключениями](how-to-connect-tls-ssl.md).

В следующем примере показано, как подключиться к серверу PostgreSQL с помощью служебной программы командной строки psql, Сначала необходимо установить командную строку MySQL, если она еще не установлена. Загрузите сертификат DigiCertGlobalRootCA, необходимый для SSL-соединений. Для принудительной проверки сертификата TLS/SSL используйте параметр--SSL-Mode = REQUIRED строки подключения. Передайте путь к локальному файлу сертификата параметру--SSL-CA. Замените в ней предложенные значения реальными значениями имени сервера и пароля.

```bash
sudo apt-get install mysql-client
wget --no-check-certificate https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem
mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p --ssl-mode=REQUIRED --ssl-ca=DigiCertGlobalRootCA.crt.pem
```

Если вы выполнили подготовку гибкого сервера с использованием **открытого доступа**, вы также можете использовать [Azure Cloud Shell](https://shell.azure.com/bash) для подключения к гибкому серверу с помощью предустановленного клиента MySQL, как показано ниже:

Чтобы использовать Azure Cloud Shell для подключения к гибкому серверу, вам потребуется разрешить сетевой доступ из Azure Cloud Shell к гибкому серверу. Для этого можно перейти в колонку **Сеть** на портале Azure для гибкого сервера MySQL, установить флажок для параметра "Разрешить открытый доступ от любой службы Azure к этому серверу" в разделе **Брандмауэр** и нажать кнопку "Сохранить".

 > :::image type="content" source="./media/quickstart-create-server-portal/allow-access-to-any-azure-service.png" alt-text="Снимок экрана, показывающий, как разрешить Azure Cloud Shell доступ к гибкому серверу MySQL для настройки сети общего доступа.":::
 
 
> [!NOTE]
> Устанавливайте флажок **Разрешить открытый доступ от любой службы Azure к этому серверу** только для целей разработки и тестирования. Он настраивает брандмауэр таким образом, чтобы разрешить подключения от IP-адресов, выделенных любой службе или ресурсу Azure, включая подключения из подписок других клиентов.

Щелкните **Попробовать**, чтобы запустить Azure Cloud Shell и воспользоваться следующими командами для подключения к гибкому серверу. В этой команде укажите фактические значения имени сервера, имени пользователя и пароля. 

```azurecli-interactive
wget --no-check-certificate https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem
mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p --ssl=true --ssl-ca=DigiCertGlobalRootCA.crt.pem
```
> [!IMPORTANT]
> При подключении к гибкому серверу с помощью Azure Cloud Shell вам потребуется использовать параметр --ssl=true, а не --ssl-mode=REQUIRED.
> Основная причина — Azure Cloud Shell поставляется с предварительно установленным клиентом mysql.exe из дистрибутива MariaDB, для которого требуется параметр --ssl, а для клиента MySQL из Oracle — параметр --ssl-mode.

Если вы видите следующее сообщение об ошибке при подключении к гибкому серверу (после выполнения команды ранее), вы не настроили правило брандмауэра с помощью упомянутого ранее параметра "Разрешить общий доступ от любой службы Azure к этому серверу" или не сохранили этот параметр. Настройте брандмауэр и повторите попытку.

ОШИБКА 2002 (HY000): Can't connect to MySQL server on <servername> (115) (Не удалось подключиться к серверу MySQL по адресу)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если эти ресурсы не требуются для изучения другого руководства, вы можете их удалить. Для этого выполните следующую команду:

```azurecli-interactive
az group delete --name myresourcegroup
```

Если вы хотите удалить только что созданный сервер, выполните команду `az mysql server delete`.

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Следующие шаги

>[!div class="nextstepaction"]
> [Подключение и запрос с помощью Azure CLI](connect-azure-cli.md)
> [Подключение к базе данных Azure для MySQL — гибкому серверу с зашифрованными подключениями](how-to-connect-tls-ssl.md)
> [Создание веб-приложения PHP (Laravel) с помощью MySQL](tutorial-php-database-app.md)
