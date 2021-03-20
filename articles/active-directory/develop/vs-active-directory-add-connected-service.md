---
title: Использование подключенной службы Active Directory (Visual Studio)
description: Добавление Azure Active Directory с помощью диалогового окна "Добавление подключенных служб" в Visual Studio
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.openlocfilehash: a1ba7db72743ac122a697bf271e783ec64e041e8
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88165487"
---
# <a name="add-an-azure-active-directory-by-using-connected-services-in-visual-studio"></a>Добавление Azure Active Directory с помощью Подключенные службы в Visual Studio

С помощью Azure Active Directory (Azure AD) можно поддерживать единый Sign-On (SSO) для веб-приложений ASP.NET MVC или Active Directory проверку подлинности в службах веб-API. Благодаря аутентификации Azure AD пользователи смогут подключаться к вашим веб-приложениям, используя свои учетные записи Azure Active Directory. Преимущества аутентификация Azure AD с веб-API включают повышенную безопасность данных при предоставлении API из веб-приложения. С Azure AD вам не придется управлять отдельной системой проверки подлинности с отдельным управлением пользователями и учетными записями.

В этой статье и других статьях этой серии приводятся сведения об использовании подключенной службы Visual Studio в Active Directory. Эта возможность доступна в Visual Studio 2015 и более поздних версиях.

Сейчас подключенная служба Active Directory не поддерживает приложения ASP.NET Core.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure. Если у вас ее нет, [зарегистрируйтесь для работы с бесплатной пробной версией](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A261C142F) или [активируйте преимущества для подписчиков Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A261C142F).
- **Visual Studio 2015** или более поздней версии. [Скачайте Visual Studio прямо сейчас](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs).

### <a name="connect-to-azure-active-directory-using-the-connected-services-dialog"></a>Подключение к Azure Active Directory с помощью диалогового окна подключенных служб

1. В Visual Studio создайте или откройте проект ASP.NET MVC или проект веб-API ASP.NET. Для этого вы можете использовать шаблоны MVC, веб-API, одностраничного приложения, приложения API Azure, мобильного приложения Azure и мобильных служб Azure.

1. Выберите команду меню **Проект > Добавить подключенную службу...** или дважды щелкните узел **Подключенные службы** для проекта в обозревателе решений.

1. На странице **Подключенные службы** выберите пункт **Проверка подлинности через Azure Active Directory**.

    ![Страница "Подключенные службы"](./media/vs-azure-active-directory/connected-services-add-active-directory.png)

1. На странице **Введение** выберите **Далее**. Если на этой странице вы заметите ошибки, ознакомьтесь со статьей [Диагностика ошибок с помощью мастера подключения к Azure Active Directory](vs-active-directory-error.md).

    ![Страница "Введение"](./media/vs-azure-active-directory/configure-azure-ad-wizard-1.png)

1. На странице **Единый вход** выберите домен из раскрывающегося списка **Домены**. Список содержит все домены, доступные для учетных записей, перечисленных в диалоговом окне "Параметры учетной записи" Visual Studio (**Параметры учетной записи > файлов...**). В качестве альтернативы можно ввести имя домена, если вы не нашли искомый домен, например `mydomain.onmicrosoft.com` . Вы можете создать новое приложение Azure Active Directory или использовать параметры уже существующего приложения Azure Active Directory. По завершении нажмите кнопку **Далее**.

    ![Страница "Единый вход"](./media/vs-azure-active-directory/configure-azure-ad-wizard-2.png)

1. На странице **Доступ к каталогу** выберите  параметр **Чтение данных каталога**, если нужно. Обычно разработчики включают этот параметр.

    ![Страница "Доступ к каталогу"](./media/vs-azure-active-directory/configure-azure-ad-wizard-3.png)

1. Выберите **Готово**, чтобы запустить процесс изменения проекта для включения аутентификации Azure AD. В течение этого периода в Visual Studio отображается ход выполнения:

    ![Ход выполнения для подключенной службы Active Directory](./media/vs-azure-active-directory/active-directory-connected-service-output.png)

1. Когда процесс завершится, Visual Studio откроет браузер с одной из следующих статей, в зависимости от типа проекта:

    - [Начало работы с проектами .NET MVC](vs-active-directory-dotnet-getting-started.md)
    - [Начало работы с проектами веб-API](vs-active-directory-webapi-getting-started.md)

1. Домен Active Directory отобразится также на [портале Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

## <a name="how-your-project-is-modified"></a>Какие изменения произойдут в проекте

Когда вы добавляете подключенную службу с помощью мастера, Visual Studio добавляет в проект Azure Active Directory и соответствующие ссылки. В файлы конфигурации и файлы кода в проекте вносятся изменения, обеспечивающие поддержку Azure AD. Конкретные изменения, производимые Visual Studio, зависят от типа вашего проекта. Подробные сведения см. в следующих разделах:

- [Что произошло с моим проектом .NET MVC?](vs-active-directory-dotnet-what-happened.md)
- [Что произошло с моим проектом веб-API?](vs-active-directory-webapi-what-happened.md)

## <a name="next-steps"></a>Дальнейшие действия

- [Сценарии аутентификации в Azure Active Directory](./authentication-vs-authorization.md)
- [Добавление возможности входа в веб-приложение ASP.NET с помощью учетной записи Майкрософт](quickstart-v2-aspnet-webapp.md)