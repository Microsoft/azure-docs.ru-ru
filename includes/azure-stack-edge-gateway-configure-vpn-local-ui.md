---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/27/2020
ms.author: alkohli
ms.openlocfilehash: fa65a7354112a2b220686372459b348d45832dd9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96467375"
---
Выполните следующие действия в локальном веб-ИНТЕРФЕЙСе устройства. Этот шаг занимает около 15 минут, включая отправку файла конфигурации VPN (или файла тегов службы). 

1. Перейдите в раздел **Configuration > VPN**. Нажмите кнопку **Настроить**.

    ![Настройка локального пользовательского интерфейса 1](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-1.png)

2. В колонке **Настройка VPN** выполните следующие действия:

    - Включите **параметры VPN**.
    - Укажите **общий секрет VPN**. Это общий ключ, предоставленный при создании ресурса VPN-подключения Azure.
    - Укажите **IP-адрес шлюза VPN**. Это IP-адрес шлюза локальной сети Azure.
    - Для параметра **PFS group** (Группа PFS) выберите значение **None** (Нет). 
    - Для параметра **DH group** (Группа DH) выберите значение **Group2**.
    - Для параметра **IPsec integrity method** (Метод проверки целостности IPsec) выберите значение **SHA256**.
    - Для **констант преобразования шифра IPsec** выберите **GCMAES256**.
    - Для параметра **IPsec authentication transform constants** (Константы преобразования проверки подлинности IPsec) выберите значение **GCMAES256**.
    - Для параметра **IKE encryption method** (Метод шифрования IKE) выберите значение **AES256**.
    - Нажмите кнопку **Применить**.

        ![Настройка локального пользовательского интерфейса 2](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-2.png)

    Дополнительные сведения о поддерживаемых алгоритмах шифрования см. в статье [требования к шифрованию и VPN-шлюзы Azure](../articles/vpn-gateway/vpn-gateway-about-compliance-crypto.md#ipsecike-policy-faq). 

3. Чтобы отправить файл конфигурации VPN-маршрута, выберите **Отправить**. 

    ![Настройка локального пользовательского интерфейса 3](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-3.png)

    - Перейдите *к файлу тегов* службы, который был скачан в локальной системе на предыдущем шаге.
    - Выберите регион Azure, связанный с устройством, виртуальной сетью и шлюзами.
    - Нажмите кнопку **Применить**.

        ![Настройка локального пользовательского интерфейса 4](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-4.png)
    
    Отправка занимает около 7-8 минут на устройстве.

4. Чтобы добавить клиентские маршруты, настройте диапазоны IP-адресов для доступа только с помощью VPN. 

    - В разделе **IP address ranges to be accessed using VPN only** (Доступ к диапазонам IP-адресов только с помощью VPN) выберите **Configure** (Настроить).
    - Укажите допустимый диапазон IPv4 и выберите **Add** (Добавить). Повторите шаги, чтобы добавить другие диапазоны.
    - Нажмите кнопку **Применить**.

        ![Настройка локального пользовательского интерфейса 5](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-5.png)

