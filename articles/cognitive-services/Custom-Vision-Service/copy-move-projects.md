---
title: Копирование и перемещение Пользовательское визуальное распознавание проектов
titleSuffix: Azure Cognitive Services
description: Узнайте, как использовать интерфейсы API Експортпрожект и Импортпрожект для копирования и перемещения проектов Пользовательское визуальное распознавание.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: how-to
ms.date: 09/08/2020
ms.author: pafarley
ms.openlocfilehash: 363f0062d316b22019b864972d5a830e4a838b93
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102178105"
---
# <a name="copy-and-move-your-custom-vision-projects"></a>Копирование и перемещение проектов Пользовательское визуальное распознавание

После создания и обучения проекта Пользовательское визуальное распознавание может потребоваться скопировать проект в другой ресурс. Например, может потребоваться переместить проект из среды разработки в рабочую среду или создать резервную копию проекта в учетной записи в другом регионе Azure для повышения безопасности данных.

API-интерфейсы **[експортпрожект](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)** и **[импортпрожект](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee3)** позволяют копировать проекты из одной учетной записи пользовательское визуальное распознавание в другие. В этом руководство показано, как использовать эти API-интерфейсы RESTFUL с фигурными скобками. Можно также использовать службу HTTP-запросов, например POST, чтобы выдать запросы.

## <a name="business-scenarios"></a>Бизнес-сценарии

Если приложение или бизнес зависит от использования Пользовательское визуальное распознавание проекта, рекомендуется скопировать модель в другую учетную запись Пользовательское визуальное распознавание в другом регионе. Затем, если происходит региональный сбой, вы можете получить доступ к проекту в регионе, в котором он был скопирован.

##  <a name="prerequisites"></a>Предварительные требования

- Два Пользовательское визуальное распознавание ресурсов Azure. Если у вас их нет, перейдите к портал Azure и [Создайте новый ресурс пользовательское визуальное распознавание](https://portal.azure.com/?microsoft_azure_marketplace_ItemHideKey=microsoft_azure_cognitiveservices_customvision#create/Microsoft.CognitiveServicesCustomVision?azure-portal=true).
- Ключи обучения и URL-адреса конечных точек для ресурсов Пользовательское визуальное распознавание. Эти значения можно найти на вкладке **Обзор** ресурса на портал Azure.
- Созданный проект Пользовательское визуальное распознавание. Инструкции о том, как это сделать, см. в разделе [Построение классификатора](./getting-started-build-a-classifier.md) .
* [PowerShell версии 6.0](https://docs.microsoft.com/powershell/scripting/install/installing-powershell-core-on-windows?view=powershell-7.1)или аналогичная программа командной строки.

## <a name="process-overview"></a>Общие сведения о процессе

Процесс копирования проекта состоит из следующих этапов.

1. Сначала вы получите идентификатор проекта в исходной учетной записи, которую необходимо скопировать.
1. Затем вы вызываете API **експортпрожект** , используя идентификатор проекта и ключ обучения исходной учетной записи. Вы получите временную строку токена.
1. Затем вы вызываете API **импортпрожект** , используя строку токена и ключ обучения целевой учетной записи. Затем проект будет перечислен в целевой учетной записи.

## <a name="get-the-project-id"></a>Получение идентификатора проекта

Сначала вызовите **[проект](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddead)** , чтобы просмотреть список существующих проектов пользовательское визуальное распознавание и их идентификаторов. Используйте ключ обучения и конечную точку исходной учетной записи.

```curl
curl -v -X GET "{endpoint}/customvision/v3.3/Training/projects"
-H "Training-key: {training key}"
```

Вы получите `200\OK` ответ со списком проектов и их метаданными в тексте. `"id"`Значение представляет собой строку, которую необходимо скопировать для следующих шагов.

```json
[
  {
    "id": "00000000-0000-0000-0000-000000000000",
    "name": "string",
    "description": "string",
    "settings": {
      "domainId": "00000000-0000-0000-0000-000000000000",
      "classificationType": "Multiclass",
      "targetExportPlatforms": [
        "CoreML"
      ],
      "useNegativeSet": true,
      "detectionParameters": "string",
      "imageProcessingSettings": {
        "augmentationMethods": {}
      }
    },
    "created": "string",
    "lastModified": "string",
    "thumbnailUri": "string",
    "drModeEnabled": true,
    "status": "Succeeded"
  }
]
```

## <a name="export-the-project"></a>Экспорт проекта

Вызовите **[експортпрожект](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)** , используя идентификатор проекта и исходный ключ обучения и конечную точку.

```curl
curl -v -X GET "{endpoint}/customvision/v3.3/Training/projects/{projectId}/export"
-H "Training-key: {training key}"
```

Вы получите `200/OK` ответ с метаданными о экспортированном проекте и ссылочной строке `"token"` . Скопируйте значение маркера.

```json
{
  "iterationCount": 0,
  "imageCount": 0,
  "tagCount": 0,
  "regionCount": 0,
  "estimatedImportTimeInMS": 0,
  "token": "string"
}
```

## <a name="import-the-project"></a>Импорт проекта

Вызовите **[импортпрожект](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc7548b571998fddee3)** , используя целевой ключ обучения и конечную точку, а также маркер ссылки. Можно также присвоить проекту имя в новой учетной записи.

```curl
curl -v -G -X POST "{endpoint}/customvision/v3.3/Training/projects/import"
--data-urlencode "token={token}" --data-urlencode "name={name}"
-H "Training-key: {training key}" -H "Content-Length: 0"
```

Вы получите `200/OK` ответ с метаданными о недавно импортированном проекте.

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "name": "string",
  "description": "string",
  "settings": {
    "domainId": "00000000-0000-0000-0000-000000000000",
    "classificationType": "Multiclass",
    "targetExportPlatforms": [
      "CoreML"
    ],
    "useNegativeSet": true,
    "detectionParameters": "string",
    "imageProcessingSettings": {
      "augmentationMethods": {}
    }
  },
  "created": "string",
  "lastModified": "string",
  "thumbnailUri": "string",
  "drModeEnabled": true,
  "status": "Succeeded"
}
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как копировать и перемещать проект между Пользовательское визуальное распознаваниеными ресурсами. Затем изучите документацию по API, чтобы узнать, что еще можно сделать с помощью Пользовательское визуальное распознавание.
* [Справочная документация по REST API](https://southcentralus.dev.cognitive.microsoft.com/docs/services/Custom_Vision_Training_3.3/operations/5eb0bcc6548b571998fddeb3)