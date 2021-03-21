---
title: Создание сертификатов с помощью средства проверки готовности центра Microsoft Azure Stack | Документация Майкрософт
description: В этой статье описывается, как создавать запросы на сертификаты, а затем получать и устанавливать сертификаты на устройстве Azure Stack с помощью средства проверки готовности центра Azure Stack.
services: Azure Stack Edge Pro
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: a04243093b89b6a2498efc48f80cbd7a47d57337
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102437729"
---
# <a name="create-certificates-for-your-azure-stack-edge-pro-using-azure-stack-hub-readiness-checker-tool"></a>Создание сертификатов для Azure Stack ребра Pro с помощью средства проверки готовности центра Azure Stack 

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается, как создать сертификаты для Azure Stack пограничной Pro с помощью средства проверки готовности центра Azure Stack. 

## <a name="using-azure-stack-hub-readiness-checker-tool"></a>Использование средства проверки готовности центра Azure Stack

Используйте средство проверки готовности центра Azure Stack для создания запросов на подписывание сертификатов (представители по обслуживанию клиентов) для развертывания устройств с Azure Stackм пограничным Pro. Эти запросы можно создать после того, как будет размещен заказ на устройство Pro Azure Stack и дождаться прибытия устройства.

> [!NOTE]
> Используйте это средство только для целей тестирования и разработки, а не для рабочих устройств. 

Вы можете использовать средство проверки готовности Azure Stack Hub (AzsReadinessChecker) для запроса следующих сертификатов.

- Сертификат Azure Resource Manager
- Локальный сертификат пользовательского интерфейса
- Сертификат узла
- Сертификат BLOB-объекта
- VPN-сертификат


## <a name="prerequisites"></a>Предварительные условия

Чтобы создать специалиста по обслуживанию Azure Stack для развертывания устройств с пограничным назначением, убедитесь в том, что: 

