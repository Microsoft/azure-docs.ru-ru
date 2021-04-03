---
title: Приступая к работе с Azure AD в проектах WebApi в Visual Studio
description: Как начать использовать Azure Active Directory в проектах WebApi после подключения или создания Azure AD с помощью подключенных служб Visual Studio
author: ghogen
manager: jillfra
ms.prod: visual-studio-windows
ms.technology: vs-azure
ms.workload: azure-vs
ms.topic: how-to
ms.date: 03/12/2018
ms.author: ghogen
ms.custom: devx-track-csharp, aaddev, vs-azure
ms.openlocfilehash: fb8c1f25c8a22c96679a0310353f474f6b9d5ba5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "88165436"
---
# <a name="get-started-with-azure-active-directory-webapi-projects"></a>Начало работы с Azure Active Directory (проекты WebApi)

> [!div class="op_single_selector"]
> - [Начало работы](vs-active-directory-webapi-getting-started.md)
> - [Что произошло?](vs-active-directory-webapi-what-happened.md)

В этой статье приведены дополнительные рекомендации, которые потребуются вам, когда вы добавите Active Directory в проект ASP.NET WebAPI, последовательно выбрав **Проект > Подключенные службы** в Visual Studio. Если вы еще не добавили службу в проект, это можно сделать в любое время.

Изменения, которые вносятся в проект при добавлении подключенной службы, описаны в статье [Что произошло с моим проектом WebAPI в подключенной службе Visual Studio Azure Active Directory?](vs-active-directory-webapi-what-happened.md)

## <a name="requiring-authentication-to-access-controllers"></a>Требование проверки подлинности для доступа к контроллерам

Ко всем контроллерам в проекте добавлен атрибут `[Authorize]`. Этот атрибут обеспечивает аутентификацию пользователей перед доступом к интерфейсам API, определяемым этими контроллерами. Для анонимного доступа к контроллеру удалить с него этот атрибут. Если необходимо задать разрешения на более детальном уровне, примените атрибут к каждому методу, требующему проверки подлинности, а не к классу контроллера.

## <a name="next-steps"></a>Дальнейшие действия

- [Сценарии аутентификации в Azure Active Directory](./authentication-vs-authorization.md)
- [Добавление возможности входа в веб-приложение ASP.NET с помощью учетной записи Майкрософт](quickstart-v2-aspnet-webapp.md)