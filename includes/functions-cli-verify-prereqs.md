---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: 19f3a99c087c0755a82ce7940326708fd1f6a3e5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "80673143"
---
### <a name="prerequisite-check"></a>Проверка предварительных условий

+ В окне терминала или командном окне запустите `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 2.7.1846 или более поздняя.

+ Выполните команду `az --version`, чтобы проверить, является ли версия Azure CLI версией 2.0.76 или более поздней.

+ Выполните команду `az login`, чтобы войти в Azure и проверить активную подписку.

::: zone pivot="programming-language-python"  
+ Выполните команду `python --version` (в ОС Linux и MacOS) или `py --version` (в ОС Windows), чтобы убедиться, что для Python возвращается версия 3.8.x, 3.7.x или 3.6.x.
::: zone-end