---
title: Поддержка и параметры справки для разработчиков приложений Azure AD
description: Узнайте, как получить справку и поддержку по вопросам и проблемам разработки, возникающим при создании приложения, которое интегрируется с удостоверениями Майкрософт (Azure Active Directory и учетной записью Майкрософт)
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/23/2019
ms.author: ryanwi
ms.reviewer: jmprieur, saeeda
ms.custom: aaddev
ms.openlocfilehash: ecbb7ec503011d78c2f83a76cea921e1cf438ac1
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102215108"
---
# <a name="support-and-help-options-for-developers"></a>Возможности получения поддержки и справки для разработчиков

Если вы только приступаете к интеграции с Azure Active Directory (Azure AD), удостоверениями Майкрософт или API Microsoft Graph либо реализуете в приложении новую возможность, бывают случаи, когда вам как разработчику требуется помощь сообщества или техническая поддержка. Эта статья поможет вам составить представление о том, как получить такую поддержку, в том числе:

> [!div class="checklist"]
> * Как узнать, есть ли ответ на ваш вопрос в сообществе, или найти документацию по возможности, которую вы намерены реализовать.
> * В некоторых случаях для решения конкретной проблемы вам могут потребоваться наши средства технической поддержки.
> * Если вам не удается найти нужный ответ, можно задать вопрос в *Microsoft Q&a*
> * Если возникла проблема с одной из наших библиотек аутентификации, сообщите об этом на сайте *GitHub*.
> * И, наконец, если вам требуется консультация, подайте запрос в службу поддержки.

## <a name="search"></a>Поиск

Если у вас есть вопрос, связанный с разработкой, возможно, вы сможете найти ответ в документации, [примеры GitHub](https://github.com/azure-samples)или ответы на вопросы в [Microsoft Q&](/answers/products/) .

### <a name="scoped-search"></a>Поиск в заданных областях


Для ускорения поиска в [Microsoft Q&](https://docs.microsoft.com/answers/products/) документацию и примеры кода с помощью следующего запроса в вашей любимой поисковой подсистеме:

Для ускорения поиска в [Microsoft Q&](/answers/products/)документацию и примеры кода с помощью следующего запроса в вашей любимой поисковой подсистеме:


```
{Your Search Terms} (site:http://www.docs.microsoft.com/answers/products/ OR site:docs.microsoft.com OR site:github.com/azure-samples OR site:cloudidentity.com OR site:developer.microsoft.com/graph)
```

*{Your Search Terms}*  — это ключевые слова для поиска.

## <a name="use-the-development-support-tools"></a>Использование средств технической поддержки для разработки

| Инструмент  | Описание  |
|---------|---------|
| [jwt.ms](https://jwt.ms) | Вставьте идентификатор или маркер доступа, чтобы декодировать имена и значения утверждений. |
| [Microsoft Graph Explorer](https://developer.microsoft.com/graph/graph-explorer)| Средство для выполнения запросов и получения ответов по API Microsoft Graph. |

## <a name="post-a-question-to-microsoft-qa"></a>Публикация вопроса в Microsoft Q&A

[Microsoft Q&A](/answers/products/) — предпочтительный канал для вопросов, связанных с разработкой. Здесь члены сообщества разработчиков и команды Майкрософт постараются помочь вам решить проблемы.

Если вы не можете найти ответ на свой вопрос с помощью поиска, отправьте новый вопрос в [Microsoft Q&a](/answers/products/) . При этом используйте один из указанных ниже тегов. Это поможет сообществу своевременно определить ваш вопрос и ответить на него.

|Компонент/область  | Теги |
|---------|---------|
| Библиотека ADAL | [ADAL](/answers/topics/azure-ad-adal-deprecation.html) |
| Библиотека MSAL     | [msal](/answers/topics/azure-ad-msal.html) |
| ПО промежуточного слоя OWIN  | [[Azure-Active-Directory]](/answers/topics/azure-active-directory.html) |
| [Azure B2B](../external-identities/what-is-b2b.md)  | [[azure-ad-b2b]](/answers/topics/azure-ad-b2b.html) |
| [Azure B2C](https://azure.microsoft.com/services/active-directory-b2c/)  | [[azure-ad-b2c]](/answers/topics/azure-ad-b2c.html) |
| [API Microsoft Graph](https://developer.microsoft.com/graph/) | [[Azure-AD-Graph]](/answers/topics/azure-ad-graph.html) |
| Любая другая область, которая относится к аутентификации или авторизации | [[Azure-Active-Directory]](/answers/topics/azure-active-directory.html) |

Следующие сообщения от [Microsoft Q&A](/answers/products/) содержат советы по заданию вопросов и добавлению исходного кода. Следуя этим рекомендациям, вы повысите вероятность того, что участники сообщества быстро отреагируют на ваш вопрос.

* [Разделы справки задать хороший вопрос](/answers/articles/24951/how-to-write-a-quality-question.html)
* [How to create a Minimal, Complete, and Verifiable example](/answers/articles/24907/how-to-write-a-quality-answer.html) (Как создать минимальный, полный и доступный для проверки пример)

## <a name="create-a-github-issue"></a>Сообщение о проблеме на GitHub

Если возникла ошибка или проблема, связанная с нашими библиотеками, сообщите о ней на сайте GitHub. Так как в наших библиотеках используется открытый код, вы также можете отправить запрос на вытягивание.

Список библиотек и их репозиториев GitHub см. в следующих статьях:

* Библиотеки [библиотеки проверки Подлинности Azure Active Directory (ADAL)](../azuread-dev/active-directory-authentication-libraries.md) и репозитории GitHub
* Библиотеки [проверки подлинности Microsoft (MSAL)](reference-v2-libraries.md) и репозитории GitHub

## <a name="open-a-support-request"></a>Подача запроса в службу поддержки

Если вам требуется консультация, подайте запрос в службу поддержки. Для клиентов Azure предусмотрено несколько вариантов поддержки. Сравнить планы можно на [этой странице](https://azure.microsoft.com/support/plans/). Клиенты Azure также могут получить поддержку уровня Developer. Сведения о приобретении планов для поддержки уровня Developer см. на [этой странице](https://azure.microsoft.com/support/plans/developer/).

* Если у вас уже есть план поддержки Azure, [подайте запрос в службу поддержки здесь](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

* Если вы не являетесь клиентом Azure, можно подать запрос в корпорацию Майкрософт на странице [коммерческой поддержки](https://support.serviceshub.microsoft.com/supportforbusiness).

Также можно получить поддержку или задать вопрос при помощи [виртуального агента](https://support.microsoft.com/contactus/?ws=support).