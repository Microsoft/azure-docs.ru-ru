---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 12/02/2019
ms.author: alkohli
ms.openlocfilehash: 203c977fe9109cd8b2b6de561e975e20aacf700e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96185576"
---
Если вы используете Обозреватель службы хранилища впервые, выполните следующие действия.

1. На панели команд сверху последовательно выберите **Изменить > Разработка для API Azure Stack**.

    ![Настройка Обозревателя службы хранилища](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-1.png)

2. Перезапустите Обозреватель службы хранилища, чтобы изменения вступили в силу.


Чтобы подключится к учетной записи хранения и проверить подключения, выполните следующие действия.

1. В Обозревателе службы хранилища выберите учетные записи хранения. Щелкните пункт правой кнопкой мыши и выберите параметр **Подключиться к хранилищу Azure**. 

    ![Настройка Обозревателя службы хранилища 2](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-2.png)

2. В диалоговом окне **Подключение к службе хранилища Azure** выберите **Использовать имя и ключ учетной записи хранения**.

    ![Настройка Обозревателя службы хранилища 3](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-3.png)

2. В диалоговом окне **Подключение с именем и ключом** выполните приведенные ниже действия.

    1. Введите отображаемое имя для своей учетной записи хранения Edge. 
    2. Укажите имя учетной записи хранения Edge.
    3. Вставьте ключ доступа, полученный из локальных API устройства через Azure Resource Manager.
    4. Для пункта "Домен хранилища" выберите **Другое (введите ниже)** , а затем укажите суффикс конечной точки службы BLOB-объектов в формате: `<appliance name>.<DNSdomain>`. 
    5. Установите флажок **Использовать HTTP**, так как передача осуществляется через *HTTP*. 
    6. Выберите **Далее**.

    ![Настройка Обозревателя службы хранилища 4](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-4.png)    

3. Проверьте сведения в диалоговом окне **Сводка подключения**. Щелкните **Подключить**.

    ![Настройка Обозревателя службы хранилища 5](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-5.png)

4. Успешно добавленная учетная запись отображается в левой области Обозревателя службы хранилища, а к ее имени будет добавлен текст (Внешняя, Другая). Выберите **Контейнеры больших двоичных объектов**, чтобы просмотреть контейнер.

    ![Просмотр контейнеров больших двоичных объектов](media/azure-stack-edge-gateway-verify-connection-storage-explorer/connect-with-storage-explorer-6.png)

Следующим шагом является проверка правильности передачи данных через это подключение.

Выполните следующие действия, чтобы загрузить данные в свою учетную запись хранения Edge на устройстве. Они должны автоматически получить многоуровневую привязку к сопоставленной учетной записи хранения Azure.

1. Выберите контейнер, в который вы хотите загрузить данные в учетной записи хранения Edge. Щелкните **Отправить**, а затем выберите **Отправить файлы**.

    ![Проверка передачи данных](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-1.png)

2. В диалоговом окне **Отправить файлы** перейдите к файлам, которые необходимо отправить, и выберите их. Выберите **Далее**.

    ![Проверка передачи данных 2](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-2.png)

3. Убедитесь, что файлы отправлены. Отправленные файлы появятся в контейнере.

    ![Проверка передачи данных 3](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-3.png)

4. Затем вы подключитесь к учетной записи хранения Azure, которая была сопоставлена с этой учетной записью хранения Edge. Любые данные, которые отправляются в учетную запись хранения Edge, должны автоматически распределяться по уровням в учетной записи хранения Azure. 
    
    Чтобы получить строку подключения для учетной записи хранения Azure, последовательно выберите **Учетная запись хранения Azure > Ключи доступа** и скопируйте строку подключения.

    ![Проверка передачи данных 4](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-5.png)

    Используйте строку подключения, чтобы присоединить учетную запись хранения Azure.  

    ![Проверка передачи данных 5](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-4.png)


5. Проверьте сведения в диалоговом окне **Сводка подключения**. Щелкните **Подключить**.

    ![Проверка передачи данных 6](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-6.png)

6. Вы увидите, что файлы, которые вы отправили в учетную запись хранения Edge, были перенесены в учетную запись хранения Azure.

    ![Проверка передачи данных 7](media/azure-stack-edge-gateway-verify-connection-storage-explorer/verify-data-transfer-7.png)
