---
title: Краткое руководство. Создание Azure Key Vault с помощью Azure CLI
description: Краткое руководство по созданию Azure Key Vault с помощью Azure CLI
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: quickstart
ms.date: 01/27/2021
ms.author: mbaldwin
ms.openlocfilehash: f7f6f5d82c5fda7101e80ddcb8b17dc6bdef6532
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99070296"
---
# <a name="quickstart-create-a-key-vault-using-the-azure-cli"></a>Краткое руководство. Создание хранилища ключей с помощью Azure CLI

Azure Key Vault — это облачная служба, которая предоставляет защищенное хранилище для [ключей](../keys/index.yml), [секретов](../secrets/index.yml) и [сертификатов](../certificates/index.yml). Дополнительные сведения о Key Vault см. в [этой статье](overview.md), а сведения о том, что можно хранить в хранилище ключей, см. [здесь](about-keys-secrets-certificates.md).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим кратким руководством требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create a resource group](../../../includes/key-vault-cli-rg-creation.md)]

## <a name="create-a-key-vault"></a>Создание хранилища ключей

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-kv-creation.md)]

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [Create a key vault](../../../includes/key-vault-cli-delete-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы создали Key Vault и удалили его. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](overview.md)
- Статья [Обзор системы безопасности Azure Key Vault](security-overview.md)
- Справочник по [командам az keyvault для Azure CLI](/cli/azure/keyvault)

