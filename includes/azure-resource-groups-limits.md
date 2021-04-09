---
author: tfitzmac
ms.service: azure-resource-manager
ms.topic: include
ms.date: 09/01/2020
ms.author: tomfitz
ms.openlocfilehash: 543aa50d72de5a06a9a1c7ac88ac5ecae993bc9d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98698073"
---
| Ресурс | Ограничение |
| --- | --- |
| Ресурсы на [группу ресурсов](../articles/azure-resource-manager/management/overview.md#resource-groups) | Ресурсы ограничиваются не группой ресурсов, а типом ресурса в этой группе ресурсов. См. следующую строку. |
| Ресурсы на группу ресурсов для каждого типа ресурса |800 (некоторые типы ресурсов могут превышать это ограничение). См. статью [Ресурсы без ограничения в 800 экземпляров на группу ресурсов](../articles/azure-resource-manager/management/resources-without-resource-group-limit.md). |
| Развертываний на группу ресурсов в журнале развертывания |800<sup>1</sup> |
| Ресурсов в развертывании |800 |
| Блокировки управления на уникальную [область](../articles/azure-resource-manager/management/overview.md#understand-scope)  |20 |
| Число тегов на ресурс или группу ресурсов |50 |
| Длина ключа тега |512 |
| Длина значения тега |256 |

<sup>1</sup> Развертывания автоматически удаляются из журнала, когда вы приближаетесь к ограничению. Удаление записи из журнала развертывания не влияет на развернутые ресурсы. См. статью [Автоматическое удаление из журнала развертывания](../articles/azure-resource-manager/templates/deployment-history-deletions.md).

#### <a name="template-limits"></a>Ограничения шаблонов

| Значение | Ограничение |
| --- | --- |
| Параметры |256 |
| Переменные |256 |
| Ресурсы (включая число копий) |800 |
| Выходные данные |64 |
| Выражение шаблона |24 576 символов |
| Ресурсы в экспортированных шаблонах |200 |
| Размер шаблона |4 МБ |
| Размер файла параметров |64 КБ |

Некоторые ограничения можно превысить, используя вложенные шаблоны. См. статью [Использование связанных и вложенных шаблонов при развертывании ресурсов Azure](../articles/azure-resource-manager/templates/linked-templates.md). Чтобы уменьшить число параметров, переменных или выходных данных, можно объединить несколько значений в объект. См. дополнительные сведения об [использовании объектов как параметров](/azure/architecture/guide/azure-resource-manager/advanced-templates/objects-as-parameters).