---
title: Запуск контейнера детекторов аномалий в службе "экземпляры контейнеров Azure"
titleSuffix: Azure Cognitive Services
description: Разверните контейнер детектора аномалий в экземпляре контейнера Azure и протестируйте его в веб-браузере.
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.subservice: anomaly-detector
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: mbullwin
ms.openlocfilehash: 3979bb82bfa055cc2a134bf3119097c452ffb855
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94364128"
---
# <a name="deploy-an-anomaly-detector-container-to-azure-container-instances"></a>Развертывание контейнера детекторов аномалий в службе "экземпляры контейнеров Azure"

Узнайте, как развернуть контейнер [детектора Cognitive Services аномалии](../anomaly-detector-container-howto.md) в службе " [экземпляры контейнеров](../../../container-instances/index.yml)Azure". Эта процедура демонстрирует создание ресурса детектора аномалий. Затем мы обсудим извлечение связанного образа контейнера. Наконец, мы выделим возможность выполнить согласование двух из браузера. Использование контейнеров может повлечь за собой внимание разработчиков от управления инфраструктурой до того, чтобы сосредоточиться на разработке приложений.

[!INCLUDE [Prerequisites](../../containers/includes/container-preview-prerequisites.md)]

[!INCLUDE [Create a Cognitive Services Anomaly Detector resource](../includes/create-anomaly-detector-resource.md)]

[!INCLUDE [Create an Anomaly Detector container on Azure Container Instances](../../containers/includes/create-container-instances-resource-from-azure-cli.md)]

[!INCLUDE [API documentation](../../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="next-steps"></a>Дальнейшие действия

* Проверьте раздел [Установка и запуск контейнеров](../anomaly-detector-container-configuration.md) , чтобы получить образ контейнера и запустить контейнер.
* Ознакомьтесь со статьей о [конфигурации контейнеров](../anomaly-detector-container-configuration.md).
* [Дополнительные сведения о службе API детектора аномалий](https://go.microsoft.com/fwlink/?linkid=2080698&clcid=0x409)