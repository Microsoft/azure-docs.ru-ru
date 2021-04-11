---
title: Учебник. Подготовка портала Azure и среды центра обработки данных к развертыванию Azure Stack Edge Pro R
description: Первый учебник по развертыванию Azure Stack Edge Pro R включает подготовку портала Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 01/22/2021
ms.author: alkohli
ms.openlocfilehash: 7ddd3941c3001ba5c12a06d9f8710e2fc6328433
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106059867"
---
# <a name="tutorial-prepare-to-deploy-azure-stack-edge-pro-r"></a>Руководство по подготовке к развертыванию Azure Stack Edge Pro R

Это руководство — первое в серии руководств по развертыванию, необходимых для полного развертывания Azure Stack Edge Pro R. Здесь содержатся сведения о подготовке портала Azure для развертывания ресурса Azure Stack Edge. В этом учебнике используется устройство Azure Stack Edge Pro R с одним узлом, поставляемое с источником бесперебойного питания (ИБП).

Чтобы завершить процесс установки и настройки, вам потребуются права администратора. Подготовка портала займет менее 10 минут.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Создать новый ресурс
> * Получение ключа активации.

### <a name="get-started"></a>Начало работы

Чтобы развернуть Azure Stack Edge Pro R, ознакомьтесь с приведенными ниже учебниками в указанной последовательности.

