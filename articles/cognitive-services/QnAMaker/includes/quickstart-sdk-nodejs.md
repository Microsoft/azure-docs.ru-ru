---
title: Краткое руководство. Клиентская библиотека QnA Maker для Node.js
description: В этом кратком руководстве показано, как начать работу с клиентской библиотекой QnA Maker для Node.js.
ms.topic: quickstart
ms.date: 06/18/2020
ms.custom: devx-track-js
ms.openlocfilehash: 0d0522be53ec9b1008f18725308c91a19ee24156
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105609410"
---
# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

Клиентская библиотека QnA Maker для Node.js:

* Создание базы знаний
* Обновление базы знаний
* Публикация базы знаний
* Получение ключа конечной точки среды выполнения прогнозирования
* Ожидание долго выполняющейся задачи
* Скачивание базы знаний
* Получение ответа из базы знаний
* Удаление базы знаний

[Справочная документация](/javascript/api/@azure/cognitiveservices-qnamaker/) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-qnamaker) | [Пакет (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-qnamaker) | [Примеры Node.js](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/qnamaker_quickstart.js)

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

Клиентская библиотека QnA Maker для Node.js:

* Создание базы знаний
* Обновление базы знаний
* Публикация базы знаний
* Ожидание долго выполняющейся задачи
* Скачивание базы знаний
* Получение ответа из базы знаний
* Удаление базы знаний

[Справочная документация](/javascript/api/@azure/cognitiveservices-qnamaker/) | [Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/cognitiveservices/cognitiveservices-qnamaker) | [Пакет (npm)](https://www.npmjs.com/package/@azure/cognitiveservices-qnamaker) | [Примеры Node.js](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/preview-sdk/quickstart.js)

---

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Предварительные требования

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Текущая версия [Node.js](https://nodejs.org).
* Получив подписку Azure, создайте [ресурс QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) на портале Azure, чтобы получить ключ разработки и ресурс. После развертывания ресурса выберите элемент **Перейти к ресурсу**.
    * Для подключения приложения к API QnA Maker потребуется ключ и имя созданного ресурса. Ключ и имя ресурса вы вставите в приведенный ниже код на следующих шагах в рамках этого краткого руководства.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Текущая версия [Node.js](https://nodejs.org).
* Получив подписку Azure, создайте [ресурс QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) на портале Azure, чтобы получить ключ разработки и конечную точку.
    * ПРИМЕЧАНИЕ. Обязательно установите флажок **Управляемый**.
    * После развертывания ресурса QnA Maker выберите **Перейти к ресурсу**. Для подключения приложения к API QnA Maker потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

---

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-nodejs-application"></a>создание приложения Node.js;

В окне консоли (например, cmd, PowerShell или Bash) создайте новый каталог для приложения и перейдите в него.

```console
mkdir qnamaker_quickstart && cd qnamaker_quickstart
```

Выполните команду `npm init -y`, чтобы создать приложение узла с помощью файла `package.json`.

```console
npm init -y
```

### <a name="install-the-client-library"></a>Установка клиентской библиотеки

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

Установите следующие пакеты npm:

```console
npm install @azure/cognitiveservices-qnamaker
npm install @azure/cognitiveservices-qnamaker-runtime
npm install @azure/ms-rest-js
```

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

Установите следующие пакеты npm:

```console
npm install @azure/cognitiveservices-qnamaker
npm install @azure/ms-rest-js
```

---

Файл `package.json` приложения обновлен с учетом зависимостей.

Создайте файл с именем index.js, откройте его и импортируйте следующие библиотеки:

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-javascript[Dependencies](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=Dependencies)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-javascript[Dependencies](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=Dependencies)]

---

Создайте переменную для ключа Azure и имени ресурса.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

- Ключ подписки и ключ разработки являются взаимозаменяемыми. Дополнительные сведения о ключе разработки см. в разделе [Ключи в QnA Maker](../concepts/azure-resources.md?tabs=v1#keys-in-qna-maker).

- Формат значения QNA_MAKER_ENDPOINT: `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Перейдите на портал Azure и найдите ресурс QnA Maker, созданный в рамках выполнения предварительных требований. Щелкните страницу **Ключи и конечная точка** в разделе **Управление ресурсами**, чтобы найти ключ разработки (подписка) и конечную точку QnA Maker.

 ![Конечная точка разработки QnA Maker](../media/keys-endpoint.png)

- Формат значения QNA_MAKER_RUNTIME_ENDPOINT: `https://YOUR-RESOURCE-NAME.azurewebsites.net`. Перейдите на портал Azure и найдите ресурс QnA Maker, созданный в рамках выполнения предварительных требований. Щелкните страницу **Экспорт шаблона** в разделе **Автоматизация**, чтобы найти конечную точку среды выполнения.

 ![Конечная точка среды выполнения QnA Maker](../media/runtime-endpoint.png)
   
- Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [Azure Key Vault](../../../key-vault/general/overview.md) предоставляет безопасное хранилище ключей.

[!code-javascript[Set the resource key and resource name](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=Resourcevariables)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

- Ключ подписки и ключ разработки являются взаимозаменяемыми. Дополнительные сведения о ключе разработки см. в разделе [Ключи в QnA Maker](../concepts/azure-resources.md?tabs=v2#keys-in-qna-maker).

- Формат значения QNA_MAKER_ENDPOINT: `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Перейдите на портал Azure и найдите ресурс QnA Maker, созданный в рамках выполнения предварительных требований. Щелкните страницу **Ключи и конечная точка** в разделе **Управление ресурсами**, чтобы найти ключ разработки (подписка) и конечную точку QnA Maker.

 ![Конечная точка разработки QnA Maker](../media/keys-endpoint.png)
 
- Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [Azure Key Vault](../../../key-vault/general/overview.md) предоставляет безопасное хранилище ключей.

[!code-javascript[Set the resource key and resource name](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=Resourcevariables)]

---

## <a name="object-models"></a>Объектные модели

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

В [QnA Maker](/javascript/api/@azure/cognitiveservices-qnamaker/) используются две разные объектные модели:
* **[QnAMakerClient](#qnamakerclient-object-model)**  — это объект для создания, публикации и скачивания базы знаний, а также управления ею.
* **[QnAMakerRuntime](#qnamakerruntimeclient-object-model)**  — это объект для отправки запроса в базу знаний с помощью API GenerateAnswer и новых предложенных вопросов с помощью API обучения (в рамках [активного обучения](../how-to/use-active-learning.md)).

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

В [QnA Maker](/javascript/api/@azure/cognitiveservices-qnamaker/) используется следующая объектная модель:
* **[QnAMakerClient](#qnamakerclient-object-model)**  — это объект для создания, публикации и скачивания базы знаний, а также управления ею и создания запросов к ней.

---

### <a name="qnamakerclient-object-model"></a>Объектная модель QnAMakerClient

Клиент разработки QnA Maker — это объект [QnAMakerClient](/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient), который проходит проверку подлинности в Azure с учетными данными, включающими ваш ключ.

После создания клиента используйте [Базу знаний](/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient#knowledgebase) для создания, управления и публикации своей базы знаний.

Управляйте своей базой знаний, отправляя объект JSON. Для немедленных операций метод обычно возвращает объект JSON, указав статус. Ответ для длительных операций — ИД операции. Вызовите метод [client.operations.getDetails](/javascript/api/@azure/cognitiveservices-qnamaker/operations#getdetails-string--msrest-requestoptionsbase-) с идентификатором операции, чтобы определить [состояние запроса](/javascript/api/@azure/cognitiveservices-qnamaker/operation).

### <a name="qnamakerruntimeclient-object-model"></a>Объектная модель QnAMakerRuntimeClient

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

Клиент прогнозирования QnA Maker — это объект QnAMakerRuntimeClient, который проходит проверку подлинности в Azure с помощью объекта Microsoft.Rest.ServiceClientCredentials, содержащего ключ среды выполнения прогнозирования, полученный из вызова [client.EndpointKeys.getKeys](/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeys#getkeys-msrest-requestoptionsbase-) к клиенту разработки после публикации базы знаний.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

Управляемый ресурс QnA Maker не требует обязательного использования объекта QnAMakerRuntimeClient. Вместо этого вызовите [generateAnswer](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#generateAnswer_string__QueryDTO__msRest_RequestOptionsBase_) непосредственно в объекте [QnAMakerClient](/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient).

---

## <a name="code-examples"></a>Примеры кода

Эти фрагменты кода показывают, как выполнить следующие действия с помощью клиентской библиотеки QnA Maker для .NET:

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

* [Проверка подлинности клиента разработки](#authenticate-the-client-for-authoring-the-knowledge-base)
* [Создание базы знаний](#create-a-knowledge-base)
* [Обновление базы знаний](#update-a-knowledge-base)
* [Скачивание базы знаний](#download-a-knowledge-base)
* [Публикация базы знаний](#publish-a-knowledge-base)
* [Удаление базы знаний](#delete-a-knowledge-base)
* [Получение ключа среды выполнения запроса](#get-query-runtime-key)
* [Получение состояния операции](#get-status-of-an-operation)
* [Проверка подлинности клиента среды выполнения запросов](#authenticate-the-runtime-for-generating-an-answer)
* [Создание ответа из базы знаний](#generate-an-answer-from-the-knowledge-base)

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

* [Проверка подлинности клиента разработки](#authenticate-the-client-for-authoring-the-knowledge-base)
* [Создание базы знаний](#create-a-knowledge-base)
* [Обновление базы знаний](#update-a-knowledge-base)
* [Скачивание базы знаний](#download-a-knowledge-base)
* [Публикация базы знаний](#publish-a-knowledge-base)
* [Удаление базы знаний](#delete-a-knowledge-base)
* [Получение состояния операции](#get-status-of-an-operation)
* [Создание ответа из базы знаний](#generate-an-answer-from-the-knowledge-base)

---

## <a name="authenticate-the-client-for-authoring-the-knowledge-base"></a>Проверка подлинности клиента для создания базы знаний

Создайте экземпляр клиента с конечной точкой и ключом. Создайте объект ServiceClientCredentials с использованием вашего ключа и примените его с конечной точкой, чтобы создать объект [QnAMakerClient](/javascript/api/@azure/cognitiveservices-qnamaker/qnamakerclient).

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-javascript[Create QnAMakerClient object with key and endpoint](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=AuthorizationAuthor)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-javascript[Create QnAMakerClient object with key and endpoint](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=AuthorizationAuthor)]

---

## <a name="create-a-knowledge-base"></a>Создание базы знаний

База знаний хранит пары вопросов и ответов для объекта [Класс CreateKbDTO](/javascript/api/@azure/cognitiveservices-qnamaker/createkbdto) из трех источников:

* Для **редакционного содержимого** используйте объект [Класс QnADTO](/javascript/api/@azure/cognitiveservices-qnamaker/qnadto).
    * Чтобы использовать метаданные и дальнейшие запросы, используйте редакционный контекст, так как эти данные добавляются на уровне отдельных пар вопросов и ответов.
* Для **файлов** используйте объект [Класс FileDTO](/javascript/api/@azure/cognitiveservices-qnamaker/filedto). FileDTO включает имя файла, а также общедоступный URL-адрес для доступа к этому файлу.
* В качестве значения для параметра **URL-адреса** используйте список строк, представляющих общедоступные URL-адреса.

На этапе создания также задаются свойства базы знаний:
* `defaultAnswerUsedForExtraction` — то, что возвращается, если ответ не найден;
* `enableHierarchicalExtraction` — автоматическое создание связей запросов между извлеченными парами вопросов и ответов;
* `language` — при создании первой базы данных ресурса задайте язык для использования в индексе Поиска Azure.

Вызовите метод ["Создать"](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#create-createkbdto--servicecallback-operation--) со сведениями о базе знаний. Сведения о базе знаний по сути представляют собой объект JSON.

После возвращения метода "Создать" передайте возвращенный идентификатор операции методу [wait_for_operation](#get-status-of-an-operation) для опроса состояния. Метод "wait_for_operation" возвращает значение после завершения операции. Проанализируйте значение заголовка `resourceLocation` возвращенной операции, чтобы получить новый идентификатор базы знаний.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-javascript[Create knowledgebase](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=CreateKBMethod)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-javascript[Create knowledgebase](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=CreateKBMethod)]

---

Чтобы создать базу знаний, добавьте функцию [`wait_for_operation`](#get-status-of-an-operation) из предыдущего примера кода.

## <a name="update-a-knowledge-base"></a>Обновление базы знаний

Базу знаний можно обновить, передав идентификатор базы знаний и метод [UpdateKbOperationDTO](/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto), содержащий [добавить](/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto#add), [обновить](/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto#update) и [удалить](/javascript/api/@azure/cognitiveservices-qnamaker/updatekboperationdto#deleteproperty) объекты DTO в метод [обновить](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#update-string--updatekboperationdto--msrest-requestoptionsbase-). DTO также являются объектами JSON. Используйте метод [wait_for_operation](#get-status-of-an-operation), чтобы определить, успешно ли выполнено обновление.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-javascript[Update a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=UpdateKBMethod)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-javascript[Update a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=UpdateKBMethod)]

---

Чтобы обновить базу знаний, добавьте функцию [`wait_for_operation`](#get-status-of-an-operation) из предыдущего примера кода.

## <a name="download-a-knowledge-base"></a>Скачивание базы знаний

Используйте метод [загрузить](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#download-string--models-environmenttype--msrest-requestoptionsbase-), чтобы загрузить базу данных в виде списка [QnADocumentsDTO](/javascript/api/@azure/cognitiveservices-qnamaker/qnadocumentsdto). Это _не_ эквивалентно экспорту портала QnA Maker со страницы **настроек**, поскольку результатом этого метода не является файл TSV.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-javascript[Download a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=DownloadKB)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-javascript[Download a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=DownloadKB)]

---

## <a name="publish-a-knowledge-base"></a>Публикация базы знаний

Опубликуйте базу знаний, используя метод [опубликовать](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#publish-string--msrest-requestoptionsbase-). При этом берется текущая сохраненная и обученная модель, на которую ссылается ИД базы знаний, и публикуется ее в конечной точке. Проверьте код ответа HTTP, чтобы убедиться, что публикация прошла удачно.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-javascript[Publish a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=PublishKB)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-javascript[Publish a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=PublishKB)]

---

## <a name="query-a-knowledge-base"></a>Запрос к базе знаний

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

### <a name="get-query-runtime-key"></a>Получение ключа среды выполнения запроса

После публикации базы знаний необходимо получить ключ среды выполнения запроса, чтобы отправить в нее запрос. Этот ключ отличается о того, который использовался для создания исходного объекта клиента.

Используйте метод [EndpointKeys.getKeys](/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeys), чтобы получить класс [EndpointKeysDTO](/javascript/api/@azure/cognitiveservices-qnamaker/endpointkeysdto).

Для запроса к базе знаний используйте любое свойство ключа, возвращаемое в объекте.

[!code-javascript[Get query runtime key](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=GetQueryEndpointKey)]

### <a name="authenticate-the-runtime-for-generating-an-answer"></a>Проверка подлинности среды выполнения для создания ответа

Создайте QnAMakerRuntimeClient, чтобы отправить в базу знаний запрос для создания ответа или обучения по методу активного обучения.

[!code-javascript[Authenticate the runtime](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=AuthorizationQuery)]

Используйте QnAMakerRuntimeClient, чтобы получить ответ от базы знаний или отправить в нее новые предложенные вопросы для [активного обучения](../concepts/active-learning-suggestions.md).

### <a name="generate-an-answer-from-the-knowledge-base"></a>Создание ответа из базы знаний

Создайте ответ из опубликованной базы знаний с помощью метода RuntimeClient.runtime.generateAnswer. Этот метод принимает идентификатор базы знаний и [QueryDTO](/javascript/api/@azure/cognitiveservices-qnamaker/querydto). Получите дополнительные свойства QueryDTO, например Top и Context, чтобы использовать их в чат-боте.

[!code-javascript[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=GenerateAnswer)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

### <a name="generate-an-answer-from-the-knowledge-base"></a>Создание ответа из базы знаний

Сгенерируйте ответ из опубликованной базы знаний с использованием метода QnAMakerClient.knowledgebase.generateAnswer. Этот метод принимает идентификатор базы знаний и [QueryDTO](/javascript/api/@azure/cognitiveservices-qnamaker/querydto). Получите дополнительные свойства QueryDTO, например Top и Context, чтобы использовать их в чат-боте.

[!code-javascript[Generate an answer from a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=GenerateAnswer)]

---

Это простой пример отправки запроса в базу знаний. Чтобы ознакомиться с расширенными сценариями запросов, см. [другие примеры запросов](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md?pivots=url-test-tool-curl#use-curl-to-query-for-a-chit-chat-answer).

## <a name="delete-a-knowledge-base"></a>Удаление базы знаний

Удалите базу знаний с помощью метода [удалить](/javascript/api/@azure/cognitiveservices-qnamaker/knowledgebase#deletemethod-string--msrest-requestoptionsbase-) с параметром идентификатора базы знаний.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-javascript[Delete a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=DeleteKB)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-javascript[Delete a knowledge base](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=DeleteKB)]

---

## <a name="get-status-of-an-operation"></a>Получение состояния операции

Некоторые методы, такие как "create" и "update", могут занять достаточно времени, чтобы вместо ожидания завершения процесса был возвращен [класс операции](/javascript/api/@azure/cognitiveservices-qnamaker/operations). Используйте [свойство Operation.OperationId](/javascript/api/@azure/cognitiveservices-qnamaker/operation#operationid) из операции для опроса (с логикой повторных попыток), чтобы определить состояние исходного метода.

Вызов _delayTimer_ в следующем блоке кода используется для имитации логики повторных попыток. Замените его на собственную логику повторных попыток.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-javascript[Monitor an operation](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/qnamaker_quickstart.js?name=MonitorOperation)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-javascript[Monitor an operation](~/cognitive-services-quickstart-code/javascript/QnAMaker/sdk/preview-sdk/quickstart.js?name=MonitorOperation)]

---

## <a name="run-the-application"></a>Выполнение приложения

Запустите приложение с командой `node index.js` из каталога приложения.

```console
node index.js
```

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/qnamaker_quickstart.js).

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/javascript/QnAMaker/sdk/preview-sdk/quickstart.js).

---
