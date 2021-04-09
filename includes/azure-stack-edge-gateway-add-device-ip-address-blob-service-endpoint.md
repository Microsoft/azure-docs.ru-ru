---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 11/21/2019
ms.author: alkohli
ms.openlocfilehash: 0c6845f081ccbe42e70964eaa939d58597e3b69b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96185549"
---
1. Перейдите в локальный веб-интерфейс устройства и войдите на устройство. Убедитесь, что устройство разблокировано.

2. Перейдите на страницу **Параметры сети**. Запишите IP-адрес устройства для сетевого интерфейса, используемого для подключения к клиенту.

3. Если используется клиент Windows для удаленного доступа, запустите **Блокнот** от имени администратора и откройте файл hosts, расположенный в папке `C:\Windows\System32\Drivers\etc`.

4. Добавьте следующую запись в ваш файл hosts: `<Device IP address> <Blob service endpoint>`

    У вас есть конечная точка службы BLOB-объектов в учетной записи хранения Edge, созданной на портале Azure. Вам потребуется только суффикс конечной точки службы BLOB-объектов.

    Для ссылки используйте следующее изображение. Сохраните файл `hosts`.

    ![Изменение файла hosts в клиенте Windows](media/azure-stack-edge-gateway-add-device-ip-address-blob-service-endpoint/hosts-file-1.png)