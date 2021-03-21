---
title: Интеграция Azure Key Vault с Azure Stackной пограничным ресурсом и активацией устройства
description: Описывает, как Azure Key Vault связана с управлением секретами во время активации Azure Stack устройства Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: c18f875dde43ab6df512375152d76a31c30a9e77
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102439873"
---
# <a name="azure-key-vault-integration-with-azure-stack-edge"></a>Интеграция Azure Key Vault с Azure Stackным ребром 

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Azure Key Vault интегрирована с Azure Stackным ресурсом периметра для управления секретами. В этой статье содержатся сведения о том, как создается Azure Key Vault для ресурса Azure Stackного периметра при активации устройства, а затем используется для управления секретами. 


## <a name="about-key-vault-and-azure-stack-edge"></a>Сведения о хранилище ключей и Azure Stack границе

Azure Key Vault облачная служба используется для безопасного хранения и управления доступом к маркерам, паролям, сертификатам, ключам API и другим секретам. Key Vault также упрощает создание и управление ключами шифрования, используемыми для шифрования данных. Дополнительные сведения о разрешенных транзакциях и соответствующих расходах см. в разделе [цены на Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/).

Для службы Azure Stack ребра используется один из секретов — ключ целостности канала (ЦИК). Этот ключ позволяет шифровать секреты. С интеграцией хранилища ключей ЦИК безопасно хранится в хранилище ключей. Дополнительные сведения см. в статье [безопасное хранение секретов и ключей](../key-vault/general/overview.md#securely-store-secrets-and-keys).


## <a name="key-vault-creation"></a>Создание хранилища ключей

Хранилище ключей создается для ресурса Azure Stack ребра в процессе создания ключа активации. 

- При создании Azure Stack пограничных ресурсов необходимо зарегистрировать поставщик ресурсов *Microsoft. KeyVault* . Поставщик ресурсов регистрируется автоматически, если у вас есть права владельца или участника подписки. Хранилище ключей создается в той же подписке и группе ресурсов, что и ресурс Azure Stackного периметра. 

- При создании ресурса Azure Stack ребра также создается Управляемое удостоверение службы (MSI), сохраняющаяся в течение времени существования ресурса и взаимодействующая с поставщиком ресурсов в облаке. 

    Когда MSI-файл включен, Azure создает доверенное удостоверение для ресурса Azure Stack ребра.

- После создания ресурса Azure Stack ребра и создания ключа активации из портал Azure создается хранилище ключей. Это хранилище ключей используется для управления секретами и сохраняется до тех пор, пока существует ресурс Azure Stack ребра. 

    ![Key Vault, созданные при создании ключа активации](media/azure-stack-edge-gpu-deploy-prep/azure-stack-edge-resource-3.png)

- Вы можете принять имя ключа по умолчанию или указать пользовательское имя для хранилища ключей. Имя хранилища ключей должно иметь длину от 3 до 24 символов. Вы не можете использовать хранилище ключей, которое уже используется. <!--The MSI is then used to authenticate to key vault to retrieve secrets.--> 

    ![MSI создан во время создания ресурса Azure Stack ребра](media/azure-stack-edge-gpu-deploy-prep/create-resource-8.png)

- Чтобы перейти к хранилищу ключей Azure, перейдите к **свойствам** в ресурсе Azure Stack ребра и выберите имя хранилища ключей. 

- Во избежание случайного удаления блокировка ресурса включается в хранилище ключей. Обратимое удаление также включено в хранилище ключей, которое позволяет восстановить хранилище ключей в течение 90 дней при случайном удалении. Дополнительные сведения см. в статье [обзор Azure Key Vault обратимого удаления](../key-vault/general/soft-delete-overview.md) .

    Если хранилище ключей случайно удалено, а срок действия защиты от очистки в 90 дней не истек, выполните следующие действия для [восстановления хранилища ключей](../key-vault/general/key-vault-recovery.md#list-recover-or-purge-soft-deleted-secrets-keys-and-certificates). 

- Если у вас уже есть ресурс Azure Stack ребра до того, как Azure Key Vault был интегрирован с Azure Stackным ресурсом, это не повлияет. Вы можете продолжить использовать существующий ресурс Azure Stack ребра. 

- При удалении ресурса Azure Stack ребра Azure Key Vault также удаляется вместе с ресурсом. Появляется сообщение с предложением подтвердить операцию. Если вы не планируете удалять это хранилище ключей, можно выбрать не предоставлять согласие. Удаляется только ресурс Azure Stack ребра, при этом хранилище ключей остается без изменений. 

- Если это хранилище ключей использовалось для хранения других ключей, вы по-прежнему можете восстановить его в течение 90 дней после удаления. В течение периода бездействия очистки имя хранилища ключей нельзя использовать для создания нового хранилища ключей.

Если возникли проблемы, связанные с хранилищем ключей и активацией устройств, см. статью [Устранение неполадок при активации устройств](azure-stack-edge-gpu-troubleshoot-activation.md).

<!--## Key vault secret management

When you generate an activation key, the following events occur:

1. You request an activation key in the Azure portal. The request is then sent to Key Vault resource provider. 
1. A standard tier key vault with access policy is created and is locked by default. This key vault uses the default name or the custom name that you specified.
1. The key vault authenticates with MSI the request to generate activation key. The MSI is also added to the key vault access policy and a channel integrity key is generated and placed in the key vault.
1. The activation key is returned to the Azure portal. You can then copy this key and use it in the local UI to activate your device.-->



## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [создании ключа активации](azure-stack-edge-gpu-deploy-prep.md#get-the-activation-key).