---
title: Настройка службы "Аттестация Azure" с помощью Azure CLI
description: Сведения о том, как установить и настроить поставщик аттестации с помощью Azure CLI.
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: quickstart
ms.date: 11/20/2020
ms.author: mbaldwin
ms.openlocfilehash: ae283785b4d4dc80c6b9b6c3997aaf82c9ff0f2f
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102178717"
---
# <a name="quickstart-set-up-azure-attestation-with-azure-cli"></a>Краткое руководство. Настройка службы "Аттестация Azure" с помощью Azure CLI

Начните работу со [службой "Аттестация Azure" с помощью Azure CLI](/cli/azure/ext/attestation/attestation).

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="get-started"></a>Начало работы

1. Установите это расширение с помощью команды CLI, приведенной ниже.

   ```azurecli
   az extension add --name attestation
   ```
   
1. Проверка версии

   ```azurecli
   az extension show --name attestation --query version
   ```

1. Примените следующую команду, чтобы войти в Azure:

   ```azurecli
   az login
   ```

1. Если нужно, перейдите к подписке для службы "Аттестация Azure":

   ```azurecli
   az account set --subscription 00000000-0000-0000-0000-000000000000
   ```

1. Зарегистрируйте в этой подписке поставщик ресурсов Microsoft.Attestation с помощью команды [az provider register](/cli/azure/provider#az_provider_register):

   ```azurecli
   az provider register --name Microsoft.Attestation
   ```

   Дополнительные сведения о поставщиках ресурсов Azure, настройке этих поставщиков и управлении ими см. в статье [Поставщики и типы ресурсов Azure](../azure-resource-manager/management/resource-providers-and-types.md).

   > [!NOTE]
   > Для каждой подписки поставщик ресурсов регистрируется однократно.

1. Создайте группу ресурсов для поставщика аттестации. В эту же группу ресурсов можно поместить и другие ресурсы Azure, в том числе виртуальные машины с экземплярами клиентского приложения. Выполните команду [az group create](/cli/azure/group#az_group_create), чтобы создать группу ресурсов, или примените существующую группу ресурсов:

   ```azurecli
   az group create --name attestationrg --location uksouth
   ```

## <a name="create-and-manage-an-attestation-provider"></a>Создание поставщика аттестации и управление им

Ниже приведены команды, которые можно использовать для создания поставщика аттестации и управления им.

1. Выполните команду [az attestation create](/cli/azure/ext/attestation/attestation#ext_attestation_az_attestation_create), чтобы создать поставщик аттестации с требованием подписывания политик:

   ```azurecli
   az attestation create --name "myattestationprovider" --resource-group "MyResourceGroup" --location westus
   ```
   
1. Выполните команду [az attestation show](/cli/azure/ext/attestation/attestation#ext_attestation_az_attestation_show), чтобы получить сведения о свойствах поставщика аттестации, включая сведения о состоянии и URI аттестации:

   ```azurecli
   az attestation show --name "myattestationprovider" --resource-group "MyResourceGroup"
   ```

   С помощью этой команды отображаются значения, аналогичные приведенным ниже:

   ```output
   Id:/subscriptions/MySubscriptionID/resourceGroups/MyResourceGroup/providers/Microsoft.Attestation/attestationProviders/MyAttestationProvider
   Location: MyLocation
   ResourceGroupName: MyResourceGroup
   Name: MyAttestationProvider
   Status: Ready
   TrustModel: AAD
   AttestUri: https://MyAttestationProvider.us.attest.azure.net
   Tags:
   TagsTable:
   ```

Поставщик аттестации можно удалить с помощью команды [az attestation delete](/cli/azure/ext/attestation/attestation#ext_attestation_az_attestation_delete):

```azurecli
az attestation delete --name "myattestationprovider" --resource-group "sample-resource-group"
```

## <a name="policy-management"></a>Управление политикой

Управлять политиками для поставщика аттестации можно с помощью описанных здесь команд (один тип аттестации за раз).

Команда [az attestation policy show](/cli/azure/ext/attestation/attestation/policy#ext_attestation_az_attestation_policy_show) возвращает текущую политику для указанной среды TEE:

```azurecli
az attestation policy show --name "myattestationprovider" --resource-group "MyResourceGroup" --attestation-type SGX-IntelSDK
```

> [!NOTE]
> С помощью этой команды политика отображается в форматах текста и JWT.

Поддерживаются следующие типы TEE:

- `SGX-IntelSDK`
- `SGX-OpenEnclaveSDK`
- `TPM`

Используйте команду [az attestation policy set](/cli/azure/ext/attestation/attestation/policy#ext_attestation_az_attestation_policy_set), чтобы задать новую политику для указанного типа аттестации.

Чтобы задать политику в текстовом формате для заданного типа аттестации с помощью пути к файлу, сделайте следующее:

```azurecli
az attestation policy set --name testatt1 --resource-group testrg --attestation-type SGX-IntelSDK --new-attestation-policy-file "{file_path}"
```

Чтобы задать политику в формате JWT для заданного типа аттестации с помощью пути к файлу, сделайте следующее:

```azurecli
az attestation policy set --name "myattestationprovider" --resource-group "MyResourceGroup" \
--attestation-type SGX-IntelSDK -f "{file_path}" --policy-format JWT
```

## <a name="next-steps"></a>Дальнейшие действия

- [Как создать и подписать файл политики аттестации](author-sign-policy.md)
- [Статья о реализации аттестации с анклавом SGX с примерами кода](/samples/browse/?expanded=azure&terms=attestation)
