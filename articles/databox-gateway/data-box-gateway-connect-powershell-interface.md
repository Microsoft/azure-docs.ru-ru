---
title: Использование Windows PowerShell для подключения к устройству Шлюз Azure Data Box и управления им
description: Описывает подключение к Шлюз Data Box и управление им с помощью интерфейса Windows PowerShell.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: how-to
ms.date: 10/20/2020
ms.author: alkohli
ms.openlocfilehash: f3b93bfc9af9bce50c301c10bd372567360d7223
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96582740"
---
# <a name="manage-an-azure-data-box-gateway-device-via-windows-powershell"></a>Управление устройством Шлюз Azure Data Box с помощью Windows PowerShell

Шлюз Azure Data Box решение позволяет передавать данные по сети в Azure. В этой статье описаны некоторые задачи по настройке и управлению для устройства Шлюз Data Box. Для управления устройством можно использовать портал Azure, локальный веб-интерфейс или интерфейс Windows PowerShell.

В этой статье рассматриваются задачи, выполняемые с помощью интерфейса PowerShell.

Эта статья содержит следующие процедуры.

- Подключитесь к интерфейсу PowerShell.
- Создать пакет поддержки.
- Передача сертификата
- Загрузка в среде без DHCP
- Просмотр сведений об устройстве

## <a name="connect-to-the-powershell-interface"></a>Подключитесь к интерфейсу PowerShell.

[!INCLUDE [Connect to admin runspace](../../includes/data-box-gateway-connect-minishell.md)]

## <a name="create-a-support-package"></a>Создать пакет поддержки.

[!INCLUDE [Create a support package](../../includes/data-box-gateway-create-support-package.md)]

## <a name="upload-certificate"></a>Передача сертификата

[!INCLUDE [Upload certificate](../../includes/data-box-gateway-upload-certificate.md)]

## <a name="boot-up-in-non-dhcp-environment"></a>Загрузка в среде без DHCP

[!INCLUDE [Boot up in non-DHCP environment](../../includes/data-box-gateway-boot-non-dhcp.md)]

## <a name="view-device-information"></a>Просмотр сведений об устройстве

[!INCLUDE [View device information](../../includes/data-box-gateway-view-device-info.md)]

## <a name="next-steps"></a>Дальнейшие действия

- Разверните [Шлюз Azure Data Box](data-box-gateway-deploy-prep.md) на портале Azure.
