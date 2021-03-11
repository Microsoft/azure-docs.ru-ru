---
title: API запросов к отчетам
description: Используйте этот API для выполнения запроса отчета для отчетов по анализу коммерческих рынков.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 0db212be06182128bbd8a3bf694a2f893ce82eae
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102583931"
---
# <a name="try-report-queries-api"></a>API запросов к отчетам

Этот API выполняет инструкцию запроса отчета. API возвращает только 10 записей, которые вы как партнер может использовать для проверки ожидаемых данных.

> [!IMPORTANT]
> Время ожидания выполнения запроса для этого API составляет 100 секунд. Если вы заметили, что API занимает более 100 секунд, очень вероятно, что запрос синтаксически верный, или же вы получили код ошибки, отличный от 200. Фактическое создание отчета пройдет, если синтаксис запроса правильный.

**Синтаксис запроса**

| **Метод** | **Универсальный код ресурса (URI) запроса** |
| --- | --- |
| GET | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledQueries/testQueryResult?exportQuery={query text}` |
|||

**Заголовок запроса**

| **Header** | **Тип** | **Описание** |
| --- | --- | --- |
| Авторизация | строка | Обязательный. Маркер доступа Azure Active Directory (Azure AD) в форме `Bearer <token>` |
| Content-Type | строка | `Application/JSON` |
|||

**QueryParameter**

| **Имя параметра** | **Тип** | **Описание** |
| --- | --- | --- |
| `exportQuery` | string | Строка запроса отчета, которую необходимо выполнить |
| `queryId` | строка | Допустимый идентификатор существующего запроса |
|||

**Параметр** пути  

Нет

**Полезные данные запроса**

Нет

**Словарь терминов**

Нет

**Ответ**

Полезные данные ответа структурированы следующим образом:

Код ответа: 200, 400, 401, 403, 404, 500

Полезные данные ответа: 10 основных строк выполнения запроса
