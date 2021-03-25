---
title: Учебник по подключению к приложению Azure Stack Edge Mini R на портале Azure
description: Сведения о том, как подключиться к устройству Azure Stack Edge Mini R с помощью локального пользовательского веб-интерфейса.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/20/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to connect and activate Azure Stack Edge Mini R so I can use it to transfer data to Azure.
ms.openlocfilehash: fe76391a5cfce8d7d39e47131db108ab87e5aed5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96464994"
---
# <a name="tutorial-connect-to-azure-stack-edge-mini-r"></a>Руководство по подключению к Azure Stack Edge Mini R

В этом учебнике описывается, как подключиться к устройству Azure Stack Edge Mini R, используя локальный пользовательский веб-интерфейс.

Процесс подключения может занять около 5 минут.

В этом руководстве вы рассмотрите следующее:

> [!div class="checklist"]
>
> * Предварительные условия
> * подключение к физическому устройству;



## <a name="prerequisites"></a>Предварительные требования

Перед установкой и настройкой устройства Azure Stack Edge проверьте следующее.

* Вы установили физическое устройство, как описано в статье [Руководство по установке Azure Stack Edge](azure-stack-edge-mini-r-deploy-install.md).


## <a name="connect-to-the-local-web-ui-setup"></a>Подключение к настройке локального пользовательского веб-интерфейса

1. Настройте адаптер Ethernet на компьютере, который используется для подключения к устройству Azure Stack Edge Pro, указав статический IP-адрес 192.168.100.5 и подсеть 255.255.255.0.

2. Подключите компьютер к порту 1 на вашем устройстве. При подключении компьютера к устройству напрямую (без коммутатора) используйте перекрестный кабель или USB-адаптер Ethernet. С помощью иллюстрации ниже определите порт 1 устройства.

    ![Подключение кабелей для Wi-Fi](./media/azure-stack-edge-mini-r-deploy-install/wireless-cabled.png)

[!INCLUDE [azure-stack-edge-gateway-delpoy-connect](../../includes/azure-stack-edge-gateway-deploy-connect.md)]


## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * Предварительные условия
> * подключение к физическому устройству;


Сведения о настройке параметров сети на устройстве см. в следующей статье:

> [!div class="nextstepaction"]
> [Настройка сети](./azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy.md)
