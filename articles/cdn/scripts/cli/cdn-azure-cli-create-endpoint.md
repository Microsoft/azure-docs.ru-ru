---
title: Создание профиля и конечной точки сети доставки содержимого Azure (CDN) с помощью Azure CLI
description: Примеры сценариев Azure CLI для создания профиля, конечной точки, группы источников, источника и личного домена Azure CDN.
author: asudbring
ms.author: allensu
manager: danielgi
ms.date: 03/09/2021
ms.topic: sample
ms.service: azure-cdn
ms.devlang: azurecli
ms.custom: devx-track-azurecli
ms.openlocfilehash: ce01c1f851a5dbaedc85d848b12bf0b0831a16ef
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "104723555"
---
# <a name="create-an-azure-cdn-profile-and-endpoint-using-the-azure-cli"></a>Создание профиля и конечной точки Azure CDN с помощью Azure CLI

В качестве альтернативы порталу Azure для управления следующими операциями CDN можно использовать такие примеры сценариев Azure CLI:

- создание профиля CDN;
- создание конечной точки CDN;
- создание группы источников CDN и оглашение ее группой по умолчанию;
- создание источника CDN;
- создание личного домена и включение протокола HTTPS.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="sample-scripts"></a>Примеры сценариев

Если у вас еще нет группы ресурсов для профиля CDN, создайте ее с помощью команды `az group create`.

```azurecli
# Create a resource group to use for the CDN.
az group create --name MyResourceGroup --location eastus

```

Следующий сценарий Azure CLI создает профиль CDN и конечную точку CDN:

```azurecli
# Create a CDN profile.
az cdn profile create --resource-group MyResourceGroup --name MyCDNProfile --sku Standard_Microsoft

# Create a CDN endpoint.
az cdn endpoint create --resource-group MyResourceGroup --name MyCDNEndpoint --profile-name MyCDNProfile --origin www.contoso.com

```

Следующий сценарий Azure CLI создает группу источников CDN, задает группу источников по умолчанию для конечной точки и создает новый источник:

```azurecli
# Create an origin group.
az cdn origin-group create --resource-group MyResourceGroup --endpoint-name MyCDNEndpoint --profile-name MyCDNProfile --name MyOriginGroup --origins origin-0

# Make the origin group the default group of an endpoint.
az cdn endpoint update --resource-group MyResourceGroup --name MyCDNEndpoint --profile-name MyCDNProfile --default-origin-group MyOriginGroup
                           
# Create another origin for an endpoint.
az cdn origin create --resource-group MyResourceGroup --endpoint-name MyCDNEndpoint --profile-name MyCDNProfile --name origin-1 --host-name example.contoso.com

```

Следующий сценарий Azure CLI создает личный домен CDN и включает протокол HTTPS. Прежде чем связывать личный домен с конечной точкой Azure CDN, необходимо создать запись канонического имени (CNAME) с помощью Azure DNS или поставщика DNS, чтобы указать конечную точку CDN. Дополнительные сведения см. в разделе [Создание записи CNAME DNS](../../../cdn/cdn-map-content-to-custom-domain.md#create-a-cname-dns-record).

```azurecli
# Associate a custom domain with an endpoint.
az cdn custom-domain create --resource-group MyResourceGroup --endpoint-name MyCDNEndpoint --profile-name MyCDNProfile --name MyCustomDomain --hostname www.example.com

# Enable HTTPS on the custom domain.
az cdn custom-domain enable-https --resource-group MyResourceGroup --endpoint-name MyCDNEndpoint --profile-name MyCDNProfile --name MyCustomDomain

```

## <a name="clean-up-resources"></a>Очистка ресурсов

После выполнения примеры скриптов вы сможете удалить группу ресурсов и все связанные с ней ресурсы, выполнив следующую команду.

```azurecli
# Delete the resource group.
az group delete --name MyResourceGroup

```

## <a name="azure-cli-commands-used-in-this-article"></a>Команды Azure CLI, используемые в этой статье

- [az cdn endpoint create](/cli/azure/cdn/endpoint#az_cdn_endpoint_create)
- [az cdn endpoint update](/cli/azure/cdn/endpoint#az_cdn_endpoint_update)
- [az cdn origin create](/cli/azure/cdn/origin#az_cdn_origin_create)
- [az cdn origin-group create](/cli/azure/cdn/origin-group#az_cdn_origin_group_create)
- [az cdn profile create](/cli/azure/cdn/profile#az_cdn_profile_create)
- [az group create](/cli/azure/group#az_group_create)
- [az group delete](/cli/azure/group#az_group_delete)
- [az cdn custom-domain create](/cli/azure/cdn/custom-domain#az_cdn_custom_domain_create)
- [az cdn custom-domain enable-https](/cli/azure/cdn/custom-domain#az_cdn_custom_domain_enable_https)
