---
author: billmath
ms.service: active-directory
ms.subservice: cloud-provisioning
ms.topic: include
ms.date: 10/16/2019
ms.author: billmath
ms.openlocfilehash: 6d95e40623f17a39145778a2fc067dccc68fd872
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95556821"
---
## <a name="steps-to-enable-single-sign-on"></a>Действия по включению единого входа
Подготовка облака работает с единым входом.  В настоящее время невозможно включить единый вход при установке агента, однако можно выполнить приведенные ниже действия, чтобы включить единый вход и использовать его. 

### <a name="step-1-download-and-extract-azure-ad-connect-files"></a>Шаг 1. скачивание и извлечение файлов Azure AD Connect
1.  Сначала скачайте последнюю версию [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594)
2.  Откройте командную строку с правами администратора и перейдите к только что загруженному MSI.
3.  Выполните следующую команду:  `msiexec /a C:\filepath\AzureADConnect.msi /qb TARGETDIR=C:\filepath\extractfolder`
4. Измените FilePath и екстрактфолдер, чтобы они соответствовали пути к файлу и имени папки извлечения.  Теперь содержимое должно находиться в папке извлечения.

### <a name="step-2-import-the-seamless-sso-powershell-module"></a>Шаг 2. импорт модуля PowerShell для простого единого входа

1. Скачайте и установите [Azure AD PowerShell](/powershell/azure/active-directory/overview).
2. Перейдите в папку `%programfiles%\Microsoft Azure Active Directory Connect`.
3. Импортируйте модуль PowerShell для простого единого входа с помощью следующей команды: `Import-Module .\AzureADSSO.psd1`.

### <a name="step-3-get-the-list-of-active-directory-forests-on-which-seamless-sso-has-been-enabled"></a>Шаг 3. Получение списка Active Directory лесов, в которых включен простой единый вход

1. Откройте сеанс PowerShell от имени администратора. В PowerShell вызовите `New-AzureADSSOAuthenticationContext`. При запросе введите учетные данные глобального администратора своего клиента.
2. Вызовите процедуру `Get-AzureADSSOStatus`. Эта команда выводит список лесов Active Directory (см. список "Домены"), в которых включена эта функция.

### <a name="step-4-enable-seamless-sso-for-each-active-directory-forest"></a>Шаг 4. Включение простого единого входа для каждого леса Active Directory

1. Вызовите процедуру `Enable-AzureADSSOForest`. При запросе введите свои учетные данные администратора домена для нужного леса Active Directory.

   > [!NOTE]
   >Имя пользователя администратора домена необходимо вводит в формате имени учетной записи SAM (contoso\johndoe или contoso.com\johndoe). Мы используем доменную часть имени пользователя, чтобы через DNS найти контроллер домена для администратора домена.

   >[!NOTE]
   >Указанная учетная запись не должна входить в группу защищенных пользователей. В противном случае операция завершится ошибкой.

2. Повторите предыдущие шаги для каждого леса Active Directory, в котором должна быть настроена эта функция.

### <a name="step-5-enable-the-feature-on-your-tenant"></a>Шаг 5. Включить функцию в своем клиенте

Чтобы включить эту функцию на клиенте, вызовите `Enable-AzureADSSO -Enable $true`.