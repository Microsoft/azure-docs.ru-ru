---
title: Конфигурация службы — QnA Maker
description: Узнайте, как и где следует настраивать ресурсы.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 11/9/2020
ms.openlocfilehash: da46084c8c2616284c31ef155927e8dbcbd19e0e
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102209365"
---
# <a name="service-configuration"></a>Конфигурация службы

Каждая версия QnA Maker использует другой набор ресурсов Azure (служб). В этой статье описаны поддерживаемые настройки для этих служб. 

## <a name="app-service"></a>Служба приложений

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/v1)

QnA Maker использует службу приложений для предоставления среды выполнения запросов, используемой [API женератеансвер](/rest/api/cognitiveservices/qnamaker4.0/runtime/generateanswer).

Эти параметры доступны в портал Azure для службы приложений. Параметры доступны, выбрав **Параметры**, а затем — **Конфигурация**.

Можно задать отдельный параметр в списке Параметры приложения или изменить несколько параметров, выбрав пункт **дополнительно изменить**.

|Ресурс|Параметр|
|--|--|
|азуресеарчадминкэй|Когнитивный поиск — используется для хранения пар QnA и ранжирования #1|
|азуресеарчнаме|Когнитивный поиск — используется для хранения пар QnA и ранжирования #1|
|дефаултансвер|Текст ответа, если совпадений не найдено|
|усераппинсигхтсаппид|Журнал разговора и данные телеметрии|
|усераппинсигхтскэй|Журнал разговора и данные телеметрии|
|усераппинсигхтснаме|Журнал разговора и данные телеметрии|
|QNAMAKER_EXTENSION_VERSION|Всегда имеет значение " _Последняя_". Этот параметр позволяет инициализировать расширение сайта QnAMaker в службе приложений.|

После завершения внесения изменений необходимо **перезапустить** службу на странице **обзора** портал Azure.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/v2)

Настройки службы приложений не применяются к QnA Maker Managed (Предварительная версия).

---

## <a name="qna-maker-service"></a>Служба QnA Maker

Служба QnA Maker предоставляет конфигурацию для следующих пользователей для совместной работы с одной QnA Makerной службой и всеми ее базами знаний.

Узнайте [, как добавить в службу участников совместной работы](./reference-role-based-access-control.md) .

## <a name="change-azure-cognitive-search"></a>Изменение Когнитивный поиск Azure

Узнайте [, как изменить службу когнитивный Поиск,](./how-to/configure-QnA-Maker-resources.md#configure-qna-maker-to-use-different-cognitive-search-resource) связанную со службой QnA Maker.

## <a name="change-default-answer"></a>Изменение ответа по умолчанию

Узнайте [, как изменить текст ответов по умолчанию](How-To/change-default-answer.md). 

## <a name="telemetry"></a>Телеметрия

# <a name="qna-maker-ga-stable-release"></a>[Общедоступная версия QnA Maker (стабильный выпуск)](#tab/v1)

Application Insights используется для мониторинга телеметрии с QnA Maker. Нет параметров конфигурации, относящихся к QnA Maker.

# <a name="qna-maker-managed-preview-release"></a>[Управляемая служба QnA Maker (предварительный выпуск)](#tab/v2)

Узнайте [, как добавить телеметрию в службу QnA Maker Managed (Предварительная версия)](How-To/get-analytics-knowledge-base.md). 

---

## <a name="app-service-plan"></a>План службы приложений

# <a name="qnamaker-ga-stable-release"></a>[QnAMaker (стабильный выпуск)](#tab/v1)

План службы приложений не имеет параметров конфигурации, относящихся к QnA Maker.

# <a name="qnamaker-managed-preview-release"></a>[Управляемый QnAMaker (предварительный выпуск)](#tab/v2)

План службы приложений не используется с управляемым QnA Maker (Предварительная версия).

---

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [форматах](reference-document-format-guidelines.md) документов и URL-адресов, которые необходимо импортировать в базу знаний.
