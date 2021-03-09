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
ms.openlocfilehash: 43990952f6cbe90c729ac2df421c682fe8d42b1b
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102517955"
---
# <a name="support-and-help-options-for-developers"></a>Возможности получения поддержки и справки для разработчиков

Если вы только приступаете к интеграции с Azure Active Directory (Azure AD), удостоверениями Майкрософт или API Microsoft Graph либо реализуете в приложении новую возможность, бывают случаи, когда вам как разработчику требуется помощь сообщества или техническая поддержка. Ниже приведены рекомендации по разработке решений для платформ Microsoft Identity.

## <a name="create-an-azure-support-request"></a>Создание запроса на поддержку Azure

<div class='icon is-large'>
    <img alt='Azure support' src='https://docs.microsoft.com/media/logos/logo_azure.svg'>
</div>

Изучите различные [варианты поддержки Azure и выберите план](https://azure.microsoft.com/support/plans), который лучше всего отвечает вашим потребностям. Мы предлагаем варианты для широкого круга пользователей — от разработчиков, которые только начинают пользоваться облаком, до крупных организаций, развертывающих стратегические и критически важные для бизнеса приложения. Клиенты Azure могут создавать запросы на получение поддержки и управлять ими на портале Azure.

- Если у вас уже есть план поддержки Azure, отправьте [запрос в службу поддержки здесь](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

- Если вы не являетесь клиентом Azure, можно подать запрос в корпорацию Майкрософт на странице [коммерческой поддержки](https://support.serviceshub.microsoft.com/supportforbusiness).

## <a name="post-a-question-to-microsoft-qa"></a>Публикация вопроса на веб-сайте Microsoft Q&A
<div class='icon is-large'>
    <img alt='Microsoft Q&A' src='./media/common/question-mark-icon.png'>
</div>             

Получите ответы на вопросы о разработке приложений для идентификации непосредственно у инженеров Майкрософт, наиболее ценных специалистов Azure (MVP) и участников нашего экспертного сообщества.

[Microsoft Q&A](/answers/products/) — это рекомендуемый источник поддержки сообщества Azure.

Если вы не можете найти ответ на проблему, выполнив поиск в Microsoft Q&A, отправьте новый вопрос. Используйте один из следующих тегов при запросе [высокого качества](https://docs.microsoft.com/answers/articles/24951/how-to-write-a-quality-question.html):

| Компонент/область| Теги  |
|------------|---------------------------|
| Библиотека проверки подлинности Active Directory (ADAL)                              | [ADAL](https://docs.microsoft.com/answers/topics/azure-ad-adal-deprecation.html)                |
| Библиотека проверки подлинности Майкрософт (MSAL)                                     | [msal](https://docs.microsoft.com/answers/topics/azure-ad-msal.html)                            |
| Открытие веб-интерфейса для .NET (OWIN) по промежуточного слоя                               | [[Azure-Active-Directory]](https://docs.microsoft.com/answers/topics/azure-active-directory.html) |
| [Azure AD B2B/внешние удостоверения](../external-identities/what-is-b2b.md) | [[azure-ad-b2b]](https://docs.microsoft.com/answers/topics/azure-ad-b2b.html)                     |
| [Azure AD B2C](https://azure.microsoft.com/services/active-directory-b2c/)  | [[azure-ad-b2c]](https://docs.microsoft.com/answers/topics/azure-ad-b2c.html)                     |
| [API Microsoft Graph](https://developer.microsoft.com/graph/)               | [[Azure-AD-Graph]](https://docs.microsoft.com/answers/topics/azure-ad-graph.html)                 |
| Все остальные области проверки подлинности и авторизации                            | [[Azure-Active-Directory]](https://docs.microsoft.com/answers/topics/azure-active-directory.html) |

## <a name="create-a-github-issue"></a>Сообщение о проблеме на GitHub

<div class='icon is-large'>
    <img alt='GitHub-image' src='./media/common/github.svg'>
</div>

Если вам нужна помощь с одной из библиотек проверки подлинности Майкрософт (MSAL), откройте ошибку в своем репозитории на сайте GitHub.

| Библиотека MSAL | URL-адрес проблем GitHub|
| --- | --- |
| MSAL для Android | https://github.com/AzureAD/microsoft-authentication-library-for-android/issues |
| MSALный угловой | https://github.com/AzureAD/microsoft-authentication-library-for-js/issues |
| MSAL для iOS и MacOS| https://github.com/AzureAD/microsoft-authentication-library-for-objc/issues |
| MSAL Java | https://github.com/AzureAD/microsoft-authentication-library-for-java/issues |
| MSAL.js | https://github.com/AzureAD/microsoft-authentication-library-for-js/issues |
|MSAL.NET| https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues |
| MSAL Node | https://github.com/AzureAD/microsoft-authentication-library-for-js/issues |
| MSAL Python | https://github.com/AzureAD/microsoft-authentication-library-for-python/issues |
| MSAL реагировать | https://github.com/AzureAD/microsoft-authentication-library-for-js/issues |

## <a name="submit-feedback-on-azure-feedback"></a>Отправка отзыва в "Отзыв по Azure"

<div class='icon is-large'>
    <img alt='UserVoice' src='https://docs.microsoft.com/media/logos/logo-uservoice.svg'>
</div>

Чтобы выполнить запрос на новые функции, опубликуйте их в службе "Отзыв по Azure". Поделитесь своими идеями, чтобы сделать платформу Microsoft Identity более эффективной для разрабатываемых приложений.

| Служба                       | URL-адрес службы "Отзыв по Azure" |
|-------------------------------|---------------|
| Azure Active Directory | https://feedback.azure.com/forums/169401-azure-active-directory |
| Опыт разработки Azure Active Directory             | https://feedback.azure.com/forums/169401-azure-active-directory?category_id=164757 |
| Azure Active Directory — проверка подлинности             | https://feedback.azure.com/forums/169401-azure-active-directory?category_id=167256 |

## <a name="stay-informed-of-updates-and-new-releases"></a>Будьте в курсе обновлений и новых выпусков

<div class='icon is-large'>
    <img alt='Stay informed' src='https://docs.microsoft.com/media/common/i_blog.svg'>
</div>

- [Обновления Azure](https://azure.microsoft.com/updates/?category=identity): сведения о важных обновлениях, стратегиях и объявлениях продуктов.

- [Новые возможности в](https://docs.microsoft.com/azure/active-directory/develop/whats-new-docs)документации: Узнайте о новых возможностях платформы идентификации Майкрософт.

- [Блог Azure Active Directory Identity](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/bg-p/Identity): получение новостей и сведений об Azure AD.

- [Технический сообщество](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/bg-p/Identity/): Поделитесь своими впечатлениями, общайтесь и изучите экспертов.


