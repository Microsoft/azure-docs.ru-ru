---
title: включить файл
description: включить файл
services: azure-resource-manager
author: tfitzmac
ms.service: cost-management-billing
ms.topic: include
ms.date: 03/15/2021
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: f02b7f0f80cfb875cc6207b542db90607b379b67
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800321"
---
| Ресурс | Ограничение |
| --- | --- |
| Подписки, [связанные с клиентом Azure Active Directory](../articles/active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md) | Неограниченно |
| [Соадминистраторы](../articles/cost-management-billing/manage/add-change-subscription-administrator.md) на подписку |Неограниченно |
| Количество [групп ресурсов](../articles/azure-resource-manager/management/overview.md) на подписку |980 |
| Размер запроса API Azure Resource Manager |4 194 304 байт |
| Количество тегов на подписку<sup>1</sup> |50 |
| Уникальный тег вычисления для каждой подписки<sup>1</sup> | 80 000 |
| Количество [развертываний на уровне подписки](../articles/azure-resource-manager/templates/deploy-to-subscription.md) на расположение | 800<sup>2</sup> |

<sup>1</sup> Вы можете применить до 50 тегов непосредственно к подписке. Но подписка может содержать неограниченное количество тегов, применяемых к группам ресурсов и ресурсам в рамках подписки. Количество тегов на ресурс или группу ресурсов ограничено значением 50. Resource Manager возвращает [список уникальных имен и значений тегов](/rest/api/resources/tags) в подписке, только если число тегов не превышает 80 000. Но вы по-прежнему можете найти ресурс по тегу, если это число превышает 80 000.

<sup>2</sup> Если вы достигли предела в 800 развертываний, удалите из журнала развертывания, которые больше не нужны. Удалить развертывания на уровне подписки можно с помощью [Remove-AzDeployment](/powershell/module/az.resources/Remove-AzDeployment) или [az deployment sub delete](/cli/azure/deployment/sub#az_deployment_sub_delete).
