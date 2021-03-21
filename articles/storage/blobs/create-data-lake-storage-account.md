---
title: Создание учетной записи хранения для Azure Data Lake Storage 2-го поколения
description: Узнайте, как создать учетную запись хранения для использования с Azure Data Lake Storage 2-го поколения.
author: normesta
ms.topic: how-to
ms.author: normesta
ms.date: 08/31/2020
ms.service: storage
ms.reviewer: stewu
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: 712f1dc0679ee49791831e782fb68c39a757870a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98624343"
---
# <a name="create-a-storage-account-to-use-with-azure-data-lake-storage-gen2"></a>Создание учетной записи хранения для использования с Azure Data Lake Storage 2-го поколения

Чтобы использовать Data Lake Storage 2-го поколения возможности, создайте учетную запись хранения с иерархическим пространством имен.

## <a name="choose-a-storage-account-type"></a>Выбор типа учетной записи хранения

Data Lake Storage возможности поддерживаются в следующих типах учетных записей хранения:

- Общего назначения версии 2
- BlockBlobStorage

Сведения о выборе между ними см. в разделе [Общие сведения об учетной записи хранения](../common/storage-account-overview.md).

## <a name="create-a-storage-account-with-a-hierarchical-namespace"></a>Создание учетной записи хранения с иерархическим пространством имен

Создайте [учетную запись общего назначения v2](../common/storage-account-create.md) или учетную запись [блоккблобстораже](storage-blob-create-account-block-blob.md) с включенным параметром **иерархического пространства имен** .

Разблокируйте Data Lake Storage возможности при создании учетной записи, включив параметр **иерархического пространства имен** на вкладке **Дополнительно** на странице **Создание учетной записи хранения** . Этот параметр необходимо включить при создании учетной записи. Его нельзя включить позже.

На следующем рисунке показан этот параметр на странице **Создание учетной записи хранения** .

> [!div class="mx-imgBorder"]
> ![Параметр иерархического пространства имен](./media/create-data-lake-storage-account/hierarchical-namespace-feature.png)

Если у вас есть учетная запись хранения, которую вы хотите использовать с Data Lake Storage а параметр иерархического пространства имен отключен, необходимо перенести данные в новую учетную запись хранения с включенным параметром.

> [!NOTE]
> Невозможно одновременно включить **защиту данных** и **Иерархическое пространство имен** .

## <a name="next-steps"></a>Дальнейшие действия

- [Общие сведения об учетной записи хранения](../common/storage-account-overview.md)
- [Использование Azure Data Lake Storage 2-го поколения для обеспечения соответствия требованиям больших данных](data-lake-storage-data-scenarios.md)
- [Контроль доступа в Azure Data Lake Storage 2-го поколения](data-lake-storage-access-control.md)
