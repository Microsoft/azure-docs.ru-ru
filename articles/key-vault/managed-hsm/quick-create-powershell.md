---
title: Создание и получение атрибутов управляемого ключа в Azure Key Vault с помощью Azure PowerShell
description: Краткое руководство по настройке и получению управляемого ключа из Azure Key Vault с помощью Azure PowerShell
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.date: 01/26/2021
ms.author: mbaldwin
ms.openlocfilehash: 943555e9f7a26530a075aee27dd310d96974e652
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99072792"
---
# <a name="quickstart-set-and-retrieve-a-managed-key-from-azure-key-vault-using-powershell"></a>Краткое руководство. Настройка и получение управляемого ключа из Azure Key Vault с помощью PowerShell

Из этого краткого руководства вы узнаете, как создать хранилище ключей в Azure Key Vault с помощью Azure PowerShell. Azure Key Vault — это облачная служба, которая работает как защищенное хранилище секретов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Дополнительные сведения о хранилище ключей см. в статье [обзора](../general/overview.md). Azure PowerShell используется для создания ресурсов Azure и управления ими с помощью скриптов и команд. Создав ресурс, вы сможете сохранить в нем ключ.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Чтобы установить и использовать PowerShell локально для работы с этим руководством, понадобится модуль Azure PowerShell 1.0.0 или более поздней версии. Чтобы узнать версию, введите `$PSVersionTable.PSVersion`. Если вам необходимо выполнить обновление, ознакомьтесь со статьей, посвященной [установке модуля Azure PowerShell](/powershell/azure/install-az-ps). Если модуль PowerShell запущен локально, необходимо также выполнить командлет `Login-AzAccount`, чтобы создать подключение к Azure.

```azurepowershell-interactive
Login-AzAccount
```

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. С помощью командлета [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) Azure PowerShell создайте группу ресурсов с именем *myResourceGroup* в регионе *westus*. 

```azurepowershell-interactive
New-AzResourceGroup -Name "myResourceGroup" -Location "WestUS"

## Get your principal ID

To create a managed HSM, you will need your Azure Active Directory principal ID.  To obtain your ID, use the Azure PowerShell [Get-AzADUser](/powershell/module/az.resources/get-azaduser) cmdlet, passing your email address to the "UserPrincipalName" parameter:

```azurepowershell-interactive
Get-AzADUser -UserPrincipalName "<your@email.address>"
```

Будет возвращен идентификатор участника в формате "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx".

## <a name="create-a-managed-hsm"></a>Создание управляемого устройства HSM

Выполните командлет [New-AzKeyVaultManagedHsm](/powershell/module/az.keyvault/new-azkeyvaultmanagedhsm) в Azure PowerShell, чтобы создать новое устройство HSM под управлением Key Vault. Необходимо будет указать следующие сведения:

- Имя управляемого устройства HSM строка длиной от 3 до 24 знаков, которая может содержать только цифры (0–9), буквы (a-z, A-Z) и дефисы (-).

  > [!Important]
  > Каждое управляемое устройство HSM должно иметь уникальное имя. В следующих примерах замените заполнитель <your-unique-managed-hsm-name> именем своего управляемого устройства HSM.

- Имя группы ресурсов: **myResourceGroup**.
- Расположение: **EastUS**.
- Идентификатор субъекта. Передайте в параметре Administrator идентификатор субъекта Azure Active Directory, который вы получили при изучении предыдущего раздела. 

```azurepowershell-interactive
New-AzKeyVaultManagedHsm -Name "<your-unique-managed-hsm-name>" -ResourceGroupName "myResourceGroup" -Location "West US" -Administrator "<your-principal-ID>"
```

В выходных данных командлета отобразятся свойства созданного управляемого устройства HSM. Запишите значения двух указанных ниже свойств.

- **Имя управляемого устройства HSM** Имя, которое вы указали в параметре --name выше.
- **URI хранилища**. В нашем примере это https://&lt;your-unique-managed-hsm-name&gt;.vault.azure.net/. Необходимо, чтобы приложения, использующие ваше хранилище через REST API, использовали этот URI.

На этом этапе любые операции в этом хранилище ключей может выполнять только учетная запись Azure.

## <a name="activate-your-managed-hsm"></a>Активация управляемого устройства HSM

Все команды плоскости данных неактивны до тех пор, пока не будет активировано устройство HSM. Вы не сможете создавать ключи или назначать роли. Только администраторы, назначенные во время выполнения команды создания, могут активировать HSM. Чтобы активировать HSM, необходимо загрузить [домен безопасности](security-domain.md).

Чтобы активировать необходимый модуль HSM, выполните следующие действия:
- получите минимум 3 пары ключей RSA (максимум 10);
- укажите минимальное число ключей, необходимых для расшифровки домена безопасности (кворум).

Чтобы активировать устройство HSM, отправьте в него по крайней мере 3 (максимум 10) открытых ключей RSA. HSM шифрует домен безопасности с помощью этих ключей и отправляет их обратно. После успешного завершения загрузки домена безопасности устройство HSM готово к использованию. Также необходимо указать кворум, который является минимальным числом закрытых ключей, необходимых для расшифровки домена безопасности.

В приведенном ниже примере показано, как с помощью `openssl` (версия для Windows доступна [здесь](https://slproweb.com/products/Win32OpenSSL.html)) создать 3 самозаверяющих сертификата.

```console
openssl req -newkey rsa:2048 -nodes -keyout cert_0.key -x509 -days 365 -out cert_0.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_1.key -x509 -days 365 -out cert_1.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_2.key -x509 -days 365 -out cert_2.cer
```

> [!IMPORTANT]
> Безопасно создайте и сохраните пары ключей RSA и файл домена безопасности.

Выполните командлет Azure PowerShell [Export-AzKeyVaultSecurityDomain](/powershell/module/az.keyvault/export-azkeyvaultsecuritydomain), чтобы скачать домен безопасности и активировать управляемое устройство HSM. В приведенном ниже примере используются 3 пары ключей RSA (для этой команды требуются только открытые ключи) и кворум со значением 2.

```azurepowershell-interactive
Export-AzKeyVaultSecurityDomain -Name "<your-unique-managed-hsm-name>" -Certificates "cert_0.cer", "cert_1.cer", "cert_2.cer" -OutputPath "MHSMsd.ps.json" -Quorum 2
```

Надежно сохраните файл домена безопасности и пары ключей RSA. Они понадобятся вам для аварийного восстановления или создания другого управляемого устройства HSM, которое использует тот же домен безопасности (поэтому они могут совместно использовать ключи).

После успешной загрузки домена безопасности устройство HSM будет находиться в активном состоянии и готово к использованию.

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Create a key vault](../../../includes/key-vault-powershell-delete-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать Key Vault и сохранить в нем сертификат. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- Справочник по [командлетам Azure PowerShell для Azure Key Vault](/powershell/module/az.keyvault/)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)
