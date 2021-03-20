---
title: Просмотр или удаление данных Пользовательская служба визуального распознавания
titleSuffix: Azure Cognitive Services
description: Вы сохраняете полный контроль над данными. В этой статье объясняется, как можно просматривать, экспортировать или удалять данные в Пользовательская служба визуального распознавания.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: custom-vision
ms.topic: conceptual
ms.date: 03/21/2019
ms.author: pafarley
ms.openlocfilehash: fe17fa4349085be47201974b418493fe8d912ece
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90527400"
---
# <a name="view-or-delete-user-data-in-custom-vision"></a>Просмотр или удаление данных пользователя в Пользовательское визуальное распознавание

Пользовательское визуальное распознавание собирает данные пользователя для работы со службой, но клиенты имеют полный контроль над просмотром и удалением данных с помощью [API-интерфейсов для обучения](https://go.microsoft.com/fwlink/?linkid=865446)пользовательское визуальное распознавание.

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

Сведения о просмотре и удалении данных пользователя в Пользовательское визуальное распознавание см. в следующей таблице.

| Данные | Просмотр операции | Операция удаления |
| ---- | ---------------- | ---------------- |
| Сведения об учетной записи (ключи подписки) | [GetAccountInfo](https://go.microsoft.com/fwlink/?linkid=865446) | Удаляются с помощью портала Azure (подписки Azure) Или с помощью кнопки "удалить учетную запись" на странице параметров CustomVision.ai (подписки на учетные записи Майкрософт) | 
| Сведения об итерациях | [GetIteration](https://go.microsoft.com/fwlink/?linkid=865446) | [DeleteIteration](https://go.microsoft.com/fwlink/?linkid=865446) |
| Сведения о производительности итерации | [GetIterationPerformance](https://go.microsoft.com/fwlink/?linkid=865446) | [DeleteIteration](https://go.microsoft.com/fwlink/?linkid=865446) | 
| Список итераций | [GetIterations](https://go.microsoft.com/fwlink/?linkid=865446) | [DeleteIteration](https://go.microsoft.com/fwlink/?linkid=865446) |
| Проекты и сведения о проектах | [GetProject](https://go.microsoft.com/fwlink/?linkid=865446) и [GetProjects](https://go.microsoft.com/fwlink/?linkid=865446) | [DeleteProject](https://go.microsoft.com/fwlink/?linkid=865446) | 
| Теги образов | [GetTag](https://go.microsoft.com/fwlink/?linkid=865446) и [GetTags](https://go.microsoft.com/fwlink/?linkid=865446) | [DeleteTag](https://go.microsoft.com/fwlink/?linkid=865446) | 
| Изображения | [GetTaggedImages](https://go.microsoft.com/fwlink/?linkid=865446) (предоставляется URI для скачивания изображения) и [GetUntaggedImages](https://go.microsoft.com/fwlink/?linkid=865446) (предоставляется URI для скачивания изображения) | [DeleteImages](https://go.microsoft.com/fwlink/?linkid=865446) | 
| Экспортированные итерации | [GetExports](https://go.microsoft.com/fwlink/?linkid=865446) | Удаляются при удалении учетной записи |
