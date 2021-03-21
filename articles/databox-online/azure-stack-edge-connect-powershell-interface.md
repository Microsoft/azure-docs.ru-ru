---
title: Подключение к устройству Pro с помощью интерфейса Windows PowerShell и управление им Microsoft Azure Stack Документация Майкрософт
description: Описывает, как подключиться к Azure Stack пограничному Pro через интерфейс Windows PowerShell и затем управлять им.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/30/2020
ms.author: alkohli
ms.openlocfilehash: c9c6247f021b7af4cfdd899ffd4b6bd178f2256c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96345599"
---
# <a name="manage-an-azure-stack-edge-pro-fpga-device-via-windows-powershell"></a>Управление устройством с Azure Stack пограничными устройствами FPGA с помощью Windows PowerShell

Azure Stack пограничных решений Pro позволяет обрабатывать данные и передавать их по сети в Azure. В этой статье описываются некоторые задачи по настройке и управлению для устройства на Azure Stack пограничной Pro. Для управления устройством можно использовать портал Azure, локальный веб-интерфейс или интерфейс Windows PowerShell.

В этой статье рассматриваются задачи, выполняемые с помощью интерфейса PowerShell. 

Эта статья содержит следующие процедуры.

- Подключитесь к интерфейсу PowerShell.
- Создать пакет поддержки.
- Передача сертификата
- Сброс устройства
- Просмотр сведений об устройстве
- Получение журналов вычислений
- Мониторинг и устранение неполадок модулей вычислений

## <a name="connect-to-the-powershell-interface"></a>Подключитесь к интерфейсу PowerShell.

[!INCLUDE [Connect to admin runspace](../../includes/data-box-edge-gateway-connect-minishell.md)]

## <a name="create-a-support-package"></a>Создать пакет поддержки.

[!INCLUDE [Create a support package](../../includes/data-box-edge-gateway-create-support-package.md)]

## <a name="upload-certificate"></a>Передача сертификата

[!INCLUDE [Upload certificate](../../includes/data-box-edge-gateway-upload-certificate.md)]

Можно также отправить сертификаты IoT Edge, чтобы обеспечить безопасное подключение между устройством IoT Edge и подчиненными устройствами, которые могут к нему подключаться. Необходимо установить три файла (формат *PEM* ):

- Сертификат корневого ЦС или ЦС владельца
- Сертификат ЦС устройства
- Закрытый ключ устройства 

В следующем примере показано использование этого командлета для установки сертификатов IoT Edge:

```
Set-HcsCertificate -Scope IotEdge -RootCACertificateFilePath "\\hcfs\root-ca-cert.pem" -DeviceCertificateFilePath "\\hcfs\device-ca-cert.pem\" -DeviceKeyFilePath "\\hcfs\device-private-key.pem" -Credential "username"
```
При запуске этого командлета вам будет предложено ввести пароль для сетевой папки.

Дополнительные сведения о сертификатах см. в [подAzure IoT Edge сертификаты](../iot-edge/iot-edge-certs.md) или [Установка сертификатов на шлюзе](../iot-edge/how-to-create-transparent-gateway.md).

## <a name="view-device-information"></a>Просмотр сведений об устройстве
 
[!INCLUDE [View device information](../../includes/data-box-edge-gateway-view-device-info.md)]

## <a name="reset-your-device"></a>выполнить сброс устройства;

[!INCLUDE [Reset your device](../../includes/data-box-edge-gateway-deactivate-device.md)]

## <a name="get-compute-logs"></a>Получение журналов вычислений

Если на устройстве настроена роль вычислений, можно также получить журналы вычислений с помощью интерфейса PowerShell.

1. [Подключитесь к интерфейсу PowerShell](#connect-to-the-powershell-interface).
2. Используйте `Get-AzureDataBoxEdgeComputeRoleLogs` для получения журналов вычислений для устройства.

    В следующем примере показано использование этого командлета:

    ```powershell
    Get-AzureDataBoxEdgeComputeRoleLogs -Path "\\hcsfs\logs\myacct" -Credential "username" -FullLogCollection
    ```

    Ниже приведено описание параметров, используемых для командлета.
    - `Path`: Укажите сетевой путь к общей папке, в которой нужно создать пакет журнала вычислений.
    - `Credential`: Укажите имя пользователя для сетевой папки. При выполнении этого командлета необходимо указать пароль для общей папки.
    - `FullLogCollection`: Этот параметр гарантирует, что пакет журнала будет содержать все журналы вычислений. По умолчанию пакет журналов содержит только подмножество журналов.

## <a name="monitor-and-troubleshoot-compute-modules"></a>Мониторинг и устранение неполадок модулей вычислений

[!INCLUDE [Monitor and troubleshoot compute modules](../../includes/azure-stack-edge-monitor-troubleshoot-compute.md)]

## <a name="exit-the-remote-session"></a>Выход из удаленного сеанса

Чтобы выйти из удаленного сеанса PowerShell, закройте окно PowerShell.

## <a name="next-steps"></a>Дальнейшие действия

- Разверните [Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md) на портале Azure.