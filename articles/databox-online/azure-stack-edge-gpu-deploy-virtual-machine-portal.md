---
title: Развертывание виртуальных машин на Azure Stack пограничных Pro с помощью портал Azure
description: Узнайте, как создавать виртуальные машины на Azure Stack крае Pro и управлять ими с помощью портал Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure compute on Azure Stack Edge Pro device so I can use it to transform the data before sending it to Azure.
ms.openlocfilehash: c11a89d91693075ca54c0689223dcf2af06df521
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105568517"
---
# <a name="deploy-vms-on-your-azure-stack-edge-pro-gpu-device-via-the-azure-portal"></a>Развертывание виртуальных машин на устройстве с Azure Stack ребра Pro GPU с помощью портал Azure

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

Вы можете создавать виртуальные машины и управлять ими на Azure Stack пограничном устройстве с помощью портал Azure, шаблонов, Azure PowerShell командлетов и с помощью сценариев Azure CLI/Python. В этой статье описывается создание виртуальной машины на Azure Stack пограничном устройстве и управление ею с помощью портал Azure. 

Эта статья относится к Azure Stackным графическим процессорам версии Pro, Azure Stackм пограничных Pro R и Azure Stack пограничных устройств R. 

> [!IMPORTANT] 
> Рекомендуется включить многофакторную проверку подлинности для пользователя, который управляет виртуальными машинами, развернутыми на устройстве, из облака.
        
## <a name="vm-deployment-workflow"></a>Рабочий процесс развертывания виртуальной машины

Общая сводка по рабочему процессу развертывания включает следующие действия:

1. Включите сетевой интерфейс для вычислений на устройстве Azure Stack пограничных устройств. Будет создан виртуальный коммутатор на указанном сетевом интерфейсе.
1. Включение облачного управления виртуальными машинами из портал Azure.
1. Отправьте виртуальный жесткий диск в учетную запись хранения Azure с помощью Обозреватель службы хранилища. 
1. Используйте переданный виртуальный жесткий диск для скачивания виртуального жесткого диска на устройство и создайте образ виртуальной машины на основе виртуального жесткого диска. 
1. Используйте ресурсы, созданные на предыдущих шагах.
    1. Созданный образ виртуальной машины.
    1. VSwitch, связанный с сетевым интерфейсом, на котором вы включили вычисление.
    1. Подсеть, связанная с VSwitch.

    Создайте или укажите следующие ресурсы в строке:
    1. Имя виртуальной машины, выберите поддерживаемый размер виртуальной машины и учетные данные для входа для виртуальной машины. 
    1. Создайте новые диски данных или подключите существующие диски данных.
    1. Настройте статический или динамический IP-адрес для виртуальной машины. При предоставлении статического IP-адреса выберите из бесплатного IP-адреса в диапазоне подсети сетевого интерфейса, включенного для вычислений.

    Используйте приведенные выше ресурсы для создания виртуальной машины.


## <a name="prerequisites"></a>Предварительные условия

Прежде чем приступить к созданию виртуальных машин на устройстве и управлению ими с помощью портал Azure, убедитесь в том, что:

