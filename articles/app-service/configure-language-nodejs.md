---
title: Настройка Node.js приложений
description: Узнайте, как настроить Node.js приложение в собственных экземплярах Windows или в предварительно созданном контейнере Linux в службе приложений Azure. В этой статье показаны наиболее распространенные задачи настройки.
ms.custom: devx-track-js, devx-track-azurecli
ms.devlang: nodejs
ms.topic: article
ms.date: 06/02/2020
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: 6a6f782768db12c2ce75f5cf1e66100222f24446
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101095213"
---
# <a name="configure-a-nodejs-app-for-azure-app-service"></a>Настройка приложения Node.js для Службы приложений Azure

Node.js приложения должны быть развернуты со всеми необходимыми зависимостями NPM. Модуль развертывания службы приложений автоматически запускается `npm install --production` при развертывании [репозитория Git](deploy-local-git.md)или [ZIP-пакета](deploy-zip.md) с включенной автоматизацией сборок. Однако при развертывании файлов с помощью [FTP/S](deploy-ftp.md)нужные пакеты необходимо загрузить вручную.

Это краткое описание содержит основные понятия и инструкции для Node.js разработчиков, развернутых в службе приложений. Если вы никогда не использовали службу приложений Azure, сначала следуйте инструкциям в [ кратком](quickstart-nodejs.md) руководстве поNode.js и [Node.js с MongoDB](tutorial-nodejs-mongodb-app.md) .

## <a name="show-nodejs-version"></a>Показывать Node.js версию

::: zone pivot="platform-windows"  

Чтобы отобразить текущую версию Node.js, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config appsettings list --name <app-name> --resource-group <resource-group-name> --query "[?name=='WEBSITE_NODE_DEFAULT_VERSION'].value"
```

Чтобы отобразились сведения обо всех поддерживаемых версиях Node.js, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes | grep node
```

::: zone-end

::: zone pivot="platform-linux"

Чтобы отобразить текущую версию Node.js, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

Чтобы отобразились сведения обо всех поддерживаемых версиях Node.js, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes --linux | grep NODE
```

::: zone-end

## <a name="set-nodejs-version"></a>Задать версию Node.js

::: zone pivot="platform-windows"  

Чтобы настроить для приложения [поддерживаемую версию Node.js](#show-nodejs-version), выполните следующую команду в [Cloud Shell](https://shell.azure.com) , чтобы установить `WEBSITE_NODE_DEFAULT_VERSION` поддерживаемую версию:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings WEBSITE_NODE_DEFAULT_VERSION="10.15"
```

Этот параметр определяет используемую версию Node.js во время выполнения и во время автоматического восстановления пакета во время автоматизации сборки службы приложений.

> [!NOTE]
> Необходимо задать версию Node.js в проекте `package.json` . Модуль развертывания выполняется в отдельном процессе, который содержит все поддерживаемые версии Node.js.

::: zone-end

::: zone pivot="platform-linux"

