---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 03/25/2020
ms.author: glenga
ms.openlocfilehash: f102a5dd5b7dccba6643176d06d17a2a65171c90
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96904086"
---
## <a name="configure-your-local-environment"></a>Настройка локальной среды

Перед началом работы убедитесь, что у вас есть такие компоненты.

+ Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-java,programming-language-other"  
+ [Azure Functions Core Tools](../articles/azure-functions/functions-run-local.md#v2) версии 2.7.1846 или более поздней.
::: zone-end  
::: zone pivot="programming-language-python"
+ Версия Azure Functions Core Tools, соответствующая установленной версии Python:

   | Версия Python | Версия Core Tools |
   | -------------- | ------------------ |
   | Python 3.8     | [Версия 3.x](../articles/azure-functions/functions-run-local.md#v2) |
   | Python 3.6<br/>Python 3.7 | [Версия 2.7.1846 или более поздняя](../articles/azure-functions/functions-run-local.md#v2) |
  
::: zone-end

+ [Azure CLI](/cli/azure/install-azure-cli) 2.4 или более поздней версии. 
::: zone pivot="programming-language-javascript,programming-language-typescript"
+ [Node.js](https://nodejs.org/), активная версия LTS и версия Maintenance LTS (рекомендуются версии 8.11.1 и 10.14.1).
::: zone-end

::: zone pivot="programming-language-python"
+ [Python 3.8 (64-разрядная версия)](https://www.python.org/downloads/release/python-382/), [Python 3.7 (64-разрядная версия)](https://www.python.org/downloads/release/python-375/), [Python 3.6 (64-разрядная версия)](https://www.python.org/downloads/release/python-368/), которые поддерживаются решением "Функции Azure". 
::: zone-end
::: zone pivot="programming-language-powershell"
+ Пакет [SDK для .NET Core 3.1](https://www.microsoft.com/net/download).
::: zone-end
::: zone pivot="programming-language-java"  
+ [Java Developer Kit](/azure/developer/java/fundamentals/java-jdk-long-term-support) версии 8 или 11. 

+ [Apache Maven](https://maven.apache.org) 3.0 или более поздней версии.

::: zone-end
::: zone pivot="programming-language-other"
+ Средства разработки для используемого вами языка. В этом учебнике в качестве примера используется [язык программирования R](https://www.r-project.org/).
::: zone-end