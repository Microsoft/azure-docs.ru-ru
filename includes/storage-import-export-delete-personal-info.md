---
title: включить файл
description: включить файл
services: azure-policy
author: craigshoemaker
ms.service: azure-policy
ms.topic: include
ms.date: 05/18/2018
ms.author: cshoe
ms.custom: include file
ms.openlocfilehash: 137aca7c6c857ee6e833c359b710e1c1848d15ed
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95556058"
---
## <a name="deleting-personal-information"></a>Удаление персональных данных

[!INCLUDE [gdpr-intro-sentence.md](gdpr-intro-sentence.md)]

Персональные данные относятся к службе импорта и экспорта (с помощью портала и API) и используются во время операций импорта и экспорта. Данные, используемые в ходе этих процессов, включают в себя:

- Имя контактного лица
- Номер телефона
- Email
- Адрес по улице
- Город
- почтовый индекс;
- Состояние
- страна, область, край, округ, регион;
- идентификатор диска;
- номер счета перевозчика;
- номер отслеживания доставки.

При создании задания импорта или экспорта пользователи указывают контактные данные и адрес доставки. Персональные данные хранятся в задании. Кроме того, они могут храниться в параметрах портала. Персональные данные сохраняются в параметрах портала, только если вы установили флажок **Save carrier and return address as default** (Сохранить перевозчика и обратный адрес по умолчанию) в разделе *Сведения о возврате* при экспорте.

Персональные контактные данные могут быть удалены следующими способами.

- Данные, сохраненные с заданием, удаляются вместе с заданием. Пользователи могут удалять задания вручную. Завершенные задания автоматически удаляются по истечении 90 дней. Можно вручную удалять задания с помощью REST API или портала Azure. Чтобы удалить задание на портале Azure, перейдите к этому заданию импорта или экспорта и щелкните *Удалить* на панели команд. Дополнительные сведения о том, как удалить задание импорта или экспорта с помощью REST API, приведены в разделе [Отмена и удаление заданий службы импорта и экспорта Azure](/previous-versions/azure/storage/common/storage-import-export-cancelling-and-deleting-jobs).

- Контактные данные, сохраненные в параметрах портала, можно удалить, удалив эти параметры портала. Чтобы удалить параметры портала, сделайте следующее.
  - Войдите на [портал Azure](https://portal.azure.com).
  - Щелкните значок *Параметры*. ![Значок параметров Azure](media/storage-import-export-delete-personal-info/azure-settings-icon.png)
  - Щелкните *Экспортировать все параметры* (чтобы сохранить текущие параметры в файл с расширением `.json`).
  - Щелкните *Удалить все параметры и частные панели мониторинга*, чтобы удалить все параметры, включая сохраненные контактные данные.

Дополнительные сведения см. в политике конфиденциальности Майкрософт в [центре управления безопасностью](https://www.microsoft.com/trustcenter) .