---
title: Руководство по Создание бота вопросов и ответов с помощью Службы Azure Bot
description: В этом руководстве показано, как создать бот вопросов и ответов с помощью QnA Maker и Службы Azure Bot без необходимости писать код.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: tutorial
ms.date: 08/31/2020
ms.openlocfilehash: ab6607175c596a0d82cf75f0ad786a76e85b6959
ms.sourcegitcommit: d135e9a267fe26fbb5be98d2b5fd4327d355fe97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102612156"
---
# <a name="tutorial-create-an-faq-bot-with-azure-bot-service"></a>Руководство по Создание бота вопросов и ответов с помощью Службы Azure Bot
Создайте бот вопросов и ответов с помощью QnA Maker и [Службы Azure Bot](https://azure.microsoft.com/services/bot-service/) без необходимости писать код.

В этом руководстве описано следующее:

<!-- green checkmark -->
> [!div class="checklist"]
> * связывание базы знаний QnA Maker со Службой Azure Bot;
> * развертывание бота;
> * общение с ботом в веб-чате;
> * подключение бота к поддерживаемым каналам.

## <a name="create-and-publish-a-knowledge-base"></a>Создание и публикация базы знаний

Чтобы создать базу знаний, следуйте указаниям в [кратком руководстве](../Quickstarts/create-publish-knowledge-base.md). После успешной публикации базы знаний вы получите доступ к следующей странице.

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/v1)

![Снимок экрана с успешной публикацией](../media/qnamaker-create-publish-knowledge-base/publish-knowledge-base-to-endpoint.png)

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/v2)

![Снимок экрана с успешной публикацией (управляемая служба)](../media/qnamaker-create-publish-knowledge-base/publish-knowledge-base-to-endpoint-managed.png)

---

## <a name="create-a-bot"></a>Создание бота

После публикации можно создать бота на странице **Publish** (Публикация):

* Можно создать несколько ботов, которые будут работать с одной базой знаний, для разных регионов или тарифных планов.
* Если нужен только один бот для базы знаний, перейдите по ссылке **View all your bots on the Azure portal** (Просмотреть всех ботов на портале Azure), чтобы просмотреть список текущих ботов.

При внесении изменений в базу знаний и ее повторной публикации не требуется предпринимать какие-либо действия с ботом. Он уже настроен для работы с базой знаний и будет учитывать все ее последующие изменения. При каждой публикации базы знаний все боты, подключенные к ней, обновляются автоматически.

1. На портале QnA Maker, на странице **Publish** (Публикация) выберите **Create bot** (Создать бота). Эта кнопка отображается только после публикации базы знаний.

     # <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/v1)

    ![Снимок экрана с созданием бота](../media/qnamaker-create-publish-knowledge-base/create-bot-from-published-knowledge-base-page.png)

    # <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/v2)

    ![Снимок экрана с созданием бота (управляемая служба, предварительная версия)](../media/qnamaker-create-publish-knowledge-base/create-bot-from-published-knowledge-base-page-managed.png)

    ---
    

1. В новой вкладке браузера откроется страница создания службы Azure Bot на портале Azure. Настройте службу Azure Bot. Бот и QnA Maker могут иметь общий план службы веб-приложений, но не могут совместно использовать одно веб-приложение. Это означает, что **имя приложения** для бота должно отличаться от имени приложения для службы QnA Maker.

    * **Рекомендуется**
        * Измените дескриптор бота, если он не уникален.
        * Выберите язык пакета SDK. После создания программы-робота можно загрузить код в локальную среду разработки и продолжить разработку.
    * **Не рекомендуется**
        * Создавая бот, не изменяйте следующие параметры на портале Azure. Их значения автоматически заполняются для базы знаний:
           * ключ проверки подлинности QnA;
           * расположение и план службы приложений;


1. После создания бота откройте ресурс **Bot service** (Служба бота).
1. В разделе **управление ботом** выберите **Test in Web Chat** (Тестировать в веб-чате).
1. В приглашении чата **Type your message** (Тип сообщения), введите:

    `Azure services?`

    Чат-бот ответит, используя ответ из базы знаний.

    :::image type="content" source="../media/qnamaker-create-publish-knowledge-base/test-web-chat.png" alt-text="Введите запрос пользователя в тестовый веб-чат.":::
1. Подключите бот к другим [поддерживаемым каналам](/azure/bot-service/bot-service-manage-channels).