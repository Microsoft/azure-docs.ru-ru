---
title: Удаление ресурсов из Azure
description: Удаление ресурсов из Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 60c5ddcc67db6e4a0649458cfbd5c2949aa9a32a
ms.sourcegitcommit: dda0d51d3d0e34d07faf231033d744ca4f2bbf4a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102202048"
---
# <a name="delete-resources-from-azure"></a>Удаление ресурсов из Azure

> [!NOTE]
>  Параметры удаления ресурсов в этой статье необратимы.

> [!NOTE]
>  Поскольку единственный режим подключения, предлагаемый для служб данных с поддержкой службы "Дуга Azure", в настоящее время является непрямым подключенным режимом, удаление экземпляра из Kubernetes не приведет к его удалению из Azure и удаление экземпляра из Azure не приведет к удалению из Kubernetes.  В настоящее время удаление ресурса выполняется в два этапа, и это будет улучшено в будущем.  В дальнейшем Kubernetes станет источником истинности, и Azure будет обновлен в соответствии с этим.

В некоторых случаях может потребоваться вручную удалить ресурсы служб данных, включенные в службу Arc Azure, в Azure Resource Manager (ARM).  Эти ресурсы можно удалить с помощью любого из следующих параметров.

- [Удаление ресурсов из Azure](#delete-resources-from-azure)
  - [Удаление всей группы ресурсов](#delete-an-entire-resource-group)
  - [Удаление конкретных ресурсов в группе ресурсов](#delete-specific-resources-in-the-resource-group)
  - [Удаление ресурсов с помощью Azure CLI](#delete-resources-using-the-azure-cli)
    - [Удаление ресурсов управляемого экземпляра SQL с помощью Azure CLI](#delete-sql-managed-instance-resources-using-the-azure-cli)
    - [Удаление ресурсов группы серверов PostgreSQL Scale с помощью Azure CLI](#delete-postgresql-hyperscale-server-group-resources-using-the-azure-cli)
    - [Удаление ресурсов контроллера данных Arc Azure с помощью Azure CLI](#delete-azure-arc-data-controller-resources-using-the-azure-cli)
    - [Удаление группы ресурсов с помощью Azure CLI](#delete-a-resource-group-using-the-azure-cli)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="delete-an-entire-resource-group"></a>Удаление всей группы ресурсов
Если вы используете определенную и выделенную группу ресурсов для служб данных, поддерживающих службу "Дуга Azure", и хотите удалить *все содержимое* в группе ресурсов, можно удалить группу ресурсов, которая будет удалять все внутри нее.  

Вы можете удалить группу ресурсов в портал Azure, выполнив следующие действия.

- Перейдите к группе ресурсов в портал Azure, где были созданы ресурсы служб данных с поддержкой дуги Azure.
- Нажмите кнопку **Удалить группу ресурсов** .
- Подтвердите удаление, введя имя группы ресурсов, и нажмите кнопку **Удалить**.

## <a name="delete-specific-resources-in-the-resource-group"></a>Удаление конкретных ресурсов в группе ресурсов

Вы можете удалить определенные ресурсы службы данных, включенные в службу "Дуга Azure" в группе ресурсов в портал Azure, выполнив следующие действия.

- Перейдите к группе ресурсов в портал Azure, где были созданы ресурсы служб данных с поддержкой дуги Azure.
- Выберите все ресурсы для удаления.
- Нажмите кнопку Удалить.
- Подтвердите удаление, введя "Yes" и нажав кнопку " **Удалить**".

## <a name="delete-resources-using-the-azure-cli"></a>Удаление ресурсов с помощью Azure CLI

Вы можете удалить определенные ресурсы службы данных, включенные в службу "Дуга Azure", с помощью Azure CLI.

### <a name="delete-sql-managed-instance-resources-using-the-azure-cli"></a>Удаление ресурсов управляемого экземпляра SQL с помощью Azure CLI

Чтобы удалить ресурсы управляемого экземпляра SQL из Azure с помощью Azure CLI замените значения заполнителей в приведенной ниже команде и запустите ее.

```azurecli
az resource delete --name <sql instance name> --resource-type Microsoft.AzureArcData/sqlManagedInstances --resource-group <resource group name>

#Example
#az resource delete --name sql1 --resource-type Microsoft.AzureArcData/sqlManagedInstances --resource-group rg1
```

### <a name="delete-postgresql-hyperscale-server-group-resources-using-the-azure-cli"></a>Удаление ресурсов группы серверов PostgreSQL Scale с помощью Azure CLI

Чтобы удалить ресурс группы серверов PostgreSQL Scale из Azure с помощью Azure CLI замените значения заполнителей в приведенной ниже команде и запустите ее.

```azurecli
az resource delete --name <postgresql instance name> --resource-type Microsoft.AzureArcData/postgresInstances --resource-group <resource group name>

#Example
#az resource delete --name pg1 --resource-type Microsoft.AzureArcData/postgresInstances --resource-group rg1
```

### <a name="delete-azure-arc-data-controller-resources-using-the-azure-cli"></a>Удаление ресурсов контроллера данных Arc Azure с помощью Azure CLI

> [!NOTE]
> Перед удалением контроллера данных ARC в Azure необходимо удалить все ресурсы экземпляра базы данных, которыми он управляет.

Чтобы удалить контроллер данных Arc Azure из Azure с помощью Azure CLI замените значения заполнителей в приведенной ниже команде и запустите ее.

```azurecli
az resource delete --name <data controller name> --resource-type Microsoft.AzureArcData/dataControllers --resource-group <resource group name>

#Example
#az resource delete --name dc1 --resource-type Microsoft.AzureArcData/dataControllers --resource-group rg1
```

### <a name="delete-a-resource-group-using-the-azure-cli"></a>Удаление группы ресурсов с помощью Azure CLI

Можно также использовать Azure CLI для [удаления группы ресурсов](../../azure-resource-manager/management/delete-resource-group.md).