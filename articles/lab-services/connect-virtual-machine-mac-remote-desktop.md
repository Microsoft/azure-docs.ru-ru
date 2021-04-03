---
title: Подключение к виртуальной машине Служб лаборатории Azure из Mac | Документация Майкрософт
description: Узнайте, как подключиться с компьютера Mac к виртуальной машине в Службах лаборатории Azure.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 674bea13093e185ae991148c2cf96599a273d0bc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "85444630"
---
# <a name="connect-to-a-vm-using-remote-desktop-protocol-on-a-mac"></a>Подключение к виртуальной машине с помощью протокола удаленного рабочего стола на Mac
В этом разделе показано, как учащийся может подключиться к виртуальной машине задания с компьютера Mac с помощью RDP.

## <a name="install-microsoft-remote-desktop-on-a-mac"></a>Установка удаленного рабочего стола (Майкрософт) на компьютере Mac
1. Откройте App Store на компьютере Mac и выполните поиск по запросу **удаленный рабочий стол (Майкрософт)** .

    ![Удаленный рабочий стол (Майкрософт)](./media/how-to-use-classroom-lab/install-ms-remote-desktop.png)
1. Установите последнюю версию удаленного рабочего стола (Майкрософт). 

## <a name="access-the-vm-from-your-mac-using-rdp"></a>Доступ к виртуальной машине с компьютера Mac с помощью RDP
1. Откройте файл **RDP**, скачанный на компьютере с установленным **удаленным рабочим столом (Майкрософт)** . При этом должно начаться подключение к виртуальной машине. 

    ![Подключение к виртуальной машине](./media/how-to-use-classroom-lab/connect-linux-vm.png)
1. Выберите **Continue** (Продолжить), если появится следующее предупреждение. 

    ![Предупреждение о проблеме с сертификатом](./media/how-to-use-classroom-lab/certificate-error.png)
1. Должна отобразиться виртуальная машина. 

    > [!NOTE]
    > Ниже приведен пример для виртуальной машины Linux CentOS. 

    ![ВМ](./media/how-to-use-classroom-lab/vm-ui.png)


## <a name="next-steps"></a>Дальнейшие действия
Сведения о подключении к виртуальным машинам Linux с помощью протокола RDP см. в статье [Использование удаленного рабочего стола для виртуальных машин Linux](how-to-use-remote-desktop-linux-student.md)


