---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 11/27/2020
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 01e62685fa73a7f547ef5246b28fdfdf659e7afa
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102623401"
---
|Язык                                 |1.x         |2.x| 3.x |
|-----------------------------------------|------------|---| --- |
|[C#](../articles/azure-functions/functions-reference-csharp.md)|Общедоступная версия (.NET Framework 4.7)|Общедоступная версия (.NET Core 2.2<sup>1</sup>)| Общедоступная версия (.NET Core 3.1)<br/>[Предварительная версия (.NET 5.0)](../articles/azure-functions/dotnet-isolated-process-guide.md) |
|[JavaScript](../articles/azure-functions/functions-reference-node.md#node-version)|Общедоступная версия (Node 6)|Общедоступная версия (Node 10 и Node 8)| Общедоступная версия (Node 14, 12 и 10) |
|[F#](../articles/azure-functions/functions-reference-fsharp.md)|Общедоступная версия (.NET Framework 4.7)|Общедоступная версия (.NET Core 2.2<sup>1</sup>)| Общедоступная версия (.NET Core 3.1) |
|[Java](../articles/azure-functions/functions-reference-java.md)|Недоступно|Общедоступная версия (Java 8)| Общедоступная версия (Java 11 и Java 8)|
|[PowerShell](../articles/azure-functions/functions-reference-powershell.md) |Недоступно|Общедоступная версия (PowerShell Core 6)| Общедоступная версия (PowerShell 7 и Core 6)|
|[Python](../articles/azure-functions/functions-reference-python.md#python-version)|Недоступно|Общедоступная версия (Python 3.7 и 3.6)| Общедоступная версия (Python 3.8, 3.7 и 3.6) <br/> Предварительная версия (Python 3.9)|
|[TypeScript](../articles/azure-functions/functions-reference-node.md#typescript) |Недоступно|Общедоступная версия<sup>2</sup>| Общедоступная версия<sup>2</sup> |


<sup>1</sup> Приложения библиотеки классов .NET, предназначенные для среды выполнения версии 2.x, теперь можно запускать в .NET Core 3.1 в режиме совместимости с .NET Core 2.x. Чтобы узнать больше, ознакомьтесь с [рекомендациями по Функциям версии 2.x](../articles/azure-functions/functions-dotnet-class-library.md#functions-v2x-considerations).  
<sup>2</sup> Поддерживается посредством транскомпиляции в JavaScript.

Дополнительные сведения о поддерживаемых версиях см. в руководстве разработчика по конкретному языку.   
Сведения о запланированных изменениях поддержки языков см. в [стратегии развития Azure](https://azure.microsoft.com/roadmap/?tag=functions).
