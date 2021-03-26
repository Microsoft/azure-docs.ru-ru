---
title: Настройка TLS 1,2 на клиентах Windows, обращающихся к Azure Stack устройстве GPU Pro
description: Описание настройки TLS 1,2 на клиентах Windows, обращающихся к Azure Stack устройства GPU с пограничными устройствами.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 47a6d3bbebdf3b2b14b1c40d7ea8fc93b4d19c6f
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105565339"
---
# <a name="configure-tls-12-on-windows-clients-accessing-azure-stack-edge-pro-device"></a>Настройка TLS 1,2 на клиентах Windows, обращающихся к Azure Stack пограничному устройству Pro

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Если вы используете клиент Windows для доступа к устройству Azure Stack погранично Pro, вам потребуется настроить TLS 1,2 на клиенте. В этой статье содержатся ресурсы и рекомендации по настройке TLS 1,2 в клиенте Windows. 

Приведенные здесь рекомендации основаны на тестировании, выполняемом на клиенте под управлением Windows Server 2016.

## <a name="configure-tls-12-for-current-powershell-session"></a>Настройка TLS 1,2 для текущего сеанса PowerShell

Выполните следующие действия, чтобы настроить TLS 1,2 на клиенте.

1. Запустите оболочку PowerShell от имени администратора.
2. Чтобы задать TLS 1,2 для текущего сеанса PowerShell, введите:
  
    ```azurepowershell
    $TLS12Protocol = [System.Net.SecurityProtocolType] 'Ssl3 , Tls12'
    [System.Net.ServicePointManager]::SecurityProtocol = $TLS12Protocol
    ```
## <a name="configure-tls-12-on-client"></a>Настройка TLS 1,2 на клиенте

Если вы хотите установить TLS 1,2 для своей среды, следуйте указаниям в следующих документах:

- [Общие сведения о включении TLS 1,2](/windows-server/security/tls/tls-registry-settings#tls-12)
- [Включение TLS 1.2 на клиентах](/configmgr/core/plan-design/security/enable-tls-1-2-client)
- [Включение TLS 1.2 на серверах сайта и удаленных системах сайта](/configmgr/core/plan-design/security/enable-tls-1-2-server)
- [Протоколы в TLS/SSL (Schannel SSP)](/windows-server/security/tls/manage-tls#configuring-tls-ecc-curve-order)
- Комплекты [шифров](/windows-server/security/tls/tls-registry-settings#tls-12). в частности, [для настройки порядка](/windows-server/security/tls/manage-tls#configuring-tls-cipher-suite-order) комплектов шифров TLS убедитесь в том, что вы перечислите текущие комплекты шифров и не могли бы быть в следующем списке:

    - TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
    - TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384

    Эти наборы шифров также можно добавить, напрямую изменив параметры реестра.

    ```azurepowershell
    New-ItemProperty -Path "$HklmSoftwarePath\Policies\Microsoft\Cryptography\Configuration\SSL\00010002" -Name "Functions"  -PropertyType String -Value ("TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384, TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384,TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384")
    ```

- Как задать эллиптические кривые

    Убедитесь, что вы перечислите текущие кривые эллиптических кривых и в начале всех недостающих из следующего списка:

    - P-256 
    - P-384

    Можно также добавить эти эллиптические кривые, напрямую изменив параметры реестра.
    
    ```azurepowershell
    New-ItemProperty -Path "$HklmSoftwarePath\Policies\Microsoft\Cryptography\Configuration\SSL\00010002" -Name "EccCurves" -PropertyType MultiString -Value @("NistP256", "NistP384")
    ```
    
    - [Установите минимальный размер обмена ключами RSA равным 2048](/windows-server/security/tls/tls-registry-settings#keyexchangealgorithm---client-rsa-key-sizes).



## <a name="next-steps"></a>Дальнейшие действия

[Подключение к Azure Resource Manager.](./azure-stack-edge-gpu-connect-resource-manager.md)