1. Параметры сети на устройстве Azure Stack погранично Pro завершены, как описано в разделе [Шаг 1. настройка Azure Stack ребра Pro Device](./azure-stack-edge-gpu-connect-resource-manager.md#step-1-configure-azure-stack-edge-pro-device).

    1. Вы включили сетевой интерфейс для вычислений. IP-адрес этого сетевого интерфейса используется для создания виртуального коммутатора для развертывания виртуальной машины. В локальном пользовательском интерфейсе устройства перейдите к разделу **Вычисление**. Выберите сетевой интерфейс, который будет использоваться для создания виртуального коммутатора.

        > [!IMPORTANT] 
        > Для служб вычислений можно настроить только один порт.

    1. Включите службы вычислений на сетевом интерфейсе. Azure Stack Edge Pro создает виртуальный коммутатор, соответствующий этому сетевому интерфейсу, и управляет им.

1. У вас есть доступ к виртуальному жесткому диску Windows или Linux, который будет использоваться для создания образа ВИРТУАЛЬНОЙ машины, которую вы собираетесь создать.

## <a name="deploy-a-vm"></a>Развертывание виртуальной машины

Выполните следующие действия, чтобы создать виртуальную машину на Azure Stack пограничном устройстве.

### <a name="add-a-vm-image"></a>Добавление образа виртуальной машины

1. Отправьте виртуальный жесткий диск в учетную запись хранения Azure. Выполните действия, описанные в разделе [Отправка виртуального жесткого диска с помощью обозреватель службы хранилища Azure](../devtest-labs/devtest-lab-upload-vhd-using-storage-explorer.md).

1. В портал Azure перейдите к ресурсу Azure Stackного периметра для устройства Azure Stack. Перейдите к **границе вычисление > виртуальных машин**.

    ![Добавление виртуальной машины, образ 1](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-1.png)

1. Выберите **виртуальные машины** , чтобы открыть страницу **Обзор** . **Включите** управление облаком виртуальных машин.
    ![Добавление виртуальной машины, образ 2](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-2.png)

1. Первым шагом является Добавление образа виртуальной машины. Вы уже отправили виртуальный жесткий диск в учетную запись хранения на предыдущем шаге. Этот виртуальный жесткий диск будет использоваться для создания образа виртуальной машины.

    Выберите **Добавить образ** , чтобы скачать виртуальный жесткий диск из учетной записи хранения и добавить его на устройство. Процесс загрузки занимает несколько минут в зависимости от размера виртуального жесткого диска и пропускной способности Интернета, доступной для загрузки. 

    ![Добавление виртуальной машины, образ 3](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-3.png)

1. В колонке **Добавление изображения** введите следующие параметры. Выберите **Добавить**.


    |Параметр  |Описание  |
    |---------|---------|
    |Скачать из хранилища BLOB-объектов    |Перейдите к расположению большого двоичного объекта хранилища в учетной записи хранения, куда был отправлен виртуальный жесткий диск.         |
    |Загрузить в    | Автоматически задается текущим устройством, на котором развертывается виртуальная машина.        |
    |Сохранить образ как      | Имя образа виртуальной машины, создаваемого на основе виртуального жесткого диска, отправленного в учетную запись хранения.        |
    |Тип ОС     |Выберите Windows или Linux в качестве операционной системы виртуального жесткого диска, который будет использоваться для создания образа виртуальной машины.         |
   

    ![Добавление виртуальной машины, образ 4](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-6.png)

1. VHD скачивается, и создается образ виртуальной машины. Создание образа занимает несколько минут. Вы увидите уведомление об успешном завершении образа виртуальной машины.

    ![Добавление образа виртуальной машины 5](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-8.png)


1. После успешного создания образа виртуальной машины он добавляется в список образов в колонке **образы** .
    ![Добавление виртуальной машины, образ 6](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-9.png)

    Колонка **развертывания** обновится, чтобы указать состояние развертывания.

    ![Добавление образа виртуальной машины 7](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-10.png)

    Недавно добавленное изображение также отображается на странице **Обзор** .
    ![Добавление виртуальной машины, образ 8](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-image-11.png)


### <a name="add-a-vm"></a>Добавление виртуальной машины

Выполните следующие действия, чтобы создать виртуальную машину после создания образа виртуальной машины.

1. На странице **Обзор** выберите **добавить виртуальную машину**.

    ![Добавить виртуальную машину 1](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-1.png)

1. На вкладке **Основные сведения** введите следующие параметры.


    |Параметр |Описание  |
    |---------|---------|
    |Имя виртуальной машины     |         |
    |Образ —     | Выберите из образов виртуальных машин, доступных на устройстве.        |
    |Размер     | Выберите один из [поддерживаемых размеров виртуальных машин](azure-stack-edge-gpu-virtual-machine-sizes.md).        |
    |Имя пользователя     | Используйте имя пользователя по умолчанию *azureuser*.        |
    |Authentication type (Тип проверки подлинности)    | Выберите открытый ключ SSH или пароль, определенный пользователем.       |
    |Пароль     | Введите пароль для входа на виртуальную машину. Пароль должен иметь длину не менее 12 символов и соответствовать определенным [требованиям к сложности](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).        |
    |Подтверждение пароля    | Введите пароль еще раз.        |


    ![Добавить виртуальную машину 2](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-basics-1.png)

    По завершении выберите **Next: Диски**.

1. На вкладке **диски** присоедините диски к виртуальной машине. 
    
    1. Вы можете **создать и подключить новый диск** или **подключить существующий диск**.

        ![Добавить виртуальную машину 3](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-disks-1.png)

    1. Выберите **создать и присоединить новый диск**. В колонке **Создание нового диска** укажите имя диска и размер в гиб.

        ![Добавить виртуальную машину 4](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-disks-2.png)

    1.  Повторите указанные выше действия, чтобы добавить дополнительные диски. После создания дисков они отображаются на вкладке **диски** .

        ![Добавить виртуальную машину 5](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-disks-3.png)

        Нажмите **Далее: сеть**.

1. На вкладке " **сети** " вы настроите сетевое подключение для виртуальной машины.

    
    |Параметр  |Описание |
    |---------|---------|
    |Виртуальная сеть    | В раскрывающемся списке выберите виртуальный коммутатор, созданный на Azure Stack пограничном устройстве при включении вычислений на сетевом интерфейсе.    |
    |Подсеть     | Это поле автоматически заполняется подсетью, связанной с сетевым интерфейсом, на котором вы включили вычисление.         |
    |IP-адрес     | Укажите статический или динамический IP-адрес для виртуальной машины. Статический IP-адрес должен быть доступным свободным IP-адресом из указанного диапазона подсети.        |

    ![Добавить виртуальную машину 6](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-networking-1.png)

    По завершении выберите **Next: Просмотр + создание**.

1. На вкладке **Проверка и создание** проверьте спецификации виртуальной машины и нажмите кнопку **создать**.

    ![Добавить виртуальную машину 7](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-review-create-1.png)

1. Создание виртуальной машины начинается и может занять до 20 минут. Чтобы отслеживать создание виртуальной машины, можно переходить к разделу **развертывания** .

    ![Добавить виртуальную машину 8](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-deployments-page-1.png)

    
1. После успешного создания ВИРТУАЛЬНОЙ машины страница **обзора** обновится, чтобы ОТОБРАЗИТЬ новую виртуальную машину.

    ![Добавить виртуальную машину 9](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-overview-page-1.png)

1. Выберите только что созданную виртуальную машину для перехода к **виртуальным машинам**.

    ![Добавить виртуальную машину 10](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-page-1.png)

    Выберите виртуальную машину, чтобы просмотреть сведения. 

    ![Добавить виртуальную машину 11](media/azure-stack-edge-gpu-deploy-virtual-machine-portal/add-virtual-machine-details-1.png)

## <a name="connect-to-a-vm"></a>Подключение к виртуальной машине

В зависимости от того, была создана виртуальная машина Windows или Linux, действия для подключения могут отличаться. Вы не можете подключиться к виртуальным машинам, развернутым на устройстве, с помощью портал Azure. Чтобы подключиться к виртуальной машине Linux или Windows, необходимо выполнить следующие действия.

### <a name="connect-to-linux-vm"></a>Подключение к виртуальной машине Linux

Выполните следующие действия, чтобы подключиться к виртуальной машине Linux.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-linux.md)]

### <a name="connect-to-windows-vm"></a>Подключение к виртуальной машине Windows

Выполните следующие действия, чтобы подключиться к виртуальной машине Windows.

[!INCLUDE [azure-stack-edge-gateway-connect-vm](../../includes/azure-stack-edge-gateway-connect-virtual-machine-windows.md)]

## <a name="next-steps"></a>Дальнейшие действия

Чтобы узнать, как администрировать устройство Azure Stack пограничной Pro, см. статью[использование локального пользовательского веб-интерфейса для администрирования Azure Stack пограничной Pro](azure-stack-edge-manage-access-power-connectivity-mode.md).