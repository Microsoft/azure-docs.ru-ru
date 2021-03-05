---
title: Использование доменных служб Azure AD для авторизации доступа к данным файлов по протоколу SMB
description: Узнайте, как включить проверку подлинности на основе удостоверений через протокол SMB для службы файлов Azure с помощью доменных служб Azure Active Directory. Затем присоединенные к домену виртуальные машины Windows могут получить доступ к файловым ресурсам Azure с помощью учетных данных Azure AD.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 01/03/2021
ms.author: rogarana
ms.subservice: files
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: 6d8402f5e8a1a9af1f84ff6beff7324d1675e441
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102214411"
---
# <a name="enable-azure-active-directory-domain-services-authentication-on-azure-files"></a>Включение проверки подлинности доменных служб Azure Active Directory в службе файлов Azure

Службы [файлов Azure](storage-files-introduction.md)   поддерживает проверку подлинности на основе удостоверений через протокол SMB с помощью двух типов доменных служб: локальных домен Active Directory служб (AD DS) и Azure Active Directory доменных служб (Azure AD DS). Мы настоятельно рекомендуем ознакомиться с [разделом "как работает](./storage-files-active-directory-overview.md#how-it-works) ", чтобы выбрать подходящую службу домена для проверки подлинности. Программа установки различается в зависимости от выбранной службы домена. Эта статья посвящена включению и настройке Azure AD DS для проверки подлинности с помощью файловых ресурсов Azure.

Если вы не знакомы с файловыми ресурсами Azure, рекомендуем ознакомиться с нашим [руководством по планированию](storage-files-planning.md) , прежде чем читать следующую серию статей.

> [!NOTE]
> Служба файлов Azure поддерживает проверку подлинности Kerberos с помощью AD DS Azure только с RC4-HMAC. Шифрование AES Kerberos пока не поддерживается.
> Служба файлов Azure поддерживает аутентификацию Azure AD DS с полной синхронизацией с Azure AD. Если вы включили синхронизацию с заданной областью в Azure AD DS что синхронизирует только ограниченный набор удостоверений из Azure AD, проверка подлинности и авторизация не поддерживается.

## <a name="prerequisites"></a>Предварительные требования

Прежде чем включить Azure AD через SMB для файловых ресурсов Azure, убедитесь, что выполнены следующие предварительные требования:

1.  **Выберите или создайте клиент Azure AD.**

    Вы можете использовать новый или существующий клиент для выполнения аутентификации Azure AD по протоколу SMB. Клиент и файловый ресурс, к которому вы хотите получить доступ, должны быть связаны с одной и той же подпиской.

    Чтобы создать новый клиент Azure AD, можно [добавить клиент и подписку Azure AD](/windows/client-management/mdm/add-an-azure-ad-tenant-and-azure-ad-subscription). Если у вас уже есть клиент Azure AD, но вы хотите создать новый клиент для использования с файловыми ресурсами Azure, см. статью [создание Azure Active Directory клиента](/rest/api/datacatalog/create-an-azure-active-directory-tenant).

1.  **Включите доменные службы Azure AD в клиенте Azure AD**.

    Чтобы поддерживать аутентификацию с помощью учетных данных Azure AD, нужно включить доменные службы Azure AD для вашего клиента Azure AD. Если вы не администратор клиента Azure AD, обратитесь к администратору и следуйте пошаговым инструкциям, чтобы [включить доменные службы Azure Active Directory на портале Azure](../../active-directory-domain-services/tutorial-create-instance.md).

    Для завершения развертывания AD DS Azure обычно требуется около 15 минут. Прежде чем перейти к следующему шагу, убедитесь, что для состояния работоспособности AD DS Azure указано состояние **работает** с включенной синхронизацией хэшей паролей.

1.  **Домен: присоединение виртуальной машины Azure к Azure AD DS.**

    Чтобы получить доступ к общей папке с помощью учетных данных Azure AD на виртуальной машине, виртуальная машина должна быть присоединена к домену Azure AD DS. Дополнительные сведения см. в статье [Присоединение виртуальной машины Windows Server к управляемому домену](../../active-directory-domain-services/join-windows-vm.md).

    > [!NOTE]
    > Проверка подлинности Azure AD DS через SMB с помощью файловых ресурсов Azure поддерживается только на виртуальных машинах Azure, работающих в версиях ОС выше Windows 7 или Windows Server 2008 R2.

1.  **Выберите или создайте файловый ресурс Azure**.

    Выберите новый или существующий файловый ресурс, связанный с той же подпиской, что и ваш клиент Azure AD. Дополнительные сведения см. в статье [Создание общей папки в службе файлов Azure](storage-how-to-create-file-share.md).
    Для оптимальной производительности рекомендуется, чтобы общая папка была в том же регионе, что и виртуальная машина, из которой планируется доступ к общей папке.

1.  **Проверьте сетевое подключение службы файлов Azure путем подключения файловых ресурсов Azure с использованием ключа учетной записи хранения.**

    Чтобы убедиться, что ваша виртуальная машина и файловый ресурс настроены правильно, попробуйте установить общий доступ к файлу с помощью ключа учетной записи хранения. Дополнительные сведения см. в статье [Использование общей папки Azure в Windows](storage-how-to-use-files-windows.md).

## <a name="regional-availability"></a>Доступность по регионам

Проверка подлинности файлов Azure с помощью Azure AD DS доступна во [всех регионах Azure Public, gov и Китая](https://azure.microsoft.com/global-infrastructure/locations/).

## <a name="overview-of-the-workflow"></a>Обзор рабочего процесса

Прежде чем включить проверку подлинности Azure AD DS в SMB для файловых ресурсов Azure, убедитесь, что среды Azure AD и службы хранилища Azure настроены правильно. Рекомендуется выполнить все [необходимые условия](#prerequisites) , чтобы убедиться, что выполнены все необходимые действия.

Затем выполните следующие действия, чтобы предоставить доступ к ресурсам службы файлов Azure с помощью учетных данных Azure AD:

1. Включите проверку подлинности Azure AD DS по протоколу SMB, чтобы ваша учетная запись хранения регистрировала учетную запись хранения в связанном развертывании Azure AD DS.
2. Назначьте права доступа к общему ресурсу для удостоверения Azure AD (пользователя, группы или субъекта-службы).
3. Настройте разрешения NTFS по протоколу SMB для каталогов и файлов.
4. Подключите файловый ресурс Azure из виртуальной машины, присоединенной к домену.

На следующей схеме показан комплексный рабочий процесс для включения проверки подлинности Azure AD DS по протоколу SMB для службы файлов Azure.

![Схема, показывающая рабочий процесс включения Azure AD по протоколу SMB для службы файлов Azure](media/storage-files-active-directory-enable/azure-active-directory-over-smb-workflow.png)

## <a name="enable-azure-ad-ds-authentication-for-your-account"></a>Включение аутентификации Azure AD DS для учетной записи

Чтобы включить аутентификацию Azure AD DS по протоколу SMB для файлов Azure, можно задать свойство в учетных записях хранения с помощью портал Azure, Azure PowerShell или Azure CLI. Неявное задание этого свойства: "домен присоединяет" к учетной записи хранения с соответствующим развертыванием AD DS Azure. Затем проверка подлинности Azure AD DS через SMB включается для всех новых и существующих файловых ресурсов в учетной записи хранения.

Помните, что вы можете включить проверку подлинности Azure AD DS по протоколу SMB только после успешного развертывания AD DS Azure в клиенте Azure AD. Дополнительные сведения см. в разделе [Предварительные требования](#prerequisites).

# <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы включить проверку подлинности Azure AD DS в SMB с [портал Azure](https://portal.azure.com), выполните следующие действия.

1. В портал Azure перейдите к существующей учетной записи хранения или [Создайте учетную запись хранения](../common/storage-account-create.md).
1. В разделе **Параметры** выберите **Конфигурация**.
1. В разделе " **доступ на основе удостоверений" для файловых ресурсов** перейдите в режим **Azure Active Directory службы домена (AAD DS)** в значение " **включено**".
1. Нажмите кнопку **Сохранить**.

На следующем рисунке показано, как включить проверку подлинности Azure AD DS по протоколу SMB для вашей учетной записи хранения.

![Включите проверку подлинности Azure AD DS по протоколу SMB в портал Azure](media/storage-files-active-directory-enable/portal-enable-active-directory-over-smb.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Чтобы включить проверку подлинности Azure AD DS в SMB с Azure PowerShell, установите последний модуль AZ (2,4 или более поздней версии) или AZ. Storage Module (1,5 или более поздней версии). Дополнительные сведения об установке PowerShell см. [в статье установка Azure PowerShell в Windows с помощью PowerShellGet](/powershell/azure/install-Az-ps).

Чтобы создать новую учетную запись хранения, вызовите командлет [New-азсторажеаккаунт](/powershell/module/az.storage/New-azStorageAccount?view=azps-2.5.0), а затем установите для параметра **енаблеазуреактиведиректоридомаинсервицесфорфиле** **значение true**. В следующем примере не забудьте заменить значения заполнителей собственными значениями. (Если использовался предыдущий модуль предварительного просмотра, параметр для включения этой функции — **енаблеазурефилесаадинтегратионфорсмб**.)

```powershell
# Create a new storage account
New-AzStorageAccount -ResourceGroupName "<resource-group-name>" `
    -Name "<storage-account-name>" `
    -Location "<azure-region>" `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableAzureActiveDirectoryDomainServicesForFile $true
```

Чтобы включить эту функцию для существующих учетных записей хранения, используйте следующую команду:

```powershell
# Update a storage account
Set-AzStorageAccount -ResourceGroupName "<resource-group-name>" `
    -Name "<storage-account-name>" `
    -EnableAzureActiveDirectoryDomainServicesForFile $true
```


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы включить проверку подлинности Azure AD по протоколу SMB с Azure CLI, установите последнюю версию CLI (2.0.70 или более поздней версии). Дополнительные сведения об установке Azure CLI см. [в разделе установка Azure CLI](/cli/azure/install-azure-cli).

Чтобы создать новую учетную запись хранения, вызовите команду [AZ Storage Account Create](/cli/azure/storage/account#az-storage-account-create)и задайте `--enable-files-aadds` для свойства **значение true**. В следующем примере не забудьте заменить значения заполнителей собственными значениями. (Если использовался предыдущий модуль предварительного просмотра, параметром для включения функции является **File-AAD**.)

```azurecli-interactive
# Create a new storage account
az storage account create -n <storage-account-name> -g <resource-group-name> --enable-files-aadds $true
```

Чтобы включить эту функцию для существующих учетных записей хранения, используйте следующую команду:

```azurecli-interactive
# Update a new storage account
az storage account update -n <storage-account-name> -g <resource-group-name> --enable-files-aadds $true
```
---

[!INCLUDE [storage-files-aad-permissions-and-mounting](../../../includes/storage-files-aad-permissions-and-mounting.md)]

Вы успешно включили проверку подлинности Azure AD DS по протоколу SMB и назначили настраиваемую роль, которая предоставляет доступ к файловому ресурсу Azure с удостоверением Azure AD. Чтобы предоставить дополнительным пользователям доступ к общему файловому ресурсу, следуйте инструкциям в разделе [Назначение прав доступа](#assign-access-permissions-to-an-identity) для использования удостоверения и [Настройка разрешений NTFS в разделах SMB](#configure-ntfs-permissions-over-smb).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о службе файлов Azure и использовании Azure AD через SMB см. в следующих ресурсах:

- [Общие сведения о поддержке проверки подлинности на основе удостоверения для доступа к SMB в Файлах Azure](storage-files-active-directory-overview.md).
- [Часто задаваемые вопросы](storage-files-faq.md)
