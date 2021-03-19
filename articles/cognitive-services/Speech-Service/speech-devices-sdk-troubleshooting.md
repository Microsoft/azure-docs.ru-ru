---
title: Устранение неполадок в пакете SDK для речевых устройств — служба речи
titleSuffix: Azure Cognitive Services
description: Эта статья содержит сведения, которые помогут решить проблемы, которые могут возникнуть при использовании пакета SDK для речевых устройств.
services: cognitive-services
author: mswellsi
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: wellsi
ms.openlocfilehash: c590e0972de289a36890a75b220eddbded701ea8
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "74815560"
---
# <a name="troubleshoot-the-speech-devices-sdk"></a>Устранение неполадок пакета SDK для устройств, подключаемых к службе "Речь"

Эта статья содержит сведения, которые помогут решить проблемы, которые могут возникнуть при использовании пакета SDK для речевых устройств.

## <a name="certificate-failures"></a>Ошибки сертификата

Если при использовании службы распознавания речи возникают ошибки сертификатов, убедитесь, что устройство имеет правильные дату и время:

1. Перейдите в меню **Параметры**. В разделе **Система** выберите **Дата и время**.

    ![В разделе "Параметры" выберите "Дата и время".](media/speech-devices-sdk/qsg-12.png)

1. Оставьте параметр **Automatic date & time** (Автоматическая установка даты и времени) выбранным. В разделе **Выбор часового пояса** выберите нужный часовой пояс.

    ![Выбор параметров даты и часового пояса](media/speech-devices-sdk/qsg-13.png)

    Когда время, установленное в комплекте разработки, соответствует времени ПК, это означает, что комплект разработки подключен к Интернету.

## <a name="next-steps"></a>Дальнейшие действия

* [Просмотр заметки о выпуске](devices-sdk-release-notes.md)
