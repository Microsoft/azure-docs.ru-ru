---
title: Учебник по шифрованию и активации устройства Azure Stack Edge Mini R на портале Azure | Документация Майкрософт
description: В учебнике по развертыванию Azure Stack Edge Mini R описано, как выполнить шифрование физического устройства и активировать его.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/22/2020
ms.author: alkohli
ms.openlocfilehash: 92dfff13d964418ea5519d30c7a1b91621d5e98e
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106061644"
---
# <a name="tutorial-activate-azure-stack-edge-mini-r"></a>Руководство по активации Azure Stack Edge Mini R

В этом учебнике описывается, как активировать устройство Azure Stack Edge Mini R, используя локальный пользовательский веб-интерфейс.

Процесс активации может занять около 15 минут.

Из этого учебника вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * Предварительные требования
> * Активация физического устройства

## <a name="prerequisites"></a>Предварительные требования

Перед настройкой устройства Azure Stack Edge Mini R убедитесь в следующем:

* Для физического устройства: 
    
    - Вы установили физическое устройство, как описано в статье [Установка Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-install.md).
    - Вы настроили сеть и параметры вычислительной сети, как описано в статье [Учебник. Настройка сети для Azure Stack Edge с GPU](azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy.md).
    - Вы загрузили собственные сертификаты на своем устройстве, когда изменили имя устройства или домена DNS на странице **Устройства**. Эти действия подробно описаны в статье о [настройке сертификатов, VPN и шифрования неактивных данных](azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption.md). Если вы этого не сделали, активация будет заблокирована.
    - Вы настроили параметры шифрования неактивных данных для устройства. Если вы не выполнили этот шаг, во время активации устройства вы увидите сообщение об ошибке, и активация будет заблокирована. Дополнительные сведения см. в статье о [настройке сертификатов, VPN и шифрования неактивных данных](azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption.md).
    
* У вас есть ключ активации из службы Azure Stack Edge, который вы создали для управления устройством Azure Stack Edge Mini R. Дополнительные сведения см. в статье [Подготовка к развертыванию Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-prep.md).


## <a name="activate-the-device"></a>Активация устройства

1. В локальном веб-интерфейсе устройства перейдите на страницу **Начало работы**.
2. На плитке **Активация** выберите **Активировать**. 

    ![Экран со страницей 1 "Сведения об облаке" в локальном пользовательском веб-интерфейсе](./media/azure-stack-edge-mini-r-deploy-activate/activation-1.png)
    
3. На панели **Активация** сделайте следующее:
    1. Введите **ключ активации**, который вы получили при работе с разделом [Получение ключа активации для Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md#get-the-activation-key).

    1. Вы можете включить упреждающий сбор журналов, чтобы разрешить корпорации Майкрософт собирать журналы в зависимости от состояния работоспособности устройства. Журналы, собранные таким образом, отправляются в учетную запись хранения Azure.
    
    1. Нажмите кнопку **Применить**.

    ![Страница "Сведения об облаке" в локальном пользовательском веб-интерфейсе 2](./media/azure-stack-edge-mini-r-deploy-activate/activation-2.png)


5. Сначала устройство активируется. Затем вам будет предложено скачать файл ключа.
    
    ![Страница "Сведения об облаке" в локальном пользовательском веб-интерфейсе 3](./media/azure-stack-edge-mini-r-deploy-activate/activation-3.png)
    
    Выберите **Download and continue** (Скачать и продолжить) и сохраните файл *device-serial-no.json* в безопасном расположении вне устройства. **Этот файл ключа содержит ключи восстановления для диска ОС и дисков данных на вашем устройстве.** Эти ключи могут потребоваться для быстрого восстановления системы в будущем.

    Вот содержимое файла *json*:

        
    ```json
    {
      "Id": "<Device ID>",
      "DataVolumeBitLockerExternalKeys": {
        "hcsinternal": "<BitLocker key for data disk>",
        "hcsdata": "<BitLocker key for data disk>"
      },
      "SystemVolumeBitLockerRecoveryKey": "<BitLocker key for system volume>",
      "SEDEncryptionExternalKey": "xZM/vC7GxjqHZ3VMo7xSs/wH9rRsF/PNM+mOsZ+GaL0=",
      "ServiceEncryptionKey": "<Azure service encryption key>"
    }
    ```        
 
    Различные ключи описаны в следующей таблице:
    
    |Поле  |Описание  |
    |---------|---------|
    |`Id`    | Это идентификатор устройства.        |
    |`DataVolumeBitLockerExternalKeys`|Это ключи BitLocker для дисков данных, которые используются для восстановления локальных данных на устройстве.|
    |`SystemVolumeBitLockerRecoveryKey`| Это ключ BitLocker для системного тома. Этот ключ помогает восстановить конфигурацию системы и системные данные для устройства. |
    |`SEDEncryptionExternalKey`| Указанный пользователем или созданный системой ключ используется для защиты самозашифрованных дисков данных со встроенным шифрованием. |
    |`ServiceEncryptionKey`| Этот ключ защищает данные, проходящие через службу Azure. Этот ключ гарантирует, что компрометация службы Azure не приведет к компрометации хранимой информации. |

6. Перейдите на страницу **Обзор**. Состояние устройства должно отображаться как **Активировано**.

    ![Страница "Сведения об облаке" в локальном пользовательском веб-интерфейсе 4](./media/azure-stack-edge-mini-r-deploy-activate/activation-4.png)
 
Активация устройства завершена. Теперь в него можно добавить общие папки.

Если во время активации возникнут проблемы, ознакомьтесь со статьей [Устранение неполадок при активации на устройстве Azure Stack Edge Pro с GPU](azure-stack-edge-gpu-troubleshoot-activation.md#activation-errors).

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * Предварительные требования
> * Активация физического устройства

Чтобы узнать, как передавать данные с помощью устройства Azure Stack Edge Mini R, см. следующее руководство:

> [!div class="nextstepaction"]
> [Передача данных с помощью Azure Stack Edge Mini R](./azure-stack-edge-gpu-deploy-add-shares.md)