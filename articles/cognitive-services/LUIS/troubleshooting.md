---
title: Часто задаваемые вопросы — LUIS
description: В этой статье приведены ответы на часто задаваемые вопросы о службе "Распознавание речи" (LUIS).
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: troubleshooting
ms.date: 05/06/2020
ms.openlocfilehash: b5e25e9ed25ced96d38994928bcb6275ce79420f
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102612802"
---
# <a name="language-understanding-frequently-asked-questions-faq"></a>Часто задаваемые вопросы о службе "Распознавание речи"

В этой статье приведены ответы на часто задаваемые вопросы о службе "Распознавание речи" (LUIS).

## <a name="whats-new"></a>Новое

[Узнайте больше](whats-new.md) о новых возможностях распознавание речи (Luis).

<a name="luis-authoring"></a>

## <a name="authoring"></a>Разработка

### <a name="what-are-the-luis-best-practices"></a>Какие лучшие методики работы с LUIS?
Начните с [цикла разработки](luis-concept-app-iteration.md), а затем ознакомьтесь с [лучшими методиками](luis-concept-best-practices.md).

### <a name="what-is-the-best-way-to-start-building-my-app-in-luis"></a>Каким способом лучше начать создание приложения в LUIS?

Создавать приложение рекомендуется [поэтапно](luis-concept-app-iteration.md).

### <a name="what-is-a-good-practice-to-model-the-intents-of-my-app-should-i-create-more-specific-or-more-generic-intents"></a>Каким образом лучше моделировать намерения приложения? Следует создавать более конкретные или более общие намерения?

Создавайте не слишком общие, чтобы не допускать перекрывания, и не слишком конкретные намерения, чтобы служба LUIS могла легко различать их, если они похожи. Создание дискриминационных конкретных целей является одним из лучших способов моделирования LUIS.

### <a name="is-it-important-to-train-the-none-intent"></a>Важно ли обучать намерение None?

Да, следует обучить намерение **None** с помощью дополнительных высказываний, добавляя дополнительные метки для других намерений. Хорошее соотношение — 1 или 2 метки, добавленные к **None**, на каждые 10 меток, добавленных к намерению. Это отношение повышает дискриминационный уровень LUIS.

### <a name="how-can-i-correct-spelling-mistakes-in-utterances"></a>Каким образом можно исправить орфографические ошибки в высказываниях?

Ознакомьтесь с руководством [Исправление орфографических ошибок с помощью API Bing для проверки орфографии](luis-tutorial-bing-spellcheck.md). Служба LUIS применяет ограничения, накладываемые API Bing для проверки орфографии версии 7.

