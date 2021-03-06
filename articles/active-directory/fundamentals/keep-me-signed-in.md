---
title: Настроить параметр "остаться в системе?" запрос учетных записей Azure Active Directory
description: Подробнее о команде "оставаться в системе" (функции "оставаться), которая отображает" остаться в системе "? запрос, Настройка на портале Azure Active Directory и устранение неполадок при входе.
services: active-directory
author: CelesteDG
manager: daveba
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: fundamentals
ms.date: 06/05/2020
ms.author: celested
ms.reviewer: asteen, jlu, hirsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: bed6bc43dfc15abf2bdf9f38a5de2240d348d6fb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89320262"
---
# <a name="configure-the-stay-signed-in-prompt-for-azure-ad-accounts"></a>Настроить параметр "остаться в системе?" запрос учетных записей Azure AD

Оставаться в системе (функции "оставаться) отображается сообщение" **оставаться в системе?** ". После успешного входа пользователь покажет запрос. Если пользователь ответит на этот запрос, служба "оставаться в системе **"** выдаст им постоянный [маркер обновления](../develop/developer-glossary.md#refresh-token). Для федеративных клиентов запрос будет отображаться после успешной проверки подлинности пользователя в Федеративной службе удостоверений.

На следующей схеме показан поток входа пользователя для управляемого клиента и федеративного клиента, а также новый запрос "оставаться в системе". Этот поток содержит смарт-логику, поэтому параметр " **оставаться в** системе" не отображается, если система машинного обучения обнаруживает вход с высоким риском или вход с общего устройства.

:::image type="content" source="./media/keep-me-signed-in/kmsi-workflow.png" alt-text="Схема, демонстрирующая последовательность входа пользователя для управляемого и федеративного клиента":::

> [!NOTE]
> Для настройки параметра "оставаться в системе" необходимо использовать Azure Active Directory (Azure AD) Premium 1, Premium 2 или Basic Edition или лицензию на Microsoft 365. Дополнительные сведения о лицензировании и выпусках см. в разделе [Регистрация для использования Azure AD Premium](active-directory-get-started-premium.md).<br><br>Выпуски Azure AD Premium и Basic доступны для клиентов в Китае с использованием международных экземпляров Azure AD. Эти выпуски в настоящее время не поддерживаются службой Azure под управлением 21Vianet в Китае. Для получения дополнительных сведений обратитесь к нам с помощью [форума Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory/).

## <a name="configure-kmsi"></a>Настройка функции "оставаться

1. Войдите на [портал Azure](https://portal.azure.com/) с учетной записью глобального администратора каталога.
1. Выберите **Azure Active Directory**, выберите **Корпоративная фирменная символика**, а затем щелкните **настроить**.
1. В разделе **Дополнительные параметры** найдите параметр Показывать, **чтобы остаться в** системе.

   Этот параметр позволяет выбрать, будут ли пользователи входить в Azure AD до тех пор, пока они не будут явно выходить из нее.
   * Если вы выберете " **нет**", параметр " **остаться в** системе" скрыт после успешного входа пользователя, и пользователь должен войти в систему при каждом закрытии и повторном открытии браузера.
   * Если выбрано **значение Да**, то пользователю будет доступен параметр **всегда оставаться в системе?** .

    :::image type="content" source="./media/keep-me-signed-in/kmsi-company-branding-advanced-settings-kmsi-1.png" alt-text="Снимок экрана: параметр &quot;Показать параметр для сохранения&quot;":::

## <a name="troubleshoot-sign-in-issues"></a>Устранение неполадок при входе

Если пользователь не работает со статьей " **остаться в системе?** ", как показано на следующей схеме, но отменяет попытки входа, вы увидите запись журнала входа, которая указывает на прерывание.

:::image type="content" source="./media/keep-me-signed-in/kmsi-stay-signed-in-prompt.png" alt-text="Показывает, что вы вошли? предупреждение":::

Сведения об ошибке входа приведены ниже и выделены в примере.

* **Код ошибки при входе**: 50140
* **Причина сбоя**: Эта ошибка возникла из-за прерывания "оставаться в системе" при входе пользователя.

:::image type="content" source="./media/keep-me-signed-in/kmsi-sign-ins-log-entry.png" alt-text="Пример записи журнала входа с прерыванием &quot;оставаться в системе&quot;":::

Вы можете запретить пользователям видеть прерывание, установив для параметра " **Показывать"** значение " **нет** " в расширенной настройке фирменной символики. Это приведет к отключению запроса функции "оставаться для всех пользователей в каталоге Azure AD.

Вы также можете использовать постоянные элементы управления сеансом браузера в условном доступе, чтобы запретить пользователям видеть запрос функции "оставаться. Этот параметр позволяет отключить запрос функции "оставаться для выделенной группы пользователей (например, глобальных администраторов), не влияя на поведение входа для остальных пользователей в каталоге. Дополнительные сведения см. [в разделе Частота входа пользователя в систему](../conditional-access/howto-conditional-access-session-lifetime.md). 

Чтобы убедиться, что запрос функции "оставаться отображается только в том случае, если он может помочь пользователю, запрос функции" оставаться намеренно не показан в следующих сценариях:

* Пользователь вошел с помощью простого единого входа и встроенной проверки подлинности Windows (IWA)
* Пользователь вошел с помощью службы федерации Active Directory (AD FS) и IWA
* Пользователь является гостевым в клиенте
* Высокий показатель риска пользователя
* Вход выполняется во время потока согласия пользователя или администратора
* Постоянное управление сеансом браузера настраивается в политике условного доступа.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте о других параметрах, влияющих на время ожидания сеанса входа:

* Microsoft 365 — [время ожидания бездействующего сеанса](/sharepoint/sign-out-inactive-users)
* Условный доступ Azure AD — [Частота входа пользователя](../conditional-access/howto-conditional-access-session-lifetime.md)
* Портал Azure — [время ожидания бездействия на уровне каталога](../../azure-portal/set-preferences.md#change-the-directory-timeout-setting-admin)