---
title: Поддерживает ли моя страница входа Azure AD учетные записи Майкрософт | Документация Майкрософт
description: Как отображаются поддерживаемые варианты имени пользователя при входе в систему
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.subservice: enterprise-users
ms.workload: identity
ms.topic: overview
ms.date: 12/02/2020
ms.author: curtand
ms.reviewer: kexia
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2c3b1c8d35c24d78abb8a519ddc8790649eb5f2b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96547871"
---
# <a name="sign-in-options-for-microsoft-accounts-in-azure-active-directory"></a>Варианты входа для учетных записей Майкрософт в Azure Active Directory

Страница входа Microsoft 365 для Azure Active Directory (Azure AD) поддерживает рабочие и учебные учетные записи, а также учетные записи Майкрософт. Но в зависимости от ситуации пользователя может поддерживаться один или оба варианта. Например, страница входа в Azure AD поддерживает:

* приложения, в которые можно войти с помощью обоих типов учетных записей;
* организации, которые принимают гостей.

## <a name="identification"></a>Идентификация
Определить, поддерживает ли используемая организацией страница входа учетные записи Майкрософт, можно по тексту подсказки в поле для ввода имени пользователя. Если в тексте подсказки указано "Электронная почта, номер телефона или Skype", то страница входа поддерживает учетные записи Майкрософт.

![Различие между страницами входа](./media/signin-account-support/ui-prompt.png)

[Дополнительные варианты входа можно использовать только для личных учетных записей Майкрософт](https://azure.microsoft.com/updates/microsoft-account-signin-options/ ). Они не поддерживаются при входе в ресурсы с помощью рабочей или учебной учетной записи.

## <a name="next-steps"></a>Дальнейшие действия

[Добавление имени личного домена с помощью портала Azure Active Directory](../fundamentals/add-custom-domain.md)