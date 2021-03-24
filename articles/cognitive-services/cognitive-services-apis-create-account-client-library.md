---
title: Создание ресурса Cognitive Services с помощью клиентской библиотеки управления Azure
titleSuffix: Azure Cognitive Services
description: Сведения о том, как создать ресурсы Cognitive Services и управлять ими с помощью клиентской библиотеки управления Azure.
services: cognitive-services
keywords: Cognitive Services, когнитивня аналитика, когнитивные решения, службы ИИ
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: quickstart
ms.date: 03/15/2021
ms.author: pafarley
zone_pivot_groups: programming-languages-set-ten
ms.openlocfilehash: e042ac263d3a30a9790ba6a3a3d404e5e9cb9aad
ms.sourcegitcommit: 66ce33826d77416dc2e4ba5447eeb387705a6ae5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/15/2021
ms.locfileid: "103472157"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-the-azure-management-client-library"></a>Краткое руководство. Создание ресурса Cognitive Services с помощью клиентской библиотеки управления Azure

В этом кратком руководстве описано, как создать ресурсы Cognitive Services и управлять ими с помощью клиентской библиотеки управления Azure.

Azure Cognitive Services — это семейство облачных служб с REST API и клиентскими библиотеками, которые помогают разработчикам интегрировать когнитивные средства искусственного интеллекта в свои приложения. Для достижения успеха разработчикам не требуется опыт работы со средствами искусственного интеллекта (ИИ) или обработки и анализа данных. С помощью Azure Cognitive Services разработчики могут без усилий добавлять в свои приложения когнитивные функции, создавая когнитивные решения, которые могут видеть, слышать, говорить, понимать и даже в некоторой степени размышлять.

Отдельные службы ИИ представлены [ресурсами](../azure-resource-manager/management/manage-resources-portal.md) Azure, которые вы создаете в своей подписке Azure. После создания ресурса вы можете использовать созданные ключи и конечную точку для аутентификации приложений.

::: zone pivot="programming-language-csharp"

[!INCLUDE [C# SDK quickstart](includes/quickstarts/management-csharp.md)]

::: zone-end

::: zone pivot="programming-language-java"

[!INCLUDE [Java SDK quickstart](includes/quickstarts/management-java.md)]

::: zone-end

::: zone pivot="programming-language-javascript"

[!INCLUDE [NodeJS SDK quickstart](includes/quickstarts/management-node.md)]

::: zone-end

::: zone pivot="programming-language-python"

[!INCLUDE [Python SDK quickstart](includes/quickstarts/management-python.md)]

::: zone-end
