---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 02/11/2021
ms.author: alkohli
ms.openlocfilehash: b2c1ebe390b1a2dec7be678b5d6f3a991056a23b
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102517580"
---
1. Перед тем как начать, убедитесь в следующем.

    1. Вы настроили и [активировали устройство Azure Stack пограничной Pro](../articles/databox-online/azure-stack-edge-gpu-deploy-activate.md) с помощью ресурса Azure Stackного периметра в Azure.
    1. Вы [настроили вычисление на этом устройстве в портал Azure](../articles/databox-online/azure-stack-edge-deploy-configure-compute.md#configure-compute).
    
1. [Подключитесь к интерфейсу PowerShell](../articles/databox-online/azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).
1. Используйте следующую команду, чтобы включить MPS на устройстве.

    ```powershell
    Start-HcsGpuMPS
    ```