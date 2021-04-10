---
title: включить файл
description: включить файл
author: tfitzmac
ms.service: governance
ms.topic: include
ms.date: 03/26/2020
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: cdcf6215973755444da9e513761de7ac71e479d4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98738889"
---
| Ресурс | Ограничение |
| --- | --- |
| Группы управления на арендатор Azure AD | 10 000 |
| Подписки на группу управления | Без ограничений. |
| Уровни иерархии групп управления | Корневой уровень плюс 6 уровней<sup>1</sup> |
| Непосредственная родительская группа управления на группу управления | Один |
| [Развертывания уровня группы управления](../articles/azure-resource-manager/templates/deploy-to-management-group.md) на расположение | 800<sup>2</sup> |

<sup>1</sup> 6 уровней не включают уровень подписки.

<sup>2</sup> Если вы достигли предела в 800 развертываний, удалите из журнала те развертывания, которые больше не нужны. Чтобы удалить развертывания уровня группы управления, выполните [Remove-AzManagementGroupDeployment](/powershell/module/az.resources/Remove-AzManagementGroupDeployment) или [az deployment mg delete](/cli/azure/deployment/mg#az-deployment-mg-delete).
