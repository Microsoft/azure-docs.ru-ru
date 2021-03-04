---
title: Включение шифрования на основе узла в службе Kubernetes Azure (AKS)
description: Узнайте, как настроить шифрование на основе узла в кластере Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/03/2021
ms.openlocfilehash: 66e71dfd6a76fb4e6b464eb5c44dcc809fb9be38
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102039739"
---
# <a name="host-based-encryption-on-azure-kubernetes-service-aks-preview"></a>Шифрование на основе узла в службе Azure Kubernetes Service (AKS) (Предварительная версия)

При шифровании на основе узла данные, хранящиеся на узле виртуальной машины агента AKS, шифруются в неактивных потоках и передаются в зашифрованном виде в службу хранилища. Это означает, что временные диски шифруются при хранении с помощью ключей, управляемых платформой. Кэш ОС и дисков данных шифруется при хранении с помощью управляемых платформой ключей или ключей, управляемых клиентом, в зависимости от типа шифрования, установленного на этих дисках. По умолчанию при использовании AKS диски операционной системы и данных шифруются при хранении с помощью ключей, управляемых платформой. Это означает, что кэши для этих дисков также шифруются по умолчанию с помощью ключей, управляемых платформой.  Вы можете указать собственные управляемые ключи [, используя собственные ключи (BYOK) с дисками Azure в службе Kubernetes Azure](azure-disk-customer-managed-keys.md). Кэш для этих дисков будет также зашифрован с помощью ключа, указанного на этом шаге.


## <a name="before-you-begin"></a>Перед началом

Этот компонент можно задать только при создании кластера или во время создания пула узлов.

> [!NOTE]
> Шифрование на основе узла доступно в [регионах Azure][supported-regions] , которые поддерживают шифрование на стороне сервера для управляемых дисков Azure и только с конкретными [поддерживаемыми размерами виртуальных машин][supported-sizes].

### <a name="prerequisites"></a>Предварительные требования

- Убедитесь, что `aks-preview` установлено расширение CLI v 0.4.73 или более поздней версии.
- Убедитесь, что в `EnableEncryptionAtHostPreview` разделе включено установлен флаг компонента `Microsoft.ContainerService` .

Чтобы иметь возможность использовать шифрование на узле для виртуальных машин или масштабируемых наборов виртуальных машин, необходимо включить эту функцию в подписке. Отправьте электронное письмо **encryptionAtHost@microsoft.com** с идентификаторами подписки, чтобы включить функцию для ваших подписок. 

> [!IMPORTANT]
> **encryptionAtHost@microsoft.com** Чтобы включить функцию для ресурсов для вычислений, необходимо отправить по электронной почте идентификатор подписки. Вы не можете включить его самостоятельно для ресурсов вычислений.


### <a name="install-aks-preview-cli-extension"></a>Установка расширения интерфейса командной строки предварительной версии AKS

Чтобы создать кластер AKS с шифрованием на основе узла, необходимо установить последнее расширение CLI *AKS-Preview* . Установите расширение Azure CLI *AKS-Preview* с помощью команды [AZ Extension Add][az-extension-add] или проверьте наличие доступных обновлений с помощью команды [AZ Extension Update][az-extension-update] .

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="limitations"></a>Ограничения

- Можно включить только для новых пулов узлов или новых кластеров.
- Можно включить только в [регионах Azure][supported-regions] , которые поддерживают шифрование управляемых дисков Azure на стороне сервера и только с конкретными [поддерживаемыми размерами виртуальных машин][supported-sizes].
- Требуется кластер AKS и пул узлов на основе масштабируемых наборов виртуальных машин (VMSS) в качестве *типа набора виртуальных машин*.

## <a name="use-host-based-encryption-on-new-clusters-preview"></a>Использовать шифрование на основе узла в новых кластерах (Предварительная версия)

Настройте узлы агента кластера для использования шифрования на основе узла при создании кластера. 

```azurecli-interactive
az aks create --name myAKSCluster --resource-group myResourceGroup -s Standard_DS2_v2 -l westus2 --enable-encryption-at-host
```

Если вы хотите создавать кластеры без шифрования на основе узла, это можно сделать, опустив `--enable-encryption-at-host` параметр.

## <a name="use-host-based-encryption-on-existing-clusters-preview"></a>Использовать шифрование на основе узла в существующих кластерах (Предварительная версия)

Вы можете включить шифрование на основе узла в существующих кластерах, добавив новый пул узлов в кластер. Настройте новый пул узлов для использования шифрования на основе узла с помощью `--enable-encryption-at-host` параметра.

```azurecli
az aks nodepool add --name hostencrypt --cluster-name myAKSCluster --resource-group myResourceGroup -s Standard_DS2_v2 -l westus2 --enable-encryption-at-host
```

Если вы хотите создать новые пулы узлов без функции шифрования на основе узла, это можно сделать, опустив `--enable-encryption-at-host` параметр.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [рекомендациями по безопасности кластера AKS][best-practices-security] Подробнее о [шифровании на основе узла](../virtual-machines/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).


<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: ./operator-best-practices-cluster-security.md
[supported-regions]: ../virtual-machines/disk-encryption.md#supported-regions
[supported-sizes]: ../virtual-machines/disk-encryption.md#supported-vm-sizes
[azure-cli-install]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
