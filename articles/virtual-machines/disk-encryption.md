---
title: Шифрование управляемых дисков Azure на стороне сервера
description: Служба хранилища Azure защищает ваши данные путем шифрования неактивных данных перед их сохранением в кластерах хранилища. Вы можете использовать ключи, управляемые клиентом, для управления шифрованием с помощью собственных ключей, или же вы можете полагаться на ключи, управляемые корпорацией Майкрософт, для шифрования управляемых дисков.
author: roygara
ms.date: 03/02/2021
ms.topic: conceptual
ms.author: rogarana
ms.service: virtual-machines
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: a1fbd536943023d3e6724b9c1638f7a0bd97d847
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102036951"
---
# <a name="server-side-encryption-of-azure-disk-storage"></a>Шифрование Хранилище дисков Azure на стороне сервера

Шифрование на стороне сервера (SSE) защищает данные и помогает соблюсти корпоративные обязательства по обеспечению безопасности и соответствия. SSE автоматически шифрует данные, хранящиеся на управляемых дисках Azure (диски операционной системы и дисков данных), по умолчанию при сохранении их в облаке. 

Данные в управляемых дисках Azure шифруются прозрачно с использованием 256-разрядного [шифрования AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), одного из наиболее сильных блочных шифров, который совместим со стандартом FIPS 140-2. Дополнительные сведения о криптографических модулях, лежащих в основе функции управляемых дисков Azure, см. в статье [Cryptography API: Next Generation](/windows/desktop/seccng/cng-portal) (API Cryptography: следующее поколение).

Шифрование на стороне сервера не влияет на производительность управляемых дисков и не дает дополнительных затрат. 

> [!NOTE]
> Временные диски не являются управляемыми дисками и не шифруются с помощью SSE, если только не включено шифрование на узле.

## <a name="about-encryption-key-management"></a>Об управлении ключами шифрования

Вы можете использовать ключи, управляемые платформой, для шифрования управляемого диска или управлять шифрованием с помощью собственных ключей. Если вы решили управлять шифрованием с помощью собственных ключей, вы можете указать *управляемый клиентом ключ*, который будет использоваться для шифрования и расшифровки всех данных на управляемых дисках. 

В следующих разделах подробно описаны все варианты управления ключами.

### <a name="platform-managed-keys"></a>Ключи, управляемые платформой

По умолчанию управляемые диски используют управляемые платформой ключи шифрования. Все управляемые диски, моментальные снимки, изображения и данные, записанные на существующие управляемые диски, автоматически шифруются с помощью ключей, управляемых платформой.

### <a name="customer-managed-keys"></a>Ключи, управляемые клиентом

[!INCLUDE [virtual-machines-managed-disks-description-customer-managed-keys](../../includes/virtual-machines-managed-disks-description-customer-managed-keys.md)]

#### <a name="restrictions"></a>Ограничения

В настоящее время ключи, управляемые клиентом, имеют следующие ограничения.

