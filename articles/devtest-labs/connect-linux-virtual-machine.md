---
title: Подключение к виртуальным машинам Linux в Azure DevTest Labs
description: Узнайте, как подключиться к виртуальной машине Linux в лаборатории (Azure DevTest Labs)
ms.topic: how-to
ms.date: 07/17/2020
ms.openlocfilehash: 52fe245f85034a4c6300615ad8fb6040c1168298
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86532188"
---
# <a name="connect-to-a-linux-vm-in-your-lab-azure-devtest-labs"></a>Подключение к виртуальной машине Linux в лаборатории (Azure DevTest Labs)
В этой статье показано, как подключиться к виртуальной машине Linux в лаборатории. 

## <a name="connect-to-a-linux-vm"></a>Подключение к виртуальной машине Linux
1. Войдите на [портал Azure](https://portal.azure.com).
1. В строке поиска найдите и выберите **DevTest Labs**. 

    :::image type="content" source="./media/connect-linux-virtual-machine/search-select.png" alt-text="Найдите и выберите DevTest Labs.":::    
1. В списке лабораторий выберите свою **лабораторию**.

    :::image type="content" source="./media/connect-linux-virtual-machine/select-lab.png" alt-text="Выбор лаборатории":::            
1. На домашней странице лаборатории выберите виртуальную машину Linux из списка **мои виртуальные машины** . 

    :::image type="content" source="./media/connect-linux-virtual-machine/select-linux-vm.png" alt-text="Выбор виртуальной машины Linux":::        
5. На странице **Обзор** можно увидеть полное доменное имя (FQDN) или IP-адрес виртуальной машины. Вы также можете просмотреть порт, как показано на следующем рисунке.

    :::image type="content" source="./media/connect-linux-virtual-machine/vm-overview.png" alt-text="Полное доменное имя виртуальной машины":::    

    Обратите внимание, что кнопка **подключить** неактивна, несмотря на то, что виртуальная машина запущена. Это по разработке.
6.  Используйте SSH для подключения к виртуальной машине Linux. В следующем примере выполняется подключение к виртуальной машине с полным доменным именем `mydtl07172452621450000.eastus.cloudapp.azure.com` с именем пользователя `vmuser` и портом `51637` . Введите пароль для пользователя, чтобы подключиться к виртуальной машине. 

    ```bash
    ssh vmuser@mydtl07172452621450000.eastus.cloudapp.azure.com -p 51637
    ```

    Для подключения к виртуальной машине можно использовать такие средства [, как](https://www.putty.org/) выводимый или любой другой SSH-клиент. 

    После подключения с помощью SSH можно установить и настроить среду рабочего стола ([Xfce](https://www.xfce.org)) и удаленный рабочий стол ([xrdp](http://xrdp.org)).  Подробные сведения см. в [статье Установка и настройка удаленный рабочий стол для подключения к виртуальной машине Linux в Azure](../virtual-machines/linux/use-remote-desktop.md). 

## <a name="next-steps"></a>Дальнейшие действия
[Как подключиться к виртуальной машине Windows](connect-windows-virtual-machine.md)
