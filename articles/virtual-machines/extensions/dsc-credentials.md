---
title: Передача учетных данных в Azure с помощью настройки требуемого состояния
description: Сведения о безопасной передаче учетных данных для виртуальных машин Azure с помощью настройки требуемого состояния PowerShell (DSC).
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: bobbytreed
ms.author: robreed
ms.collection: windows
ms.date: 05/02/2018
ms.openlocfilehash: 6817dd6baacd835b7d433177ff18af1238ee44a6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102560085"
---
# <a name="pass-credentials-to-the-azure-dscextension-handler"></a>Передача учетных данных в обработчик Azure DSCExtension

В этой статье описывается расширение Desired State Configuration (DSC) для Azure. Обзор обработчика расширения DSC см. в статье [Общие сведения об обработчике расширения Desired State Configuration в Azure](dsc-overview.md).

 

## <a name="pass-in-credentials"></a>Передача учетных данных

В рамках процесса настройки может потребоваться настройка учетных записей пользователей, доступ к службам или установка программы в пользовательском контексте. Чтобы выполнить эти действия, необходимо указать учетные данные.

DSC можно использовать для настройки параметризованных конфигураций. В параметризованной конфигурации учетные данные передаются в конфигурацию и надежно хранятся в MOF-файлах. Обработчик расширений Azure упрощает управление учетными данными, предоставляя автоматическое управление сертификатами.

Следующий скрипт конфигурации DSC создает учетную запись локального пользователя с указанным паролем.

```powershell
configuration Main
{
    param(
        [Parameter(Mandatory=$true)]
        [ValidateNotNullorEmpty()]
        [PSCredential]
        $Credential
    )
    Node localhost {
        User LocalUserAccount
        {
            Username = $Credential.UserName
            Password = $Credential
            Disabled = $false
            Ensure = "Present"
            FullName = "Local User Account"
            Description = "Local User Account"
            PasswordNeverExpires = $true
        }
    }
}
```

Важно включить **node localhost** в конфигурацию. Обработчик расширений специально ищет инструкцию **node localhost**. Если эта инструкция отсутствует, следующие шаги не работают. Важно также добавить приведение типа **[PsCredential]**, так как этот конкретный тип активирует в расширении шифрование учетных данных.

Для публикации этого скрипта в хранилище BLOB-объектов Azure:

`Publish-AzVMDscConfiguration -ConfigurationPath .\user_configuration.ps1`

Для установки расширения Azure DSC и указания учетных данных:

```powershell
$configurationName = 'Main'
$configurationArguments = @{ Credential = Get-Credential }
$configurationArchive = 'user_configuration.ps1.zip'
$vm = Get-AzVM -Name 'example-1'

$vm = Set-AzVMDscExtension -VMName $vm -ConfigurationArchive $configurationArchive -ConfigurationName $configurationName -ConfigurationArgument @configurationArguments

$vm | Update-AzVM
```

## <a name="how-a-credential-is-secured"></a>Как обеспечивается безопасность учетных данных

При выполнении этого кода запрашиваются учетные данные. После предоставления учетных данных они недолго хранятся в памяти. При публикации с помощью командлета **Set-AzVMDscExtension** учетные данные передаются на виртуальную машину по протоколу HTTPS, где Azure сохраняет их в зашифрованном виде на диске с использованием локального сертификата виртуальной машины. Затем они быстро расшифровываются в памяти и повторно шифруются для передачи в DSC.

Этот процесс отличается от [использования безопасных конфигураций без обработчика расширений](/powershell/scripting/dsc/pull-server/securemof). Среда Azure предоставляет способ безопасной передачи данных конфигурации с помощью сертификатов. При использовании обработчика расширений DSC нет необходимости указывать запись **$CertificatePath** либо **$CertificateID**/ **$Thumbprint** в **ConfigurationData**.

## <a name="next-steps"></a>Дальнейшие действия

- Ознакомьтесь с [общими сведениями об обработчике расширения Desired State Configuration в Azure](dsc-overview.md).
- Изучите [шаблон Azure Resource Manager для расширения DSC](dsc-template.md).
- Дополнительные сведения о PowerShell DSC см. в [центре документации по PowerShell](/powershell/scripting/dsc/overview/overview).
- Дополнительные функциональные возможности, которыми можно управлять с помощью PowerShell DSC, и ресурсы по DSC см. в [коллекции PowerShell](https://www.powershellgallery.com/packages?q=DscResource&x=0&y=0).