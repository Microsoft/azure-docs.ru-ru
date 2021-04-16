---
title: Краткое руководство Azure. Создание управляемого устройства HSM с помощью шаблона Azure Resource Manager
description: Краткое руководство по созданию управляемого устройства HSM в Azure Key Vault с помощью шаблона Resource Manager
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.custom: mvc
ms.date: 09/15/2020
ms.author: mbaldwin
ms.openlocfilehash: b2cd8578599fe1e80044f1eb388cddae7a292166
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107376874"
---
# <a name="quickstart-create-an-key-vault-managed-hsm-using-an-azure-resource-manager-template"></a>Краткое руководство. Создание управляемого устройства HSM в Key Vault с помощью шаблона Azure Resource Manager

Управляемое устройство HSM — это полностью управляемая, высокодоступная, однотенантная, соответствующая стандартам облачная служба, которая позволяет защищать криптографические ключи для облачных приложений, используя устройства HSM, отвечающие стандартам **FIPS 140-2 уровня 3**.  

В этом кратком руководстве рассматривается процесс развертывания шаблона Resource Manager для создания управляемого устройства HSM.  [Шаблон Resource Manager](../../azure-resource-manager/templates/overview.md) является файлом нотации объектов JavaScript (JSON), определяющими инфраструктуру и конфигурацию вашего проекта. Шаблон использует декларативный синтаксис, который позволяет указать объект, который вы собираетесь развернуть. При этом, для развертывания объекта, не нужно писать последовательность команд. Дополнительные сведения о разработке шаблонов Resource Manager см. в разделе [Документация по Azure Resource Manager](../../azure-resource-manager/index.yml) и в [справочнике по шаблонам](/azure/templates/microsoft.keyvault/allversions).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этой статье, вам потребуется следующее:

- подписка на Microsoft Azure. Если у вас ее нет, зарегистрируйтесь, чтобы воспользоваться [бесплатной пробной версией](https://azure.microsoft.com/pricing/free-trial).
- Azure CLI 2.12.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Сведения об установке или обновлении Azure CLI см. в [этой статье]( /cli/azure/install-azure-cli).


[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Вход в Azure

Чтобы войти в Azure с помощью CLI, введите следующее:

```azurecli
az login
```

Дополнительные сведения о вариантах входа с помощью Azure CLI см. в [этой статье](/cli/azure/authenticate-azure-cli).

## <a name="create-a-manage-hsm"></a>Создание управляемого устройства HSM

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/101-managed-hsm-create/).

Ресурс Azure, определенный в этом шаблоне:

* **Microsoft.KeyVault/managedHSMs**: создание управляемого устройства HSM в Azure Key Vault.

Другие примеры шаблонов хранилища Azure Key Vault можно найти [здесь](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Keyvault).

Для шаблона требуется идентификатор объекта, связанный с вашей учетной записью. Чтобы найти его, используйте команду Azure CLI [az ad user show](/cli/azure/ad/user#az_ad_user_show) для передачи адреса электронной почты в параметр `--id`. Вы можете ограничить выходные данные с помощью параметра `--query`, оставив только идентификатор объекта.

```azurecli-interactive
az ad user show --id <your-email-address> --query "objectId"
```

Вам также может потребоваться идентификатор клиента. Чтобы найти его, используйте команду Azure CLI [az ad user show](/cli/azure/account#az_account_show). Вы можете ограничить выходные данные с помощью параметра `--query`, оставив только идентификатор клиента.

 ```azurecli-interactive
 az account show --query "tenantId"
 ```

1. Выберите следующее изображение, чтобы войти на портал Azure и открыть шаблон. Шаблон создает хранилище ключей и секрет.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-managed-hsm-create%2Fazuredeploy.json"><img src="../media/deploy-to-azure.svg" alt="deploy to azure"/></a>

2. Введите или выберите следующие значения.

    Если значение не указано, используйте для создания хранилища ключей и секрета значение по умолчанию.

    - **Подписка**: Выберите подписку Azure.
    - **Группа ресурсов.** Щелкните **Создать**, введите уникальное имя группы ресурсов и нажмите кнопку **ОК**.
    - **Расположение.** Выберите расположение. Например, **центрально-южная часть США**.
    - **managedHSMName**: введите имя управляемого устройства HSM.
    - **Идентификатор арендатора**: функция шаблона автоматически получает идентификатор клиента. Не изменяйте значение по умолчанию.  Если значение отсутствует, введите идентификатор клиента, полученный в разделе [Необходимые компоненты](#prerequisites).
    * **initialAdminObjectIds**: введите идентификатор объекта, полученный в разделе [Необходимые компоненты](#prerequisites).

3. Щелкните **Приобрести**. После успешного развертывания хранилища ключей вы получите уведомление:

Для развертывания шаблона используется портал Azure. В дополнение к порталу Azure можно также использовать Azure PowerShell, Azure CLI и REST API. Дополнительные сведения о других методах развертывания см. в статье о [развертывании с использованием шаблонов](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="next-steps"></a>Дальнейшие действия

Используя сведения этого краткого руководства, вы создали управляемое устройство HSM. Возможности этого устройства будут ограничены до его активации. Сведения о том, как активировать управляемое устройство HSM, см. в [этом разделе](quick-create-cli.md#activate-your-managed-hsm).

- Ознакомьтесь с [обзором управляемого устройства HSM](overview.md).
- Просмотрите дополнительные сведения об [управлении ключами в управляемом устройстве HSM](key-management.md).
- Изучите [рекомендации по использованию управляемого устройства HSM](best-practices.md).
