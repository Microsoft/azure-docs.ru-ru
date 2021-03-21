---
title: Извлечение DOCKER для контейнера работоспособности
titleSuffix: Azure Cognitive Services
description: Команда DOCKER Pull для Анализ текста контейнере работоспособности
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 07/07/2020
ms.author: aahi
ms.openlocfilehash: a0b2c9548f9c1289ae0abd61a72d7146a3bbca29
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94965155"
---
Заполните и отправьте [форму запроса Cognitive Services](https://aka.ms/csgate) , чтобы запросить доступ к анализ текста для общедоступной предварительной версии работоспособности.  Это приложение применимо как для контейнера, так и для общедоступной предварительной версии API размещенного веб-интерфейса.
В форме нужно указать сведения о себе, компании и пользовательском сценарии, для которого будет использоваться контейнер. После отправки формы команда Azure Cognitive Services просматривает ее и проверяет, соответствуете ли вы критериям доступа к частному реестру контейнеров.

> [!IMPORTANT]
> * В форме необходимо использовать адрес электронной почты, связанный с ИДЕНТИФИКАТОРом подписки Azure.
> * Ресурс Azure, используемый для запуска контейнера, должен быть создан с утвержденным ИДЕНТИФИКАТОРом подписки Azure. 
> * Проверьте электронную почту (папки "Входящие" и "спам") на наличие обновлений для приложения от корпорации Майкрософт.

После утверждения вам будет отправлено сообщение электронной почты с учетными данными для доступа к частному реестру контейнеров.  Используйте команду DOCKER login с учетными данными, предоставленными в электронной почте для подключения к закрытому реестру контейнеров для Cognitive Services контейнеров.


```Docker
docker login containerpreview.azurecr.io -u <username> -p <password>
```

Используйте [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) команду, чтобы скачать этот образ контейнера из закрытого реестра контейнеров.

```
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-healthcare:latest
```
