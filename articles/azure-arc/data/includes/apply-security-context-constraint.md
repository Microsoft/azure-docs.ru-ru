---
author: MikeRayMSFT
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.topic: include
ms.date: 03/02/2021
ms.author: mikeray
ms.openlocfilehash: 0fca43f76b24a08ca96be749f7f2a822b0be2418
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101687600"
---
В этом разделе объясняется, как применить ограничение контекста безопасности (SCC). В предварительной версии это ослабляет ограничения безопасности. 

1. Скачайте настраиваемое ограничение контекста безопасности (SCC). Используйте одно из следующих значений: 
   - [GitHub](https://github.com/microsoft/azure_arc/tree/main/arc_data_services/deploy/yaml/arc-data-scc.yaml) 
   - [Raw](https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/arc-data-scc.yaml)
   - `curl`
   
      Следующая команда скачивает Arc-Data-SCC. YAML:

      ```console
      curl https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/arc-data-scc.yaml -o arc-data-scc.yaml
      ```

1. Создайте SCC.

   ```console
   oc create -f arc-data-scc.yaml
   ```

1. Примените SCC к учетной записи службы.

   > [!NOTE]
   > Используйте то же пространство имен здесь и в `azdata arc dc create` приведенной ниже команде. Пример: `arc` .

   ```console
   oc adm policy add-scc-to-user arc-data-scc --serviceaccount default --namespace arc
   ```

   ```console
   oc create rolebinding arc-data-rbac --clusterrole=system:openshift:scc:arc-data-scc --serviceaccount=arc:default
   ```