| Действие | Документы для изучения |
| --- | --- |
| **Подготовка** |При подготовке к предстоящему развертыванию необходимо выполнить следующие шаги. |
| **[Контрольный список конфигурации развертывания](#deployment-configuration-checklist)** |Этот контрольный список служит для сбора и записи информации до и во время развертывания. |
| **[Предварительные условия для развертывания](#prerequisites)** |Эти предварительные требования служат для проверки готовности среды к развертыванию. |
|  | |
|**Учебники по развертыванию** |Эти учебники необходимы для развертывания устройства Azure Stack Edge Pro R в рабочей среде. |
|**[1. Подготовка портала Azure для устройства](azure-stack-edge-pro-r-deploy-prep.md)** |Создайте и настройте ресурсы Azure Stack Edge, прежде чем устанавливать физическое устройство Azure Stack Box Edge. |
|**[2. Установка устройства](azure-stack-edge-pro-r-deploy-install.md)**|Осмотрите физическое устройство и подключите к нему кабели.  |
|**[3. Подключение к устройству](azure-stack-edge-pro-r-deploy-connect.md)** |После установки устройства подключитесь к локальному пользовательскому веб-интерфейсу устройства.  |
|**[4. Настройка параметров сети](azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy.md)** |Настройте сеть, включая параметры вычислительной сети и веб-прокси для вашего устройства.   |
|**[5. Настройка параметров устройства](azure-stack-edge-pro-r-deploy-set-up-device-update-time.md)** |Назначьте имя устройства и домен DNS, настройте сервер обновлений и время устройства. |
|**[6. Настройка параметров безопасности](azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption.md)** |Настройте сертификат, VPN и шифрование неактивных данных для устройства. Используйте сертификаты, созданные устройством, или подключите собственные сертификаты.   |
|**[7. Активация устройства](azure-stack-edge-pro-r-deploy-activate.md)** |Используйте ключ активации из службы, чтобы активировать устройство. Устройство готово к настройке общих папок SMB или NFS или подключению через REST. |
|**[8. Настройка вычислений](azure-stack-edge-gpu-deploy-configure-compute.md)** |Настройте роль вычислений на устройстве. При этом также создается кластер Kubernetes. |

Теперь можно приступать к настройке портала Azure.

## <a name="deployment-configuration-checklist"></a>Контрольный список конфигурации развертывания

Перед развертыванием устройства Azure Stack Edge Pro необходимо будет собрать данные для настройки программного обеспечения на нем. Заблаговременная подготовка некоторых из этих сведений поможет оптимизировать процесс развертывания устройства в вашей среде. Используйте [контрольный список конфигурации развертывания Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-checklist.md) для записи сведений о конфигурации при развертывании устройства.

## <a name="prerequisites"></a>Предварительные требования

Ниже расположены настройки, необходимые для вашего ресурса Azure Stack Edge, устройства Azure Stack Edge и сети центров обработки данных.

### <a name="for-the-azure-stack-edge-resource"></a>Для ресурса Azure Stack Edge

[!INCLUDE [Azure Stack Edge resource prerequisites](../../includes/azure-stack-edge-gateway-resource-prerequisites.md)]

### <a name="for-the-azure-stack-edge-device"></a>Для устройства Azure Stack Edge

Перед развертыванием физического устройства убедитесь, что:

- Вы просмотрели сведения о безопасности для этого устройства, указанные в статье: [Рекомендации по безопасности устройства Azure Stack Edge Pro R](azure-stack-edge-pro-r-safety.md).
[!INCLUDE [Azure Stack Edge device prerequisites](../../includes/azure-stack-edge-gateway-device-prerequisites.md)] 



### <a name="for-the-datacenter-network"></a>Для сети центра обработки данных

Перед тем как начать, убедитесь в следующем.

- Сеть в центре обработки данных настроена в соответствии с требованиями к сетевым характеристикам устройства Azure Stack Edge. Дополнительные сведения см. в статье [Требования к системе для Azure Stack Edge Pro R](azure-stack-edge-pro-r-system-requirements.md).

- Для нормальных рабочих условий устройства у вас должна быть:

    - пропускная способность не менее 10 Мбит/с для скачивания, чтобы обеспечить обновление данных на устройстве;
    - выделенная пропускная способность не менее 20 Мбит/с для передачи и скачивания файлов.

## <a name="create-a-new-resource"></a>Создать новый ресурс

Если у вас есть ресурс Azure Stack Edge для управления физическими устройствами, пропустите этот шаг и перейдите к [получению ключа активации](#get-the-activation-key).

### <a name="portal"></a>[Портал](#tab/azure-portal)

Чтобы создать ресурс Azure Stack Edge, выполните указанные ниже действия на портале Azure.

1. Используйте свои учетные данные Microsoft Azure для входа на портал Azure по этому URL-адресу: [https://portal.azure.com](https://portal.azure.com).

2. В левой области выберите **+ Создать ресурс**. Найдите и выберите **Azure Stack Edge или Шлюз Azure Data Box**. Нажмите кнопку **создания**. 

3. Выберите подписку, которую вы хотите использовать для устройства Azure Stack Edge Pro. Выберите страну, в которую вы хотите отправить это физическое устройство. Выберите **Показать устройства**.

    ![Создание ресурса 1](media/azure-stack-edge-pro-r-deploy-prep/create-resource-1.png)

4. Выберите тип устройства. В разделе **Azure Stack Edge** выберите **Azure Stack Edge Pro R**, а затем щелкните **Выбрать**. Если вы столкнетесь с проблемами или не сможете выбрать тип устройства, перейдите к разделу [Устранение проблем с заказами](azure-stack-edge-troubleshoot-ordering.md).

    ![Создание ресурса 2](media/azure-stack-edge-pro-r-deploy-prep/create-resource-2.png)

5. В зависимости от бизнес-потребности вы можете выбрать **Azure Stack Edge Pro R с одним узлом** или **Azure Stack Edge Pro R с одним узлом и ИБП**.  

    ![Создание ресурса 3](media/azure-stack-edge-pro-r-deploy-prep/create-resource-3.png)

6. На вкладке **Основы** введите или выберите следующие **сведения о проекте**.
    
    |Параметр  |Значение  |
    |---------|---------|
    |Подписка    |Сведения о подписке заполняются автоматически на основе ранее выбранного значения. Подписка привязана к учетной записи выставления счетов. |
    |Группа ресурсов  |Создайте группу или выберите имеющуюся.<br>Узнайте больше о [группах ресурсов Azure](../azure-resource-manager/management/overview.md).     |

7. Введите или выберите следующие **сведения об экземпляре**.

    |Параметр  |Значение  |
    |---------|---------|
    |Имя   | Понятное имя для идентификации ресурса.<br>Имя включает от 2 до 50 символов, содержащих буквы, цифры и дефисы.<br> Имя начинается и заканчивается буквой или цифрой.        |
    |Регион     |Список всех регионов, в которых доступны ресурсы Azure Stack Edge, приведен на странице [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). При использовании Azure для государственных организаций доступны все регионы для государственных организаций, как показано на странице [Регионы Azure](https://azure.microsoft.com/global-infrastructure/regions/).<br> Выберите расположение, ближайшее к географическому региону, в котором вы хотите развернуть устройство.|

    ![Создание ресурса 4](media/azure-stack-edge-pro-r-deploy-prep/create-resource-4.png)


8. По завершении выберите **Next: Shipping address** (Далее. Адрес доставки).

    - Если у вас уже есть устройство, установите флажок **I have a Azure Stack Edge Pro device** (У меня есть устройство Azure Stack Edge Pro R).

        ![Создание ресурса 5](media/azure-stack-edge-pro-r-deploy-prep/create-resource-5.png)

    - Если вы заказываете новое устройство, введите имя контактного лица, название компании, адрес для отправки устройства и контактные данные.

        ![Создание ресурса 6](media/azure-stack-edge-pro-r-deploy-prep/create-resource-6.png)

9. По завершении выберите **Next: Теги**. При желании укажите здесь теги для распределения ресурсов по категориям и консолидированного выставления счетов. По завершении выберите **Next: Отзыв и создание**.

10. На вкладке **Просмотр и создание** просмотрите **сведения о ценах**, **условия использования** и подробные сведения о ресурсе. Установите флажок, чтобы подтвердить ознакомление с **условиями конфиденциальности**.

    ![Создание ресурса 7](media/azure-stack-edge-pro-r-deploy-prep/create-resource-7.png) 

    Вы также получите уведомление о том, что во время создания ресурса включается Управляемое удостоверение службы (MSI), которое позволяет выполнять проверку подлинности в облачных службах. Это удостоверение существует, пока существует ресурс.

11. Нажмите кнопку **создания**.

    Создание ресурса займет несколько минут. Кроме того, создается MSI-файл, позволяющий устройству Azure Stack Edge взаимодействовать с поставщиком ресурсов в Azure.

    После успешного создания и развертывания ресурса вы получите уведомление. Выберите **Перейти к ресурсу**.

    ![Переход к ресурсу Azure Stack Edge Pro](media/azure-stack-edge-pro-r-deploy-prep/azure-stack-edge-resource-1.png)

После размещения заказа корпорация Майкрософт проверит его и отправит вам сведения о доставке (по электронной почте).

<!--![Notification for review of the Azure Stack Edge Pro order](media/azure-stack-edge-gpu-deploy-prep/azure-stack-edge-resource-2.png) - If this is restored, it must go above "After the resource is successfully created." The azure-stack-edge-resource-1.png would seem superfluous in that case.--> 

> [!NOTE]
> Чтобы создать несколько заказов за один раз или клонировать существующий заказ, используйте [скрипты из репозитория примеров для Azure](https://github.com/Azure-Samples/azure-stack-edge-order). Дополнительные сведения см. в файле сведений.

Если во время процесса заказа возникли проблемы, ознакомьтесь со статьей [Устранение неполадок с заказами в Azure Stack Edge](azure-stack-edge-troubleshoot-ordering.md).

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

При необходимости подготовьте среду для работы с Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

Чтобы создать ресурс Azure Stack Edge, выполните указанные ниже команды в Azure CLI.

1. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create) или воспользуйтесь существующей группой ресурсов:

   ```azurecli
   az group create --name myasepgpu1 --location eastus
   ```

1. Чтобы создать устройство, воспользуйтесь командой [az databoxedge device create](/cli/azure/databoxedge/device#az_databoxedge_device_create):

   ```azurecli
   az databoxedge device create --resource-group myasepgpu1 \
      --device-name myasegpu1 --location eastus --sku EdgePR_Base
   ```

   Выберите расположение, ближайшее к географическому региону, в котором вы хотите развернуть устройство. В регионе хранятся только метаданные для управления устройством. Фактические данные могут храниться в любой учетной записи хранения.

   Список всех регионов, в которых доступны ресурсы Azure Stack Edge, приведен на странице [Доступность продуктов по регионам](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). При использовании Azure для государственных организаций доступны все регионы для государственных организаций, как показано на странице [Регионы Azure](https://azure.microsoft.com/global-infrastructure/regions/).

1. Чтобы создать заказ, выполните команду [az databoxedge order create](/cli/azure/databoxedge/order#az_databoxedge_order_create):

   ```azurecli
   az databoxedge order create --resource-group myasepgpu1 \
      --device-name myasegpu1 --company-name "Contoso" \
      --address-line1 "1020 Enterprise Way" --city "Sunnyvale" \
      --state "California" --country "United States" --postal-code 94089 \
      --contact-person "Gus Poland" --email-list gus@contoso.com --phone 4085555555
   ```

Создание ресурса займет несколько минут. Чтобы просмотреть заказ, выполните команду [az databoxedge order show](/cli/azure/databoxedge/order#az_databoxedge_order_show):

```azurecli
az databoxedge order show --resource-group myasepgpu1 --device-name myasegpu1 
```

После размещения заказа корпорация Майкрософт проверит его и отправит вам данные о доставке по электронной почте.

---

## <a name="get-the-activation-key"></a>Получение ключа активации.

После того как ресурс Azure Stack Edge будет настроен и запущен, вам потребуется получить ключ активации. Этот ключ используется для активации устройства Azure Stack Edge Pro и его подключения к ресурсу. Этот ключ можно получить, пока вы находитесь на портале Azure.

1. Выберите созданный вами ресурс, а затем выберите элемент **Обзор**.

2. В области справа укажите имя хранилища Azure Key Vault или примите имя по умолчанию. Длина имени хранилища ключей должна составлять от 3 до 24 знаков.

   Хранилище ключей создается для каждого ресурса Azure Stack Edge, который активируется на устройстве. Хранилище ключей позволяет хранить секреты и получать к ним доступ. Например, в хранилище ключей хранится ключ целостности канала (CIK) для службы.

   Указав имя хранилища ключей, щелкните элемент **Generate activation key** (Создать ключ активации), чтобы создать ключ активации.

   ![Получение ключа активации](media/azure-stack-edge-pro-r-deploy-prep/azure-stack-edge-resource-3.png)

   Создание хранилища ключей и ключа активации занимает несколько минут. Щелкните значок копирования, чтобы скопировать ключ и сохранить его для последующего использования.<!--Verify that the new screen has a copy icon.-->

> [!IMPORTANT]
> - Срок действия ключа активации истекает через три дня после создания.
> - Если срок действия ключа истек, создайте другой ключ. Ключ с истекшим сроком действия является недействительным.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве были освещены следующие темы относительно Azure Stack Edge.

> [!div class="checklist"]
> * Создать новый ресурс
> * Получение ключа активации.

Перейдите к следующему руководству, чтобы узнать, как установить службу Azure Stack Edge.

> [!div class="nextstepaction"]
> [Руководство по установке Azure Stack Edge](./azure-stack-edge-pro-r-deploy-install.md)
