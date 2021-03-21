---
title: Заметки о выпуске Шлюз Azure Data Box & Azure Data Box Edge 1906 | Документация Майкрософт
description: Описываются критические открытые проблемы и способы их устранения для Шлюз Azure Data Box и Azure Data Box Edge, в которых используется выпуск 1906.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 11/11/2020
ms.author: alkohli
ms.openlocfilehash: 51f4995ffd6a86022d95df15ae0eb7694d878c60
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96582838"
---
# <a name="azure-data-box-edge-and-azure-data-box-gateway-1906-release-notes"></a>Заметки о выпуске Azure Data Box Edge и Шлюз Azure Data Box 1906

В следующих заметках о выпуске указываются критические открытые проблемы и разрешенные проблемы для выпуска 1906 для Azure Data Box Edge и Шлюз Azure Data Box. 

Заметки о выпуске постоянно обновляются: обнаруживаются и добавляются критические проблемы, требующие временного решения. Перед развертыванием Data Box Edge или Шлюз Data Box внимательно проверьте сведения, содержащиеся в заметках о выпуске.

Этот выпуск соответствует версиям программного обеспечения:

- **Шлюз Data Box 1906 (1.6.978.743)**
- **Data Box Edge 1906 (1.6.978.743)**

> [!NOTE]
> Обновление 1906 можно применять только к Data Box Edge устройствам, на которых выполняется общедоступная версия программного обеспечения (GA) или 1905.

## <a name="whats-new"></a>Новое

- **Исправление ошибки в рабочем процессе управления ключами восстановления** . в предыдущем выпуске обнаружена ошибка за нечувствительного, к которой ключ восстановления не был применен. В этом обновлении эта ошибка устранена. Настоятельно рекомендуется применять это обновление, так как ключ восстановления позволяет восстанавливать данные на устройстве в случае, если устройство не загружается. Дополнительные сведения см. в статье [Сохранение ключа восстановления при развертывании Data Box EDGE или шлюз Data Box](../databox-online/azure-stack-edge-deploy-connect-setup-activate.md#set-up-and-activate-the-physical-device).
- **Улучшения ведения журнала для программируемых массивов шлюзов (FPGA)** . Начиная с версии 1905, были сделаны улучшения в журналах и оповещениях, связанных с FPGA. Это обязательное обновление для Data Box Edge, если в FPGA используется функция COMPUTE для граничных вычислений. Дополнительные сведения см. в разделе [Преобразование данных с помощью граничных вычислений на Data Box Edge](../databox-online/azure-stack-edge-deploy-configure-compute-advanced.md).

## <a name="known-issues-in-ga-release"></a>Известные проблемы в общедоступном выпуске

В этом выпуске не указаны новые проблемы. Все указанные в выпуске проблемы были перенесены из предыдущих выпусков. Чтобы просмотреть список известных проблем, перейдите к разделу [Известные проблемы в общедоступном выпуске](data-box-gateway-release-notes.md#known-issues-in-ga-release).


## <a name="next-steps"></a>Дальнейшие действия

- [Подготовка к развертыванию Шлюз Azure Data Box](data-box-gateway-deploy-prep.md)
- [Подготовка к развертыванию Azure Data Box Edge](../databox-online/azure-stack-edge-deploy-prep.md)
