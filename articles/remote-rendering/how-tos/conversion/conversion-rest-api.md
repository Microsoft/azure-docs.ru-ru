---
title: Преобразование ресурса REST API
description: Описание преобразования ресурса с помощью REST API
author: florianborn71
ms.author: flborn
ms.date: 02/04/2020
ms.topic: how-to
ms.openlocfilehash: e3db4b9c9b4a05142f1327f681b067748cb1a2f9
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104951646"
---
# <a name="use-the-model-conversion-rest-api"></a>Использование REST API преобразования модели

Управление службой [преобразования модели](model-conversion.md) осуществляется с помощью [REST API](https://en.wikipedia.org/wiki/Representational_state_transfer). Этот API можно использовать для создания преобразований, получения свойств преобразования и перечисления существующих преобразований.

## <a name="rest-api-reference"></a>Справочник по REST API

Справочную документацию по удаленной подготовке REST API можно найти [здесь](/rest/api/mixedreality/2021-01-01/remoterendering), а определения Swagger — [здесь](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/mixedreality/data-plane/Microsoft.MixedReality).

Мы предоставляем скрипт PowerShell в [репозитории образцов arr](https://github.com/Azure/azure-remote-rendering) в папке *Scripts* с именем *Conversion.ps1*, который демонстрирует использование нашей службы. Скрипт и его конфигурация описаны здесь: [примеры сценариев PowerShell](../../samples/powershell-example-scripts.md). Мы также предоставляем пакеты SDK для [.NET](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/remoterendering/Azure.MixedReality.RemoteRendering/README.md) и [Java]( https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/remoterendering/azure-mixedreality-remoterendering/README.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Использование хранилища BLOB-объектов Azure для преобразования модели](blob-storage.md)
- [Преобразование модели](model-conversion.md)