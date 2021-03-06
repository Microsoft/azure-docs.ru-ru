---
title: Изменение метода и параметров двухфакторной проверки подлинности — Azure Active Directory
description: Узнайте, как изменить метод и параметры проверки безопасности для рабочей или учебной учетной записи на странице "Дополнительная проверка безопасности".
services: active-directory
author: curtand
manager: daveba
ms.reviewer: richagi
ms.assetid: d3372d9a-9ad1-4609-bdcf-2c4ca9679a3b
ms.workload: identity
ms.service: active-directory
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 07/06/2020
ms.author: curtand
ms.openlocfilehash: e0a6c566e8e0dfb77b5899f735020d0f1facf3d1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88798386"
---
# <a name="change-your-two-factor-verification-method-and-settings"></a>Изменение метода и параметров двухфакторной проверки подлинности

Настроив методы проверки безопасности для рабочей или учебной учетной записи, вы можете изменить любой из этих параметров, включая:

- метод проверки безопасности по умолчанию;

- параметры метода проверки безопасности, например номер телефона;

- настройка приложения Authenticator или удаление устройства из приложения Authenticator.

## <a name="using-the-additional-security-verification-page"></a>Использование страницы "Дополнительная проверка безопасности"

Если в вашей организации настроены определенные процедуры для включения двухфакторной проверки подлинности и управления ею, выполните их. Если нет, параметры метода проверки безопасности вы можете найти на странице [Дополнительная проверка безопасности](./multi-factor-authentication-end-user-first-time.md).

>[!Note]
>Если изображение на экране не соответствует тому, что описано в этой статье, значит ваш администратор включил интерфейс **Сведения для защиты (предварительная версия)** или в организации используется собственный портал. Дополнительную информацию о новом интерфейсе сведений для защиты см. в статье [Обзор сведений для защиты (предварительная версия)](./security-info-setup-signin.md). Чтобы узнать о портале вашей организации, обратитесь в службу технической поддержки.

### <a name="to-get-to-the-additional-security-verification-page"></a>Переход на страницу "Дополнительная проверка безопасности"

