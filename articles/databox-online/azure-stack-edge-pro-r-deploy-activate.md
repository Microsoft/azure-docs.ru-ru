---
title: Учебник. Активация устройства Azure Stack Edge Pro R на портале Azure
description: В учебнике по развертыванию Azure Stack Edge Pro R описывается, как активировать физическое устройство.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 02/23/2021
ms.author: alkohli
ms.openlocfilehash: 2908f9c3cb39debed4f5a1bea38d15ccf6a22822
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106059961"
---
# <a name="tutorial-activate-azure-stack-edge-pro-r-device"></a>Руководство по активации устройства Azure Stack Edge Pro R

В этом учебнике описывается, как активировать устройство Azure Stack Edge Pro R, используя локальный пользовательский веб-интерфейс.

Процесс активации может занять около 5 минут.  

Из этого учебника вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * Предварительные требования
> * Активация физического устройства

## <a name="prerequisites"></a>Предварительные требования

Перед настройкой устройства Azure Stack Edge Pro R проверьте следующее:

* Для физического устройства: 
    
    - Вы установили физическое устройство, как описано в статье [Установка Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-install.md).
    - Вы настроили сеть и параметры вычислительной сети, как описано в статье [Учебник. Настройка сети для Azure Stack Edge с GPU](azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy.md).
    - Вы загрузили собственные или созданные сертификаты устройства на своем устройстве, когда изменили имя устройства или домена DNS на странице **Устройства**. Если вы не выполнили этот шаг, во время активации устройства вы увидите сообщение об ошибке, и активация будет заблокирована. Дополнительные сведения см. в статье [Учебник. Настройка сертификатов для Azure Stack Edge с GPU](azure-stack-edge-placeholder.md).
    
* У вас есть ключ активации из службы Azure Stack Edge, который вы создали для управления устройством Azure Stack Edge Pro R. Дополнительные сведения см. в статье [Подготовка к развертыванию Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md).


## <a name="activate-the-device"></a>Активация устройства

1. В локальном веб-интерфейсе устройства перейдите на страницу **Начало работы**.
2. На плитке **Активация** выберите **Активировать**. 

    ![Экран со страницей 1 "Сведения об облаке" в локальном пользовательском веб-интерфейсе](./media/azure-stack-edge-pro-r-deploy-activate/activate-1.png)
    
3. На панели **Активация** сделайте следующее:
    1. Введите **ключ активации**, который вы получили при работе с разделом [Получение ключа активации для Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md#get-the-activation-key).

    1. Вы можете включить упреждающий сбор журналов, чтобы разрешить корпорации Майкрософт собирать журналы в зависимости от состояния работоспособности устройства. Журналы, собранные таким образом, отправляются в учетную запись хранения Azure.
    
    1. Нажмите кнопку **Применить**.

    ![Страница "Сведения об облаке" в локальном пользовательском веб-интерфейсе 2](./media/azure-stack-edge-pro-r-deploy-activate/activate-2.png)


5. Сначала устройство активируется. Затем вам будет предложено скачать файл ключа.
    
    ![Страница "Сведения об облаке" в локальном пользовательском веб-интерфейсе 3](./media/azure-stack-edge-pro-r-deploy-activate/activate-3.png)
    
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
      "ServiceEncryptionKey": "<Azure service encryption key>"
    }
    ```
        
 
    Различные ключи описаны в следующей таблице:
    
    |Поле  |Описание  |
    |---------|---------|
    |`Id`    | Это идентификатор устройства.        |
    |`DataVolumeBitLockerExternalKeys`|Это ключи BitLockers для дисков данных, которые используются для восстановления локальных данных на устройстве.|
    |`SystemVolumeBitLockerRecoveryKey`| Это ключ BitLocker для системного тома. Этот ключ помогает восстановить конфигурацию системы и системные данные для устройства. |
    |`ServiceEncryptionKey`| Этот ключ защищает данные, проходящие через службу Azure. Этот ключ гарантирует, что компрометация службы Azure не приведет к компрометации хранимой информации. |

6. Перейдите на страницу **Обзор**. Состояние устройства должно отображаться как **Активировано**.

    ![Страница "Сведения об облаке" в локальном пользовательском веб-интерфейсе 4](./media/azure-stack-edge-gpu-deploy-activate/activate-4.png)
 
Активация устройства завершена. Теперь в него можно добавить общие папки.

Если во время активации возникнут проблемы, ознакомьтесь со статьей [Устранение неполадок при активации на устройстве Azure Stack Edge Pro с GPU](azure-stack-edge-gpu-troubleshoot-activation.md#activation-errors).

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как выполнять такие задачи:

> [!div class="checklist"]
> * Предварительные требования
> * Активация физического устройства

Чтобы узнать, как передавать данные с помощью устройства Azure Stack Edge Pro R, см. следующее руководство:

> [!div class="nextstepaction"]
> [Передача данных с помощью Azure Stack Edge Pro R](./azure-stack-edge-gpu-deploy-add-shares.md)