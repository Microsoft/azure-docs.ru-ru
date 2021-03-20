---
title: включить файл
description: включить файл
services: virtual-machines-windows
author: rothja
manager: craigg
tags: azure-service-management
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: include
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 04/30/2018
ms.author: jroth
ms.custom: include file
ms.openlocfilehash: 3509185baa3a9d7be90c1fa4bd8000da4a8a6fe5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95557857"
---
## <a name="prepare-for-akv-integration"></a>Подготовка к интеграции AKV
Чтобы использовать интеграцию хранилища ключей Azure для настройки виртуальной машины с SQL Server, необходимо выполнить несколько предварительных условий: 

1. [Установка Azure PowerShell](#install)
2. [Создание Azure Active Directory](#register)
3. [Создайте хранилище ключей.](#createkeyvault)

В следующих разделах описаны эти предварительные условия и представлены сведения, необходимые для использования командлетов PowerShell в дальнейшем.

[!INCLUDE [updated-for-az](./updated-for-az.md)]

### <a name="install-azure-powershell"></a><a id="install"></a> Установка Azure PowerShell
Убедитесь, что установлен последний модуль Azure PowerShell. Подробнее: [Установка и настройка Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="register-an-application-in-your-azure-active-directory"></a><a id="register"></a> Регистрация приложения в Azure Active Directory

Во-первых, в вашей подписке должна быть служба [Azure Active Directory](https://azure.microsoft.com/trial/get-started-active-directory/) (AAD). Среди прочих преимуществ она дает возможность предоставить доступ к вашему хранилищу ключей для определенных пользователей и приложений.

Далее необходимо зарегистрировать приложение в AAD. Это позволит получить учетную запись субъекта-службы, имеющую доступ к вашему хранилищу ключей, которое потребуется виртуальной машине. В Azure Key Vault статье эти действия можно найти в разделе [Регистрация приложения с Azure Active Directory](../articles/key-vault/general/manage-with-cli2.md#registering-an-application-with-azure-active-directory) . Кроме того, вы можете ознакомиться с снимками экрана в **разделе Получение удостоверения для приложения** [этой записи блога](/archive/blogs/kv/azure-key-vault-step-by-step). Перед выполнением этих шагов учтите, что во время этой регистрации необходимо собрать следующие сведения, которые понадобятся в дальнейшем при включении интеграции Azure Key Vault на виртуальной машине SQL.

* После добавления приложения найдите **идентификатор приложения** (также известный как AAD ClientID или AppID) в колонке **зарегистрированное приложение** .
    Идентификатор приложения позже назначается параметру **$spName** (имя субъекта-службы) в сценарии PowerShell, чтобы обеспечить интеграцию Azure Key Vault.

   ![Идентификатор приложения](./media/virtual-machines-sql-server-akv-prepare/aad-application-id.png)

* Во время выполнения этой процедуры при создании ключа скопируйте секрет ключа, как показано на приведенном ниже снимке экрана. Этот секрет ключа позже назначается параметру **$spSecret** (секрет субъекта-службы) в сценарии PowerShell.

   ![Секрет AAD](./media/virtual-machines-sql-server-akv-prepare/aad-sp-secret.png)

* Идентификатор и секрет приложения также будут использоваться для создания учетных данных в SQL Server.

* Необходимо авторизовать этот новый идентификатор приложения (или идентификатор клиента), чтобы иметь следующие разрешения на доступ: **Get**, **wrapKey**, **unwrapKey**. Это делается с помощью командлета [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy). Дополнительные сведения см. в статье [Что такое хранилище ключей Azure?](../articles/key-vault/general/overview.md)

### <a name="create-a-key-vault"></a><a id="createkeyvault"></a> Создание хранилища ключей
Чтобы использовать хранилище ключей Azure для хранения ключей, которые будут использоваться для шифрования на виртуальной машине, необходим доступ к хранилищу ключей. Если вы еще не настроили ваше хранилище ключей, создайте его, выполнив следующие действия, описанные в статье [Приступая к работе с хранилищем ключей Azure](../articles/key-vault/general/overview.md). Перед выполнением этих шагов учтите, что во время этой настройки необходимо собрать некоторые сведения, которые понадобятся в дальнейшем при включении интеграции Azure Key Vault на виртуальной машине SQL.

```azurepowershell
New-AzKeyVault -VaultName 'ContosoKeyVault' -ResourceGroupName 'ContosoResourceGroup' -Location 'East Asia'
```

При создании хранилища ключей запишите возвращенное свойство **vaultUri** , которое является URL-адресом хранилища ключей. В примере, приведенном на этом шаге и показанном ниже, именем хранилища ключей является ContosoKeyVault, поэтому URL-адрес хранилища ключей — https://contosokeyvault.vault.azure.net/.

URL-адрес хранилища ключей позже назначается параметру **$akvURL** в сценарии PowerShell для включения интеграции хранилища ключей Azure.

После создания хранилища ключей нужно добавить в него ключ. На этот ключ в дальнейшем будет добавлена ссылка при создании асимметричного ключа SQL Server.