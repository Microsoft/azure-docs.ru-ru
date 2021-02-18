---
title: Руководство по настройке сертификатов для устройства Azure Stack Edge Mini R на портале Azure | Документация Майкрософт
description: Руководство по развертыванию Azure Stack Edge Mini R поможет вам настроить сертификаты на физическом устройстве.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/21/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure certificates for Azure Stack Edge Mini R  so I can use it to transfer data to Azure.
ms.openlocfilehash: c3a09242b895234c96c64d9e23449d980e47e387
ms.sourcegitcommit: 5a999764e98bd71653ad12918c09def7ecd92cf6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/16/2021
ms.locfileid: "100546743"
---
# <a name="tutorial-configure-certificates-vpn-encryption-for-your-azure-stack-edge-mini-r"></a>Руководство по Настройка сертификатов, VPN и шифрования для Azure Stack Edge Mini R

В этом руководстве объясняется, как настроить сертификаты, VPN и шифрование неактивных данных для устройства Azure Stack Edge Mini R, используя локальный пользовательский веб-интерфейс.

Необходимое время на выполнение этого шага зависит от выбранного варианта и существующего потока управления сертификатами в вашей среде.

В этом руководстве вы рассмотрите следующее:

> [!div class="checklist"]
>
> * Предварительные требования
> * Настройка сертификатов для физического устройства
> * Настройка VPN
> * Настройка шифрования неактивных данных

## <a name="prerequisites"></a>Предварительные требования

Перед настройкой устройства Azure Stack Edge Mini R удостоверьтесь в том, что выполнено следующее:

* Установлено физическое устройство, как описано в статье об [установке Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-install.md).

* Если вы планируете применить собственные сертификаты:
    - эти сертификаты должны быть подготовлены в соответствующем формате, включая сертификат цепочки подписывания. Дополнительные сведения о сертификате см. в статье [Управление сертификатами](azure-stack-edge-gpu-manage-certificates.md).

    - Если ваше устройство развернуто в Azure для государственных организаций, Azure Government Secret или Azure Government Top Secret, а не в общедоступном облаке Azure, перед активацией устройства необходимо получить сертификат цепочки подписывания. 
    Дополнительные сведения см. в статье об [управлении сертификатами](azure-stack-edge-gpu-manage-certificates.md).


## <a name="configure-certificates-for-device"></a>Настройка сертификатов для устройства

