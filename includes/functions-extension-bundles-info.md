---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 02/09/2020
ms.author: glenga
ms.openlocfilehash: 1fc37c6f93fba34944caa7a91c2a89ce5dcdc398
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "78201025"
---
::: zone pivot="programming-language-python,programming-language-javascript,programming-language-powershell,programming-language-typescript"  
> [!TIP]
> Во время запуска узел скачивает и устанавливает [расширение привязки хранилища](../articles/azure-functions/functions-bindings-storage-queue.md#functions-2x-and-higher) и другие расширения привязки Майкрософт. Эта установка происходит, потому что расширения привязки включены по умолчанию в файле *host.json* со следующими свойствами:
>
> ```json
> {
>     "version": "2.0",
>     "extensionBundle": {
>         "id": "Microsoft.Azure.Functions.ExtensionBundle",
>         "version": "[1.*, 2.0.0)"
>     }
> }
> ```
>
> Если возникнут ошибки, связанные с расширениями привязки, убедитесь, что указанные выше свойства есть в файле *host.json*.
::: zone-end  