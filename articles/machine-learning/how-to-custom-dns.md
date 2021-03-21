---
title: Использовать пользовательский DNS-сервер
titleSuffix: Azure Machine Learning
description: Как настроить пользовательский DNS-сервер для работы с Машинное обучение Azure рабочей областью и частной конечной точкой.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: jhirono
author: jhirono
ms.date: 03/12/2021
ms.topic: conceptual
ms.custom: how-to
ms.openlocfilehash: 67341fcc7be21e4fa62c6e52a921fe397c8ffeb6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103417626"
---
# <a name="how-to-use-your-workspace-with-a-custom-dns-server"></a>Использование рабочей области с пользовательским DNS-сервером

При использовании рабочей области Машинное обучение Azure с частной конечной точкой существует [несколько способов обработки разрешения имен DNS](../private-link/private-endpoint-dns.md). По умолчанию Azure автоматически обрабатывает разрешение имен для рабочей области и частной конечной точки. Если вместо этого _используется собственный настраиваемый DNS-сервер_, необходимо вручную создать записи DNS или использовать условные серверы пересылки для рабочей области.

> [!IMPORTANT]
> В этой статье описывается, как найти полное доменное имя и IP-адреса для этих записей, которые не предоставляют сведений о настройке записей DNS для этих элементов. Сведения о добавлении записей см. в документации по программному обеспечению DNS.

## <a name="prerequisites"></a>Предварительные требования

- Виртуальная сеть Azure, использующая [собственный DNS-сервер](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

- Рабочая область Машинное обучение Azure с частной конечной точкой. Дополнительные сведения см. в статье [создание машинное обучение Azure рабочей области](how-to-manage-workspace.md).

- Знакомство с использованием [сетевой изоляции во время обучения & вывода](./how-to-network-security-overview.md).

- Знакомство с [конфигурацией зоны DNS в частной конечной точке Azure](../private-link/private-endpoint-dns.md)

- При необходимости [Azure CLI](/cli/azure/install-azure-cli) или [Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="public-regions"></a>Общедоступные регионы

В следующем списке содержатся полные доменные имена (FQDN), используемые рабочей областью, если она находится в общедоступном регионе:

* `<workspace-GUID>.workspace.<region>.cert.api.azureml.ms`
* `<workspace-GUID>.workspace.<region>.api.azureml.ms`
* `ml-<workspace-name>-<region>-<workspace-guid>.notebooks.azure.net`
* `<instance-name>.<region>.instances.azureml.ms`

    > [!NOTE]
    > Доступ к экземплярам вычислений можно получить только в пределах виртуальной сети.

## <a name="azure-china-21vianet-regions"></a>Регионы Azure для китайского региона (21Vianet)

Следующие полные доменные имена предназначены для регионов Azure для китайского региона (21Vianet):

* `<workspace-GUID>.workspace.<region>.cert.api.ml.azure.cn`
* `<workspace-GUID>.workspace.<region>.api.ml.azure.cn`
* `ml-<workspace-name, truncated>-<region>-<workspace-guid>.notebooks.chinacloudapi.cn`

    > [!NOTE]
    > Имя рабочей области для этого FQDN может быть усечено. Усечение выполняется, чтобы длина полного доменного имени не превышала 63 символов.
* `<instance-name>.<region>.instances.ml.azure.cn`
## <a name="find-the-ip-addresses"></a>Поиск IP-адресов

Чтобы найти внутренние IP-адреса для полных доменных имен в виртуальной сети, используйте один из следующих методов.

> [!NOTE]
> Полные доменные имена и IP-адреса будут отличаться в зависимости от конфигурации. Например, значение GUID в имени домена будет соответствовать вашей рабочей области.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az network private-endpoint show --endpoint-name <endpoint> --resource-group <resource-group> --query 'customDnsConfigs[*].{FQDN: fqdn, IPAddress: ipAddresses[0]}' --output table
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
$workspaceDns=Get-AzPrivateEndpoint -Name <endpoint> -resourcegroupname <resource-group>
$workspaceDns.CustomDnsConfigs | format-table
```

# <a name="azure-portal"></a>[Портал Azure](#tab/azure-portal)

1. В [портал Azure](https://portal.azure.com)выберите __рабочую область__ машинное обучение Azure.
1. В разделе __Параметры__ выберите __частные конечные точки подключения__.
1. Выберите ссылку в столбце __закрытая конечная точка__ , который отображается.
1. Список полных доменных имен (FQDN) и IP-адреса для частной конечной точки рабочей области находятся в нижней части страницы.

:::image type="content" source="./media/how-to-custom-dns/private-endpoint-custom-dns.png" alt-text="Список полных доменных имен на портале":::

---

Сведения, возвращаемые из всех методов, одинаковы. список полного доменного имени и частного IP-адреса для ресурсов. Следующий пример относится к глобальному региону Azure:

| Полное доменное имя. | IP-адрес |
| ----- | ----- |
| `fb7e20a0-8891-458b-b969-55ddb3382f51.workspace.eastus.api.azureml.ms` | `10.1.0.5` |
| `ml-myworkspace-eastus-fb7e20a0-8891-458b-b969-55ddb3382f51.notebooks.azure.net` | `10.1.0.6` |

> [!IMPORTANT]
> Некоторые полные доменные имена не отображаются в списке частной конечной точки, но требуются для рабочей области в eastus, southcentralus и westus2. Эти полные доменные имена перечислены в следующей таблице, и их также необходимо добавить на DNS-сервер и (или) в зону Частная зона DNS Azure.
>
> * `<workspace-GUID>.workspace.<region>.cert.api.azureml.ms`
> * `<workspace-GUID>.workspace.<region>.experiments.azureml.net`
> * `<workspace-GUID>.workspace.<region>.modelmanagement.azureml.net`
> * `<workspace-GUID>.workspace.<region>.aether.ms`
> * Если у вас есть вычислительный экземпляр, используйте `<instance-name>.<region>.instances.azureml.ms` , где `<instance-name>` — имя вычислительного экземпляра. Используйте частный IP-адрес закрытой конечной точки рабочей области. Обратите внимание, что доступ к вычислительному экземпляру можно получить только в пределах виртуальной сети.
>
> Для всех этих IP-адресов используйте тот же адрес, что и для `*.api.azureml.ms` записей, возвращенных предыдущими шагами.

В следующей таблице приведены примеры IP-адресов из регионов Azure для китайского региона (21Vianet):

| Полное доменное имя. | IP-адрес |
| ----- | ----- |
| `52882c08-ead2-44aa-af65-08a75cf094bd.workspace.chinaeast2.api.ml.azure.cn` | `10.1.0.5` |
| `ml-mype-pltest-chinaeast2-52882c08-ead2-44aa-af65-08a75cf094bd.notebooks.chinacloudapi.cn` | `10.1.0.6` |
## <a name="next-steps"></a>Следующие шаги

Дополнительные сведения об использовании Машинное обучение Azure с виртуальной сетью см. в [обзоре виртуальной сети](how-to-network-security-overview.md).

Дополнительные сведения об интеграции частных конечных точек в конфигурацию DNS см. в статье [Настройка DNS для частной конечной точки Azure](../private-link/private-endpoint-dns.md).