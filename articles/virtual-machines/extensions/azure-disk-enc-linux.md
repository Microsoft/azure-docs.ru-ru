---
title: Шифрование дисков Azure для Linux
description: Развертывание шифрования дисков Azure на виртуальной машине Linux с помощью расширения виртуальной машины.
ms.topic: article
ms.service: virtual-machines
ms.subservice: disks
author: ejarvi
ms.author: ejarvi
ms.date: 03/19/2020
ms.collection: linux
ms.openlocfilehash: 7c79391e3459804a4b5ce72c2230d17af3269641
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102566266"
---
# <a name="azure-disk-encryption-for-linux-microsoftazuresecurityazurediskencryptionforlinux"></a>Шифрование дисков Azure для Linux (Microsoft.Azure.Security.AzureDiskEncryptionForLinux)

## <a name="overview"></a>Обзор

Шифрование дисков Azure использует подсистему dm-crypt в Linux для шифрования всего диска в [выбранных дистрибутивах Azure Linux](../linux/disk-encryption-overview.md).  Решение интегрируется с Azure Key Vault, обеспечивая управление секретами и ключами шифрования диска.

## <a name="prerequisites"></a>Предварительные условия

Полный список необходимых компонентов см. в разделах [Шифрование дисков Azure для виртуальных машин Linux](../linux/disk-encryption-overview.md), в частности следующие разделы:

