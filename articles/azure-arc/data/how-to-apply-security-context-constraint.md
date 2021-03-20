---
title: Как применить ограничение контекста безопасности
description: Применение ограничения контекста безопасности для платформы контейнеров Azure Red Hat OpenShift или Red Hat OpenShift
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: dnethi
ms.author: dinethi
ms.reviewer: mikeray
ms.date: 01/15/2021
ms.topic: how-to
ms.openlocfilehash: a6751a9ff96b82f9d4fdaacd449c8a766afaa381
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98697842"
---
# <a name="apply-a-security-context-constraint-for-azure-arc-enabled-data-services-on-openshift"></a>Применение ограничения контекста безопасности для служб данных с поддержкой службы "Дуга Azure" на OpenShift

В этой статье описывается, как применить ограничение контекста безопасности для служб данных, поддерживающих службу "Дуга Azure". 

## <a name="applicability"></a>Сущности, к которым применяется триггер

Он применяется к развертываниям на платформе контейнеров Azure Red Hat OpenShift или Red Hat OpenShift. 

## <a name="apply-security-context-constraint"></a>Применить ограничение контекста безопасности

[!INCLUDE [apply-security-context-constraint](includes/apply-security-context-constraint.md)]

## <a name="next-steps"></a>Дальнейшие действия

- [Создание контроллера данных ARC в Azure](create-data-controller.md)
- [Создание контроллера данных в Azure Data Studio](create-data-controller-azure-data-studio.md)
- [Создание контроллера данных ARC в Azure с помощью [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]](create-data-controller-using-azdata.md)

