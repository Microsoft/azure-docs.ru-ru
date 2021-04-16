---
title: Включить файл
description: включить файл
services: functions
author: craigshoemaker
manager: gwallace
ms.service: azure-functions
ms.topic: include
ms.date: 01/28/2020
ms.author: cshoe
ms.custom: include file
ms.openlocfilehash: bcc55156ca1d03614a4ff9767d6cf3f2603c06ca
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96027940"
---
Добавьте поддержку в предпочитаемой среде разработки, используя указанные ниже методы.

| Среда разработки  | Тип приложения      | Добавление поддержки |
|--------------------------|-----------------------|----------------|
| Visual Studio            | Библиотека классов C#      | [Установка пакета NuGet](../articles/azure-functions/functions-bindings-register.md#vs) |
| Visual Studio Code       | На базе [основных средств](../articles/azure-functions/functions-run-local.md) | [Регистрация пакета расширений](../articles/azure-functions/functions-bindings-register.md#extension-bundles)<br><br>Рекомендуем установить [расширение инструментов Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack). |
| Любой другой редактор или IDE     | На базе [основных средств](../articles/azure-functions/functions-run-local.md) | [Регистрация пакета расширений](../articles/azure-functions/functions-bindings-register.md#extension-bundles) |
| Портал Azure             | Только в Интернете на портале | Устанавливается при добавлении привязки<br /><br /> См. дополнительные сведения об [обновлении существующих расширений привязки](../articles/azure-functions/functions-bindings-register.md) без необходимости переиздавать проект приложения-функции. |