- Если эта функция включена для диска, ее нельзя отключить.
    Если необходимо обойти эту ошибку, необходимо скопировать все данные с помощью [Azure PowerShell модуля](windows/disks-upload-vhd-to-managed-disk-powershell.md#copy-a-managed-disk) или [Azure CLI](linux/disks-upload-vhd-to-managed-disk-cli.md#copy-a-managed-disk), на совершенно другой управляемый диск, не использующий ключи, управляемые клиентом.
[!INCLUDE [virtual-machines-managed-disks-customer-managed-keys-restrictions](../../includes/virtual-machines-managed-disks-customer-managed-keys-restrictions.md)]

#### <a name="supported-regions"></a>Поддерживаемые регионы

Ключи, управляемые клиентом, доступны во всех регионах, где доступны управляемые диски.

Автоматическое вращение ключей находится на этапе предварительной версии и доступно только в следующих регионах:

- Восточная часть США
- восточная часть США 2
- Центрально-южная часть США
- западная часть США
- западная часть США 2
- Северная Европа
- Западная Европа
- Центральная Франция

> [!IMPORTANT]
> Управляемые клиентом ключи используют управляемые удостоверения для ресурсов Azure — функцию Azure Active Directory (Azure AD). При настройке управляемых пользователем ключей управляемое удостоверение автоматически назначается вашим ресурсам. Если впоследствии вы перемещаете подписку, группу ресурсов или управляемый диск из одного каталога Azure AD в другой, управляемое удостоверение, связанное с управляемыми дисками, не передается новому клиенту, поэтому управляемые клиентом ключи могут перестать работать. Дополнительные сведения см. в статье [Передача подписки между каталогами Azure AD](../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

Сведения о том, как включить управляемые пользователем ключи для управляемых дисков, см. в статьях, посвященных их включению с помощью [модуля Azure PowerShell](windows/disks-enable-customer-managed-keys-powershell.md), [Azure CLI](linux/disks-enable-customer-managed-keys-cli.md) или [портал Azure](disks-enable-customer-managed-keys-portal.md). Сведения о том, как включить управляемые клиентом ключи с помощью автоматического смены ключей, см. в разделе [настройка Azure Key Vault и дискенкриптионсет с автоматическим сменой ключей (Предварительная версия)](windows/disks-enable-customer-managed-keys-powershell.md#set-up-an-azure-key-vault-and-diskencryptionset-with-automatic-key-rotation-preview).

## <a name="encryption-at-host---end-to-end-encryption-for-your-vm-data"></a>Шифрование на основе сквозного шифрования данных виртуальной машины

При включении шифрования на узле это шифрование начинается на самом узле виртуальной машины, на сервере Azure, на котором размещена виртуальная машина. Данные для временных дисковых и дисковых кэшей операционной системы и данных хранятся на этом узле виртуальной машины. После включения шифрования на узле все эти данные шифруются в неактивных потоках, а потоки шифруются в службе хранилища, где они сохраняются. По сути, шифрование на узле шифрует данные от начала до конца. Шифрование на узле не использует ЦП виртуальной машины и не влияет на производительность виртуальной машины. 

При включении сквозного шифрования временные диски и эфемерные диски ОС шифруются при хранении с помощью ключей, управляемых платформой. Кэш операционной системы и диска данных шифруется при хранении с помощью управляемых клиентом или управляемых платформой ключей в зависимости от выбранного типа шифрования диска. Например, если диск зашифрован ключами, управляемыми клиентом, то кэш для диска шифруется с помощью ключей, управляемых клиентом, а если диск шифруется с помощью ключей, управляемых платформой, кэш диска шифруется с помощью ключей, управляемых платформой.

### <a name="restrictions"></a>Ограничения

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

#### <a name="supported-regions"></a>Поддерживаемые регионы

[!INCLUDE [virtual-machines-disks-encryption-at-host-regions](../../includes/virtual-machines-disks-encryption-at-host-regions.md)]

#### <a name="supported-vm-sizes"></a>Поддерживаемые размеры виртуальных машин

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

Чтобы включить сквозное шифрование с помощью шифрования на узле, ознакомьтесь с нашими статьями о том, как включить [модуль Azure PowerShell](windows/disks-enable-host-based-encryption-powershell.md), [Azure CLI](linux/disks-enable-host-based-encryption-cli.md)или [портал Azure](disks-enable-host-based-encryption-portal.md).

## <a name="double-encryption-at-rest"></a>Двойное шифрование при хранении

Клиенты с высоким уровнем безопасности, которые связаны с риском, связанным с любым конкретным алгоритмом шифрования, реализацией или скомпрометированным ключом, теперь могут использовать дополнительный уровень шифрования с использованием другого алгоритма или режима шифрования на уровне инфраструктуры, используя ключи шифрования, управляемые платформой. Этот новый слой можно применить к материализованным дискам ОС и данным, моментальным снимкам и изображениям, которые будут зашифрованы при хранении с помощью двойного шифрования.

### <a name="supported-regions"></a>Поддерживаемые регионы

Двойное шифрование доступно во всех регионах, где доступны управляемые диски.

Сведения о том, как включить двойное шифрование для управляемых дисков, см. в статьях, посвященных их включению с помощью [модуля Azure PowerShell](windows/disks-enable-double-encryption-at-rest-powershell.md), [Azure CLI](linux/disks-enable-double-encryption-at-rest-cli.md) или [портал Azure](disks-enable-double-encryption-at-rest-portal.md).

## <a name="server-side-encryption-versus-azure-disk-encryption"></a>Шифрование на стороне сервера и шифрование дисков Azure

[Шифрование дисков Azure](../security/fundamentals/azure-disk-encryption-vms-vmss.md) использует либо функцию " [DM-](https://en.wikipedia.org/wiki/Dm-crypt) Encryption" в Linux, либо функцию [BitLocker](/windows/security/information-protection/bitlocker/bitlocker-overview) в Windows для шифрования управляемых дисков с помощью управляемых клиентом ключей в гостевой виртуальной машине.  Шифрование на стороне сервера с использованием управляемых клиентом ключей улучшает работу шифрования дисков Azure, позволяя использовать любые типы ОС и образы для виртуальных машин путем шифрования данных в службе хранилища.
> [!IMPORTANT]
> Управляемые клиентом ключи используют управляемые удостоверения для ресурсов Azure — функцию Azure Active Directory (Azure AD). При настройке управляемых пользователем ключей управляемое удостоверение автоматически назначается вашим ресурсам. При перемещении подписки, группы ресурсов или управляемого диска из одного каталога Azure AD в другой управляемое удостоверение, связанное с управляемыми дисками, не передается в новый клиент, поэтому управляемые клиентом ключи могут перестать работать. Дополнительные сведения см. в статье [Передача подписки между каталогами Azure AD](../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).

## <a name="next-steps"></a>Дальнейшие действия

- Включите сквозное шифрование с помощью шифрования на узле, используя [модуль Azure PowerShell](windows/disks-enable-host-based-encryption-powershell.md), [Azure CLI](linux/disks-enable-host-based-encryption-cli.md)или [портал Azure](disks-enable-host-based-encryption-portal.md).
- Включите двойное шифрование при неактивных дисках для управляемых дисков с помощью [модуля Azure PowerShell](windows/disks-enable-double-encryption-at-rest-powershell.md), [Azure CLI](linux/disks-enable-double-encryption-at-rest-cli.md) или [портал Azure](disks-enable-double-encryption-at-rest-portal.md).
- Включите управляемые пользователем ключи для управляемых дисков с помощью [модуля Azure PowerShell](windows/disks-enable-customer-managed-keys-powershell.md), [Azure CLI](linux/disks-enable-customer-managed-keys-cli.md) или [портал Azure](disks-enable-customer-managed-keys-portal.md).
- [Изучите шаблоны Azure Resource Manager для создания зашифрованных дисков с помощью управляемых клиентом ключей](https://github.com/ramankumarlive/manageddiskscmkpreview)
- [Что такое хранилище ключей Azure?](../key-vault/general/overview.md)