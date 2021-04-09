---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 12/07/2018
ms.author: alkohli
ms.openlocfilehash: d01cf90c42efbe611748027d0ea47ff8af3e5621
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94553213"
---
1. Войдите на устройство Data Box. Убедитесь, что оно разблокировано.

    ![Снимок экрана: панель мониторинга, на которой для устройства отображается состояние "Разблокировано".](media/data-box-add-device-ip/data-box-connect-via-rest-1.png)

2. Перейдите к **Задать сетевые интерфейсы**. Запишите IP-адрес устройства для сетевого интерфейса, используемого для подключения к клиенту.

    ![Снимок экрана: область "Параметры сети", в которой отображается IP-адрес.](media/data-box-add-device-ip/data-box-connect-via-rest-2.png)

3. Перейдите в пункт меню **Подключиться и копировать** и щелкните **Rest**.

    ![Снимок экрана: область "Подключиться и копировать", где можно выбрать параметр доступа Rest.](media/data-box-add-device-ip/data-box-connect-via-rest-3.png)

4. В диалоговом окне **Доступ к учетной записи хранения и отправка данных** скопируйте **конечную точку службы BLOB-объектов**.

    ![Снимок экрана: диалоговое окно "Доступ к учетной записи хранения и отправка данных", где можно скопировать конечную точку службы больших двоичных объектов.](media/data-box-add-device-ip/data-box-connect-via-rest-4.png)

5. Откройте **Блокнот** от имени администратора, а затем откройте файл **hosts**, расположенный в `C:\Windows\System32\Drivers\etc`.
6. Добавьте следующую запись в ваш файл **hosts**: `<device IP address> <Blob service endpoint>`
7. Для ссылки используйте следующее изображение. Сохраните файл **hosts**.

    ![Снимок экрана: документ в Блокноте с добавленным IP-адресом и конечной точкой службы BLOB-объектов.](media/data-box-add-device-ip/data-box-connect-via-rest-5.png)
