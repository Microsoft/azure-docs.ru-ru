---
title: Руководство по приложению Node.js с MongoDB
description: Узнайте, как разработать приложение Node.js с подключением к базе данных MongoDB в Azure (Cosmos DB). В этом руководстве используется MEAN.js.
ms.assetid: 0b4d7d0e-e984-49a1-a57a-3c0caa955f0e
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 06/16/2020
ms.custom: mvc, cli-validate, seodec18, devx-track-js, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: b1dcd413f301f25460cb29f1bb20e67a37ac6ebb
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767330"
---
# <a name="tutorial-build-a-nodejs-and-mongodb-app-in-azure"></a>Руководство по Разработка приложения на основе Node.js и MongoDB в Azure

::: zone pivot="platform-windows"  

[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости. В этом руководстве показано, как создать приложение Node.js в Службе приложений в Windows и подключить его к базе данных MongoDB. После выполнения действий, описанных в этом руководстве, у вас появится приложение MEAN (MongoDB, Express, AngularJS и Node.js), выполняющееся в [службе приложений Azure](overview.md). Для простоты в примере приложения используется [веб-платформа MEAN.js](https://meanjs.org/).

::: zone-end

::: zone pivot="platform-linux"


[Служба приложений Azure](overview.md) — это высокомасштабируемая служба размещения с самостоятельной установкой исправлений на основе операционной системы Linux. В этом руководстве показано, как создать приложение Node.js в Службе приложений в Linux, подключить его локально к базе данных MongoDB, а затем развернуть для базы данных в API Azure Cosmos DB для MongoDB. Выполнив действия, описанные в этом руководстве, вы получите приложение MEAN (MongoDB, Express, AngularJS и Node.js), работающее в службе приложений на платформе Linux. Для простоты в примере приложения используется [веб-платформа MEAN.js](https://meanjs.org/).

::: zone-end

![Приложение MEAN.js, которое запущено в службе приложений Azure](./media/tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Из этого руководства вы узнаете, как выполнять такие задачи:

> [!div class="checklist"]
> * Создание базы данных MongoDB в Azure.
> * Подключение приложения Node.js к базе данных MongoDB.
> * Развертывание приложения в Azure
> * Обновление модели данных и повторное развертывание приложения.
> * Потоковая передача журналов диагностики из Azure.
> * Управление приложением на портале Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

- [установите Git](https://git-scm.com/);
- [установите Node.j и NPM](https://nodejs.org/).
- [установите Bower](https://bower.io/) (требуется для [MEAN.js](https://meanjs.org/docs/0.5.x/#getting-started));
- [Gulp.js](https://gulpjs.com/) (требуется для [MEAN.js](https://meanjs.org/docs/0.5.x/#getting-started));
- [Установка и запуск MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/)
[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)] 

## <a name="test-local-mongodb"></a>Проверка локальной базы данных MongoDB

Откройте окно терминала и `cd` в каталог установки MongoDB `bin`. Используйте это окно терминала для выполнения всех команд в рамках этого руководства.

Выполните команду `mongo` в окне терминала для подключения к локальному серверу MongoDB.

```bash
mongo
```

Если подключение успешно установлено, это означает, что локальная база данных MongoDB запущена. Если это не так, запустите локальную базу данных MongoDB, выполнив инструкции, описанные в статье [Install MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/) (Установка MongoDB Community Edition). Часто база данных MongoDB установлена, но ее необходимо запустить, выполнив команду `mongod`. 

Закончив проверку базы данных MongoDB, введите команду `Ctrl+C` в окне терминала. 

## <a name="create-local-nodejs-app"></a>Создание локального приложения Node.js

На этом шаге вы настроите локальный проект Node.js.

### <a name="clone-the-sample-application"></a>Клонирование примера приложения

Откройте окно терминала и c помощью команды `cd` перейдите в рабочий каталог.  

Выполните команду ниже, чтобы клонировать репозиторий с примером. 

```bash
git clone https://github.com/Azure-Samples/meanjs.git
```

Этот пример репозитория содержит копию [репозитория MEAN.js](https://github.com/meanjs/mean). В него внесены изменения для запуска службы приложений. Дополнительные сведения см. в [файле сведений](https://github.com/Azure-Samples/meanjs/blob/master/README.md) в репозитории MEAN.js.

### <a name="run-the-application"></a>Выполнение приложения

Выполните команды ниже, чтобы установить необходимые пакеты и запустить приложение.

```bash
cd meanjs
npm install
npm start
```

Игнорируйте это предупреждение config.domain. После полной загрузки приложения вы увидите следующее сообщение:

<pre>
--
MEAN.JS — среда разработки

Среда — сервер разработки:          http://0.0.0.0:3000 База данных: mongodb://localhost/mean-dev Версия приложения:     Версия 0.5.0 MEAN.JS: 0.5.0 —
</pre>

Откройте браузер и перейдите по адресу `http://localhost:3000`. В верхнем меню щелкните **Зарегистрироваться** и создайте тестового пользователя. 

В примере приложения MEAN.js данные пользователя хранятся в базе данных. Если вы создали пользователя и вошли в приложение, это означает, что приложение записывает данные в локальную базу данных MongoDB.

![MEAN.js успешно подключается к базе данных MongoDB](./media/tutorial-nodejs-mongodb-app/mongodb-connect-success.png)

Чтобы добавить несколько статей, выберите **Администрирование > Manage Articles** (Управление статьями).

Чтобы остановить Node.js в любое время, введите `Ctrl+C` в окне терминала. 

## <a name="create-production-mongodb"></a>Создание рабочей базы данных MongoDB

На этом шаге вы создадите базу данных MongoDB в Azure. При развертывании приложения в Azure используется эта облачная база данных.

В качестве MongoDB в этом руководстве используется [Azure Cosmos DB](/azure/documentdb/). Cosmos DB поддерживает подключения клиентов MongoDB.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)] 

### <a name="create-a-cosmos-db-account"></a>Создание учетной записи Cosmos DB

> [!NOTE]
> За создание баз данных Azure Cosmos DB, используемых в этом руководстве, взимается плата в вашей подписке Azure. Чтобы получить бесплатную учетную запись Azure Cosmos DB на семь дней, можно использовать [бесплатную пробную версию Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/). Просто нажмите кнопку **Создать** на плитке MongoDB, чтобы создать бесплатную базу данных MongoDB в Azure. После создания базы данных перейдите к разделу **Строка подключения** на портале и извлеките строку подключения к Azure Cosmos DB для дальнейшего использования в этом руководстве.
>

В Cloud Shell создайте учетную запись Cosmos DB при помощи команды [`az cosmosdb create`](/cli/azure/cosmosdb#az_cosmosdb_create).

В следующей команде замените заполнитель *\<cosmosdb-name>* уникальным именем базы данных Cosmos DB. Это имя используется как часть конечной точки Cosmos DB (`https://<cosmosdb-name>.documents.azure.com/`), поэтому оно должно быть уникальным для всех учетных записей Cosmos DB в Azure. В нем могут использоваться только строчные буквы, цифры и дефис (-). Его длина должна быть от 3 до 50 знаков.

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

Параметр *--kind MongoDB* разрешает клиентские подключения MongoDB.

После создания учетной записи Cosmos DB в Azure CLI отображаются следующие сведения.

<pre>
{
  "consistencyPolicy":
  {
    "defaultConsistencyLevel": "Session",
    "maxIntervalInSeconds": 5,
    "maxStalenessPrefix": 100
  },
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://&lt;cosmosdb-name&gt;.documents.azure.com:443/",
  "failoverPolicies": 
  ...
  &lt; Output truncated for readability &gt;
}
</pre>

## <a name="connect-app-to-production-mongodb"></a>Подключение приложения к рабочей базе данных MongoDB

На этом шаге вы подключите пример приложения MEAN.js к только что созданной базе данных Cosmos DB с помощью строки подключения MongoDB. 

### <a name="retrieve-the-database-key"></a>Получение ключа базы данных

Для подключения к базе данных Cosmos DB потребуется ключ базы данных. Чтобы получить первичный ключ, выполните в Cloud Shell команду [`az cosmosdb list-keys`](/cli/azure/cosmosdb#az_cosmosdb_list_keys).

```azurecli-interactive
az cosmosdb list-keys --name <cosmosdb-name> --resource-group myResourceGroup
```

В Azure CLI отображаются следующие сведения:

<pre>
{
  "primaryMasterKey": "RS4CmUwzGRASJPMoc0kiEvdnKmxyRILC9BWisAYh3Hq4zBYKr0XQiSE4pqx3UchBeO4QRCzUt1i7w0rOkitoJw==",
  "primaryReadonlyMasterKey": "HvitsjIYz8TwRmIuPEUAALRwqgKOzJUjW22wPL2U8zoMVhGvregBkBk9LdMTxqBgDETSq7obbwZtdeFY7hElTg==",
  "secondaryMasterKey": "Lu9aeZTiXU4PjuuyGBbvS1N9IRG3oegIrIh95U6VOstf9bJiiIpw3IfwSUgQWSEYM3VeEyrhHJ4rn3Ci0vuFqA==",
  "secondaryReadonlyMasterKey": "LpsCicpVZqHRy7qbMgrzbRKjbYCwCKPQRl0QpgReAOxMcggTvxJFA94fTi0oQ7xtxpftTJcXkjTirQ0pT7QFrQ=="
}
</pre>

Скопируйте значение `primaryMasterKey`. Эти сведения потребуются на следующем шаге.

<a name="devconfig"></a>
### <a name="configure-the-connection-string-in-your-nodejs-application"></a>Настройка строки подключения в приложении Node.js

В локальном репозитории MEAN.js создайте файл с именем _local-production.js_ в папке _config/env/_ . Чтобы этот файл хранился вне репозитория, уже настроен _.gitignore_. 

Скопируйте в него следующий код: Также замените два заполнителя *\<cosmosdb-name>* именем базы данных Cosmos DB, а заполнитель *\<primary-master-key>* замените ключом, скопированным на предыдущем шаге.

```javascript
module.exports = {
  db: {
    uri: 'mongodb://<cosmosdb-name>:<primary-master-key>@<cosmosdb-name>.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false'
  }
};
```

Параметр `ssl=true` обязателен, так как для [Cosmos DB требуется протокол SSL](../cosmos-db/connect-mongodb-account.md#connection-string-requirements). 

Сохраните изменения.

### <a name="test-the-application-in-production-mode"></a>Тестирование приложения в рабочем режиме 

В окне терминала на локальном компьютере выполните команду ниже, чтобы уменьшить размер скриптов и объединить их в пакет для рабочей среды. При этом создаются файлы, необходимые для рабочей среды.

```bash
gulp prod
```

В окне терминала на локальном компьютере выполните команду ниже, чтобы использовать строку подключения, которая была настроена в _config/env/local-production.js_. Игнорируйте сообщение об ошибке сертификата и предупреждение config.domain.

```bash
# Bash
NODE_ENV=production node server.js

# Windows PowerShell
$env:NODE_ENV = "production" 
node server.js
```

`NODE_ENV=production` устанавливает переменную среды, указывающую, что Node.js необходимо запустить в рабочей среде.  `node server.js` запускает сервер Node.js с `server.js` в корневой папке репозитория. Так приложение Node.js загружается в Azure. 

Когда приложение будет загружено, убедитесь, что оно запущено в рабочей среде:

<pre>
--
MEAN.JS

Среда: рабочая Сервер:          http://0.0.0.0:8443 База данных:        mongodb://&lt; cosmosdb-name&gt;:&lt; primary-master-key&gt;@&lt; cosmosdb-name&gt;.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false App version:     Версия 0.5.0 MEAN.JS: 0.5.0
</pre>

Откройте браузер и перейдите по адресу `http://localhost:8443`. В верхнем меню щелкните **Зарегистрироваться** и создайте тестового пользователя. Если вы создали пользователя и вошли в приложение, это означает, что приложение записывает данные в локальную базу данных Cosmos DB в Azure. 

Чтобы остановить Node.js, введите `Ctrl+C` в окне терминала. 

## <a name="deploy-app-to-azure"></a>Развертывание приложения в Azure

На этом шаге вы развернете приложение Node.js, подключенное к MongoDB, в службе приложений Azure.

### <a name="configure-a-deployment-user"></a>Настойка пользователя развертывания

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

::: zone pivot="platform-windows"  

[!INCLUDE [Create app service plan no h](../../includes/app-service-web-create-app-service-plan-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]

::: zone-end

<a name="create"></a>
### <a name="create-a-web-app"></a>Создание веб-приложения

::: zone pivot="platform-windows"  

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-nodejs-no-h.md)] 

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-nodejs-linux-no-h.md)] 

::: zone-end

### <a name="configure-an-environment-variable"></a>Настройка переменной среды

По умолчанию _config/env/local-production.js_ хранится в проекте MEAN.js вне репозитория Git. Поэтому для приложения Azure вам нужно использовать параметры приложения, чтобы определить строку подключения MongoDB.

Чтобы задать параметры приложения, выполните команду [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) в Cloud Shell. 

В следующем примере настраивается параметр приложения `MONGODB_URI` в приложении Azure. Замените заполнители *\<app-name>* , *\<cosmosdb-name>* и *\<primary-master-key>* .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings MONGODB_URI="mongodb://<cosmosdb-name>:<primary-master-key>@<cosmosdb-name>.documents.azure.com:10250/mean?ssl=true"
```

В коде Node.js для [доступа к этому параметру приложения](configure-language-nodejs.md#access-environment-variables), как и к любой переменной среды, используется `process.env.MONGODB_URI`. 

В локальном репозитории MEAN.js откройте файл _config/env/production.js_ (не _config/env/local-production.js_), который содержит конфигурацию для конкретной рабочей среды. Приложение MEAN.js по умолчанию уже настроено для использования созданной переменной среды `MONGODB_URI`.

```javascript
db: {
  uri: ... || process.env.MONGODB_URI || ...,
  ...
},
```

### <a name="push-to-azure-from-git"></a>Публикация в Azure из Git

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 489 bytes | 0 bytes/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id '6c7c716eee'.
remote: Running custom deployment command...
remote: Running deployment command...
remote: Handling node.js deployment.
.
.
.
remote: Deployment successful.
To https://&lt;app-name&gt;.scm.azurewebsites.net/&lt;app-name&gt;.git
 * [new branch]      main -> main
</pre>

Вы можете заметить, что в ходе развертывания после `npm install` запускается [Gulp](https://gulpjs.com/). Служба приложений не запускает задачи Gulp или Grunt во время развертывания, поэтому для запуска скрипта в этом примере в корневом каталоге репозитория расположены два дополнительных файла: 

- _.deployment_ — этот файл указывает службе приложений запустить `bash deploy.sh` как пользовательский сценарий развертывания.
- _deploy.sh_ — пользовательский сценарий развертывания. Если открыть этот файл, вы увидите, что он запускает `gulp prod` после `npm install` и `bower install`. 

Точно так же можно добавить любые действия в развертывание на основе Git. При перезапуске приложения Azure в любой момент времени Служба приложений не перезапускает эти задачи автоматизации. Подробные сведения см. в разделе [Run Grunt/Bower/Gulp](configure-language-nodejs.md#run-gruntbowergulp) (Запуск Grunt/Bower/Gulp).

### <a name="browse-to-the-azure-app"></a>Переход к приложению Azure 

Перейдите к развернутому приложению в веб-браузере. 

```bash 
http://<app-name>.azurewebsites.net 
``` 

Щелкните **Зарегистрироваться** в верхнем меню и создайте фиктивного пользователя. 

Если все прошло успешно и вы автоматически вошли в приложение с созданным пользователем, это означает, что приложение MEAN.js в Azure подключено к базе данных MongoDB (Cosmos DB). 

![Приложение MEAN.js, которое запущено в службе приложений Azure](./media/tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Чтобы добавить несколько статей, выберите **Администрирование > Manage Articles** (Управление статьями). 

**Поздравляем!** Вы запустили управляемое данными приложение Node.js в службе приложений Azure.

## <a name="update-data-model-and-redeploy"></a>Обновление модели данных и повторное развертывание

На этом шаге вы внесете изменения в модель данных `article` и опубликуете свои изменения в Azure.

### <a name="update-the-data-model"></a>Обновление модели данных

В локальном репозитории MEAN.js откройте файл _modules/articles/server/models/article.server.model.js_.

В `ArticleSchema` добавьте тип `String` с именем `comment`. Когда все будет готово, код схемы должен выглядеть следующим образом:

```javascript
const ArticleSchema = new Schema({
  ...,
  user: {
    type: Schema.ObjectId,
    ref: 'User'
  },
  comment: {
    type: String,
    default: '',
    trim: true
  }
});
```

### <a name="update-the-articles-code"></a>Обновление кода статей

Обновите остальную часть кода `articles`, чтобы в ней использовался `comment`.

Вам потребуется изменить пять файлов: файл сервера контроллера и четыре файла представлений клиентов. 

Откройте файл _modules/articles/server/controllers/articles.server.controller.js_.

Добавьте в функцию `update` присваивание для `article.comment`. В следующем коде приведена готовая функция `update`:

```javascript
exports.update = function (req, res) {
  let article = req.article;

  article.title = req.body.title;
  article.content = req.body.content;
  article.comment = req.body.comment;

  ...
};
```

Откройте файл _modules/articles/client/views/view-article.client.view.html_.

Прямо перед закрывающим тегом `</section>` добавьте следующую строку для отображения `comment` вместе с остальными данными статьи:

```html
<p class="lead" ng-bind="vm.article.comment"></p>
```

Откройте файл _modules/articles/client/views/list-articles.client.view.html_.

Прямо перед закрывающим тегом `</a>` добавьте следующую строку для отображения `comment` вместе с остальными данными статьи:

```html
<p class="list-group-item-text" ng-bind="article.comment"></p>
```

Откройте файл _modules/articles/client/views/admin/list-articles.client.view.html_.

В элементе `<div class="list-group">` и прямо перед закрывающим тегом `</a>` добавьте следующую строку для отображения `comment` вместе с остальными данными статьи:

```html
<p class="list-group-item-text" data-ng-bind="article.comment"></p>
```

Откройте файл _modules/articles/client/views/admin/form-article.client.view.html_.

Найдите элемент `<div class="form-group">`, который содержит кнопку "Отправить" и выглядит следующим образом:

```html
<div class="form-group">
  <button type="submit" class="btn btn-default">{{vm.article._id ? 'Update' : 'Create'}}</button>
</div>
```

Прямо перед этим тегом добавьте еще один элемент `<div class="form-group">`, который позволяет пользователям изменять поле `comment`. Новый элемент должен выглядеть следующим образом:

```html
<div class="form-group">
  <label class="control-label" for="comment">Comment</label>
  <textarea name="comment" data-ng-model="vm.article.comment" id="comment" class="form-control" cols="30" rows="10" placeholder="Comment"></textarea>
</div>
```

### <a name="test-your-changes-locally"></a>Проверьте изменения локально.

Сохраните изменения.

В окне терминала на локальном компьютере снова проверьте изменения в рабочем режиме.

```bash
# Bash
gulp prod
NODE_ENV=production node server.js

# Windows PowerShell
gulp prod
$env:NODE_ENV = "production" 
node server.js
```

Откройте в браузере адрес `http://localhost:8443` и убедитесь, что вход выполнен.

Выберите **Администрирование > Управление статьями** и добавьте статью, нажав кнопку **+** .

Должно появиться новое текстовое поле `Comment`.

![В статьи добавлено поле комментария](./media/tutorial-nodejs-mongodb-app/added-comment-field.png)

Чтобы остановить Node.js, введите `Ctrl+C` в окне терминала. 

### <a name="publish-changes-to-azure"></a>Публикация изменений в Azure

В окне терминала на локальном компьютере зафиксируйте изменения в Git, а затем отправьте изменение кода в Azure.

```bash
git commit -am "added article comment"
git push azure main
```

После выполнения команды `git push` перейдите к приложению Azure и проверьте новые функции.

![Изменения модели и базы данных, опубликованные в Azure](media/tutorial-nodejs-mongodb-app/added-comment-field-published.png)

Если вы добавляли статьи ранее, то вы увидите, что они по-прежнему отображаются. Существующие данные в Cosmos DB не теряются. Кроме того, изменения в схеме данных не влияют на существующие данные.

## <a name="stream-diagnostic-logs"></a>Потоковая передача журналов диагностики 

::: zone pivot="platform-windows"  

При запуске приложения Node.js в службе приложений Azure можно направить журналы консоли в свой терминал. Таким образом, вы будете получать те же диагностические сообщения, которые помогут устранить ошибки приложения.

Чтобы настроить потоки для журналов, выполните команду [`az webapp log tail`](/cli/azure/webapp/log#az_webapp_log_tail) в Cloud Shell.

```azurecli-interactive
az webapp log tail --name <app-name> --resource-group myResourceGroup
``` 

После настройки потоков обновите приложение Azure в браузере, чтобы получить немного трафика. Вы должны увидеть, что журналы консоли теперь направляются в терминал.

Чтобы в любое время отменить потоковую передачу журналов, введите `Ctrl+C`. 

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end

## <a name="manage-your-azure-app"></a>Управление приложением Azure

Перейдите на [портал Azure](https://portal.azure.com), чтобы увидеть созданное приложение.

В меню слева щелкните **Службы приложений**, а затем — имя своего приложения Azure.

![Переход к приложению Azure на портале](./media/tutorial-nodejs-mongodb-app/access-portal.png)

По умолчанию на портале отображается страница **Обзор** приложения. Здесь вы можете наблюдать за работой приложения. Вы также можете выполнять базовые задачи управления: обзор, завершение, запуск, перезагрузку и удаление. На вкладках в левой части страницы отображаются различные страницы конфигурации, которые можно открыть.

![Страница службы приложений на портале Azure](./media/tutorial-nodejs-mongodb-app/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Дальнейшие действия

Вы научились выполнять следующие задачи:

> [!div class="checklist"]
> * Создание базы данных MongoDB в Azure.
> * Подключение приложения Node.js к базе данных MongoDB.
> * Развертывание приложения в Azure
> * Обновление модели данных и повторное развертывание приложения.
> * Потоковая передача журналов из Azure в окно терминала.
> * Управление приложением на портале Azure.

Перейдите к следующему руководству, чтобы научиться сопоставлять пользовательские DNS-имена с приложением.

> [!div class="nextstepaction"] 
> [Сопоставление существующего настраиваемого DNS-имени со Службой приложений Azure](app-service-web-tutorial-custom-domain.md)

Также ознакомьтесь с другими ресурсами:

> [!div class="nextstepaction"]
> [Настройка приложения Node.js](configure-language-nodejs.md)
