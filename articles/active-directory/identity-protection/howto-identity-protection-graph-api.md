---
title: Пакет SDK для Microsoft Graph PowerShell и Защита идентификации Azure Active Directory
description: Узнайте, как запрашивать Microsoft Graph обнаружения рисков и связанные сведения из Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: identity-protection
ms.topic: how-to
ms.date: 01/25/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: sahandle
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2db8cfe652c0fca4b68b00d846e345c1b60cd05d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98880242"
---
# <a name="azure-active-directory-identity-protection-and-the-microsoft-graph-powershell-sdk"></a>Защита идентификации Azure Active Directory и пакет SDK для Microsoft Graph PowerShell

Microsoft Graph — это конечная точка Unified API (Майкрософт) и источник интерфейсов API [защиты идентификации Azure Active Directory](./overview-identity-protection.md). В этой статье показано, как использовать [пакет SDK для Microsoft Graph PowerShell](/graph/powershell/get-started) для получения сведений о рискованных пользователях с помощью PowerShell. Организации, которые хотят запрашивать API-интерфейсы Microsoft Graph напрямую, могут использовать статью [руководство. выявление и устранение рисков с помощью Microsoft Graph API-интерфейсов](/graph/tutorial-riskdetection-api) для начала этого пути.


## <a name="connect-to-microsoft-graph"></a>Подключение к Microsoft Graph

Получить доступ к данным защиты идентификации с помощью Microsoft Graph можно в четыре этапа:

- [Создание сертификата](#create-a-certificate)
- [Создание регистрации приложения](#create-a-new-app-registration)
- [Настройка разрешений API](#configure-api-permissions)
- [Настройка допустимых учетных данных](#configure-a-valid-credential)

### <a name="create-a-certificate"></a>Создание сертификата

В рабочей среде вы будете использовать сертификат из вашего рабочего центра сертификации, но в этом примере мы будем использовать самозаверяющий сертификат. Создайте и экспортируйте сертификат, используя следующие команды PowerShell.

```powershell
$cert = New-SelfSignedCertificate -Subject "CN=MSGraph_ReportingAPI" -CertStoreLocation "Cert:\CurrentUser\My" -KeyExportPolicy Exportable -KeySpec Signature -KeyLength 2048 -KeyAlgorithm RSA -HashAlgorithm SHA256
Export-Certificate -Cert $cert -FilePath "C:\Reporting\MSGraph_ReportingAPI.cer"
```

### <a name="create-a-new-app-registration"></a>Создание регистрации приложения

1. В портал Azure перейдите к **Azure Active Directory**  >  **Регистрация приложений**.
1. Выберите **Новая регистрация**.
1. На странице **Создание** выполните следующие действия.
   1. В текстовом поле **имя** введите имя приложения (например, API обнаружения рисков Azure AD).
   1. В разделе **Поддерживаемые типы учетных записей** выберите тип учетных записей, которые будут использовать API.
   1. Выберите **Зарегистрировать**.
1. Запишите **идентификатор приложения (клиента)** и **идентификатор каталога (клиента)** , так как эти элементы понадобятся вам позже.

### <a name="configure-api-permissions"></a>Настройка разрешений API

В этом примере мы настроим разрешения приложения, позволяющие автоматически использовать этот пример. При предоставлении разрешений пользователю, который будет входить в систему, вместо этого выберите делегированные разрешения. Дополнительные сведения о различных типах разрешений можно найти в статье, [разрешения и согласие на платформе Microsoft Identity](../develop/v2-permissions-and-consent.md#permission-types).

1. В созданном **приложении** выберите **разрешения API**.
1. На странице **настроенные разрешения** на панели инструментов в верхней части страницы щелкните **Добавить разрешение**.
1. На странице **Добавить доступ через API** щелкните **Выбор API**.
1. На странице **Выбор API** выберите **Microsoft Graph**, а затем нажмите кнопку **Выбор**.
1. На странице **разрешения API запроса** выполните следующие действия. 
   1. Выберите **Разрешения приложений**.
   1. Установите флажки рядом с `IdentityRiskEvent.Read.All` и `IdentityRiskyUser.Read.All` .
   1. Выберите **Добавить разрешения**.
1. Выберите параметр **предоставить согласие администратора для домена** . 

### <a name="configure-a-valid-credential"></a>Настройка допустимых учетных данных

1. В созданном **приложении** выберите **Сертификаты & секреты**.
1. В разделе **Сертификаты** выберите **отправить сертификат**.
   1. Выберите ранее экспортированный сертификат в открывшемся окне.
   1. Выберите **Добавить**.
1. Запишите **отпечаток** сертификата, так как эти сведения понадобятся вам на следующем шаге.

## <a name="list-risky-users-using-powershell"></a>Вывод списка рискованных пользователей с помощью PowerShell

Чтобы обеспечить возможность запроса Microsoft Graph, необходимо установить `Microsoft.Graph` модуль в окне PowerShell с помощью `Install-Module Microsoft.Graph` команды.

Измените следующие переменные, включив в них сведения, созданные на предыдущих шагах, а затем выполните их как единое целое, чтобы получить сведения о рискованных пользователях с помощью PowerShell.

```powershell
$ClientID       = "<your client ID here>"        # Application (client) ID gathered when creating the app registration
$tenantdomain   = "<your tenant domain here>"    # Directory (tenant) ID gathered when creating the app registration
$Thumbprint     = "<your client secret here>"    # Certificate thumbprint gathered when configuring your credential

Select-MgProfile -Name "beta"
  
Connect-MgGraph -ClientId $ClientID -TenantId $tenantdomain -CertificateThumbprint $Thumbprint

Get-MgRiskyUser -All
```

## <a name="next-steps"></a>Дальнейшие действия

- [Приступая к работе с пакетом SDK для Microsoft Graph PowerShell](/graph/powershell/get-started)
- [Учебник. обнаружение и устранение рисков с помощью Microsoft Graph API](/graph/tutorial-riskdetection-api)
- [Overview of Microsoft Graph (Обзор Microsoft Graph)](https://developer.microsoft.com/graph/docs)
- [Получение доступа без пользователя](/graph/auth-v2-service)
- [Azure AD Identity Protection Service Root (Корень службы защиты идентификации Azure AD)](/graph/api/resources/identityprotectionroot)
- [Защита идентификации Azure Active Directory](./overview-identity-protection.md)