Вы можете открыть страницу "Дополнительная проверка безопасности" по [этой ссылке](https://account.activedirectory.windowsazure.com/proofup.aspx?proofup=1).

![Страница "Дополнительная проверка безопасности", где указаны сведения о доступных методах проверки безопасности](./media/multi-factor-authentication-end-user-manage-settings/mfa-security-verification-page.png)

Также на страницу **Дополнительная проверка безопасности** можно перейти, сделав следующее:

1. Войдите на портал [https://myapps.microsoft.com](https://myapps.microsoft.com).

1. В правом верхнем углу выберите имя учетной записи и **профиль**.

1. Выберите элемент **Дополнительная проверка безопасности**.  

    ![Ссылка на страницу "Дополнительная проверка безопасности" в разделе "Мои приложения"](./media/multi-factor-authentication-end-user-manage-settings/mfa-myapps-link.png)

>[!Note]
>Сведения об использовании раздела **Пароли приложений** на странице **Дополнительная проверка безопасности** см. в статье [Управление паролями приложения для двухфакторной проверки подлинности](multi-factor-authentication-end-user-app-passwords.md). Пароли приложений следует использовать только для тех приложений, которые не поддерживают двухфакторную проверку подлинности.

## <a name="change-your-default-security-verification-method"></a>Изменение метода проверки безопасности, используемого по умолчанию

Когда вы введете допустимое сочетание имени пользователя и пароля для входа в рабочую или учебную учетную запись, вам будет автоматически предложен выбранный метод проверки безопасности. В зависимости от требований организации это может быть уведомление или код проверки в приложении Authentificator, текстовое сообщение или телефонный звонок.

Если вы решили изменить используемый по умолчанию метод проверки безопасности, это можно сделать прямо здесь.

### <a name="to-change-your-default-security-verification-method"></a>Изменение метода проверки безопасности, используемого по умолчанию

1. На странице **Дополнительная проверка безопасности** выберите нужный метод в списке **Ваш предпочтительный вариант**. Здесь отображаются все варианты, но доступными будут только те, которые поддерживаются в вашей организации.

    - **Уведомить меня с помощью приложения.** Вы получите в приложении Authentificator уведомление от том, что вам поступил запрос на подтверждение.

    - **Позвонить на телефон для проверки подлинности.** Вы получите на мобильное устройство телефонный звонок с просьбой подтвердить информацию.

    - **Отправить код в SMS на телефон для проверки подлинности.** Вы получите на мобильное устройство код, включенный в текстовое сообщение. Этот код необходимо ввести в запрос проверки для рабочей или учебной учетной записи.

    - **Позвонить на мой рабочий телефон.** Вы получите на рабочий телефон звонок с просьбой подтвердить информацию.

    - **Использовать код проверки из приложения.** Вам нужно будет открыть приложение для проверки подлинности и получить код проверки, а затем ввести его в запрос для рабочей или учебной учетной записи.

2. Щелкните **Сохранить**.

## <a name="add-or-change-your-phone-number"></a>Добавление или изменение номера телефона

На странице **Дополнительная проверка безопасности** вы можете добавить новый номер телефона или изменить существующие номера.

>[!Important]
>Мы настоятельно рекомендуем добавить дополнительный номер телефона, чтобы избежать блокировки учетной записи при утере, краже или замене телефона, т. е. при утрате доступа к основному номеру телефона по любой причине.

### <a name="to-change-your-phone-numbers"></a>Смена номеров телефона

1. В разделе **Выберите способ ответа** на странице **Дополнительная проверка безопасности** обновите сведения о номерах телефона в полях **Телефон для проверки подлинности** (основное мобильное устройство) и **Рабочий телефон**.

1. Установите флажок рядом с параметром **альтернативный телефон для проверки подлинности** , а затем введите дополнительный номер телефона, по которому можно будет получать телефонные звонки, если вы не можете получить доступ к основному устройству.

1. Щелкните **Сохранить**.

## <a name="add-a-new-account-to-the-microsoft-authenticator-app"></a>Добавление учетной записи в приложение Microsoft Authenticator

Вы можете настроить рабочую и учебную учетную запись в приложении Microsoft Authenticator на платформах [Android](https://play.google.com/store/apps/details?id=com.azure.authenticator) и [iOS](https://apps.apple.com/app/microsoft-authenticator/id983156458).

Если вы уже настроили рабочую и учебную учетную запись в приложении Microsoft Authenticator, вам не нужно это делать снова.

1. В разделе **Выберите способ ответа** на странице **Дополнительная проверка безопасности** нажмите кнопку **Настроить приложение Authenticator**.

    ![Настройка рабочей или учебной учетной записи в приложении Microsoft Authenticator](./media/multi-factor-authentication-end-user-manage-settings/mfa-security-verification-page-auth-app.png)

1. Следуйте инструкциям на экране, отсканируйте QR-код с мобильного устройства, а затем щелкните **Далее**.

    Вам будет предложено подтвердить уведомление в приложении Microsoft Authenticator, чтобы проверить введенные сведения.

1. Щелкните **Сохранить**.

## <a name="delete-your-account-or-device-from-the-microsoft-authenticator-app"></a>Удаление учетной записи или устройства из приложения Microsoft Authenticator

Вы можете удалить учетную запись из приложения Microsoft Authenticator или удалить устройство из рабочей или учебной учетной записи. Обычно удаление устройства выполняется для окончательного удаления из учетной записи утерянного, украденного или устаревшего устройства, а удаление учетной записи — для устранения проблем с подключением или при изменении данных в учетной записи, например при смене имени пользователя.

### <a name="to-delete-your-device-from-your-work-or-school-account"></a>Удаление устройства из рабочей или учебной учетной записи

1. В разделе **Выберите способ ответа** на странице **Дополнительная проверка безопасности** нажмите кнопку **Настроить приложение Authenticator**.

1. Щелкните **Сохранить**.

### <a name="to-delete-your-account-from-the-microsoft-authenticator-app"></a>Удаление учетной записи из приложения Microsoft Authenticator

В приложении Microsoft Authenticator нажмите кнопку **Удалить** рядом с устройством, которое вы хотите удалить.

## <a name="turn-on-two-factor-verification-prompts-on-a-trusted-device"></a>Включение запросов на двухфакторную проверку подлинности на доверенных устройствах

В зависимости от политик организации при выполнении двухфакторной проверки подлинности в браузере может отображаться флажок **Не спрашивать в течение X дней**. Если вы установите этот флажок, чтобы не получать запросы на двухфакторную проверку подлинности, а затем потеряете устройство или оно окажется потенциально скомпрометированным, нужно снова включить двухфакторную проверку подлинности, чтобы защитить вашу учетную запись. Запросы можно включить только для всех устройств одновременно. К сожалению, нет возможности включать запросы для отдельного устройства.

### <a name="to-turn-two-factor-verification-prompts-back-on-for-your-devices"></a>Включение запросов на двухфакторную проверку подлинности на устройствах

На [странице **Дополнительная проверка безопасности**](#to-get-to-the-additional-security-verification-page) выберите **Восстановление многофакторной проверки подлинности на ранее доверенных устройствах**. При следующем входе в систему на любом устройстве вы снова увидите запрос на выполнение двухфакторной проверки подлинности.

## <a name="next-steps"></a>Дальнейшие действия

После добавления или обновления параметров двухфакторной проверки подлинности вы сможете управлять паролями приложений, входить в систему или получать помощь по некоторым распространенным проблемам, связанным с двухфакторной проверкой подлинности.

- [Управление паролями приложений для двухфакторной проверки подлинности](multi-factor-authentication-end-user-app-passwords.md) позволяет работать с приложениями, которые не поддерживают двухфакторную проверку подлинности.

- [Сведения о входе с помощью двухфакторной проверки подлинности](multi-factor-authentication-end-user-signin.md)

- [Устранение распространенных проблем с двухфакторной проверкой подлинности](multi-factor-authentication-end-user-troubleshoot.md)