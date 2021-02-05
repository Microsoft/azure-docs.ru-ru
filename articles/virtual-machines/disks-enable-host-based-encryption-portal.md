---
title: Включение сквозного шифрования с помощью шифрования на управляемых дисках Host-портал Azure
description: Используйте шифрование на узле, чтобы включить сквозное шифрование на управляемых дисках Azure — портал Azure.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 08/24/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: ba7d6d8deb2034f8b2a853cf74635687561c41ea
ms.sourcegitcommit: 1f1d29378424057338b246af1975643c2875e64d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/05/2021
ms.locfileid: "99573608"
---
# <a name="use-the-azure-portal-to-enable-end-to-end-encryption-using-encryption-at-host"></a>Использование портал Azure для включения сквозного шифрования с помощью шифрования на узле

При включении шифрования на узле данные, хранящиеся на узле виртуальной машины, шифруются при хранении и передаются в зашифрованном виде в службу хранилища. Основные сведения о шифровании на узле, а также о других типах шифрования управляемых дисков см. в следующих статьях:

* Linux: [Шифрование на уровне узла — сквозное шифрование данных виртуальной машины](./disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

* Windows: [Шифрование на уровне узла — сквозное шифрование данных виртуальной машины](./disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

## <a name="restrictions"></a>Ограничения

[!INCLUDE [virtual-machines-disks-encryption-at-host-restrictions](../../includes/virtual-machines-disks-encryption-at-host-restrictions.md)]

### <a name="supported-regions"></a>Поддерживаемые регионы

[!INCLUDE [virtual-machines-disks-encryption-at-host-regions](../../includes/virtual-machines-disks-encryption-at-host-regions.md)]

### <a name="supported-vm-sizes"></a>Поддерживаемые размеры виртуальных машин

[!INCLUDE [virtual-machines-disks-encryption-at-host-suported-sizes](../../includes/virtual-machines-disks-encryption-at-host-suported-sizes.md)]

## <a name="prerequisites"></a>Предварительные требования

Чтобы иметь возможность использовать шифрование на узле для виртуальных машин или масштабируемых наборов виртуальных машин, необходимо включить эту функцию в подписке. Отправьте сообщение электронной почты encryptionAtHost@microsoft.com с идентификаторами подписки, чтобы включить функцию для ваших подписок.

Войдите в портал Azure с помощью [предоставленной ссылки](https://aka.ms/diskencryptionupdates).

> [!IMPORTANT]
> Для доступа к портал Azure необходимо использовать [предоставленную ссылку](https://aka.ms/diskencryptionupdates) . Шифрование на узле в настоящее время не отображается в общедоступной портал Azure без использования ссылки.

### <a name="create-an-azure-key-vault-and-disk-encryption-set"></a>Создание Azure Key Vault и набора шифрования дисков

После включения этой функции необходимо настроить Azure Key Vault и набор шифрования дисков, если это еще не сделано.

[!INCLUDE [virtual-machines-disks-encryption-create-key-vault-portal](../../includes/virtual-machines-disks-encryption-create-key-vault-portal.md)]

## <a name="deploy-a-vm"></a>Развертывание виртуальной машины

Чтобы включить шифрование на узле, необходимо развернуть новую виртуальную машину. она не может быть включена на существующих виртуальных машинах.

1. Выполните поиск **виртуальных машин** и выберите **+ Добавить** , чтобы создать виртуальную машину.
1. Создайте новую виртуальную машину, выберите соответствующий регион и поддерживаемый размер виртуальной машины.
1. Заполните остальные значения в колонке **Основная** , а затем перейдите к колонке **диски** .

    :::image type="content" source="media/virtual-machines-disks-encryption-at-host-portal/disks-encryption-at-host-basic-blade.png" alt-text="Снимок экрана: колонка &quot;основы создания виртуальной машины&quot;, &quot;регион&quot; и &quot;размер V M&quot; выделены.":::

1. В колонке **диски** выберите **Да** для **шифрования на узле**.
1. Сделайте оставшиеся варианты выбора нужным образом.

    :::image type="content" source="media/virtual-machines-disks-encryption-at-host-portal/disks-encryption-at-host-disk-blade.png" alt-text="Снимок экрана: колонка &quot;диски создания виртуальной машины&quot;, &quot;шифрование на узле&quot; выделена.":::

1. Завершите процесс развертывания виртуальной машины, чтобы выбрать параметры, соответствующие вашей среде.

Теперь вы развернули виртуальную машину с включенным шифрованием на узле. все связанные с ним диски будут зашифрованы с помощью шифрования на узле.

## <a name="next-steps"></a>Следующие шаги

[Примеры шаблонов Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)