### <a name="how-do-i-edit-my-luis-app-programmatically"></a>Как изменить приложение LUIS программным способом?
Чтобы изменить приложение LUIS программным способом, используйте [API разработки](https://go.microsoft.com/fwlink/?linkid=2092087). Примеры вызова API разработки см. в статьях [Руководство по добавлению фраз в приложение с использованием Node.js](./get-started-get-model-rest-apis.md) и [Создание приложения LUIS программным способом с помощью Node.js](./luis-tutorial-node-import-utterances-csv.md). Для применения API разработки необходимо использовать [ключ разработки](luis-how-to-azure-subscription.md#azure-resources-for-luis), а не ключ конечной точки. Программный способ разработки позволяет осуществлять до 1 000 000 звонков в месяц и пять транзакций в секунду. Дополнительные сведения об используемых с LUIS ключах см. в статье [Keys in LUIS](./luis-how-to-azure-subscription.md) (Ключи в LUIS).

### <a name="where-is-the-pattern-feature-that-provided-regular-expression-matching"></a>Где находится компонент "Шаблон", который сопоставляется с предоставленным регулярным выражением?
Сейчас не рекомендуется использовать предыдущий компонент **Шаблон**, замененный компонентом **[Шаблоны](luis-concept-patterns.md)**.

### <a name="how-do-i-use-an-entity-to-pull-out-the-correct-data"></a>Как использовать сущность для отделения правильных данных?
Ознакомьтесь со сведениями о [сущностях](luis-concept-entity-types.md) и [извлечении данных](luis-concept-data-extraction.md).

### <a name="should-variations-of-an-example-utterance-include-punctuation"></a>Должны ли вариации примерного высказывания включать пунктуацию?
Используйте одно из следующих решений:
* Игнорировать [знаки препинания](luis-reference-application-settings.md#punctuation-normalization)
* Добавьте различные варианты, например фразы продолжительностью, в цель.
* Добавьте шаблон примера utterance с [синтаксисом, чтобы игнорировать](luis-concept-patterns.md#pattern-syntax) знаки препинания.

### <a name="does-luis-currently-support-cortana"></a>Осуществляется ли поддержка Cortana в LUIS в данный момент?

Предварительно созданное приложение Cortana было внесено в список нерекомендуемых в 2017 г. Оно больше не поддерживается.

### <a name="how-do-i-transfer-ownership-of-a-luis-app"></a>Как передавать права владельца приложения LUIS?
Чтобы переместить приложение LUIS в другую подписку Azure, экспортируйте приложение LUIS и импортируйте его, используя новую учетную запись. Обновите идентификатор приложения LUIS в клиентском приложении, которое его вызывает. Возвращаемые новым приложением оценки LUIS могут немного отличаться от оценок исходного приложения.

### <a name="a-prebuilt-entity-is-tagged-in-an-example-utterance-instead-of-my-custom-entity-how-do-i-fix-this"></a>Предварительно созданная сущность помечается в примере utterance вместо моей пользовательской сущности. Как устранить эту проблему?

На портале LUIS можно пометить текст для точной сущности, которую нужно извлечь. Если на портале LUIS не отображается правильный прогноз сущностей, может потребоваться добавить дополнительные фразы продолжительностью и пометить сущность в тексте или добавить функцию.

### <a name="i-tried-to-import-an-app-or-version-file-but-i-got-an-error-what-happened"></a>При попытке импортировать файл приложения или версии возникла ошибка, что случилось?

Дополнительные сведения об [ошибках импорта версий](luis-how-to-manage-versions.md#import-errors).

<a name="luis-collaborating"></a>

## <a name="collaborating-and-contributing"></a>Совместная работа и участие

### <a name="how-do-i-give-collaborators-access-to-luis-with-azure-active-directory-azure-ad-or-azure-role-based-access-control-azure-rbac"></a>Разделы справки предоставить совместный доступ к LUIS с помощью Azure Active Directory (Azure AD) или управления доступом на основе ролей Azure (Azure RBAC)?

Дополнительные сведения о том, как предоставить доступ для участников совместной работы, см. в разделах [Ресурсы Azure Active Directory](luis-how-to-collaborate.md#azure-active-directory-resources) и [Пользователь клиента Azure Active Directory](luis-how-to-collaborate.md#azure-active-directory-tenant-user).

<a name="luis-endpoint"></a>

## <a name="endpoint"></a>Конечная точка

### <a name="i-received-an-http-403-error-status-code-how-do-i-fix-it"></a>Я получил код состояния ошибки HTTP 403. Как ее исправить?

Вы получаете коды состояния ошибок 403 и 429 при превышении числа транзакций в секунду или транзакций в месяц для ценовой категории. Увеличьте ценовую категорию или используйте [контейнеры](luis-container-howto.md)распознавание речи.

При использовании всех этих бесплатных запросов к конечной точке 1000 или превышении квоты месячных транзакций для ценовой категории вы получаете код состояния ошибки HTTP 403.

Чтобы устранить эту ошибку, необходимо либо [изменить ценовую](luis-how-to-azure-subscription.md#change-the-pricing-tier) категорию на более высокий уровень, либо [создать новый ресурс](get-started-portal-deploy-app.md#create-the-endpoint-resource) и [назначить его приложению](get-started-portal-deploy-app.md#assign-the-resource-key-to-the-luis-app-in-the-luis-portal).

Ниже приведены решения для этой ошибки.

* В [портал Azure](https://portal.azure.com)на ресурсе распознавание речи в **ценовой категории управление ресурсами — >** измените ценовую категорию на более высокий уровень технической спецификации. Вам не нужно ничего делать на портале Распознавание речи, если ресурс уже назначен вашему Распознавание речи приложению.
*  Если вы превысили максимальную ценовую категорию, добавьте дополнительные Распознавание речи ресурсы с подсистемой балансировки нагрузки перед ними. В этом может помочь [контейнер распознавание речи](luis-container-howto.md) с Kubernetes или DOCKER Compose.

### <a name="i-received-an-http-429-error-status-code-how-do-i-fix-it"></a>Я получил код состояния ошибки HTTP 429. Как ее исправить?

Вы получаете коды состояния ошибок 403 и 429 при превышении числа транзакций в секунду или транзакций в месяц для ценовой категории. Увеличьте ценовую категорию или используйте [контейнеры](luis-container-howto.md)распознавание речи.

Этот код состояния возвращается, если количество транзакций в секунду превышает вашу ценовую категорию.

Решения включают:

* Вы можете [увеличить ценовую категорию](luis-how-to-azure-subscription.md#change-the-pricing-tier), если вы не на самом высшем уровне.
* Если вы превысили максимальную ценовую категорию, добавьте дополнительные Распознавание речи ресурсы с подсистемой балансировки нагрузки перед ними. В этом может помочь [контейнер распознавание речи](luis-container-howto.md) с Kubernetes или DOCKER Compose.
* Вы можете настроить запрос клиентского приложения с помощью [политики повтора](/azure/architecture/best-practices/transient-faults#general-guidelines) , которая реализуется при получении этого кода состояния.

### <a name="my-endpoint-query-returned-unexpected-results-what-should-i-do"></a>Запрос конечной точки вернул непредвиденные результаты.   Что следует делать?

Непредвиденные результаты прогнозирования, полученные при запросе, основаны на состоянии опубликованной модели. Чтобы скорректировать модель, потребуется изменить ее, обучить и опубликовать повторно.

Начните корректировку модели с [активного обучения](luis-how-to-review-endpoint-utterances.md).

Можно удалить результаты недетерминированного обучения. Для этого настройте [API параметров версии приложения](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) для использования всех данных обучения.

Другие советы см. в [статье с рекомендациями](luis-concept-best-practices.md).

### <a name="why-does-luis-add-spaces-to-the-query-around-or-in-the-middle-of-words"></a>Почему LUIS добавляет пробелы вокруг или между словами запроса?
LUIS [помечает](luis-glossary.md#token) высказывание на основе [языка и региональных параметров](luis-language-support.md#tokenization). Исходное и помеченное значения можно использовать для [извлечения данных](luis-concept-data-extraction.md#tokenized-entity-returned).

### <a name="how-do-i-create-and-assign-a-luis-endpoint-key"></a>Как создать и назначить ключ конечной точки LUIS?
[Создайте ключ конечной точки](luis-how-to-azure-subscription.md) в Azure для уровня вашей [службы](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/). [Назначьте ключ](luis-how-to-azure-subscription.md) на странице **[ресурсов Azure](luis-how-to-azure-subscription.md)** . Это действие можно выполнить с помощью любого API. Затем необходимо изменить HTTP-запрос к конечной точке, чтобы [использовать новый ключ конечной точки](luis-how-to-azure-subscription.md).

### <a name="how-do-i-interpret-luis-scores"></a>Как интерпретировать оценки LUIS?
В системе должно использоваться намерение с самой высокой оценкой, независимо от его значения. Например, оценка ниже 0,5 (меньше 50%) не означает, что данные LUIS имеют низкую степень достоверности. Предоставление дополнительных обучающих данных поможет увеличить [показатель](luis-concept-prediction-score.md) наиболее вероятной цели.

### <a name="why-dont-i-see-my-endpoint-hits-in-my-apps-dashboard"></a>Почему для меня недоступны обращения конечной точки на панели мониторинга приложения?
Общее количество обращений конечной точки на панели мониторинга приложения периодически обновляется, но более часто обновляются показатели, связанные с ключом конечной точки LUIS на портале Azure.

Если вам недоступны обновленные данные обращения конечной точки на панели мониторинга, войдите на портал Azure, найдите ресурс, связанный с ключом конечной точки LUIS, и откройте **Метрики**, чтобы установить метрику **Всего вызовов**. При использовании ключа конечной точки для более чем одного приложения LUIS метрики на портале Azure отображается совокупное количество вызовов от всех приложений LUIS, использующих этот ключ.

### <a name="is-there-a-powershell-command-get-to-the-endpoint-quota"></a>Существует ли команда получения PowerShell для запроса сведений о квоте конечной точки?

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Чтобы просмотреть данные о квоте конечной точки, используйте следующую команду PowerShell:

```powershell
Get-AzCognitiveServicesAccountUsage -ResourceGroupName <your-resource-group> -Name <your-resource-name>
```

### <a name="my-luis-app-was-working-yesterday-but-today-im-getting-403-errors-i-didnt-change-the-app-how-do-i-fix-it"></a>Вчера приложение LUIS работало, но сейчас отображаются ошибки 403. Я не изменял приложение. Как ее исправить?
Выполните эти [инструкции](#how-do-i-create-and-assign-a-luis-endpoint-key) , чтобы создать ключ КОНЕЧНОЙ точки Luis и назначить его приложению. Затем необходимо изменить HTTP-запрос клиентского приложения на конечную точку, чтобы [использовать новый ключ конечной точки](luis-how-to-azure-subscription.md). Если вы создали новый ресурс в другом регионе, измените регион запроса клиента HTTP.

### <a name="how-do-i-secure-my-luis-endpoint"></a>Как включить защиту конечной точки LUIS?
Сведения об этом можно просмотреть в разделе [Защита конечной точки](luis-how-to-azure-subscription.md#securing-the-endpoint).

## <a name="working-within-luis-limits"></a>Работа в пределах ограничений LUIS

### <a name="what-is-the-maximum-number-of-intents-and-entities-that-a-luis-app-can-support"></a>Какое максимальное число намерений и сущностей, поддерживаемых приложением LUIS?
Подробные сведения см. в разделе по [этой](luis-limits.md) ссылке.

### <a name="i-want-to-build-a-luis-app-with-more-than-the-maximum-number-of-intents-what-should-i-do"></a>Я хочу создать приложение LUIS с количеством намерений, которое превышает допустимое количество.   Что следует делать?

Подробнее см. в [этом](luis-concept-intent.md#if-you-need-more-than-the-maximum-number-of-intents) разделе с рекомендациями.

### <a name="i-want-to-build-an-app-in-luis-with-more-than-the-maximum-number-of-entities-what-should-i-do"></a>Я хочу создать в LUIS приложение с количеством сущностей, которое превышает допустимое количество.   Что следует делать?

Подробнее см. в [этом](luis-concept-entity-types.md#if-you-need-more-than-the-maximum-number-of-entities) разделе с рекомендациями.

### <a name="what-are-the-limits-on-the-number-and-size-of-phrase-lists"></a>Какие ограничения на количество и размер списков фраз?
Максимально полный [список фраз](./luis-concept-feature.md) см. в разделе по[этой](luis-limits.md) ссылке.

### <a name="what-are-the-limits-on-example-utterances"></a>Каковы ограничения на примеры высказываний?
Подробные сведения см. в разделе по [этой](luis-limits.md) ссылке.

## <a name="testing-and-training"></a>Обучение и тестирование

### <a name="i-see-some-errors-in-the-batch-testing-pane-for-some-of-the-models-in-my-app-how-can-i-address-this-problem"></a>На панели пакетного тестирования для некоторых из моделей приложений отображаются ошибки. Как решить эту проблему?

Ошибки свидетельствуют о наличии некоторых расхождений между метками и прогнозированием моделей. Чтобы устранить эту проблему, выполните одну или обе следующие задачи:
* Чтобы LUIS лучше дискриминировал намерения, добавьте дополнительные метки.
* Чтобы LUIS обучался быстрее, добавьте компоненты списка фраз, которые вводят словарь на основе доменов.

Дополнительные сведения см. в руководстве по [пакетному тестированию](./luis-how-to-batch-test.md).

### <a name="when-an-app-is-exported-then-reimported-into-a-new-app-with-a-new-app-id-the-luis-prediction-scores-are-different-why-does-this-happen"></a>Если приложение экспортировано, затем повторно импортировано в новое приложение (с новым идентификатором приложения), прогнозирующие оценки LUIS отличаются. Почему это происходит?

Дополнительные сведения см. в разделе, посвященном [прогнозированию различий между копиями одного приложения](luis-concept-prediction-score.md#review-intents-with-similar-scores).

### <a name="some-utterances-go-to-the-wrong-intent-after-i-made-changes-to-my-app-the-issue-seems-to-disappear-at-random-how-do-i-fix-it"></a>После внесения изменений в приложение некоторые высказывания отправляются в неправильное намерение. Проблема периодически исчезает. Как ее исправить?

См. раздел [Обучение на основе всех данных](luis-how-to-train.md#train-with-all-data).

## <a name="app-publishing"></a>Публикация приложений

### <a name="what-is-the-tenant-id-in-the-add-a-key-to-your-app-window"></a>Какой идентификатор клиента в окне Add a key to your app (Добавить ключ для приложения)?
В Azure клиентом считается клиент или организация, связанные со службой. На портале Azure найдите идентификатор клиента в поле **Идентификатор каталога**, выбрав **Azure Active Directory** > **Управление** > **Свойства**.

![Идентификатор клиента на портале Azure](./media/luis-manage-keys/luis-assign-key-tenant-id.png)

<a name="why-are-there-more-subscription-keys-on-my-apps-publish-page-than-i-assigned-to-the-app"></a>
<a name="why-are-there-more-endpoint-keys-on-my-apps-publish-page-than-i-assigned-to-the-app"></a>


### <a name="why-are-there-more-endpoint-keys-assigned-to-my-app-than-i-assigned"></a>Почему приложению назначено больше ключей конечной точки, чем назначено мной?
Из соображений удобства для каждого приложения LUIS в списке конечных точек указывается ключ разработки (начальный ключ). Этот ключ позволяет выполнить только несколько обращений к конечной точке для пробного использования LUIS.

Если приложение было создано до выхода общедоступной версии LUIS, ключи конечных точек LUIS в подписке назначаются автоматически. Это упрощает процесс миграции GA. Все новые ключи конечных точек LUIS на портале Azure _не_ назначаются в LUIS автоматически.

## <a name="key-management"></a>Управление ключами

### <a name="how-do-i-know-what-key-i-need-where-i-get-it-and-what-i-do-with-it"></a>Как я узнаю, какой ключ мне нужен, где его получить и что мне с ним делать?

Сведения о различиях между ключом разработки и ключом среды выполнения прогнозирования см. [в разделе Создание и прогнозирование запросов в ключах конечных точек в Luis](luis-how-to-azure-subscription.md) .

### <a name="i-got-an-error-about-being-out-of-quota-how-do-i-fix-it"></a>Я получил ошибку с сообщением об отсутствии квоты. Как ее исправить?

Дополнительные сведения см. в разделе Исправление кода состояния HTTP [403](#i-received-an-http-403-error-status-code-how-do-i-fix-it) и [429](#i-received-an-http-429-error-status-code-how-do-i-fix-it) .

### <a name="i-need-to-handle-more-endpoint-queries-how-do-i-do-that"></a>Мне нужно обрабатывать больше запросов конечной точки. Как это сделать?

Дополнительные сведения см. в разделе Исправление кода состояния HTTP [403](#i-received-an-http-403-error-status-code-how-do-i-fix-it) и [429](#i-received-an-http-429-error-status-code-how-do-i-fix-it) .

### <a name="i-created-an-authoring-key-but-it-isnt-showing-in-the-luis-portal-what-happened"></a>Я создал ключ разработки, но он не отображается на портале LUIS. Что произошло?

Ключи разработки доступны на портале LUIS после [перехода на процесс создания ключа разработки](luis-migration-authoring.md).

## <a name="app-management"></a>Управление приложениями

### <a name="how-do-i-download-a-log-of-user-utterances"></a>Как скачать журнал выражений пользователя?
По умолчанию приложение LUIS регистрирует выражения пользователей. Чтобы загрузить журнал высказываний, которые пользователи отправляют в приложение LUIS, перейдите к **Мои приложения** и выберите приложение. На контекстной панели инструментов выберите **Export Endpoint Logs** (Экспортировать журналы конечной точки). Журнал форматируется в файл данных с разделителями-запятыми (CSV-файл).

### <a name="how-can-i-disable-the-logging-of-utterances"></a>Как отключить ведение журнала выражений?
Ведение журнала выражений пользователя можно отключить, задав `log=false` в URL-адресе конечной точки, который клиентское приложение использует для отправки запросов в LUIS. Однако отключение ведения журнала не позволяет приложению LUIS предлагать выражения или повышать производительность на основе [активного обучения](luis-concept-review-endpoint-utterances.md#what-is-active-learning). Если задать `log=false` по соображениям конфиденциальности данных, не удается скачать запись этих выражений пользователя из LUIS или использовать эти выражения для улучшения приложения.

Выражения хранятся только в журналах.

### <a name="why-dont-i-want-all-my-endpoint-utterances-logged"></a>Почему не нужно регистрировать все выражения конечной точки?
При использовании журналов для прогнозирующего анализа не сохраняйте в них собранные данные тестирования выражений.

## <a name="data-management"></a>Управление данными

### <a name="can-i-delete-data-from-luis"></a>Можно ли удалить данные из LUIS?

* Вы можете удалить высказывания, использовавшиеся для обучения LUIS, в любой момент. При удалении примера высказывания из приложения LUIS высказывание удаляется из веб-службы LUIS и недоступно для экспорта.
* Вы можете удалить высказывания из списка высказываний пользователя, предлагаемых LUIS на странице **Review endpoint utterances** (Проверка высказываний конечной точки). Высказывания, удаленные из этого списка, больше не будут выводиться как предлагаемые, но останутся в журналах.
* При удалении учетной записи удаляются все приложения вместе с их примерами высказываний и журналами. Данные хранятся на серверах 60 дней, а затем удаляются без возможности восстановления.

### <a name="how-does-microsoft-manage-data-i-send-to-luis"></a>Как корпорация Майкрософт управляет данными, отправляемыми мною в LUIS?

В [центре управления безопасностью](https://www.microsoft.com/trustcenter) описаны наши обязательства, а также и доступные вам варианты управления данными и доступа в службы Azure.

## <a name="language-and-translation-support"></a>Поддержка языка и преобразования

### <a name="i-have-an-app-in-one-language-and-want-to-create-a-parallel-app-in-another-language-what-is-the-easiest-way-to-do-so"></a>У меня есть приложение на одном языке и я хочу создать параллельное приложение на другом языке. Какой самый простой способ сделать это?
1. Экспортируйте приложение.
2. Преобразуйте помеченные выражения в JSON-файл экспортированного приложения в целевой язык.
3. Может понадобиться изменить имена намерений и сущностей, или оставить их как есть.
4. Наконец, выполните импорт приложения, чтобы создать приложение LUIS на целевом языке.

## <a name="app-notification"></a>Уведомление приложения

### <a name="why-did-i-get-an-email-saying-im-almost-out-of-quota"></a>Почему пришло сообщение электронной почты о том, что я почти превысил квоту?
Ваш ключ разработки или начальный ключ может запрашивать только 1000 конечных точек в месяц. Создайте ключ конечной точки LUIS (бесплатный или платный) и используйте его при создании запросов конечных точек. При создании запроса конечной точки из бота или другого клиентского приложения необходимо изменить ключ конечной точки LUIS.

## <a name="bots"></a>Боты

### <a name="my-luis-bot-isnt-working-what-do-i-do"></a>Бот LUIS не работает. Что делать?

Первая трудность состоит в изоляции ошибки, которая связана с LUIS или случилась за пределами ПО промежуточного слоя LUIS.

#### <a name="resolve-issue-in-luis"></a>Устранение проблемы в LUIS
Выполните передачу требуемого речевого фрагмента в LUIS из [конечной точки LUIS ](luis-get-started-create-app.md#query-the-v2-api-prediction-endpoint). Если произошла ошибка, устраните ее в LUIS таким образом, чтобы предотвратить ее повторное появление. Ниже перечислены распространенные ошибки.

* `Out of call volume quota. Quota will be replenished in <time>.`. Данная ошибка указывает о необходимости изменения ключа разработки на [ключ конечной точки](luis-how-to-azure-subscription.md) или изменения [уровней служб](luis-how-to-azure-subscription.md#change-the-pricing-tier).

#### <a name="resolve-issue-in-azure-bot-service"></a>Устранение ошибок в службе Azure Bot

Когда при использовании службы Azure Bot возникает ошибка в которой **тестирование в веб-чате** возвращает сообщение `Sorry, my bot code is having an issue`, проверьте свои журналы.

1. Для своего бота, на портале Azure, в разделе **Bot management** (Управление ботом), выберите **Сборка**.
1. Откройте сетевой редактор кода.
1. Выберите имя бота в верхней части синей панели навигации (второй элемент справа).
1. В раскрывающемся итоговом списке выберите **Open Kudu Console** (Открыть консоль Kudu).
1. Последовательно выберите **LogFiles**, а затем **Приложение**. Просмотр всех файлов журналов. Если в папке приложения не появляется ошибка, просмотрите файлы журнала в **LogFiles**.
1. Обязательно перестраивайте проект при использовании компилируемого языка, например C#.

> [!Tip]
> Установку пакетов также можно выполнять с помощью консоли.

#### <a name="resolve-issue-while-debugging-on-local-machine-with-bot-framework"></a>Во время отладки на локальной машине устраняйте проблему с помощью Bot Framework.

Дополнительные сведения о локальной отладке бота см. в статье [Отладка бота](/azure/bot-service/bot-service-debug-bot).

## <a name="integrating-luis"></a>Интеграция LUIS

### <a name="where-is-my-luis-app-created-during-the-azure-web-app-bot-subscription-process"></a>Где находится приложение LUIS, созданное во время подписки на бота веб-приложения Azure?
Если выбрать шаблон LUIS и нажать кнопку **Выбрать** в области шаблонов, на панели слева отображается тип шаблона и вопрос о расположении для создания шаблона LUIS. Однако процесс бота веб-приложения не приводит к созданию подписки LUIS.

![Регион шаблона LUIS бота веб-приложения](./media/luis-faq/web-app-bot-location.png)

### <a name="what-luis-regions-support-bot-framework-speech-priming"></a>Какие регионы LUIS поддерживают подготовку речи Bot Framework?
[Подготовка речи](/bot-framework/bot-service-manage-speech-priming) поддерживается только для приложений LUIS в центральном регионе экземпляра (США).

## <a name="api-programming-strategies"></a>Стратегии программирования API

### <a name="how-do-i-programmatically-get-the-luis-region-of-a-resource"></a>Как с помощью программных средств узнать регион для ресурса LUIS?

Используйте пример LUIS, чтобы [узнать регион](https://github.com/Azure-Samples/cognitive-services-language-understanding/tree/master/documentation-samples/find-region) с помощью программных средств для C# или Node.Js.

## <a name="luis-service"></a>Служба LUIS

### <a name="is-language-understanding-luis-available-on-premises-or-in-private-cloud"></a>Доступна ли служба "Распознавание речи" (LUIS) в локальной среде или в частном облаке?

Да. Вы можете использовать [контейнер](luis-container-howto.md) LUIS в таких сценариях, если у вас есть подключение для измерения использования.

## <a name="migrating-to-the-next-version"></a>Переход на следующую версию

### <a name="how-do-i-migrate-to-preview-v3-api"></a>Разделы справки перейти на предварительную версию API V3?

См. [инструкции по миграции для приложений Luis в API версии 2 и 3](luis-migration-api-v3.md)

## <a name="build-2019-conference-announcements"></a>Объявления о конференции для сборки 2019

На конференции Build 2019 г были представлены следующие возможности:

* [Предварительная версия инструкции по миграции API версии 3](luis-migration-api-v3.md)
* [Улучшенная панель мониторинга аналитики](luis-how-to-use-dashboard.md)
* [Улучшенные предварительно созданные домены](luis-reference-prebuilt-domains.md)
* [Сущности динамического списка](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time)
* [Внешние сущности](luis-migration-api-v3.md#external-entities-passed-in-at-prediction-time)

Видеоролики:

* [Как использовать службу Azure Conversational AI для масштабирования вашего бизнеса для следующего поколения](https://www.youtube.com/watch?v=_k97jd-csuk&feature=youtu.be)

## <a name="next-steps"></a>Следующие шаги

Дополнительную информацию о LUIS см. в следующих ресурсах:
* [Вопросы Stack Overflow, помеченные LUIS](https://stackoverflow.com/questions/tagged/luis)
* [Страница вопросов и ответов по Microsoft Q&для MSDN Распознавание речи Intelligent Services (LUIS)](/answers/topics/azure-language-understanding.html)