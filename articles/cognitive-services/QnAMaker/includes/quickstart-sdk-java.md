---
title: Краткое руководство. Клиентская библиотека QnA Maker для Java
description: В этом кратком руководстве показано, как начать работу с клиентской библиотекой QnA Maker для Java.
author: v-jaswel
manager: chrhoder
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: include
ms.date: 09/04/2020
ms.author: v-jawe
ms.openlocfilehash: 755e4fcb9d5b9f1711bc615a29489037ea377d43
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105609494"
---
# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

Клиентская библиотека QnA Maker для Java позволяет выполнить указанные ниже задачи.

* Создание базы знаний
* Обновление базы знаний
* Публикация базы знаний
* Получение ключа конечной точки среды выполнения прогнозирования
* Ожидание долго выполняющейся задачи
* Скачивание базы знаний
* Получение ответа из базы знаний
* Удаление базы знаний

[Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker) | [Пакет](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-qnamaker/1.0.0-beta.1) | [Примеры](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/qnamaker/sdk/quickstart.java)

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

Клиентская библиотека QnA Maker для Java позволяет выполнить указанные ниже задачи.

* Создание базы знаний
* Обновление базы знаний
* Публикация базы знаний
* Ожидание долго выполняющейся задачи
* Скачивание базы знаний
* Получение ответа из базы знаний
* Удаление базы знаний

[Исходный код библиотеки](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker) | [Пакет](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-qnamaker/1.0.0-beta.2) | [Примеры](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/java/qnamaker/sdk/preview-sdk/quickstart.java)

---