Чтобы настроить приложение на [поддерживаемую версию Node.js](#show-nodejs-version), выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --linux-fx-version "NODE|10.14"
```

Этот параметр определяет используемую версию Node.js как во время выполнения, так и при автоматическом восстановлении пакетов в KUDU.

> [!NOTE]
> Необходимо задать версию Node.js в проекте `package.json` . Модуль развертывания выполняется в отдельном контейнере, который содержит все поддерживаемые версии Node.js.

::: zone-end

## <a name="get-port-number"></a>Получение номера порта

Для получения входящих запросов приложение Node.js должно прослушивать правильный порт.

::: zone pivot="platform-windows"  

В службе приложений в Windows Node.js приложения размещаются с помощью [IISNode](https://github.com/Azure/iisnode), а Node.js приложение должно прослушивать порт, указанный в `process.env.PORT` переменной. В следующем примере показано, как это сделать в простом приложении Express:

::: zone-end

::: zone pivot="platform-linux"  

Служба приложений устанавливает переменную среды `PORT` в контейнере Node.js и перенаправляет входящие запросы в контейнер по этому номеру порта. Чтобы получать запросы, приложение должно прослушивать этот порт с помощью `process.env.PORT` . В следующем примере показано, как это сделать в простом приложении Express:

::: zone-end

```javascript
const express = require('express')
const app = express()
const port = process.env.PORT || 3000

app.get('/', (req, res) => {
  res.send('Hello World!')
})

app.listen(port, () => {
  console.log(`Example app listening at http://localhost:${port}`)
})
```

::: zone pivot="platform-linux"

## <a name="customize-build-automation"></a>Настройка автоматизации сборки

Если приложение развертывается с использованием Git или ZIP-пакетов с включенной автоматизацией сборки, то автоматизация сборки cлужбы приложений проходит в такой последовательности:

1. Запустите пользовательский скрипт, если он указан `PRE_BUILD_SCRIPT_PATH`.
1. Запустите `npm install` без флагов, включая NPM `preinstall` и `postinstall` сценарии, а также устанавливает `devDependencies` .
1. Выполните, `npm run build` Если в *package.jsна* выбран скрипт сборки.
1. Выполните `npm run build:azure` , если сборка: Скрипт Azure указан в *package.jsна*.
1. Запустите пользовательский скрипт, если он указан `POST_BUILD_SCRIPT_PATH`.

> [!NOTE]
> Как описано в [документах NPM](https://docs.npmjs.com/misc/scripts), скрипты с именами `prebuild` и `postbuild` выполняются до и после `build` , соответственно, если они указаны. `preinstall` и `postinstall` выполняются до и после `install` соответственно.

`PRE_BUILD_COMMAND` и `POST_BUILD_COMMAND` являются переменными среды, которые по умолчанию пустые. Чтобы выполнить команды перед сборкой, определите `PRE_BUILD_COMMAND`. Чтобы выполнить команды после сборки, определите `POST_BUILD_COMMAND`.

В следующем примере указываются две переменные для ряда команд, разделенных запятыми.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PRE_BUILD_COMMAND="echo foo, scripts/prebuild.sh"
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings POST_BUILD_COMMAND="echo foo, scripts/postbuild.sh"
```

Дополнительные переменные среды для настройки автоматизации сборки см. в статье [Конфигурация Oryx](https://github.com/microsoft/Oryx/blob/master/doc/configuration.md).

Дополнительные сведения о том, как служба приложений работает и создает Node.js приложений в Linux, см. в [документации по Орикс: как обнаруживаются и строятся приложения Node.js](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/nodejs.md).

## <a name="configure-nodejs-server"></a>Настройка сервера Node.js

Контейнеры Node.js поставляются с [PM2](https://pm2.keymetrics.io/), диспетчером рабочих процессов. Вы можете настроить приложение для запуска с PM2 или с помощью NPM или с помощью пользовательской команды.

- [Выполнить пользовательскую команду](#run-custom-command)
- [Запустить NPM запуск](#run-npm-start)
- [Запуск с помощью PM2](#run-with-pm2)

### <a name="run-custom-command"></a>Выполнить пользовательскую команду

Служба приложений может запустить приложение с помощью пользовательской команды, такой как исполняемый файл, например *Run.sh*. Например, чтобы выполнить `npm run start:prod` , выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "npm run start:prod"
```

### <a name="run-npm-start"></a>Запустить NPM запуск

Чтобы запустить приложение с помощью `npm start` , просто убедитесь, что `start` сценарий находится в *package.js* файле. Пример:

```json
{
  ...
  "scripts": {
    "start": "gulp",
    ...
  },
  ...
}
```

Чтобы использовать в проекте настраиваемый *package.js* , выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<filename>.json"
```

### <a name="run-with-pm2"></a>Запуск с помощью PM2

Контейнер автоматически запускает приложение с PM2, когда в проекте обнаруживается один из общих файлов Node.js:

- *bin/www*
- *server.js*
- *app.js*
- *index.js*
- *hostingstart.js*
- Один из следующих [файлов PM2](https://pm2.keymetrics.io/docs/usage/application-declaration/#process-file): *process.json* и *ecosystem.config.js*

Можно также настроить пользовательский файл запуска со следующими расширениями:

- *JS* -файл
- [Файл PM2](https://pm2.keymetrics.io/docs/usage/application-declaration/#process-file) с расширением *JSON*, *.config.js*, *YAML* или *. yml*

Чтобы добавить пользовательский начальный файл, выполните следующую команду в [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<filname-with-extension>"
```

## <a name="debug-remotely"></a>Удаленная отладка

> [!NOTE]
> Удаленная отладка сейчас доступна в предварительной версии.

Вы можете выполнить удаленную отладку приложения Node.js в [Visual Studio Code](https://code.visualstudio.com/) , если он настроен для [работы с PM2](#run-with-pm2), за исключением случая, когда он выполняется с помощью * .config.js, *. yml или *. YAML*.

В большинстве случаев для приложения не требуется дополнительная настройка. Если приложение запускается с *process.js* файла (по умолчанию или настраиваемого), оно должно иметь `script` свойство в корне JSON. Пример:

```json
{
  "name"        : "worker",
  "script"      : "./index.js",
  ...
}
```

Чтобы настроить Visual Studio Code для удаленной отладки, установите [расширение службы приложений](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice). Следуйте инструкциям на странице расширение и войдите в Azure в Visual Studio Code.

В Azure Explorer найдите приложение, которое нужно отладить, щелкните его правой кнопкой мыши и выберите команду **запустить удаленную отладку**. Нажмите кнопку **Да** , чтобы включить его для приложения. Служба приложений запускает прокси-сервер туннеля и присоединяет отладчик. Затем можно выполнить запросы к приложению и увидеть, что отладчик приостанавливается в точках останова.

После завершения отладки закройте отладчик, выбрав **Отключить**. При появлении запроса нажмите кнопку **Да** , чтобы отключить удаленную отладку. Чтобы отключить его позже, снова щелкните свое приложение в Azure Explorer и выберите **Отключить удаленную отладку**.

::: zone-end

## <a name="access-environment-variables"></a>Доступ к переменным среды

В Службе приложений можно [задать параметры приложения](configure-common.md) вне кода приложения. Затем к ним можно получить доступ с помощью стандартного шаблона Node.js. Например, для доступа к параметру приложения с именем `NODE_ENV` используйте следующий код:

```javascript
process.env.NODE_ENV
```

## <a name="run-gruntbowergulp"></a>Запуск grunt/Bower/gulp

По умолчанию Автоматизация сборки службы приложений выполняется `npm install --production` при обнаружении Node.jsного приложения, которое развертывается с помощью Git или ZIP-развертывания с включенной автоматизацией сборки. Если приложению требуются какие-либо популярные средства автоматизации, такие как grunt, Bower или gulp, необходимо предоставить [Пользовательский скрипт развертывания](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script) для его запуска.

Чтобы разрешить репозиторию запускать эти средства, необходимо добавить их в зависимости в *package.js.* Пример:

```json
"dependencies": {
  "bower": "^1.7.9",
  "grunt": "^1.0.1",
  "gulp": "^3.9.1",
  ...
}
```

В окне терминала перейдите в каталог корня репозитория и выполните следующие команды:

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

## <a name="detect-https-session"></a>Обнаружение сеанса HTTPS

В Службе приложений [завершение SSL-запросов](https://wikipedia.org/wiki/TLS_termination_proxy) происходит в подсистеме балансировки нагрузки сети, поэтому все HTTPS-запросы достигают вашего приложения в виде незашифрованных HTTP-запросов. Если логика вашего приложения проверяет, зашифрованы ли пользовательские запросы, проверяйте заголовок `X-Forwarded-Proto`.

Популярные веб-платформы позволяют получить доступ к информации `X-Forwarded-*` в стандартном шаблоне приложения. В [Express](https://expressjs.com/)можно использовать [Доверенные прокси-серверы](https://expressjs.com/guide/behind-proxies.html). Пример:

```javascript
app.set('trust proxy', 1)
...
if (req.secure) {
  // Do something when HTTPS is used
}
```

## <a name="access-diagnostic-logs"></a>Доступ к журналам диагностики

::: zone pivot="platform-windows"  

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

::: zone-end


::: zone pivot="platform-linux"

## <a name="monitor-with-application-insights"></a>Мониторинг с помощью Application Insights

Application Insights позволяет отслеживать производительность приложения, исключения и использование без внесения изменений в код. Чтобы подключить агент App Insights, перейдите к веб-приложению на портале и выберите **Application Insights** в разделе **Параметры**, а затем выберите **включить Application Insights**. Затем выберите существующий ресурс App Insights или создайте новый. Наконец, нажмите кнопку **Применить** в нижней части страницы. Инструкции по инструментированию веб-приложения с помощью PowerShell см. [здесь](../azure-monitor/app/azure-web-apps.md?tabs=netcore#enabling-through-powershell) .

Этот агент отслеживает Node.js приложение на стороне сервера. Чтобы отслеживать клиентский сценарий JavaScript, [добавьте в проект пакет SDK для JavaScript](../azure-monitor/app/javascript.md). 

Дополнительные сведения см. в [заметках о выпуске расширения Application Insights](../azure-monitor/app/web-app-extension-release-notes.md).

::: zone-end

## <a name="troubleshooting"></a>Устранение неполадок

Когда работающий Node.js приложение работает иначе в службе приложений или с ошибками, попробуйте выполнить следующие действия.

- [Получите доступ к потоку журнала](#access-diagnostic-logs).
- Протестируйте приложение локально в рабочем режиме. Служба приложений запускает приложения Node.js в рабочем режиме, поэтому вам необходимо убедиться в том, что проект работает надлежащим образом локально в рабочем режиме. Пример:
    - В зависимости от *package.jsв* рабочем режиме могут быть установлены различные пакеты ( `dependencies` VS `devDependencies` ).
    - Некоторые веб-платформы в рабочем режиме могут выполнять другие операции при развертывании статических файлов.
    - Некоторые веб-платформы могут использовать специальные скрипты запуска в рабочем режиме.
- Запустите приложение в службе приложений в режиме разработки. Например, в [MEAN.js](https://meanjs.org/)можно настроить приложение в режиме разработки в среде выполнения, [задав `NODE_ENV` параметр приложения](configure-common.md).

::: zone pivot="platform-linux"

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

::: zone-end

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство. Приложение Node.js с MongoDB](tutorial-nodejs-mongodb-app.md)

::: zone pivot="platform-linux"

> [!div class="nextstepaction"]
> [Служба приложений под управлением Linux: вопросы и ответы](faq-app-service-linux.md)

::: zone-end
