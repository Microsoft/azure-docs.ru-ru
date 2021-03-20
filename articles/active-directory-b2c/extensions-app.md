---
title: Приложение расширений в Azure Active Directory B2C | Документация Майкрософт
description: Восстановление приложения b2c-extensions-app.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/06/2017
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: dc536fa4292d794e8d89a2564ad10a3c10dd0a3d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94560866"
---
# <a name="azure-ad-b2c-extensions-app"></a>Azure AD B2C: приложение расширений

После создания каталога Azure AD B2C в нем автоматически создается приложение `b2c-extensions-app. Do not modify. Used by AADB2C for storing user data.`. Это приложение, **b2c-extensions-app**, отображается в разделе *Регистрация приложений*. Служба Azure AD B2C использует его для хранения информации о пользователях и пользовательских атрибутах. При удалении приложения Azure AD B2C и рабочая среда не будут правильно работать.

> [!IMPORTANT]
> Не удаляйте приложение b2c-extensions-app, если вы не планируете сразу же удалить клиент. Если приложение удалено более 30 дней назад, сведения о пользователе будут безвозвратно утеряны.

## <a name="verifying-that-the-extensions-app-is-present"></a>Проверка наличия приложения расширений

Чтобы проверить наличие приложения b2c-extensions-app, сделайте следующее:

1. В клиенте Azure AD B2C в меню навигации слева щелкните **Все службы**.
1. Найдите и откройте элемент **Регистрация приложений**.
1. Найдите приложение, имя которого начинается с **b2c-extensions-app**

## <a name="recover-the-extensions-app"></a>Восстановление приложения расширений

Если вы случайно удалили приложение b2c-extensions-app, его можно восстановить в течение 30 дней. Восстановить приложение можно с помощью API Graph:

1. Перейдите по ссылке [https://graphexplorer.azurewebsites.net/](https://graphexplorer.azurewebsites.net/) .
1. Войдите на сайт как глобальный администратор каталога Azure AD B2C, в котором необходимо восстановить удаленное приложение. У этого глобального администратора должен быть адрес электронной почты следующего образца: `username@{yourTenant}.onmicrosoft.com`.
1. Выполните запрос HTTP GET для URL-адреса `https://graph.windows.net/myorganization/deletedApplications` с версией API 1.6. При выполнении этой операции отображается список всех приложений, удаленных за последние 30 дней.
1. Найдите в списке приложение, имя которого начинается с "B2C-Extensions-App", и скопируйте его `objectid` значение свойства.
1. Выполните запрос HTTP POST для URL-адреса `https://graph.windows.net/myorganization/deletedApplications/{OBJECTID}/restore`. Замените часть `{OBJECTID}` URL-адреса на `objectid` из предыдущего шага.

Теперь [восстановленное приложение может появиться](#verifying-that-the-extensions-app-is-present) на портале Azure.

> [!NOTE]
> Приложение можно восстановить, только если оно было удалено менее 30 дней назад. Если прошло более 30 дней, данные будут утеряны. Для получения дополнительной помощи отправьте запрос в службу поддержки.
