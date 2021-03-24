---
author: baanders
description: включаемый файл для отправки модели в экземпляр Digital двойников Azure
ms.service: digital-twins
ms.topic: include
ms.date: 3/23/2021
ms.author: baanders
ms.openlocfilehash: 987409f070f34f0fd9ee212fab8cc57e889e0146
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104950610"
---
Модель выглядит следующим образом:
:::code language="json" source="~/digital-twins-docs-samples/models/Thermostat.json":::

Чтобы **передать эту модель в экземпляр двойников**, выполните следующую команду Azure CLI, которая передает указанную выше модель в виде встроенного JSON. Можно выполнить команду в [Azure Cloud Shell](../articles/cloud-shell/overview.md) в браузере или на компьютере, если интерфейс командной строки [установлен локально](/cli/azure/install-azure-cli).

```azurecli-interactive
az dt model create --models '{  "@id": "dtmi:contosocom:DigitalTwins:Thermostat;1",  "@type": "Interface",  "@context": "dtmi:dtdl:context;2",  "contents": [    {      "@type": "Property",      "name": "Temperature",      "schema": "double"    }  ]}' -n {digital_twins_instance_name}
```

> [!Note]
> При использовании Cloud Shell в среде PowerShell может потребоваться escape-последовательность символов кавычек во встроенных полях JSON, чтобы их значения были проанализированы должным образом. Ниже приведена команда для передачи модели с этим изменением.
>
> Модель передачи:
> ```azurecli-interactive
> az dt model create --models '{  \"@id\": \"dtmi:contosocom:DigitalTwins:Thermostat;1\",  \"@type\": \"Interface\",  \"@context\": \"dtmi:dtdl:context;2\",  \"contents\": [    {      \"@type\": \"Property\",      \"name\": \"Temperature\",      \"schema\": \"double\"    }  ]}' -n {digital_twins_instance_name}
> ```