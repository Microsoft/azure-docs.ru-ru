---
title: Подключение функции Java к службе хранилища Azure
description: Узнайте, как подключить функцию Java, активируемую HTTP, к службе хранилища Azure с помощью выходной привязки хранилища очередей.
author: KarlErickson
ms.custom: devx-track-java
ms.author: karler
ms.date: 10/14/2019
ms.topic: quickstart
zone_pivot_groups: java-build-tools-set
ms.openlocfilehash: b4381c4acbea8a3b7d86d9c32aaf9cea24cf15fa
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93422660"
---
# <a name="connect-your-java-function-to-azure-storage"></a>Подключение функции Java к службе хранилища Azure

[!INCLUDE [functions-add-storage-binding-intro](../../includes/functions-add-storage-binding-intro.md)]

В этой статье приведены сведения об интеграции функции, которая была создана в [предыдущей статье](./create-first-function-cli-java.md?tabs=bash,browser), с помощью очереди службы хранилища Azure. Выходная привязка, которая была добавлена в эту функцию, записывает в сообщение очереди данные HTTP-запроса.

Большинство привязок требуют сохраненную строку подключения, которая будет использоваться Функциями Azure для доступа к привязанным службам. Чтобы упростить это подключение, вам следует использовать учетную запись хранения, созданную в приложении-функции. Подключение к этой учетной записи уже хранится в параметрах приложения под названием `AzureWebJobsStorage`.  

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы с этой статьей следует завершить все шаги из [части 1 краткого руководства по Java](./create-first-function-cli-java.md?tabs=bash,browser).

## <a name="download-the-function-app-settings"></a>Загрузка параметров приложения-функции

[!INCLUDE [functions-app-settings-download-cli](../../includes/functions-app-settings-download-local-cli.md)]

## <a name="enable-extension-bundles"></a>Включение пакетов расширений

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

Теперь вы можете добавить выходную привязку службы хранилища в проект.

## <a name="add-an-output-binding"></a>Добавление выходной привязки

[!INCLUDE [functions-add-output-binding-java-cli](../../includes/functions-add-output-binding-java-cli.md)]

## <a name="add-code-that-uses-the-output-binding"></a>Добавление кода, который использует выходную привязку

[!INCLUDE [functions-add-output-binding-java-code](../../includes/functions-add-output-binding-java-code.md)]

[!INCLUDE [functions-add-output-binding-java-test-cli](../../includes/functions-add-output-binding-java-test-cli.md)]

Теперь новую выходную привязку можно испытать локально.

## <a name="run-the-function-locally"></a>Локальное выполнение функции

Как и прежде, чтобы построить проект и запустить среду выполнения функций локально, используйте следующую команду.

# <a name="maven"></a>[Maven](#tab/maven)
```bash
mvn clean package 
mvn azure-functions:run
```
# <a name="gradle"></a>[Gradle](#tab/gradle) 
```bash
gradle jar --info
gradle azureFunctionsRun
```
---

> [!NOTE]  
> Так как пакеты расширений включены в файл host.json, во время запуска вместе с другими расширениями привязки Майкрософт было также загружено и установлено [расширение привязки службы хранилища](functions-bindings-storage-blob.md#add-to-your-functions-app).

Как и прежде, запустите функцию из командной строки, используя cURL в новом окне терминала:

```CMD
curl -w "\n" http://localhost:7071/api/HttpTrigger-Java --data AzureFunctions
```

На этот раз в хранилище очередей выходная привязка также создает очередь с именем `outqueue` и с помощью этой же строки добавляет сообщение.

Затем Azure CLI следует использовать для просмотра новой очереди и проверки добавленного сообщения. Кроме того, очередь можно просматривать с помощью [Обозревателя службы хранилища Microsoft Azure][Azure Storage Explorer] или на [портале Azure](https://portal.azure.com).

[!INCLUDE [functions-storage-account-set-cli](../../includes/functions-storage-account-set-cli.md)]

[!INCLUDE [functions-query-storage-cli](../../includes/functions-query-storage-cli.md)]

### <a name="redeploy-the-project"></a>Повторное развертывание проекта 

Чтобы обновить опубликованное приложение, выполните следующую команду еще раз:  

# <a name="maven"></a>[Maven](#tab/maven)  
```bash
mvn azure-functions:deploy
```
# <a name="gradle"></a>[Gradle](#tab/gradle)  
```bash
gradle azureFunctionsDeploy
```
---

Чтобы проверить развернутую функцию, можно снова использовать cURL. Как и ранее, передайте значение `AzureFunctions` в тексте запроса POST к URL-адресу, как показано в следующем примере:

```bash
curl -w "\n" https://fabrikam-functions-20190929094703749.azurewebsites.net/api/HttpTrigger-Java?code=zYRohsTwBlZ68YF.... --data AzureFunctions
```

Чтобы убедиться, что выходная привязка снова создает новое сообщение в очереди, как и ожидалось, можно [проверить сообщение очереди службы хранилища](#query-the-storage-queue).

[!INCLUDE [functions-cleanup-resources](../../includes/functions-cleanup-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

Вы обновили функцию, активируемую HTTP, которую теперь можно использовать для записи данных в очередь службы хранилища. Чтобы получить дополнительные сведения о разработке Функций Azure с помощью Java, просмотрите статьи [Руководство разработчика Java по Функциям Azure](functions-reference-java.md) и [Основные понятия триггеров и привязок в Функциях Azure](functions-triggers-bindings.md). Примеры завершенных проектов Функций в Java см. в [этой статье](/samples/browse/?products=azure-functions&languages=Java). 

Затем для мониторинга приложения-функции следует включить Application Insights.

> [!div class="nextstepaction"]
> [Включение интеграции с Application Insights](configure-monitoring.md#add-to-an-existing-function-app)


[Azure Storage Explorer]: https://storageexplorer.com/