- У вас есть клиент под управлением Windows 10 или Windows Server 2016 или более поздней версии. 
- Средство проверки готовности к установке центра Microsoft Azure Stack загружено [из коллекция PowerShell](https://aka.ms/AzsReadinessChecker) в этой системе.
- Для сертификатов имеются следующие сведения.
  - Имя устройства
  - Серийный номер узла
  - Внешнее полное доменное имя (FQDN)

## <a name="generate-certificate-signing-requests"></a>Создание запросов на подпись сертификата

Выполните следующие действия, чтобы подготовить сертификаты устройств для Azure Stack пограничной Pro:

1. Запустите PowerShell от имени администратора (5,1 или более поздней версии).
2. Установите средство проверки готовности центра Azure Stack. В командной строке PowerShell введите следующую команду: 

    ```azurepowershell
    Install-Module -Name Microsoft.AzureStack.ReadinessChecker
    ```

    Чтобы получить установленную версию, введите:  

    ```azurepowershell
    Get-InstalledModule -Name Microsoft.AzureStack.ReadinessChecker  | ft Name, Version 
    ```

3. Создайте каталог для всех сертификатов, если у вас его еще нет. Тип: 
    
    ```azurepowershell
    New-Item "C:\certrequest" -ItemType Directory
    ``` 
    
4. Чтобы создать запрос на сертификат, укажите следующие сведения. При создании VPN-сертификата некоторые из этих входных данных не применяются.
    
    |Входные данные |Описание  |
    |---------|---------|
    |`OutputRequestPath`|Путь к файлу на локальном клиенте, на котором должны быть созданы запросы на сертификат.        |
    |`DeviceName`|Имя устройства на странице **устройства** в локальном веб-интерфейсе вашего устройства. <br> Это поле не требуется для VPN-сертификата.         |
    |`NodeSerialNumber`|Серийный номер узла устройства на странице " **сеть** " в локальном пользовательском веб-интерфейсе вашего устройства. <br> Это поле не требуется для VPN-сертификата.       |
    |`ExternalFQDN`|Значение Днсдомаин на странице **устройства** в локальном веб-интерфейсе вашего устройства.         |
    |`RequestType`|Тип запроса может быть для `MultipleCSR` разных сертификатов для различных конечных точек, или `SingleCSR` для всех конечных точек — один сертификат. <br> Это поле не требуется для VPN-сертификата.     |

    Для всех сертификатов, кроме VPN-сертификата, введите: 
    
    ```azurepowershell
    $edgeCSRparams = @{
        CertificateType = 'AzureStackEdgeDEVICE'
        DeviceName = 'myTEA1'
        NodeSerialNumber = 'VM1500-00025'
        externalFQDN = 'azurestackedge.contoso.com'
        requestType = 'MultipleCSR'
        OutputRequestPath = "C:\certrequest"
    }
    New-AzsCertificateSigningRequest @edgeCSRparams
    ```

    При создании VPN-сертификата введите: 

    ```azurepowershell
    $edgeCSRparams = @{
        CertificateType = 'AzureStackEdgeVPN'
        externalFQDN = 'azurestackedge.contoso.com'
        OutputRequestPath = "C:\certrequest"
    }
    New-AzsCertificateSigningRequest @edgeCSRparams
    ```

    
5. Файлы запросов сертификатов находятся в каталоге, указанном в приведенном выше параметре Аутпутрекуестпас. При использовании `MultipleCSR` параметра вы увидите следующие четыре файла с `.req` расширением:

    
    |Имена файлов  |Тип запроса на сертификат  |
    |---------|---------|
    |Начиная с `DeviceName`     |Локальный запрос сертификата веб-интерфейса      |
    |Начиная с `NodeSerialNumber`     |Запрос сертификата узла         |
    |Начиная с `login`     |Запрос сертификата конечной точки Azure Resource Manager       |
    |Начиная с `wildcard`     |Запрос сертификата хранилища BLOB-объектов. Он содержит подстановочный знак, так как он охватывает все учетные записи хранения, которые можно создать на устройстве.          |
    |Начиная с `AzureStackEdgeVPNCertificate`     |Запрос сертификата VPN-клиента.         |

    Вы также увидите папку INF. Он содержит файл сведений об управлении. <ребр-DeviceName> в виде открытого текста, поясняющего сведения о сертификате.  


6. Отправьте эти файлы в центр сертификации (как внутренний, так и общедоступный). Убедитесь, что ваш центр сертификации создает сертификаты с помощью созданного запроса, который соответствует требованиям к сертификату для сертификатов [узлов](azure-stack-edge-gpu-manage-certificates.md#node-certificates), [конечным точкам](azure-stack-edge-gpu-manage-certificates.md#endpoint-certificates)и [локальным сертификатам пользовательского интерфейса](azure-stack-edge-gpu-manage-certificates.md#local-ui-certificates), которые соответствуют потребностям сертификата узла Azure Stack.

## <a name="prepare-certificates-for-deployment"></a>Подготовка сертификатов к развертыванию

Файлы сертификатов, получаемые от вашего центра сертификации (ЦС), должны быть импортированы и экспортированы с помощью свойств, соответствующих требованиям к сертификату устройства Azure Stack ребра Pro. Выполните следующие действия в той же системе, в которой были созданы запросы подписи сертификата.

- Чтобы импортировать сертификаты, выполните действия, описанные в статье [Импорт сертификатов на клиентах, обращающихся к устройству Azure Stack ребра Pro](azure-stack-edge-gpu-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).

- Чтобы экспортировать сертификаты, выполните действия, описанные в разделе [Экспорт сертификатов с клиента, обращающегося к устройству Azure Stack ребра Pro](azure-stack-edge-gpu-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).


## <a name="validate-certificates"></a>Проверка сертификатов

Во-первых, вы создадите правильную структуру папок и поместите сертификаты в соответствующие папки. Только после этого вы проверите сертификаты с помощью средства.

1. Запустите оболочку PowerShell от имени администратора.

2. Чтобы создать соответствующую структуру папок, введите в командной строке:

    `New-AzsCertificateFolder -CertificateType AzureStackEdge -OutputPath "$ENV:USERPROFILE\Documents\AzureStackCSR"`

3. Преобразуйте пароль PFX в защищенную строку. Тип:       

    `$pfxPassword = Read-Host -Prompt "Enter PFX Password" -AsSecureString` 

4. Затем проверьте сертификаты. Тип:

    `Invoke-AzsCertificateValidation -CertificateType AzureStackEdge -DeviceName mytea1 -NodeSerialNumber VM1500-00025 -externalFQDN azurestackedge.contoso.com -CertificatePath $ENV:USERPROFILE\Documents\AzureStackCSR\AzureStackEdge -pfxPassword $pfxPassword`

## <a name="next-steps"></a>Дальнейшие действия

[Развертывание устройства Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-prep.md)
