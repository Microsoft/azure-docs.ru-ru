---
ms.openlocfilehash: dfb887004cd29b5bd9f1d9886b7dfa5f43c83dbe
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99531143"
---
MP4-файлы записываются в каталог на пограничном устройстве, указанном в файле *.env* с помощью ключа VIDEO_OUTPUT_FOLDER_ON_DEVICE. Если использовано значение по умолчанию, результаты должны находиться в папке */var/media/* .

Чтобы воспроизвести MP4-файл:

1. Перейдите к группе ресурсов, найдите виртуальную машину, а затем подключитесь с помощью Бастиона Azure.

    ![Группа ресурсов](../../../media/quickstarts/resource-group.png)
    
    ![ВМ](../../../media/quickstarts/virtual-machine.png)
1. Войдите, используя учетные данные, созданные при [настройке ресурсов Azure](../../../detect-motion-emit-events-quickstart.md#set-up-azure-resources). 
1. В командной строке перейдите в соответствующий каталог. Расположение по умолчанию — */var/media*. В каталоге должны отобразиться файлы MP4.

    ![Выходные данные](../../../media/quickstarts/samples-output.png) 

1. Используйте [безопасное копирование (SCP)](../../../../../virtual-machines/linux/copy-files-to-linux-vm-using-scp.md) для копирования файлов на локальный компьютер. 
1. Воспроизведите файлы с помощью [проигрывателя мультимедиа VLC](https://www.videolan.org/vlc/) или любого другого проигрывателя, поддерживающего формат MP4.
