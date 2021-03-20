---
title: Установка Azure AD Connect агента подготовки облака с помощью PowerShell
description: Узнайте, как установить Azure AD Connect агент подготовки облака с помощью командлетов PowerShell.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/16/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: c20cfb96b5cd6e1d05e332fa7157fe6e0cde8656
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98614188"
---
# <a name="install-the-azure-ad-connect-provisioning-agent-using-powershell-cmdlets"></a>Установка агента подготовки Azure AD Connect с помощью командлетов PowerShell 
В следующем документе показано, как установить агент подготовки Azure AD Connect с помощью командлетов PowerShell.
 

## <a name="prerequisite"></a>Предварительные требования: 


>[!IMPORTANT]
>В следующих инструкциях по установке предполагается, что выполнены все [необходимые условия](how-to-prerequisites.md).
>
> Перед установкой агента подготовки Azure AD Connect с помощью командлетов PowerShell в Windows Server необходимо включить TLS 1,2. Чтобы включить TLS 1,2, можно выполнить действия, описанные [здесь](how-to-prerequisites.md#tls-requirements).

 

## <a name="install-the-azure-ad-connect-provisioning-agent-using-powershell-cmdlets"></a>Установка агента подготовки Azure AD Connect с помощью командлетов PowerShell 


 1. Войдите в портал Azure, а затем перейдите в **Azure Active Directory**.
 2. В меню слева выберите **Azure AD Connect**.
 3. Выберите **Управление подготовкой (предварительная версия)**  > **Просмотр всех агентов**.
 4. Скачайте Azure AD Connect агент подготовки из портал Azure в локальную систему.  

   ![Скачивание локального агента](media/how-to-install/install-9.png)</br>
 5. Для выполнения этих инструкций агент был скачан в следующую папку: "К:\провисионингсетуп". 
 6. Установка Провисионингажент в тихом режиме

   ```
   $installerProcess = Start-Process c:\temp\AADConnectProvisioningAgent.Installer.exe /quiet -NoNewWindow -PassThru 
   $installerProcess.WaitForExit()  
   ```
 7. Импортировать модуль агента подготовки PS 

   ```
   Import-Module "C:\Program Files\Microsoft Azure AD Connect Provisioning Agent\Microsoft.CloudSync.Powershell.dll" 
   ```
 8. Подключение к AzureAD с помощью учетных данных глобального администратора. Этот раздел можно настроить для выборки пароля из безопасного хранилища. 

   ```
   $globalAdminPassword = ConvertTo-SecureString -String "Global admin password" -AsPlainText -Force 

   $globalAdminCreds = New-Object System.Management.Automation.PSCredential -ArgumentList ("GlobalAdmin@contoso.onmicrosoft.com", $globalAdminPassword) 
   ```

   Connect-AADCloudSyncAzureAD $globalAdminCreds учетных данных 

 9. Добавьте учетную запись gMSA, укажите учетные данные администратора домена, чтобы создать учетную запись gMSA по умолчанию. 
 
   ```
   $domainAdminPassword = ConvertTo-SecureString -String "Domain admin password" -AsPlainText -Force 

   $domainAdminCreds = New-Object System.Management.Automation.PSCredential -ArgumentList ("DomainName\DomainAdminAccountName", $domainAdminPassword) 

   Add-AADCloudSyncGMSA -Credential $domainAdminCreds 
   ```
 10. Или используйте приведенный выше командлет, чтобы предоставить предварительно созданную учетную запись gMSA. 

 
   ```
   Add-AADCloudSyncGMSA -CustomGMSAName preCreatedGMSAName$ 
   ```
 11. Добавить домен 

   ```
   $contosoDomainAdminPassword = ConvertTo-SecureString -String "Domain admin password" -AsPlainText -Force 

   $contosoDomainAdminCreds = New-Object System.Management.Automation.PSCredential -ArgumentList ("DomainName\DomainAdminAccountName", $contosoDomainAdminPassword) 

   Add-AADCloudSyncADDomain -DomainName contoso.com -Credential $contosoDomainAdminCreds 
   ```
 12. Или используйте приведенный выше командлет, чтобы настроить предпочтительные контроллеры домена. 

   ```
   $preferredDCs = @("PreferredDC1", "PreferredDC2", "PreferredDC3") 

   Add-AADCloudSyncADDomain -DomainName contoso.com -Credential $contosoDomainAdminCreds -PreferredDomainControllers $preferredDCs 
   ```
 13. Повторите предыдущий шаг, чтобы добавить другие домены, укажите имена учетных записей и доменных имен соответствующих доменов. 
 14. Перезапустите службу. 
   ```
   Restart-Service -Name AADConnectProvisioningAgent  
   ```
 15.  Перейдите на портал Azure, чтобы создать конфигурацию синхронизации в облаке.

## <a name="provisioning-agent-gmsa-powershell-cmdlets"></a>Командлеты gMSA агента подготовки PowerShell
Теперь, когда агент установлен, можно применить более детализированные разрешения для gMSA.  Дополнительные сведения и пошаговые инструкции по настройке разрешений см. в статье [Azure AD Connect агент подготовки облака gMSA командлетов PowerShell](how-to-gmsa-cmdlets.md) .

## <a name="next-steps"></a>Дальнейшие действия 

- [Что собой представляет подготовка?](what-is-provisioning.md)
- [GMSA командлеты PowerShell для агента подготовки облака Azure AD Connect](how-to-gmsa-cmdlets.md)
- [Что представляет собой облачная синхронизация Azure AD Connect?](what-is-cloud-sync.md)