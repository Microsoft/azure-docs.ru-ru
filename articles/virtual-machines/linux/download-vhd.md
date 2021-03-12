---
title: Скачивание виртуального жесткого диска Linux из Azure
description: Скачайте виртуальный жесткий диск Linux с помощью Azure CLI и портала Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: disks
ms.collection: linux
ms.topic: how-to
ms.date: 08/03/2020
ms.author: cynthn
ms.openlocfilehash: b3435d1dabf604cf7a1394c14ee62d65b923714b
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102565943"
---
# <a name="download-a-linux-vhd-from-azure"></a>Скачивание виртуального жесткого диска Linux из Azure

Из этой статьи вы узнаете, как скачать файл виртуального жесткого диска (VHD) Linux из Azure с помощью портал Azure. 

## <a name="stop-the-vm"></a>Остановка виртуальной машины

VHD невозможно скачать из Azure, если он подключен к запущенной виртуальной машине. Для скачивания виртуального жесткого диска необходимо отключить виртуальную машину. 

1.  Войдите на [портал Azure](https://portal.azure.com/).
2.  В меню слева выберите **виртуальные машины**.
3.  Выберите виртуальную машину из списка.
4.  На странице виртуальной машины выберите пункт **прерывать**.

    :::image type="content" source="./media/download-vhd/export-stop.PNG" alt-text="Отображает кнопку меню для завершения виртуальной машины.":::

## <a name="generate-sas-url"></a>Создание URL-адреса SAS

Чтобы скачать VHD-файл, необходимо создать URL для [подписи общего доступа (SAS)](../../storage/common/storage-sas-overview.md?toc=/azure/virtual-machines/windows/toc.json) . Когда этот URL-адрес создан, ему назначается срок действия.

1. В меню страницы виртуальной машины выберите **диски**.
2. Выберите диск операционной системы для виртуальной машины и щелкните **Экспорт диска**.
1. Если требуется, обновите значение **URL-адреса истекает через (с)** , чтобы получить достаточно времени для завершения загрузки. Значение по умолчанию — 3600 секунд (один час).
3. Щелкните **Создать URL-адрес**.
 
      
## <a name="download-vhd"></a>Скачивание VHD

1.  Под созданным URL-адресом щелкните ссылку **Скачать VHD-файл**.

    :::image type="content" source="./media/download-vhd/export-download.PNG" alt-text="Отображается кнопка для скачивания виртуального жесткого диска.":::

2.  Чтобы начать загрузку, может потребоваться выбрать **сохранить** в браузере. По умолчанию VHD-файлу присваивается имя *abcd*.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [передать пользовательский диск и создать на его основе виртуальную машину Linux с помощью Azure CLI](upload-vhd.md). 
- [Управление дисками Azure с помощью Azure CLI](tutorial-manage-disks.md).
