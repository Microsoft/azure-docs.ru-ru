---
title: Включить файл
description: включить файл
services: iot-edge
author: kgremban
ms.service: iot-edge
ms.topic: include
ms.date: 01/22/2019
ms.author: kgremban
ms.custom: include file
ms.openlocfilehash: ebc23ce4238c736442fbc4507e858876f9192fd9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "76021151"
---
## <a name="associate-an-azure-storage-account-to-iot-hub"></a>Связывание учетной записи хранения Azure с Центром Интернета вещей

Так как приложение виртуального устройства отправляет файл в большой двоичный объект, необходимо связать учетную запись [хранилища Azure](../articles/storage/common/storage-account-create.md) с центром Интернета вещей. При связывании учетной записи хранения Azure с Центром Интернета вещей Центр Интернета вещей создает URI SAS. Устройство может использовать этот URI SAS для безопасной передачи файла в контейнер больших двоичных объектов. Служба Центра Интернета вещей и пакеты SDK для устройств координируют процесс создания URI SAS, делая его доступным для устройства, которое отправляет файл.

Следуйте инструкциям в статье [Настройка отправки файлов Центра Интернета вещей с помощью портала Azure](../articles/iot-hub/iot-hub-configure-file-upload.md). Убедитесь, что контейнер больших двоичных объектов связан с вашим Центром Интернета вещей и что уведомления файлов включены.

![Включение уведомлений файлов на портале](./media/iot-hub-associate-storage/file-notifications-vs2019.png)