- [Поддерживаемые виртуальные машины и операционные системы](../linux/disk-encryption-overview.md#supported-vms-and-operating-systems)
- [Дополнительные требования к виртуальным машинам](../linux/disk-encryption-overview.md#additional-vm-requirements)
- [Требования к сети](../linux/disk-encryption-overview.md#networking-requirements)
- [Требования к хранилищу ключей шифрования](../linux/disk-encryption-overview.md#encryption-key-storage-requirements)

## <a name="extension-schema"></a>Схема расширения

Существует две версии схемы расширения для шифрования дисков Azure (ADE):
- v 1.1 — Новая Рекомендуемая схема, которая не использует свойства Azure Active Directory (AAD).
- v 0,1 — более старая схема, для которой требуются свойства Azure Active Directory (AAD). 

Чтобы выбрать целевую схему, `typeHandlerVersion` свойство должно иметь значение, равное версии схемы, которую вы хотите использовать.

### <a name="schema-v11-no-aad-recommended"></a>Схема v 1.1: нет AAD (рекомендуется)

Рекомендуется использовать схему v 1.1 и не требует свойств Azure Active Directory (AAD).

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
        "publisher": "Microsoft.Azure.Security",
        "type": "AzureDiskEncryptionForLinux",
        "typeHandlerVersion": "1.1",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "DiskFormatQuery": "[diskFormatQuery]",
          "EncryptionOperation": "[encryptionOperation]",
          "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
          "KeyVaultURL": "[keyVaultURL]",
          "KeyVaultResourceId": "[KeyVaultResourceId]",
          "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
          "KekVaultResourceId": "[KekVaultResourceId",
          "SequenceVersion": "sequenceVersion]",
          "VolumeType": "[volumeType]"
        }
  }
}
```


### <a name="schema-v01-with-aad"></a>Схема v 0,1: с AAD 

Для схемы 0,1 требуется значение `AADClientID` и `AADClientSecret` `AADClientCertificate` .

Использование среды `AADClientSecret`:

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientSecret": "[aadClientSecret]",
      "Passphrase": "[passphrase]"
    },
    "publisher": "Microsoft.Azure.Security",
    "type": "AzureDiskEncryptionForLinux",
    "typeHandlerVersion": "0.1",
    "settings": {
      "AADClientID": "[aadClientID]",
      "DiskFormatQuery": "[diskFormatQuery]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    }
  }
}
```

Использование среды `AADClientCertificate`:

```json
{
  "type": "extensions",
  "name": "[name]",
  "apiVersion": "2019-07-01",
  "location": "[location]",
  "properties": {
    "protectedSettings": {
      "AADClientCertificate": "[aadClientCertificate]",
      "Passphrase": "[passphrase]"
    },
    "publisher": "Microsoft.Azure.Security",
    "type": "AzureDiskEncryptionForLinux",
    "typeHandlerVersion": "0.1",
    "settings": {
      "AADClientID": "[aadClientID]",
      "DiskFormatQuery": "[diskFormatQuery]",
      "EncryptionOperation": "[encryptionOperation]",
      "KeyEncryptionAlgorithm": "[keyEncryptionAlgorithm]",
      "KeyEncryptionKeyURL": "[keyEncryptionKeyURL]",
      "KeyVaultURL": "[keyVaultURL]",
      "SequenceVersion": "sequenceVersion]",
      "VolumeType": "[volumeType]"
    }
  }
}
```


### <a name="property-values"></a>Значения свойств

| Имя | Значение и пример | Тип данных |
| ---- | ---- | ---- |
| версия_API | 07.01.2019 | Дата |
| publisher | Microsoft.Azure.Security | строка |
| type | AzureDiskEncryptionForLinux | строка |
| typeHandlerVersion | 1,1, 0,1 | INT |
| (схема 0,1) аадклиентид | xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx | guid | 
| (схема 0,1) AADClientSecret | password | строка |
| (схема 0,1) аадклиентцертификате | thumbprint | строка |
| используемых (схема 0,1) Кодов | password | строка |
| DiskFormatQuery | {"dev_path":"","name":"","file_system":""} | Словарь JSON |
| EncryptionOperation | EnableEncryption, EnableEncryptionFormatAll | строка | 
| (необязательно-по умолчанию RSA-OAEP) кэйенкриптионалгорисм | 'RSA-OAEP', 'RSA-OAEP-256', 'RSA1_5' | строка |
| KeyVaultURL | url | строка |
| кэйваултресаурцеид | url | строка |
| используемых KeyEncryptionKeyURL | url | строка |
| используемых кекваултресаурцеид | url | строка |
| используемых секуенцеверсион | UNIQUEIDENTIFIER | строка |
| VolumeType | ОС, данные, все | строка |

## <a name="template-deployment"></a>Развертывание шаблона

Пример развертывания шаблона на основе схемы v 1.1 см. в шаблоне быстрого [запуска Azure 201-Encrypt-выполняющемуся-Linux-VM-без-AAD](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm-without-aad).

Пример развертывания шаблона на основе схемы версии 0,1 см. в шаблоне быстрого [запуска Azure 201-Encrypting-Linux-VM](https://github.com/Azure/azure-quickstart-templates/tree/master/201-encrypt-running-linux-vm).

>[!WARNING]
> - Если вы уже использовали службу "Шифрование дисков Azure" c Azure AD для шифрования виртуальной машины, применяйте этот способ шифрования виртуальной машины и далее.
> - При шифровании томов Linux ОС виртуальная машина должна считаться недоступной. Настоятельно рекомендуется избегать входа по протоколу SSH во время шифрования, чтобы избежать проблем с блокированием открытых файлов, к которым необходимо получить доступ во время процесса шифрования. Чтобы проверить ход выполнения, используйте командлет PowerShell [Get-азвмдискенкриптионстатус](/powershell/module/az.compute/get-azvmdiskencryptionstatus) или команду CLI для [шифрования виртуальных машин](/cli/azure/vm/encryption#az-vm-encryption-show) . Этот процесс может занять несколько часов для тома операционной системы 30 ГБ, а также дополнительное время для шифрования томов данных. Если не используется параметр шифрования всех форматов, время шифрования тома данных будет соответствовать размеру и количеству томов данных. 
> - Отключение шифрования на виртуальных машинах Linux поддерживается только для томов данных. Если том операционной системы зашифрован, то отключение не поддерживается для томов данных и томов ОС. 

>[!NOTE]
> Кроме того `VolumeType` , если параметр имеет значение ALL, диски данных будут шифроваться только в том случае, если они правильно подключены.

## <a name="troubleshoot-and-support"></a>Устранение неполадок и поддержка

### <a name="troubleshoot"></a>Диагностика

Дополнительные сведения см. в [руководстве по устранению неполадок с шифрованием дисков Azure](../linux/disk-encryption-troubleshooting.md).

### <a name="support"></a>Поддержка

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/community/). 

Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите в [службу поддержки Azure](https://azure.microsoft.com/support/options/) и выберите получить поддержку. Сведения об использовании службы поддержки Azure см. в статье [часто задаваемые вопросы о поддержке Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о расширениях виртуальных машин см. в обзоре [расширений и компонентов виртуальной машины для Linux](features-linux.md).
* Дополнительные сведения о шифровании дисков Azure для Linux см. в статье [виртуальные машины Linux](../../security/fundamentals/azure-disk-encryption-vms-vmss.md#linux-virtual-machines).