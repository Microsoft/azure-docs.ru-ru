---
title: включить файл
description: включить файл
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 02/03/2021
ms.author: fauhse
ms.custom: include file
ms.openlocfilehash: a086aae35c9a800c6a4cfc3e872a34438bc84095
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99569607"
---
По умолчанию общие папки категории "Стандартный" могут охватывать до 5 Тиб. Но вы можете увеличить размер общей папки до 100 ТиБ. Чтобы увеличить предельное значение для общей папки, включите параметр **Large file share** (Большая общая папка) в учетной записи хранения. Для учетных записей хранения категории "Премиум" (учетные записи *хранилища файлов*) флаг функции больших общих папок не устанавливается, так как для всех общих папок категории "Премиум" уже включена возможность подготовки до полной емкости в 100 ТиБ.

Функцию больших общих папок можно включить только в учетных записях хранения категории "Стандартный" для локально избыточного или избыточного между зонами хранилищ. После включения флага функции больших общих папок нельзя изменить уровень избыточности, выбрав геоизбыточное или геоизбыточное между зонами хранилище.

Чтобы включить большие общие папки в существующей учетной записи хранения, перейдите к разделу **Общие папки** в содержании для учетной записи хранения.
В этой колонке выберите параметр **емкости общей папки**, измените его значение на **100 Тиб** и нажмите кнопку **Сохранить**.

:::image type="content" source="media/storage-files-tiers-enable-large-shares/enable-lfs.png" alt-text="Снимок экрана: включение параметра больших общих папок на портале Azure." lightbox="media/storage-files-tiers-enable-large-shares/increase-share-capacity.png":::

Кроме того, вы можете включить общие папки размером 100 ТиБ с помощью командлета PowerShell [`Set-AzStorageAccount`](/powershell/module/az.storage/set-azstorageaccount), а также команды Azure CLI [`az storage account update`](/cli/azure/storage/account#az-storage-account-update). Подробные инструкции см. в статье [Включение функции больших общих папок и их создание](../articles/storage/files/storage-files-how-to-create-large-file-share.md).

Дополнительные сведения см. в статье о [создании общей папки Azure в новой учетной записи хранения](../articles/storage/files/storage-how-to-create-file-share.md).