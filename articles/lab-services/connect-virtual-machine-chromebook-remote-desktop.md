---
title: Подключение к виртуальной машине Служб лаборатории Azure с Chromebook | Документация Майкрософт
description: Узнайте, как подключиться с компьютера Chromebook к виртуальной машине в Службах лаборатории Azure.
services: devtest-lab, lab-services, virtual-machines
author: nicolela
ms.topic: article
ms.date: 06/26/2020
ms.author: nicolela
ms.openlocfilehash: 74135c0b36f533ebfbba6422bc79af47825a1a3b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97813230"
---
# <a name="connect-to-a-vm-using-remote-desktop-protocol-on-a-chromebook"></a>Подключение к виртуальной машине с помощью протокола удаленного рабочего стола на Chromebook

В этом разделе показано, как учащийся может подключиться к виртуальной машине задания с компьютера Chromebook с помощью RDP.

## <a name="install-microsoft-remote-desktop-on-a-chromebook"></a>Установка удаленного рабочего стола (Майкрософт) на компьютере Chromebook

1. Откройте App Store на компьютере Chromebook и выполните поиск по запросу **удаленный рабочий стол (Майкрософт)** .

    ![Удаленный рабочий стол (Майкрософт)](./media/how-to-use-classroom-lab/install-ms-remote-desktop-chromebook.png)
    
1. Установите последнюю версию удаленного рабочего стола (Майкрософт). 

## <a name="access-the-vm-from-your-chromebook-using-rdp"></a>Доступ к виртуальной машине с компьютера Chromebook с помощью RDP

1. Откройте файл **RDP**, скачанный на компьютере с установленным **удаленным рабочим столом (Майкрософт)** . При этом должно начаться подключение к виртуальной машине. 

    ![Подключение к виртуальной машине](./media/how-to-use-classroom-lab/connect-vm-chromebook.png)

1. При появлении запроса введите свой пароль.

    ![Снимок экрана входа в систему, в котором вы вводите имя пользователя и пароль.](./media/how-to-use-classroom-lab/password-chromebook.png)

1. Выберите **Continue** (Продолжить), если появится следующее предупреждение. 

    ![Предупреждение о проблеме с сертификатом](./media/how-to-use-classroom-lab/certificate-error-chromebook.png)

1. Вы должны увидеть рабочий стол виртуальной машины, к которой вы подключаетесь.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о [подключении к виртуальным машинам Linux](how-to-use-remote-desktop-linux-student.md).

