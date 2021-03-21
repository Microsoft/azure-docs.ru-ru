---
title: Запуск службы Kubernetes Azure — Анализ текста
titleSuffix: Azure Cognitive Services
description: Разверните Анализ текста образ контейнера в службе Azure Kubernetes и протестируйте его в веб-браузере.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 04/01/2020
ms.author: aahi
ms.openlocfilehash: fb33bd4fe70c51e1e2afffece10ba3b29dcb0450
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98932439"
---
# <a name="deploy-a-text-analytics-container-to-azure-kubernetes-service"></a>Развертывание контейнера Анализ текста в службе Kubernetes Azure

Узнайте, как развернуть образ контейнера [Анализ текста](./text-analytics-how-to-install-containers.md) Cognitive Services Azure в службе Kubernetes Azure (AKS). В этой процедуре показано, как создать ресурс Анализ текста, как создать связанный образ анализа тональности и как выполнить это согласование этих двух элементов из браузера. С помощью контейнеров можно не только управлять инфраструктурой, но и сосредоточиться на разработке приложений.

## <a name="prerequisites"></a>Предварительные условия

Для выполнения этой процедуры необходимо установить и запустить несколько средств локально. Не используйте Azure Cloud Shell. Кроме этого, вам потребуются:

* Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services), прежде чем начинать работу.
* Текстовый редактор, например [Visual Studio Code](https://code.visualstudio.com/download).
* [Azure CLI](/cli/azure/install-azure-cli) установлен.
* Установленная [CLI Kubernetes](https://kubernetes.io/docs/tasks/tools/install-kubectl/) .
* Ресурс Azure с правильной ценовой категорией. Не все ценовые категории поддерживают этот контейнер.
    * **Анализ текста ресурс Azure** только с ценовыми категориями F0 или Standard.
    * Ресурс **Azure Cognitive Services** с ценовой категорией S0.

[!INCLUDE [Create a Cognitive Services Text Analytics resource](../includes/create-text-analytics-resource.md)]

[!INCLUDE [Create a Text Analytics container on Azure Kubernetes Service (AKS)](../../containers/includes/create-aks-resource.md)]

#### <a name="key-phrase-extraction"></a>[извлечение ключевых фраз](#tab/keyphrase);

[!INCLUDE [Key Phrase Extraction Kubernetes config and deploy steps](../includes/key-phrase-extraction-kubernetes-config-deploy.md)]

[!INCLUDE [Verify the Key Phrase Extraction container instance](../includes/verify-key-phrase-extraction-container.md)]

#### <a name="language-detection"></a>[распознавание языка](#tab/language);

[!INCLUDE [Language Detection Kubernetes config and deploy steps](../includes/language-detection-kubernetes-config-deploy.md)]

[!INCLUDE [Verify the Language Detection container instance](../includes/verify-language-detection-container.md)]

#### <a name="sentiment-analysis"></a>[анализ тональности](#tab/sentiment);

[!INCLUDE [Sentiment Analysis Kubernetes config and deploy steps](../includes/sentiment-analysis-kubernetes-config-deploy.md)]

[!INCLUDE [Verify the Sentiment Analysis container instance](../includes/verify-sentiment-analysis-container.md)]

***

## <a name="next-steps"></a>Дальнейшие действия

* Использование большего числа [контейнеров Cognitive Services](../../cognitive-services-container-support.md)
* Использование [подключенной службы анализ текста](../index.yml)
