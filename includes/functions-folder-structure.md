---
title: Включить файл
description: включить файл
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/12/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: 5f5f682c8c31c17f1a645bcdacdc78f32bd5c001
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96018454"
---
Код всех функций конкретного приложения-функции хранится в корневой папке проекта, содержащей файл конфигурации главного узла и одну или несколько вложенных папок. Каждая вложенная папка содержит код для отдельной функции. Структура папок показана в следующем представлении:

```
FunctionApp
 | - host.json
 | - MyFirstFunction
 | | - function.json
 | | - ...  
 | - MySecondFunction
 | | - function.json
 | | - ...  
 | - SharedCode
 | - bin
```

В версии 2.x и более поздних среды выполнения Функций Azure все функции в приложении-функции следует создавать на одном языке.  

Файл [host.json](../articles/azure-functions/functions-host-json.md) содержит конфигурацию среды выполнения. Он находится в корневой папке приложения-функции. Папка *bin* содержит пакеты и другие файлы библиотек, необходимые для работы приложения-функции. См. требования конкретного языка для проекта приложения-функции:

* [Библиотека классов C# (.csproj)](../articles/azure-functions/functions-dotnet-class-library.md#functions-class-library-project)
* [Скрипт C# (.csx)](../articles/azure-functions/functions-reference-csharp.md#folder-structure)
* [Скрипт F#](../articles/azure-functions/functions-reference-fsharp.md#folder-structure)
* [Java](../articles/azure-functions/functions-reference-java.md#folder-structure)
* [JavaScript](../articles/azure-functions/functions-reference-node.md#folder-structure)
* [Python](../articles/azure-functions/functions-reference-python.md#folder-structure)
