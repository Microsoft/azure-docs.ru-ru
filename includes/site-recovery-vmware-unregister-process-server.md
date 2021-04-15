---
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: include
ms.date: 04/28/2019
ms.author: ramamill
ms.openlocfilehash: bff9457220efbb6a6b318550a5b2dd9af4ae230f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96019206"
---
Выполните шаги для своего сценария.

### <a name="unregister-a-connected-process-server"></a>Отмена регистрации подключенного сервера обработки

1. Установите удаленное подключение к серверу обработки с правами администратора.
2. На **панели управления** выберите **Программы > Удалить программу**.
3. Удалите программу **Microsoft Azure Site Recovery Mobility Service/Master Target Server** (Microsoft Azure Site Recovery Mobility Service/Главный целевой сервер).
4. Удалите программу **Microsoft Azure Site Recovery Configuration/Process Server** (Microsoft Azure Site Recovery Configuration/Сервер обработки).
5. После удаления программ в шагах 3 и 4 удалите **Microsoft Azure Site Recovery Configuration/Process Server Dependencies** (Microsoft Azure Site Recovery Configuration/Зависимости сервера обработки).

### <a name="unregister-a-disconnected-process-server"></a>Отмена регистрации отключенного сервера обработки

Выполняйте эти шаги, только если вы не можете иным способом восстановить компьютер, на котором установлен сервер обработки.

1. Войдите на сервер конфигурации с правами администратора.
2. Откройте административную командную строку и перейдите в каталог `%ProgramData%\ASR\home\svsystems\bin`.
3. Выполните эту команду, чтобы получить список с одним или несколькими серверами обработки.

    ```
    perl Unregister-ASRComponent.pl -IPAddress <IP_of_Process_Server> -Component PS
    ```
    - Х. No: серийный номер сервера обработки.
    - IP/Name: IP-адрес и имя компьютера, на котором выполняется сервер обработки.
    - Heartbeat: последний пульс от компьютера с сервером обработки.
    ![Снимок экрана: текст с данными о серверах обработки и строка "Выберите один из указанных выше серверов, чтобы отменить его регистрацию".(media/site-recovery-vmware-unregister-process-server/Unregister-cmd.PNG)

4. Укажите серийный номер сервера обработки, для которого вы хотите отменить регистрацию.
5. При отмене регистрации для сервера обработки все сведения о нем будут удалены из системы, а также отобразится следующее сообщение: **Отмена регистрации успешно выполнена для имя_сервера> (IP-адрес_сервера)** .

