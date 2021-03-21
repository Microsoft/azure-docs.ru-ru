---
title: Множественная перевращение диалогов — QnA Maker
description: Используйте приглашения и контекст для управления несколькими процессами, называемыми многоповоротными, для программы-робота с одного вопроса на другой. Множественная передача — это возможность иметь обратную и сквозную беседу, когда контекст предыдущего вопроса влияет на следующий вопрос и ответ.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 04/13/2020
ms.openlocfilehash: 313d1a390c30e7e5612b8d9bab7783b6698c35fd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102618497"
---
# <a name="use-follow-up-prompts-to-create-multiple-turns-of-a-conversation"></a>Использование дальнейших подсказок для создания диалога с несколькими шагами

Используйте запросы к исполнению и контекст для управления несколькими процессами, которые называются _многофакторными_, для программы-робота с одного вопроса на другой.

Чтобы увидеть, как работает Multi-Turn, просмотрите следующее демонстрационное видео:

[![Множественная перевращение диалога в QnA Maker](../media/conversational-context/youtube-video.png)](https://aka.ms/multiturnexample)

## <a name="what-is-a-multi-turn-conversation"></a>Что такое множественная беседа?

Ответить на некоторые вопросы можно одним из них. При проектировании бесед с клиентским приложением (Чат Bot) пользователь может задать вопрос, для которого необходимо выполнить фильтрацию или уточнение, чтобы определить правильный ответ. Вы сделаете этот поток через все возможные вопросы, предоставляя пользователю *запросы к исполнению*.

Когда пользователь запросит вопрос, QnA Maker возвращает ответ _и_ все дальнейшие подсказки. Этот ответ позволяет представлять дальнейшие вопросы как варианты выбора.

> [!CAUTION]
> Многострочные запросы не извлекаются из документов часто задаваемых вопросов. Если требуется множественное извлечение, удалите вопросительные знаки, обозначающие пары QnA, как часто задаваемые вопросы.

## <a name="example-multi-turn-conversation-with-chat-bot"></a>Пример многофункциональной беседы с помощью Chat Bot

С помощью множественного включения Bot робота управляет разговором с пользователем, чтобы определить окончательный ответ, как показано на следующем рисунке:

![Диалоговое окно с множественной обобщением с запросом, с помощью которого пользователь проходит через беседу.](../media/conversational-context/conversation-in-bot.png)

На предыдущем рисунке пользователь начал диалог, вводя **мою учетную запись**. В базе знаний есть три связанные пары вопросов и ответов. Чтобы уточнить ответ, пользователь выбирает один из трех вариантов в базе знаний. Вопрос (#1) содержит три следующих приглашения, которые отображаются в роботе чата в виде трех вариантов (#2).

Когда пользователь выбирает параметр (#3), отображается следующий список параметров уточнения (#4). Эта последовательность продолжится (#5), пока пользователь не определит правильный, окончательный ответ (#6).

### <a name="use-multi-turn-in-a-bot"></a>Использование множественного включения в роботе

После публикации базы знаний можно нажать кнопку **создать Bot** , чтобы развернуть QnA Maker Bot в службе Azure Bot. Приглашения будут отображаться в клиентах разговора, которые включены для программы-робота.

## <a name="create-a-multi-turn-conversation-from-a-documents-structure"></a>Создание многофакторной беседы из структуры документа

Когда вы создаете базу знаний, в разделе **Заполнение статьи KB** отображается флажок **Включить множественное извлечение из URL-адресов, файлов PDF или DOCX** .

![Флажок для включения множественного извлечения](../media/conversational-context/enable-multi-turn.png)

При выборе этого параметра QnA Maker извлекает иерархию, имеющуюся в структуре документа. Иерархия преобразуется в для дальнейших запросов, а корень иерархии выступает в качестве родительского QnA. В некоторых документах корневой элемент иерархии не имеет содержимого, которое может выступать в качестве ответа. Вы можете указать текст ответа по умолчанию, который будет использоваться в качестве замещающего текста ответа для извлечения таких иерархий.

Структура множественной переворачивания может быть выведена только из URL-адресов, PDF-файлов или файлов DOCX. Пример структуры см. в образе [PDF-файла пользовательской документации по Microsoft Surface](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/product-manual.pdf).

:::image type="content" source="../media/conversational-context/import-file-with-conversational-structure.png" alt-text="На снимке экрана показан пример структуры в руководстве пользователя." lightbox="../media/conversational-context/import-file-with-conversational-structure.png":::

### <a name="building-your-own-multi-turn-document"></a>Создание собственного документа с несколькими переворотами

При создании документа с несколькими переворотами учитывайте следующие рекомендации.

* Используйте заголовки и подзаголовки для обозначения иерархии. Например, можно заметить, что в качестве родительского QnA и H2 вы можете обозначить QnA, который должен быть сделан запросом. Используйте Малый размер заголовка для обозначения последующей иерархии. Не используйте стиль, цвет или какой-либо другой механизм для включения структуры в документ, QnA Maker не извлекать многострочные запросы.

* Первый символ заголовка должен быть прописным.

* Не завершайте заголовок вопросительным знаком, `?` .

* Пример [документа](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/multi-turn.docx) можно использовать в качестве примера для создания собственного документа с несколькими переворотами.

### <a name="adding-files-to-a-multi-turn-kb"></a>Добавление файлов в базу знаний с множественным включением

При добавлении иерархического документа QnA Maker определяет дальнейшие подсказки от структуры до создания диалогового потока.

1. В QnA Maker выберите существующую базу знаний, которая была создана с помощью **включения множественного извлечения из URL-адресов, файлов PDF или DOCX.** включено.
1. Перейдите на страницу **Параметры** , выберите файл или URL-адрес для добавления.
1. **Сохраните и обучите** базу знаний.

> [!Caution]
> Поддержка использования экспортированного файла базы знаний TSV или XLS с множественным включением в качестве источника данных для новой или пустой базы знаний не поддерживается. Чтобы добавить экспортированные многострочные запросы в базу знаний, необходимо **импортировать** этот тип файла на странице **Параметры** портала QnA Maker.

## <a name="create-knowledge-base-with-multi-turn-prompts-with-the-create-api"></a>Создание базы знаний с помощью многофункциональных запросов с помощью API создания

Можно создать вариант базы знаний с несколькими инструкциями с помощью [API QnA Maker Create](/rest/api/cognitiveservices/qnamaker/knowledgebase/create). Запросы добавляются в `context` `prompts` массив свойств.

## <a name="show-questions-and-answers-with-context"></a>Отображение вопросов и ответов с контекстом

Сократите отображаемые пары вопросов и ответов только с помощью контекстных диалогов.

Выберите **Параметры представления**, а затем **Показать контекст**. В списке отображаются пары вопросов и ответов, которые содержат подсказки для дальнейших действий.

![Фильтрация пар вопросов и ответов с помощью контекстных диалогов](../media/conversational-context/filter-question-and-answers-by-context.png)

Контекст множественной переворачивания отображается в первом столбце.

:::image type="content" source="../media/conversational-context/surface-manual-pdf-follow-up-prompt.png" alt-text="На снимке экрана показан выделенный раздел контекста." lightbox="../media/conversational-context/surface-manual-pdf-follow-up-prompt.png":::

На предыдущем рисунке **#1** обозначает полужирный текст в столбце, который обозначает текущий вопрос. Родительский вопрос — это верхний элемент в строке. Все приведенные ниже вопросы являются связанными парами вопросов и ответов. Эти элементы доступны для выбора, поэтому можно сразу же переходить к другим элементам контекста.

## <a name="add-an-existing-question-and-answer-pair-as-a-follow-up-prompt"></a>Добавление существующей пары вопросов и ответов в качестве дальнейших запросов

Исходный вопрос, **Моя учетная запись**, содержит дальнейшие подсказки, такие как **учетные записи и вход в** систему.

![«Учетные записи и вход в систему» — ответы и дальнейшие подсказки](../media/conversational-context/detected-and-linked-follow-up-prompts.png)

Добавление дальнейших запросов к существующей паре вопросов и ответов, которая не связана. Так как вопрос не связан с какой-либо парой вопросов и ответов, необходимо изменить параметр текущего представления.

1. Чтобы связать существующую пару "вопрос-ответ" как дальнейшие действия, выберите строку для пары "вопрос-ответ". Для руководства по рабочей области выполните поиск по слову **выход** , чтобы сократить список.
1. В строке для **выхода** в столбце **ответ** выберите команду **Добавить дальнейшие действия**.
1. В полях во всплывающем окне **запроса к исполнению** введите следующие значения:

    |Поле|Значение|
    |--|--|
    |Отображаемый текст|Введите **отключить устройство**. Это пользовательский текст, отображаемый в запросе к исполнению.|
    |Только контекст| Установите этот флажок. Ответ возвращается только в том случае, если вопрос указывает контекст.|
    |Ссылка на ответ|Введите **использовать экран входа** , чтобы найти существующую пару вопросов и ответов.|

1.  Возвращается одно совпадение. Выберите этот ответ в качестве дальнейших действий и нажмите кнопку **сохранить**.

    ![Страница "запрос к исполнению (Предварительная версия)"](../media/conversational-context/search-follow-up-prompt-for-existing-answer.png)

1. После добавления дальнейших действий выберите **сохранить и обучить** в верхней области навигации.

### <a name="edit-the-display-text"></a>Изменение отображаемого текста

При создании запроса на выдачу и вводе существующей пары "вопрос-ответ" в качестве **ссылки на ответ** можно ввести новый **отображаемый текст**. Этот текст не заменяет существующий вопрос и не добавляет новый альтернативный вопрос. Он отделен от этих значений.

1. Чтобы изменить отображаемый текст, найдите и выберите вопрос в поле **контекст** .
1. В строке для этого вопроса выберите в столбце ответ дальнейшие действия.
1. Выберите отображаемый текст, который нужно изменить, и нажмите кнопку **изменить**.

    ![Команда редактирования для отображаемого текста](../media/conversational-context/edit-existing-display-text.png)

1. Во всплывающем окне **запроса к исполнению** измените существующий отображаемый текст.
1. Завершив редактирование отображаемого текста, нажмите кнопку **сохранить**.
1. В верхней панели навигации **Сохраните и обучить**.

## <a name="add-a-new-question-and-answer-pair-as-a-follow-up-prompt"></a>Добавление новой пары "вопрос-ответ" в качестве запроса к исполнению

При добавлении новой пары "вопрос-ответ" в базу знаний каждая пара должна быть связана с существующим вопросом в качестве дальнейших запросов.

1. На панели инструментов базы знаний найдите и выберите существующую пару вопросов и ответов для **учетных записей и входа**.

1. В столбце **ответ** для этого вопроса выберите команду **Добавить дальнейшие действия**.
1. В разделе **"дальнейшие действия" (Предварительная версия)** создайте новую строку дальнейших действий, введя следующие значения:

    |Поле|Значение|
    |--|--|
    |Отображаемый текст|*Создайте учетную запись Windows*. Настраиваемый текст, отображаемый в запросе к исполнению.|
    |Только контекст|Установите этот флажок. Этот ответ возвращается только в том случае, если вопрос указывает на контекст.|
    |Ссылка на ответ|Введите следующий текст в качестве ответа:<br>*[Создайте](https://account.microsoft.com/) учетную запись Windows с новой или существующей учетной записью электронной почты*.<br>При сохранении и обучении базы данных этот текст будет преобразован. |
    |||

    ![Создание нового вопроса и ответа на запрос](../media/conversational-context/create-child-prompt-from-parent.png)

1. Выберите **создать** и нажмите кнопку **сохранить**.

    Это действие создает новую пару вопросов и ответов и связывает выбранный вопрос в качестве дальнейших запросов. В столбце **контекст** для обоих вопросов указывается связь с запросом к исполнению.

1. Выберите **Параметры представления**, а затем [**Показать контекст (Предварительная версия)**](#show-questions-and-answers-with-context).

    Новый вопрос показывает, как он связан.

    ![Создание нового запроса к исполнению](../media/conversational-context/new-qna-follow-up-prompt.png)

    Родительский вопрос отображает новый вопрос в качестве одного из вариантов.

    :::image type="content" source="../media/conversational-context/child-prompt-created.png" alt-text="На снимке экрана показан контекстный столбец для обоих вопросов." lightbox="../media/conversational-context/child-prompt-created.png":::

1. После добавления дальнейших действий выберите **сохранить и обучить** на верхней панели навигации.

<a name="enable-multi-turn-during-testing-of-follow-up-prompts"></a>

## <a name="view-multi-turn-during-testing-of-follow-up-prompts"></a>Просмотр множественного включения во время тестирования запросов к исполнению

При проверке вопроса с помощью запросов к исполнению в области **тестирования** ответ включает в себя запросы к исполнению.

![Ответ включает в себя запросы к исполнению](../media/conversational-context/test-pane-with-question-having-follow-up-prompts.png)

## <a name="a-json-request-to-return-an-initial-answer-and-follow-up-prompts"></a>Запрос JSON для возврата начального ответа и последующих запросов.

Используйте пустой `context` объект, чтобы запросить ответ на вопрос пользователя и включить в него запросы к исполнению.

```JSON
{
  "question": "accounts and signing in",
  "top": 10,
  "userId": "Default",
  "isTest": false,
  "context": {}
}
```

## <a name="a-json-response-to-return-an-initial-answer-and-follow-up-prompts"></a>Ответ JSON для возврата начального ответа и последующих запросов.

В предыдущем разделе был запрошен ответ и все последующие запросы к **учетным записям и входу**. Ответ содержит сведения о приглашении, которые находятся в *ответах [0]. Context*, и текст, отображаемый пользователю.

```JSON
{
    "answers": [
        {
            "questions": [
                "Accounts and signing in"
            ],
            "answer": "**Accounts and signing in**\n\nWhen you set up your Surface, an account is set up for you. You can create additional accounts later for family and friends, so each person using your Surface can set it up just the way he or she likes. For more info, see All about accounts on Surface.com. \n\nThere are several ways to sign in to your Surface Pro 4: ",
            "score": 100.0,
            "id": 15,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": [
                    {
                        "displayOrder": 0,
                        "qnaId": 16,
                        "qna": null,
                        "displayText": "Use the sign-in screen"
                    }
                ]
            }
        },
        {
            "questions": [
                "Sign out"
            ],
            "answer": "**Sign out**\n\nHere's how to sign out: \n\n  Go to Start, and right-click your name. Then select Sign out. ",
            "score": 38.01,
            "id": 18,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": [
                    {
                        "displayOrder": 0,
                        "qnaId": 16,
                        "qna": null,
                        "displayText": "Turn off the device"
                    }
                ]
            }
        },
        {
            "questions": [
                "Use the sign-in screen"
            ],
            "answer": "**Use the sign-in screen**\n\n1.  \n\nTurn on or wake your Surface by pressing the power button. \n\n2.  \n\nSwipe up on the screen or tap a key on the keyboard. \n\n3.  \n\nIf you see your account name and account picture, enter your password and select the right arrow or press Enter on your keyboard. \n\n4.  \n\nIf you see a different account name, select your own account from the list at the left. Then enter your password and select the right arrow or press Enter on your keyboard. ",
            "score": 27.53,
            "id": 16,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": []
            }
        }
    ]
}
```

`prompts`Массив предоставляет текст в `displayText` свойстве и `qnaId` значении. Эти ответы можно отобразить как следующие отображаемые варианты в потоке диалога, а затем отправить выбранное `qnaId` обратно в QnA Maker в следующем запросе.

<!--

The `promptsToDelete` array provides the ...

-->

## <a name="a-json-request-to-return-a-non-initial-answer-and-follow-up-prompts"></a>Запрос JSON для возврата непервоначального ответа и дальнейших запросов

Заполните `context` объект, чтобы включить предыдущий контекст.

В следующем запросе JSON текущим вопросом является *Использование Windows Hello для входа* , а предыдущий вопрос — *учетные записи и вход*.

```JSON
{
  "question": "Use Windows Hello to sign in",
  "top": 10,
  "userId": "Default",
  "isTest": false,
  "qnaId": 17,
  "context": {
    "previousQnAId": 15,
    "previousUserQuery": "accounts and signing in"
  }
}
```

##  <a name="a-json-response-to-return-a-non-initial-answer-and-follow-up-prompts"></a>Ответ JSON для возврата непервоначального ответа и последующих запросов

Ответ QnA Maker _женератеансвер_ JSON содержит запросы к исполнению в `context` свойстве первого элемента `answers` объекта:

```JSON
{
    "answers": [
        {
            "questions": [
                "Use Windows Hello to sign in"
            ],
            "answer": "**Use Windows Hello to sign in**\n\nSince Surface Pro 4 has an infrared (IR) camera, you can set up Windows Hello to sign in just by looking at the screen. \n\nIf you have the Surface Pro 4 Type Cover with Fingerprint ID (sold separately), you can set up your Surface sign you in with a touch. \n\nFor more info, see What is Windows Hello? on Windows.com. ",
            "score": 100.0,
            "id": 17,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": []
            }
        },
        {
            "questions": [
                "Meet Surface Pro 4"
            ],
            "answer": "**Meet Surface Pro 4**\n\nGet acquainted with the features built in to your Surface Pro 4. \n\nHere's a quick overview of Surface Pro 4 features: \n\n\n\n\n\n\n\nPower button \n\n\n\n\n\nPress the power button to turn your Surface Pro 4 on. You can also use the power button to put it to sleep and wake it when you're ready to start working again. \n\n\n\n\n\n\n\nTouchscreen \n\n\n\n\n\nUse the 12.3" display, with its 3:2 aspect ratio and 2736 x 1824 resolution, to watch HD movies, browse the web, and use your favorite apps. \n\nThe new Surface G5 touch processor provides up to twice the touch accuracy of Surface Pro 3 and lets you use your fingers to select items, zoom in, and move things around. For more info, see Surface touchscreen on Surface.com. \n\n\n\n\n\n\n\nSurface Pen \n\n\n\n\n\nEnjoy a natural writing experience with a pen that feels like an actual pen. Use Surface Pen to launch Cortana in Windows or open OneNote and quickly jot down notes or take screenshots. \n\nSee Using Surface Pen (Surface Pro 4 version) on Surface.com for more info. \n\n\n\n\n\n\n\nKickstand \n\n\n\n\n\nFlip out the kickstand and work or play comfortably at your desk, on the couch, or while giving a hands-free presentation. \n\n\n\n\n\n\n\nWi-Fi and Bluetooth&reg; \n\n\n\n\n\nSurface Pro 4 supports standard Wi-Fi protocols (802.11a/b/g/n/ac) and Bluetooth 4.0. Connect to a wireless network and use Bluetooth devices like mice, printers, and headsets. \n\nFor more info, see Add a Bluetooth device and Connect Surface to a wireless network on Surface.com. \n\n\n\n\n\n\n\nCameras \n\n\n\n\n\nSurface Pro 4 has two cameras for taking photos and recording video: an 8-megapixel rear-facing camera with autofocus and a 5-megapixel, high-resolution, front-facing camera. Both cameras record video in 1080p, with a 16:9 aspect ratio. Privacy lights are located on the right side of both cameras. \n\nSurface Pro 4 also has an infrared (IR) face-detection camera so you can sign in to Windows without typing a password. For more info, see Windows Hello on Surface.com. \n\nFor more camera info, see Take photos and videos with Surface and Using autofocus on Surface 3, Surface Pro 4, and Surface Book on Surface.com. \n\n\n\n\n\n\n\nMicrophones \n\n\n\n\n\nSurface Pro 4 has both a front and a back microphone. Use the front microphone for calls and recordings. Its noise-canceling feature is optimized for use with Skype and Cortana. \n\n\n\n\n\n\n\nStereo speakers \n\n\n\n\n\nStereo front speakers provide an immersive music and movie playback experience. To learn more, see Surface sound, volume, and audio accessories on Surface.com. \n\n\n\n\n",
            "score": 21.92,
            "id": 3,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": [
                    {
                        "displayOrder": 0,
                        "qnaId": 4,
                        "qna": null,
                        "displayText": "Ports and connectors"
                    }
                ]
            }
        },
        {
            "questions": [
                "Use the sign-in screen"
            ],
            "answer": "**Use the sign-in screen**\n\n1.  \n\nTurn on or wake your Surface by pressing the power button. \n\n2.  \n\nSwipe up on the screen or tap a key on the keyboard. \n\n3.  \n\nIf you see your account name and account picture, enter your password and select the right arrow or press Enter on your keyboard. \n\n4.  \n\nIf you see a different account name, select your own account from the list at the left. Then enter your password and select the right arrow or press Enter on your keyboard. ",
            "score": 19.04,
            "id": 16,
            "source": "product-manual.pdf",
            "metadata": [],
            "context": {
                "isContextOnly": true,
                "prompts": []
            }
        }
    ]
}
```

## <a name="query-the-knowledge-base-with-the-qna-maker-id"></a>Запрос к базе знаний с помощью идентификатора QnA Maker

Если вы создаете пользовательское приложение, используя функцию множественного включения. В ответе на первый вопрос возвращаются все дальнейшие подсказки и связанные с ней `qnaId` результаты. Теперь, когда у вас есть идентификатор, вы можете передать его в текст запроса дальнейших действий. Если текст запроса содержит `qnaId` , и объект контекста (который содержит предыдущие QnA Maker свойства), то женератеансвер будет возвращать точный вопрос по идентификатору, а не использовать алгоритм ранжирования для поиска ответа по тексту вопроса.

## <a name="display-order-is-supported-in-the-update-api"></a>Порядок просмотра поддерживается в API обновления

[Отображаемый текст и порядок показа](/rest/api/cognitiveservices/qnamaker/knowledgebase/update#promptdto), возвращаемые в ответе JSON, поддерживаются для редактирования [API обновления](/rest/api/cognitiveservices/qnamaker/knowledgebase/update).

## <a name="add-or-delete-multi-turn-prompts-with-the-update-api"></a>Добавление и удаление многофункциональных запросов с помощью API обновления

Вы можете добавлять или удалять многострочные запросы с помощью [API обновления QnA Maker](/rest/api/cognitiveservices/qnamaker/knowledgebase/update).  Запросы добавляются в `context` `promptsToAdd` массив свойств и `promptsToDelete` массив.

## <a name="export-knowledge-base-for-version-control"></a>Экспорт базы знаний для системы управления версиями

QnA Maker поддерживает управление версиями, включая шаги множественного включения диалога в экспортированный файл.

## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения об контекстных диалогах см. в этом [образце диалогового окна](https://github.com/microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/adaptive-dialog/07.qnamaker/QnAMaker.csproj) или дополнительные сведения о [концептуальном проектировании для нескольких диалогов](/azure/bot-service/bot-builder-conversations).

> [!div class="nextstepaction"]
> [Перенос базы знаний](../Tutorials/migrate-knowledge-base.md)