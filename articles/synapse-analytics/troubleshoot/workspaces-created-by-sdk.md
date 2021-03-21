---
title: 'Устранение неполадок: рабочие области, созданные пакетом SDK, не могут запустить синапсе Studio'
description: Инструкции по разрешению рабочих областей, созданных с помощью пакета SDK, не позволяют запустить синапсе Studio
author: julieMSFT
ms.author: jrasnick
ms.topic: troubleshooting
ms.service: synapse-analytics
ms.subservice: workspace
ms.date: 11/24/2020
ms.openlocfilehash: f5d0aae975d476da47510a1f2fd164cbc5f32945
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96466950"
---
# <a name="troubleshoot-azure-synapse-analytics-workspaces-created-using-sdk"></a>Устранение неполадок рабочих областей Azure синапсе Analytics, созданных с помощью пакета SDK

В этой статье содержатся инструкции по устранению неполадок при запуске синапсе Studio из рабочей области синапсе, которая была создана с помощью пакета средств разработки программного обеспечения (SDK).


## <a name="prerequisites"></a>Предварительные условия

- Рабочая область синапсе создана с помощью пакета SDK

## <a name="workaround"></a>Обходной путь

Чтобы запустить синапсе Studio из рабочей области созданного пакета SDK, выполните следующие действия. 
  1.    Создайте рабочую область, выполнив команду `az synapse workspace create`.
  2.    Извлеките идентификатор управляемого удостоверения, выполнив команду `$identity=$(az synapse workspace show --name {workspace name}  --resource-group {resource group name} --query "identity.principalId")`.
  3.    Предоставьте роль участника данных BLOB-объекта хранилища для управляемой учетной записи хранения удостоверений, выполнив ` az role assignment create --role "Storage Blob Data Contributor" --assignee-object-id {identity } --scope {storage account resource id}.`
  4.    Добавьте правило брандмауэра, выполнив команду ` az synapse firewall-rule create --name allowAll --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255 `.

## <a name="next-steps"></a>Дальнейшие действия

* [Что представляет собой Azure Synapse?](../overview-what-is.md)
* [Приступая к работе](../get-started.md)
