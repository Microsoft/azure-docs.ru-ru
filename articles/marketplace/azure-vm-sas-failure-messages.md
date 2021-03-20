---
title: Сообщения об ошибках SAS виртуальной машины — Azure Marketplace
description: Часто задаваемые вопросы при работе с подписанными URL-адресами (SAS).
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 10/15/2020
ms.openlocfilehash: 1c89887117c10ca77ec4c04b3adbe3e2d9923479
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93126844"
---
# <a name="virtual-machine-sas-failure-messages"></a>Сообщения об ошибках SAS виртуальной машины

Ниже перечислены некоторые общие проблемы, возникающие при работе с подписанными URL-адресами (которые используются для идентификации и совместного использования загруженных виртуальных жестких дисков для вашего решения) вместе с предлагаемыми разрешениями.

| Проблема | Сообщение об ошибке | Fix |
| --------- | ------------------- | ------- |
| *Ошибка при копировании образов* |  |  |
| "?" не найден в URI SAS | `Failure: Copying Images. Not able to download blob using provided SAS Uri.` | Обновите URI SAS с помощью рекомендуемых средств. |
| Параметры "st" и "se" отсутствуют в URI SAS | `Failure: Copying Images. Not able to download blob using provided SAS Uri.` | Обновите URI SAS с соответствующими значениями **Дата начала** и **Дата окончания** в нем. |
| "sp=rl" отсутствует в URI SAS | `Failure: Copying Images. Not able to download blob using provided SAS Uri.` | Обновите URI SAS, указав разрешения `Read` и `List`. |
| URI SAS имеет пробелы в имени виртуального жесткого диска | `Failure: Copying Images. Not able to download blob using provided SAS Uri.` | Обновите URI SAS, удалив пробелы. |
| Ошибка авторизации URI SAS | `Failure: Copying Images. Not able to download blob due to authorization error.` | Просмотрите и исправьте формат универсального кода ресурса SAS. При необходимости повторно создайте. |
| Параметры URI SAS "st" и "se" не имеют полной спецификации даты и времени | `Failure: Copying Images. Not able to download blob due to incorrect SAS Uri.` | В URI SAS параметры **Дата начала** и **Дата окончания** (подстроки `st` и `se`) должны иметь полный формат даты и времени, например `11-02-2017T00:00:00Z`. Сокращенные версии не допускаются (некоторые команды в Azure CLI могут создать сокращенное значения по умолчанию). |
|  |  |  |

Дополнительные сведения см. в разделе [Использование подписанных URL-адресов (SAS)](../storage/common/storage-sas-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Создать URI SAS](azure-vm-get-sas-uri.md)