[!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="prerequisites"></a>Предварительные требования

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* [JDK](https://www.oracle.com/java/technologies/javase-downloads.html).
* Получив подписку Azure, создайте [ресурс QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) на портале Azure, чтобы получить ключ разработки и конечную точку. После развертывания ресурса выберите элемент **Перейти к ресурсу**.
    * Для подключения приложения к API QnA Maker потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* [JDK](https://www.oracle.com/java/technologies/javase-downloads.html).
* Получив подписку Azure, создайте [ресурс QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker) на портале Azure, чтобы получить ключ разработки и конечную точку.
    * ПРИМЕЧАНИЕ. Обязательно установите флажок **Управляемый**.
    * После развертывания ресурса QnA Maker выберите **Перейти к ресурсу**. Для подключения приложения к API QnA Maker потребуется ключ и конечная точка из созданного ресурса. Ключ и конечная точка будут вставлены в приведенный ниже код в кратком руководстве.
    * Используйте бесплатную ценовую категорию (`F0`), чтобы опробовать службу, а затем выполните обновление до платного уровня для рабочей среды.

---

## <a name="setting-up"></a>Настройка

### <a name="install-the-client-libraries"></a>Установка клиентских библиотек

После установки Java можно установить клиентские библиотеки с помощью [Maven](https://maven.apache.org/) из [репозитория MVN](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-qnamaker).

### <a name="create-a-new-java-application"></a>Создание нового приложения Java

Создайте файл с именем `quickstart.java` и импортируйте указанные ниже библиотеки.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Dependencies](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=dependencies)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Dependencies](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=dependencies)]

---

Создайте переменные для конечной точки Azure и ключа ресурса.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

- Ключ подписки и ключ разработки являются взаимозаменяемыми. Дополнительные сведения о ключе разработки см. в разделе [Ключи в QnA Maker](../concepts/azure-resources.md?tabs=v1#keys-in-qna-maker).

- Формат значения QNA_MAKER_ENDPOINT: `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Перейдите на портал Azure и найдите ресурс QnA Maker, созданный в рамках выполнения предварительных требований. Щелкните страницу **Ключи и конечная точка** в разделе **Управление ресурсами**, чтобы найти ключ разработки (подписка) и конечную точку QnA Maker.

 ![Конечная точка разработки QnA Maker](../media/keys-endpoint.png)

- Формат значения QNA_MAKER_RUNTIME_ENDPOINT: `https://YOUR-RESOURCE-NAME.azurewebsites.net`. Перейдите на портал Azure и найдите ресурс QnA Maker, созданный в рамках выполнения предварительных требований. Щелкните страницу **Экспорт шаблона** в разделе **Автоматизация**, чтобы найти конечную точку среды выполнения.

 ![Конечная точка среды выполнения QnA Maker](../media/runtime-endpoint.png)
   
- Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [Azure Key Vault](../../../key-vault/general/overview.md) предоставляет безопасное хранилище ключей.

[!code-java[Resource variables](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=resourceKeys)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

- Ключ подписки и ключ разработки являются взаимозаменяемыми. Дополнительные сведения о ключе разработки см. в разделе [Ключи в QnA Maker](../concepts/azure-resources.md?tabs=v2#keys-in-qna-maker).

- Формат значения QNA_MAKER_ENDPOINT: `https://YOUR-RESOURCE-NAME.cognitiveservices.azure.com`. Перейдите на портал Azure и найдите ресурс QnA Maker, созданный в рамках выполнения предварительных требований. Щелкните страницу **Ключи и конечная точка** в разделе **Управление ресурсами**, чтобы найти ключ разработки (подписка) и конечную точку QnA Maker.

 ![Конечная точка разработки QnA Maker](../media/keys-endpoint.png)
 
- Для рабочей среды рекомендуется использовать безопасный способ хранения и доступа к учетным данным. Например, [Azure Key Vault](../../../key-vault/general/overview.md) предоставляет безопасное хранилище ключей.

[!code-java[Resource variables](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=resourceKeys)]

---

## <a name="object-models"></a>Объектные модели

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

QnA Maker использует две различные объектные модели.
* **[QnAMakerClient](#qnamakerclient-object-model)**  — это объект для создания, публикации и скачивания базы знаний, а также управления ею.
* **[QnAMakerRuntime](#qnamakerruntimeclient-object-model)**  — это объект для отправки запроса в базу знаний с помощью API GenerateAnswer и новых предложенных вопросов с помощью API обучения (в рамках [активного обучения](../how-to/use-active-learning.md)).

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

В QnA Maker используется следующая объектная модель:
* **[QnAMakerClient](#qnamakerclient-object-model)**  — это объект для создания, публикации и скачивания базы знаний, а также управления ею и создания запросов к ней.

---

[!INCLUDE [Get KBinformation](./quickstart-sdk-cognitive-model.md)]

### <a name="qnamakerclient-object-model"></a>Объектная модель QnAMakerClient

Клиент разработки QnA Maker — это объект [QnAMakerClient](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerClient.java), который проходит проверку подлинности в Azure с помощью объекта MsRest::ServiceClientCredentials, содержащего ваш ключ.

После создания клиента используйте методы свойства [Knowledgebases](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java) клиента для создания и публикации своей базы знаний, а также управления ею.

Для немедленных операций метод обычно возвращает результат, если таковой имеется. Ответ для длительных операций — объект [Operation](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/Operation.java). Вызовите метод [getDetails](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Operations.java#L32) со значением `operation.operationId`, чтобы определить [состояние запроса](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/OperationStateType.java).

### <a name="qnamakerruntimeclient-object-model"></a>Объектная модель QnAMakerRuntimeClient

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

Клиент QnA Maker среды выполнения — объект [QnAMakerRuntimeClient](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerClient.java).

После публикации базы знаний с помощью клиента разработки используйте метод [generateAnswer](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Runtimes.java#L36) клиента среды выполнения, чтобы получить ответ от базы знаний.

Чтобы создать клиент среды выполнения, вызовите метод [QnAMakerRuntimeManager.authenticate](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerRuntimeManager.java#L29) и передайте ключ конечной точки среды выполнения. Чтобы получить ключ конечной точки среды выполнения, вызовите [getKeys](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/EndpointKeys.java#L30) с помощью клиента разработки.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

Управляемый ресурс QnA Maker не требует обязательного использования объекта QnAMakerRuntimeClient. Вместо этого вызовите [generateAnswer](https://github.com/Azure/azure-sdk-for-java/blob/657e9a47e4b4c7e7e7eee4100273c09468a30c63/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L308) непосредственно в объекте [QnAMakerClient](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerClient.java).

---

## <a name="authenticate-the-client-for-authoring-the-knowledge-base"></a>Проверка подлинности клиента для создания базы знаний

Создайте экземпляр клиента с помощью конечной точки разработки и ключа подписки.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Authenticate](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=authenticate)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Authenticate](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=authenticate)]

---

## <a name="create-a-knowledge-base"></a>Создание базы знаний

База знаний хранит пары вопросов и ответов для объекта [Класс CreateKbDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/CreateKbDTO.java) из трех источников:

* Для **редакционного содержимого** используйте объект [Класс QnADTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/QnADTO.java).
    * Чтобы использовать метаданные и дальнейшие запросы, используйте редакционный контекст, так как эти данные добавляются на уровне отдельных пар вопросов и ответов.
* Для **файлов** используйте объект [Класс FileDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/FileDTO.java). FileDTO включает имя файла, а также общедоступный URL-адрес для доступа к этому файлу.
* В качестве значения для параметра **URL-адреса** используйте список строк, представляющих общедоступные URL-адреса.

Вызовите метод [create](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L173), затем передайте свойство `operationId` возвращенной операции в метод [getDetails](#get-status-of-an-operation) для опроса состояния.

Последняя строка указанного ниже кода возвращает идентификатор базы знаний.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Create knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=createKb)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Create knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=createKb)]

---

## <a name="update-a-knowledge-base"></a>Обновление базы знаний

Вы можете обновить базу знаний, вызвав метод [update](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L150), а затем передав ее идентификатор и объект [UpdateKbOperationDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/UpdateKbOperationDTO.java). Этот объект, в свою очередь, может содержать следующие методы:
- [add](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/UpdateKbOperationDTOAdd.java)
- [update](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/UpdateKbOperationDTOUpdate.java)
- [delete](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/UpdateKbOperationDTODelete.java)

Передайте свойство `operationId` возвращенной операции в метод [getDetails](#get-status-of-an-operation) для опроса состояния.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Update knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=updateKb)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Update knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=updateKb)]

---

## <a name="download-a-knowledge-base"></a>Скачивание базы знаний

Используйте метод [загрузить](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L196), чтобы загрузить базу данных в виде списка [QnADocumentsDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/QnADocumentsDTO.java). Это _не_ эквивалентно экспорту портала QnA Maker со страницы **настроек**, поскольку результатом этого метода не является файл TSV.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Download knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=downloadKb)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Download knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=downloadKb)]

---

## <a name="publish-a-knowledge-base"></a>Публикация базы знаний

Опубликуйте базу знаний, используя метод [опубликовать](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L196). При этом берется текущая сохраненная и обученная модель, на которую ссылается ИД базы знаний, и публикуется ее в конечной точке.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Publish knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=publishKb)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Publish knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=publishKb)]

---

## <a name="generate-an-answer-from-the-knowledge-base"></a>Создание ответа из базы знаний

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

После публикации базы знаний необходимо получить ключ конечной точки среды выполнения, чтобы отправить в нее запрос. Он отличается от ключа подписки, используемого для создания клиента разработки.

Используйте метод [getKeys](https://github.com/Azure/azure-sdk-for-java/blob/b637366e32edefb0fe63962983715a02c1ad2631/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/EndpointKeys.java#L30), чтобы получить объект [EndpointKeysDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/EndpointKeysDTO.java).

Создайте клиент среды выполнения, вызвав [QnAMakerRuntimeManager.authenticate](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/QnAMakerRuntimeManager.java#L29), и передайте ключ конечной точки среды выполнения из объекта EndpointKeysDTO.

Создайте ответ из опубликованной базы знаний с помощью метода [generateAnswer](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Runtimes.java#L36). Этот метод принимает идентификатор базы знаний и объект [QueryDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/QueryDTO.java).

[!code-java[Query knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=queryKb)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

Создайте ответ из опубликованной базы знаний с помощью метода [generateAnswer](https://github.com/Azure/azure-sdk-for-java/blob/657e9a47e4b4c7e7e7eee4100273c09468a30c63/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L308). Этот метод принимает идентификатор базы знаний и объект [QueryDTO](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/QueryDTO.java).

[!code-java[Query knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=queryKb)]

---

Это простой пример отправки запроса в базу знаний. Чтобы ознакомиться с расширенными сценариями запросов, см. [другие примеры запросов](../quickstarts/get-answer-from-knowledge-base-using-url-tool.md?pivots=url-test-tool-curl#use-curl-to-query-for-a-chit-chat-answer).

## <a name="delete-a-knowledge-base"></a>Удаление базы знаний

Удалите базу знаний с помощью метода [удалить](https://github.com/Azure/azure-sdk-for-java/blob/b455a61f4c6daece13590a0f4136bab3c4f30546/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/Knowledgebases.java#L81) с параметром идентификатора базы знаний.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Delete knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=deleteKb)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Delete knowledgebase](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=deleteKb)]

---

## <a name="get-status-of-an-operation"></a>Получение состояния операции

Некоторые методы, такие как "create" и "update", могут занять достаточно времени, чтобы вместо ожидания завершения процесса был возвращен [класс операции](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/cognitiveservices/ms-azure-cs-qnamaker/src/main/java/com/microsoft/azure/cognitiveservices/knowledge/qnamaker/models/Operation.java). Используйте идентификатор операции из операции для опроса (с логикой повторных попыток), чтобы определить состояние исходного метода.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Wait for operation](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=waitForOperation)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Wait for operation](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=waitForOperation)]

---

## <a name="run-the-application"></a>Выполнение приложения

Ниже приведен метод Main для приложения.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

[!code-java[Main method](~/cognitive-services-quickstart-code/java/qnamaker/sdk/quickstart.java?name=main)]

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

[!code-java[Main method](~/cognitive-services-quickstart-code/java/qnamaker/sdk/preview-sdk/quickstart.java?name=main)]

---

Запустите приложение следующим образом: Предполагается, что имя класса — `Quickstart`, а зависимости находятся во вложенной папке с именем `lib` под текущей папкой.

```console
javac Quickstart.java -cp .;lib\*
java -cp .;lib\* Quickstart
```

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/version-1)

Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/qnamaker/sdk/quickstart.java).

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/version-2)

Исходный код для этого шаблона можно найти на портале [GitHub](https://github.com/Azure-Samples/cognitive-services-quickstart-code/blob/master/java/qnamaker/sdk/preview-sdk/quickstart.java).

---
