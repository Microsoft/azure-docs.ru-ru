---
title: Краткое руководство. Настройка и получение секрета из Azure Key Vault
description: Краткое руководство по настройке и получению секрета из Azure Key Vault с помощью Azure CLI
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc, seo-javascript-september2019, seo-javascript-october2019, devx-track-azurecli
ms.date: 09/03/2019
ms.author: mbaldwin
ms.openlocfilehash: 01741540d94d906fc6d41693392f48c17dbae5ce
ms.sourcegitcommit: 2aa52d30e7b733616d6d92633436e499fbe8b069
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/06/2021
ms.locfileid: "97934821"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-azure-cli"></a>Краткое руководство. Настройка и получение секрета из Azure Key Vault с помощью Azure CLI

Из этого краткого руководства вы узнаете, как создать хранилище ключей в Azure Key Vault с помощью Azure CLI. Azure Key Vault — это облачная служба, которая работает как защищенное хранилище секретов. Вы можете безопасно хранить ключи, пароли, сертификаты и другие секреты. Дополнительные сведения о хранилище ключей см. в статье [обзора](../general/overview.md). Azure CLI используется для создания ресурсов Azure и управления ими с помощью скриптов и команд. После этого вы сохраните в нем секрет.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

 - Для работы с этим кратким руководством требуется Azure CLI версии 2.0.4 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="create-a-resource-group"></a>Создание группы ресурсов

Группа ресурсов — это логический контейнер, в котором происходит развертывание ресурсов Azure и управление ими. В следующем примере создается группа ресурсов с именем *ContosoResourceGroup* в расположении *eastus*.

```azurecli
az group create --name "ContosoResourceGroup" --location eastus
```

## <a name="create-a-key-vault"></a>Создание хранилища ключей

Затем вы создадите Key Vault в группе ресурсов, созданной на предыдущем шаге. Необходимо будет указать следующие сведения:

- В этом руководстве мы используем **Contoso-vault2**. Укажите уникальное имя в тестировании.
- Имя группы ресурсов: **ContosoResourceGroup**.
- Расположение: **восточная часть США**.

```azurecli
az keyvault create --name "Contoso-Vault2" --resource-group "ContosoResourceGroup" --location eastus
```

В выходных данных командлета будут показаны свойства созданного Key Vault. Запишите значения двух указанных ниже свойств.

- **Имя хранилища.** В нашем примере это **Contoso-Vault2**. Вы будете использовать это имя для выполнения других команд Key Vault.
- **URI хранилища**. В нашем примере это https://contoso-vault2.vault.azure.net/. Необходимо, чтобы приложения, использующие ваше хранилище через REST API, использовали этот URI.

На этом этапе любые операции в этом хранилище ключей может выполнять только учетная запись Azure.

## <a name="add-a-secret-to-key-vault"></a>Добавление секрета в Key Vault

Чтобы добавить секрет в хранилище, вам просто нужно выполнить несколько дополнительных шагов. Этот пароль может использоваться приложением. Пароль будет называться **ExamplePassword** и в нем будет храниться значение **hVFkk965BuUv**.

Затем введите следующие команды, чтобы создать секрет в Key Vault с именем **ExamplePassword** и значением **hVFkk965BuUv**.

```azurecli
az keyvault secret set --vault-name "Contoso-Vault2" --name "ExamplePassword" --value "hVFkk965BuUv"
```

Теперь пароль, добавленный в Azure Key Vault, можно вызвать, используя его URI. Используйте **https://Contoso-Vault2.vault.azure.net/secrets/ExamplePassword** , чтобы получить текущую версию. 

Чтобы просмотреть содержащееся в секрете значение в виде обычного текста, введите следующее:

```azurecli
az keyvault secret show --name "ExamplePassword" --vault-name "Contoso-Vault2"
```

Вы создали Key Vault, сохранили в нем секрет и извлекли его.

## <a name="clean-up-resources"></a>Очистка ресурсов

Другие руководства в этой серии созданы на основе этого документа. Если вы планируете продолжить работу с последующими краткими руководствами и статьями, эти ресурсы можно не удалять.
Вы можете удалить ставшие ненужными группу ресурсов и все связанные с ней ресурсы, выполнив команду [az group delete](/cli/azure/group). Удалите ресурсы следующим образом:

```azurecli
az group delete --name ContosoResourceGroup
```

## <a name="next-steps"></a>Дальнейшие действия

С помощью этого краткого руководства вы создали Key Vault и сохранили в нем секрет. Дополнительные сведения о Key Vault и его интеграции в приложения см. в следующих статьях.

- [Обзор Azure Key Vault](../general/overview.md)
- Справочник по [командам az keyvault для Azure CLI](/cli/azure/keyvault?view=azure-cli-latest)
- Статья [Обзор системы безопасности Key Vault](../general/security-overview.md)
