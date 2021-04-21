---
title: Краткое руководство. Подключение приложения MongoDB на Node.js к Azure Cosmos DB
description: В этом кратком руководстве показано, как подключить существующее приложение MongoDB, написанное на Node.js, к Azure Cosmos DB.
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: quickstart
ms.date: 05/21/2019
ms.custom: seo-javascript-september2019, seo-javascript-october2019, devx-track-js, devx-track-azurecli
ms.openlocfilehash: 588c12addbdbd686038699026bb9dccd8671717c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107765264"
---
# <a name="quickstart-migrate-an-existing-mongodb-nodejs-web-app-to-azure-cosmos-db"></a>Краткое руководство. Перенос имеющегося веб-приложения MongoDB на Node.js в Azure Cosmos DB 
[!INCLUDE[appliesto-mongodb-api](includes/appliesto-mongodb-api.md)]

> [!div class="op_single_selector"]
> * [.NET](create-mongodb-dotnet.md)
> * [Java](create-mongodb-java.md)
> * [Node.js](create-mongodb-nodejs.md)
> * [Python](./mongodb-introduction.md)
> * [Xamarin](create-mongodb-xamarin.md)
> * [Golang](create-mongodb-go.md)
>  

В этом кратком руководстве вы создадите Azure Cosmos DB для учетной записи API Mongo DB и будете управлять ею с помощью Azure Cloud Shell, а также приложением MEAN (MongoDB, Express, Angular и Node.js) приложением, клонированным из GitHub. Azure Cosmos DB — это служба многомодельной базы данных, позволяющая быстро создавать и запрашивать документы, таблицы, пары "ключ-значение" и графовые базы данных, используя возможности глобального распределения и горизонтального масштабирования.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)] Либо [воспользуйтесь пробной версией Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) без подписки Azure. Вы также можете воспользоваться [эмулятором Azure Cosmos DB](https://aka.ms/cosmosdb-emulator) со строкой подключения `.mongodb://localhost:C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==@localhost:10255/admin?ssl=true`.

- [Node.js](https://nodejs.org/) и опыт работы с Node.js.

- [Git](https://git-scm.com/downloads).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.


## <a name="clone-the-sample-application"></a>Клонирование примера приложения

Затем выполните следующие команды, чтобы клонировать репозиторий с примером. Этот репозиторий с примером содержит приложение по умолчанию [MEAN.js](https://meanjs.org/).

1. Откройте командную строку, создайте папку git-samples, а затем закройте окно командной строки.

    ```bash
    mkdir "C:\git-samples"
    ```

2. Откройте окно терминала git, например git bash, и выполните команду `cd`, чтобы перейти в новую папку для установки примера приложения.

    ```bash
    cd "C:\git-samples"
    ```

3. Выполните команду ниже, чтобы клонировать репозиторий с примером. Эта команда создает копию примера приложения на локальном компьютере. 

    ```bash
    git clone https://github.com/prashanthmadi/mean
    ```

## <a name="run-the-application"></a>Выполнение приложения

Это приложение MongoDB, написанное на Node.js, подключается к базе данных Azure Cosmos DB, которая поддерживает клиент MongoDB. Другими словами, приложению понятно, что данные хранятся в базе данных Azure Cosmos DB.

Установите необходимые пакеты и запустите приложение.

```bash
cd mean
npm install
npm start
```
Приложение попытается подключиться к источнику MongoDB, что ему не удастся. Выйдите из приложения, если в выходных данных появится ошибка [MongoError: connect ECONNREFUSED 127.0.0.1:27017].

## <a name="sign-in-to-azure"></a>Вход в Azure

Если вы используете установленную версию Azure CLI, войдите в подписку Azure с помощью команды [az login](/cli/azure/reference-index#az_login) и следуйте инструкциям на экране. Этот шаг можно пропустить, если вы используете Azure Cloud Shell.

```azurecli
az login 
``` 
   
## <a name="add-the-azure-cosmos-db-module"></a>Добавление модуля Azure Cosmos DB

Если вы используете установленную версию Azure CLI, проверьте наличие компонента `cosmosdb`, выполнив команду `az`. Если компонент `cosmosdb` включен в список базовых команд, перейдите к следующей команде. Этот шаг можно пропустить, если вы используете Azure Cloud Shell.

Если компонента `cosmosdb` нет в списке базовых команд, переустановите [Azure CLI](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Создайте [группу ресурсов](../azure-resource-manager/management/overview.md) с помощью команды [az group create](/cli/azure/group#az_group_create). Группа ресурсов Azure — это логический контейнер, в котором происходит развертывание ресурсов Azure (веб-приложений, баз данных и учетных записей хранения) и управление ими. 

В следующем примере показано создание группы ресурсов в регионе "Западная Европа". Выберите уникальное имя для группы ресурсов.

Если вы используете Azure Cloud Shell, щелкните **Попробовать**, войдите в систему, следуя указаниям на экране, а затем скопируйте команду в командную строку.

```azurecli-interactive
az group create --name myResourceGroup --location "West Europe"
```

## <a name="create-an-azure-cosmos-db-account"></a>создание учетной записи Azure Cosmos DB;

Создайте учетную запись Cosmos с помощью команды [az cosmosdb create](/cli/azure/cosmosdb#az_cosmosdb_create).

В следующей команде замените `<cosmosdb-name>` уникальным именем своей базы данных Cosmos везде, где встречается этот заполнитель. Это уникальное имя будет использоваться как часть конечной точки Cosmos DB (`https://<cosmosdb-name>.documents.azure.com/`), поэтому оно должно быть уникальным для всех учетных записей Cosmos в Azure. 

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

Параметр `--kind MongoDB` разрешает клиентские подключения MongoDB.

После создания учетной записи Azure Cosmos DB в Azure CLI отображаются сведения, схожие с теми, которые приведены ниже. 

> [!NOTE]
> В этом примере JSON по умолчанию используется как формат выходных данных Azure CLI. Чтобы использовать другой формат выходных данных, ознакомьтесь со статьей [Форматы выходных данных для команд Azure CLI](/cli/azure/format-output-azure-cli).

```json
{
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://<cosmosdb-name>.documents.azure.com:443/",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Document
DB/databaseAccounts/<cosmosdb-name>",
  "kind": "MongoDB",
  "location": "West Europe",
  "name": "<cosmosdb-name>",
  "readLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ],
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "writeLocations": [
    {
      "documentEndpoint": "https://<cosmosdb-name>-westeurope.documents.azure.com:443/",
      "failoverPriority": 0,
      "id": "<cosmosdb-name>-westeurope",
      "locationName": "West Europe",
      "provisioningState": "Succeeded"
    }
  ]
} 
```

## <a name="connect-your-nodejs-application-to-the-database"></a>Подключение приложения Node.js к базе данных

На этом шаге вы подключите пример приложения MEAN.js к только что созданной учетной записи базы данных Azure Cosmos DB. 

<a name="devconfig"></a>
## <a name="configure-the-connection-string-in-your-nodejs-application"></a>Настройка строки подключения в приложении Node.js

В репозитории MEAN.js откройте `config/env/local-development.js`.

Замените содержимое этого файла приведенным ниже кодом. Замените также два заполнителя `<cosmosdb-name>` на имя учетной записи Cosmos.

```javascript
'use strict';

module.exports = {
  db: {
    uri: 'mongodb://<cosmosdb-name>:<primary_master_key>@<cosmosdb-name>.documents.azure.com:10255/mean-dev?ssl=true&sslverifycertificate=false'
  }
};
```

## <a name="retrieve-the-key"></a>Получение ключа

Чтобы подключиться к базе данных Cosmos, вам понадобится ключ базы данных. Чтобы получить первичный ключ, выполните команду [az cosmosdb keys list](/cli/azure/cosmosdb/keys#az_cosmosdb_keys_list).

```azurecli-interactive
az cosmosdb keys list --name <cosmosdb-name> --resource-group myResourceGroup --query "primaryMasterKey"
```

В Azure CLI отображаются сведения, подобные следующим: 

```json
"RUayjYjixJDWG5xTqIiXjC..."
```

Скопируйте значение `primaryMasterKey`. Вставить это над `<primary_master_key>` в `local-development.js`.

Сохраните изменения.

### <a name="run-the-application-again"></a>Повторный запуск приложения

Еще раз запустите `npm start`. 

```bash
npm start
```

Должно появиться сообщение консоли, что среда разработки подготовлена к работе. 

Перейдите к `http://localhost:3000` в браузере. Щелкните **Зарегистрироваться** в меню сверху и создайте двух фиктивных пользователей. 

В примере приложения MEAN.js данные пользователя хранятся в базе данных. Если все получилось и вы вошли в приложение MEAN.js под созданным пользователем, это означает, что подключение Azure Cosmos DB работает. 

:::image type="content" source="./media/create-mongodb-nodejs/mongodb-connect-success.png" alt-text="MEAN.js успешно подключается к базе данных MongoDB":::

## <a name="view-data-in-data-explorer"></a>Просмотр данных в обозревателе данных

Данные, хранимые в базе данных Cosmos, доступны для просмотра и запроса на портале Azure.

Чтобы просмотреть данные пользователя, созданные на предыдущем шаге, запросить их и начать с ними работать, войдите на [портал Azure](https://portal.azure.com) с помощью своего браузера.

В поле поиска в верхней области введите **Azure Cosmos DB**. Когда откроется колонка учетной записи Cosmos, выберите свою учетную запись Cosmos. В левой области навигации щелкните **Обозреватель данных**. Разверните свою коллекцию на панели коллекций. Вы сможете увидеть документы в коллекции, запросить данные и даже создать и запустить хранимые процедуры, триггеры и определенные пользователем функции. 

:::image type="content" source="./media/create-mongodb-nodejs/cosmosdb-connect-mongodb-data-explorer.png" alt-text="Обозреватель данных на портале Azure":::


## <a name="deploy-the-nodejs-application-to-azure"></a>Развертывание приложения Node.js в Azure

На этом шаге вы развернете приложение Node.js в Cosmos DB.

Заметьте, что файл конфигурации, измененный ранее, понадобится для среды разработки (`/config/env/local-development.js`). Когда вы развернете приложение в службу приложений, оно запустится в рабочей среде по умолчанию. Таким образом, вам необходимо внести те же изменения в соответствующий файл конфигурации.

В репозитории MEAN.js откройте `config/env/production.js`.

В объекте `db` замените значение `uri`, как показано в следующем примере. Обязательно замените заполнители на фактические значения, как и в прошлый раз.

```javascript
'mongodb://<cosmosdb-name>:<primary_master_key>@<cosmosdb-name>.documents.azure.com:10255/mean?ssl=true&sslverifycertificate=false',
```

> [!NOTE] 
> Параметр `ssl=true` важен, так как он требуется для Cosmos DB. Дополнительные сведения см. в разделе [Требования к строке подключения](connect-mongodb-account.md#connection-string-requirements).
>
>

В окне терминала зафиксируйте все изменения в Git. Вы можете скопировать обе команды, чтобы выполнить их вместе.

```bash
git add .
git commit -m "configured MongoDB connection string"
```
## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [cosmosdb-delete-resource-group](../../includes/cosmos-db-delete-resource-group.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве вы узнали, как создать учетную запись API MongoDB для Azure Cosmos DB с помощью Azure Cloud Shell, а также, как создать и запустить приложение MEAN.js для добавления пользователей в учетную запись. Теперь можно импортировать дополнительные данные в учетную запись Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Перенос данных MongoDB в Azure Cosmos DB](../dms/tutorial-mongodb-cosmos-db.md?toc=%2fazure%2fcosmos-db%2ftoc.json%253ftoc%253d%2fazure%2fcosmos-db%2ftoc.json)