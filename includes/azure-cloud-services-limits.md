---
author: rothja
ms.service: cloud-services
ms.topic: include
ms.date: 11/09/2018
ms.author: jroth
ms.openlocfilehash: 390460cf6a22c164ee41a9b6679c7b7f2979d8ec
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85838875"
---
| Ресурс | Ограничение |
| --- | --- |
| [Количество рабочих или веб-ролей на одно развертывание](../articles/cloud-services/cloud-services-choose-me.md)<sup>1</sup> |25 |
| [Входные конечные точки экземпляра](/previous-versions/azure/reference/gg557552(v=azure.100)#instanceinputendpoint) на развертывание |25 |
| [Входные конечные точки](/previous-versions/azure/reference/gg557552(v=azure.100)#inputendpoint) на развертывание |25 |
| [Внутренние конечные точки](/previous-versions/azure/reference/gg557552(v=azure.100)#internalendpoint) на развертывание |25 |
| [Сертификаты размещенной службы](../articles/cloud-services/cloud-services-certs-create.md#what-are-service-certificates) на развертывание |199 |

<sup>1</sup>Каждая облачная служба Azure с рабочими и веб-ролями может иметь два развертывания: одно для производственной среды и одно для промежуточной. Это ограничение применяется к количеству уникальных ролей, то есть к конфигурации. Это ограничение не применяется к количеству экземпляров на роль, то есть к масштабированию.

