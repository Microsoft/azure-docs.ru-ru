---
title: Учебник. Фильтрация и анализ данных с помощью вычислений на устройстве Azure Stack Edge Pro с GPU | Документация Майкрософт
description: Сведения о том, как настроить роль вычислений в Azure Stack Edge Pro с GPU и использовать ее для преобразования данных перед их отправкой в Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 03/08/2021
ms.author: alkohli
ms.openlocfilehash: 626c4b192eeeeb8a07130d0a9ecf6754eebbe068
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106065690"
---
# <a name="tutorial-configure-compute-on-azure-stack-edge-pro-gpu-device"></a>Руководство по Настройка вычислений на устройстве Azure Stack Edge Pro с GPU

<!--ALPA WILL VERIFY - [!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

В этом учебнике описывается, как настроить роль вычислений и создать кластер Kubernetes на устройстве Azure Stack Edge Pro. 

Эта процедура может занять от 20 до 30 минут.


В этом руководстве описано следующее:

> [!div class="checklist"]
> * настройка вычислений;
> * Получение конечных точек Kubernetes

 
## <a name="prerequisites"></a>Предварительные требования

Прежде чем вы настроите роль вычислений на устройстве Azure Stack Edge Pro, убедитесь, что:

- Устройство Azure Stack Edge Pro активировано, как описано в статье [Активация Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).
- Убедитесь, что вы выполнили инструкции, приведенные в разделе [Включение сети вычислений](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#enable-compute-network), и сделали следующее:
    - включили сетевой интерфейс для служб вычислений;
    - назначили IP-адреса узлов Kubernetes и внешних служб Kubernetes.

## <a name="configure-compute"></a>настройка вычислений;

[!INCLUDE [configure-compute](../../includes/azure-stack-edge-gateway-configure-compute.md)]

## <a name="get-kubernetes-endpoints"></a>Получение конечных точек Kubernetes

Чтобы настроить клиент для доступа к кластеру Kubernetes, вам понадобится конечная точка Kubernetes. Выполните приведенные ниже действия, чтобы получить конечную точку API Kubernetes из локального пользовательского интерфейса устройства Azure Stack Edge Pro.

1. Перейдите на страницу **Устройства** в локальном пользовательском веб-интерфейсе устройства.
2. В разделе **Device endpoints** (Конечные точки устройства) скопируйте конечную точку **службы API Kubernetes**. Эта конечная точка представляет собой строку в следующем формате: `https://compute.<device-name>.<DNS-domain>[Kubernetes-cluster-IP-address]`. 

    ![Страница "Устройство" в локальном пользовательском интерфейсе](./media/azure-stack-edge-gpu-create-kubernetes-cluster/device-kubernetes-endpoint-1.png)

3. Сохраните строку конечной точки. Вы будете использовать ее позже при настройке клиента для доступа к кластеру Kubernetes через kubectl.

4. Находясь в локальном веб-интерфейсе, вы можете выполнить следующее.

    - Перейдите в API Kubernetes, выберите **дополнительные параметры** и скачайте расширенный файл конфигурации для Kubernetes. 

        ![Страница "Устройство" в локальном пользовательском интерфейсе 1](./media/azure-stack-edge-gpu-deploy-configure-compute/download-advanced-config-1.png)

        Вы можете использовать этот файл конфигурации, если вам предоставлен ключ от Майкрософт (он может быть у некоторых пользователей).

        ![Страница "Устройство" в локальном пользовательском интерфейсе 2](./media/azure-stack-edge-gpu-deploy-configure-compute/download-advanced-config-2.png)

    - Вы также можете перейти к конечной точке **панели мониторинга Kubernetes** и скачать файл конфигурации `aseuser`. 
    
        ![Страница "Устройство" в локальном пользовательском интерфейсе 3](./media/azure-stack-edge-gpu-deploy-configure-compute/download-aseuser-config-1.png)

        Этот файл конфигурации можно использовать для входа на панель мониторинга Kubernetes или отладки любых проблем в кластере Kubernetes. Дополнительные сведения см. в разделе [Доступ к панели мониторинга](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#access-dashboard). 


## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * настройка вычислений;
> * Получение конечных точек Kubernetes


Сведения об администрировании устройства Azure Stack Edge Pro см. в следующей статье:

> [!div class="nextstepaction"]
> [Использование локального пользовательского веб-интерфейса для администрирования Azure Stack Edge Pro](azure-stack-edge-manage-access-power-connectivity-mode.md)