1. Сертификаты настраиваются на странице **Сертификаты**. В зависимости от того, изменялось ли имя устройства или домен DNS на странице **Устройство**, вам будут доступны для выбора следующие варианты применения сертификатов.

    - Если вы на предыдущем шаге не изменяли имя устройства или домен DNS, предложенные по умолчанию, и не планируете использовать собственные сертификаты, просто пропустите этот шаг и переходите к следующему. Для начала работы устройство автоматически создает самозаверяющие сертификаты. 

       <!-- ![Local web UI "Certificates" page](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/certificate-1.png)-->

    - Если вы изменили имя устройства или домен DNS, состояние активации здесь будет иметь значение **Недопустимо**. 

        ![Страница "Сертификаты" в локальном пользовательском веб-интерфейсе (2)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/certificate-2.png)    

        Выберите сертификат, чтобы просмотреть сведения о его состоянии.

        <!--![Local web UI "Certificates" page 3](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-1a.png)-->  

        Сертификат имеет состояние **Недопустимо**, так как сертификаты не отражают обновленных имен устройства или домена DNS (которые используются в имени субъекта и альтернативном имени субъекта). Для успешной активации устройства можно применить собственные подписанные сертификаты конечной точки и соответствующие цепочки подписывания. Для этого сначала добавьте цепочку подписывания, а затем передайте сертификаты конечной точки. Дополнительные сведения см. в разделе [Использование собственных сертификатов](#bring-your-own-certificates).


    - Если вы изменили имя устройства или домен DNS, но не предоставили собственные сертификаты, **активация будет невозможна**.


#### <a name="bring-your-own-certificates"></a>Использование собственных сертификатов

Вы уже добавили цепочку подписывания на это устройство на предыдущем шаге. Теперь на него можно передать сертификаты конечной точки, сертификат узла, сертификат локального пользовательского интерфейса и сертификат VPN. Выполните описанные ниже действия, чтобы добавить собственные сертификаты.

1. Чтобы отправить сертификат, выберите на странице **Сертификат** действие **+ Добавить сертификат**.

    [![Страница "Сертификаты" в локальном пользовательском веб-интерфейсе (4)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-1.png)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-1.png#lightbox)


1. Все готово к отправке сертификатов. Например, вы можете отправить сертификаты конечных точек Azure Resource Manager и (или) Хранилища BLOB-объектов.

    ![Страница "Сертификаты" в локальном пользовательском веб-интерфейсе (6)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-3.png)

1. Кроме того, вы можете отправить сертификаты локального пользовательского веб-интерфейса. Когда вы завершите отправку сертификатов, вам будет предложено перезапустить браузер и очистить кэш. После этого войдите в локальный пользовательский веб-интерфейс.  

    ![Страница "Сертификаты" в локальном пользовательском веб-интерфейсе (7)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-4.png)

1. Кроме того, вы можете отправить сертификат узла.


    ![Страница "Сертификаты" в локальном пользовательском веб-интерфейсе (8)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-5.png)

1. Теперь можно отправить сертификат для VPN.
        
    ![Страница "Сертификаты" в локальном пользовательском веб-интерфейсе](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-6.png)

1. В любое время вы можете выбрать сертификат, просмотреть сведения о нем и убедиться, что они соответствуют отправленному сертификату.

    [![Страница "Сертификаты" в локальном пользовательском веб-интерфейсе (9)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-7.png)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-7.png#lightbox)

    Теперь на странице "Сертификаты" должны появиться все добавленные вами сертификаты.

    [![Страница "Сертификаты" в локальном пользовательском веб-интерфейсе (10)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-8.png)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-8.png#lightbox)  

    > [!NOTE]
    > Во всех облачных конфигурациях, кроме общедоступного облака Azure (то есть в Azure для государственных организаций или Azure Stack Hub), перед активацией необходимо предоставить сертификаты цепочки подписывания.


## <a name="configure-vpn"></a>Настройка VPN

1. На плитке **Security** (Безопасность) выберите команду **Configure** (Настроить) для VPN. 

    Чтобы настроить VPN, сначала убедитесь, что в Azure выполнены все необходимые настройки. Дополнительные сведения см. в статье о [настройке VPN для устройства Azure Stack Edge Mini R с помощью PowerShell](azure-stack-edge-placeholder.md). После этого можно выполнить настройку в локальном пользовательском интерфейсе.
    
    1. На странице VPN выберите элемент **Configure** (Настроить).
        ![Настройка VPN в локальном пользовательском интерфейсе, шаг 1](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/configure-vpn-1.png)

    1. В колонке **Configure VPN** (Настройка VPN) выполните следующие действия:

        1. Укажите телефонную книгу в качестве входных данных.
        2. Укажите JSON-файл с диапазоном IP-адресов для центра обработки данных Azure в качестве входных данных. Этот файл можно скачать по адресу [https://www.microsoft.com/download/details.aspx?id=56519](https://www.microsoft.com/download/details.aspx?id=56519).
        3. Выберите регион **eastus**.
        4. Нажмите кнопку **Применить**.

        ![Настройка VPN в локальном пользовательском интерфейсе, шаг 2](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/configure-vpn-2.png)
    
    1. Настройте параметр IP address ranges to be accessed using VPN only (Доступ к диапазонам IP-адресов только с помощью VPN). 
    
        - В разделе **IP address ranges to be accessed using VPN only** (Доступ к диапазонам IP-адресов только с помощью VPN), выберите элемент **Configure** (Настроить).
        - Укажите диапазон IPv4-адресов, который вы выбрали для виртуальной сети Azure.
        - Нажмите кнопку **Применить**.
    
        ![Настройка VPN в локальном пользовательском интерфейсе, шаг 3](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/configure-vpn-3.png)

Теперь устройство готово к настройке шифрования. Настройте шифрование неактивных данных.


## <a name="enable-encryption"></a>Включение шифрования

1. На плитке **Security** (Безопасность) выберите действие **Configure** (Настроить) для шифрования неактивных данных. Это обязательный параметр. Вы сможете активировать устройство только после правильной настройки этого параметра. 

    ![Страница Encryption at rest (Шифрование неактивных данных) в локальном пользовательском веб-интерфейсе, шаг 1](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-1.png)

    При производстве после развертывания образа на устройстве активируется шифрование BitLocker на уровне тома. После получения устройства необходимо настроить шифрование неактивных данных. Пул носителей и тома создаются повторно, и вы можете предоставить ключи BitLocker, чтобы включить шифрование неактивных данных и создать второй уровень шифрования для своих неактивных данных.

1. В области **Encryption-at-rest** (Шифрование неактивных данных) укажите ключ в кодировке Base-64 длиной 32 символа (AES-256 бит). Эта настройка выполняется однократно, а ключ используется для защиты фактического ключа шифрования. 

    ![Страница Encryption at rest (Шифрование неактивных данных) в локальном пользовательском веб-интерфейсе, шаг 2](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-2.png)

    Вы также можете создать этот ключ автоматически.

    ![Страница Encryption at rest (Шифрование неактивных данных) в локальном пользовательском веб-интерфейсе, шаг 3](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-3.png)

1. Нажмите кнопку **Применить**. Эта операция занимает несколько минут, а состояние операции отображается на плитке **Security** (Безопасность).

    ![Страница Encryption at rest (Шифрование неактивных данных) в локальном пользовательском веб-интерфейсе, шаг 4](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-3.png)

1. Когда отобразится состояние **Completed** (Завершено), вернитесь к странице **Get started** (Начало работы).

Теперь устройство готово к активации.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы рассмотрите следующее:

> [!div class="checklist"]
>
> * Предварительные требования
> * Настройка сертификатов для физического устройства
> * Настройка VPN
> * Настройка шифрования неактивных данных

Сведения об активировании устройства Azure Stack Edge Mini R см. в следующей статье:

> [!div class="nextstepaction"]
> [Активация устройства Azure Stack Edge Mini R](./azure-stack-edge-mini-r-deploy-activate.md)
