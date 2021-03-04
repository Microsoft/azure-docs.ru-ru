---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 01/27/2021
ms.author: mimart
ms.openlocfilehash: 41d9962657aa81dbe34a52302d1b68ec655f2893
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102095295"
---
Если у вас еще нет сертификата, можно использовать самозаверяющий сертификат. Самозаверяющий сертификат — это сертификат безопасности, не подписанный центром сертификации (ЦС) и не предоставляющий гарантии безопасности сертификата, подписанного центром сертификации. 

# <a name="windows"></a>[Windows](#tab/windows)

В Windows используйте командлет [New-SelfSignedCertificate](/powershell/module/pkiclient/new-selfsignedcertificate) PowerShell, чтобы создать сертификат.

1. Выполните эту команду PowerShell, чтобы создать самозаверяющий сертификат. Измените аргумент `-Subject`, указав реальные значения приложения и имени клиента Azure  AD B2C. Можно также скорректировать дату `-NotAfter`, чтобы указать другой срок действия сертификата.

    ```PowerShell
    New-SelfSignedCertificate `
        -KeyExportPolicy Exportable `
        -Subject "CN=yourappname.yourtenant.onmicrosoft.com" `
        -KeyAlgorithm RSA `
        -KeyLength 2048 `
        -KeyUsage DigitalSignature `
        -NotAfter (Get-Date).AddMonths(12) `
        -CertStoreLocation "Cert:\CurrentUser\My"
    ```

1. Откройте **Управление сертификатами пользователя** > **Текущий пользователь** > **Персональные** > **Сертификаты** > *имя_приложения.имя_клиента.onmicrosoft.com*.
1. Выберите сертификат, а затем выберите **действие**  >  **экспортировать все задачи**  >  .
1. Нажмите **Да** > **Далее** > **Да, экспортировать закрытый ключ** > **Далее**.
1. Примите значения по умолчанию для **формата файла экспорта**.
1. Укажите пароль для сертификата.

Чтобы Azure AD B2C принять пароль PFX-файла, пароль должен быть зашифрован с помощью параметра TripleDES – SHA1 в служебной программе экспорта хранилища сертификатов Windows, а не в AES256-SHA256.

# <a name="macos"></a>[macOS](#tab/macos)

В macOS для создания сертификата используйте [Помощник по сертификатам](https://support.apple.com/guide/keychain-access/aside/glosa3ed0609/11.0/mac/11.0) в цепочке ключей.

1. Следуйте инструкциям, чтобы [Создать Самозаверяющие сертификаты с доступом к цепочке ключей на Mac](https://support.apple.com/guide/keychain-access/kyca8916/mac).
1. В приложении для доступа к цепочке ключей на компьютере Mac выберите созданный сертификат.
1. Выберите **файл**  >  **Экспорт элементов**.
1. Выберите имя файла для сохранения сертификата. Например, **Self-заверенный-Certificate. p12**.
1. В качестве **формата файла** выберите файл **обмена личной информацией (P12)**.
1. Щелкните **Сохранить**.
1. Введите **пароль**, а затем **Проверьте** его пароль.
1. Замените расширение файла на `.pfx` . Например, **Селф-сигнед-цертификате. pfx**.

---