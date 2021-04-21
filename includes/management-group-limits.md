---
title: включить файл
description: включить файл
author: tfitzmac
ms.service: governance
ms.topic: include
ms.date: 03/26/2020
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: 53e3f37d14153f3a2d7b5886a49b08ca9052b128
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107800207"
---
| Ресурс | Ограничение |
| --- | --- |
| Группы управления на арендатор Azure AD | 10 000 |
| Подписки на группу управления | Без ограничений. |
| Уровни иерархии групп управления | Корневой уровень плюс 6 уровней<sup>1</sup> |
| Непосредственная родительская группа управления на группу управления | Один |
| [Развертывания уровня группы управления](../articles/azure-resource-manager/templates/deploy-to-management-group.md) на расположение | 800<sup>2</sup> |

<sup>1</sup> 6 уровней не включают уровень подписки.

<sup>2</sup> Если вы достигли предела в 800 развертываний, удалите из журнала те развертывания, которые больше не нужны. Чтобы удалить развертывания уровня группы управления, выполните [Remove-AzManagementGroupDeployment](/powershell/module/az.resources/Remove-AzManagementGroupDeployment) или [az deployment mg delete](/cli/azure/deployment/mg#az_deployment_mg_delete).
