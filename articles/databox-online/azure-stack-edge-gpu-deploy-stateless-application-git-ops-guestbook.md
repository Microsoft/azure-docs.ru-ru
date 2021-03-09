---
title: Развертывание `Guestbook` приложения PHP в Arc с включенной Kubernetes на устройстве Azure Stack ребра Pro GPU | Документация Майкрософт
description: В этой статье описывается развертывание `Guestbook` приложения без отслеживания состояния PHP с помощью Redis с использованием гитопс в кластере Kubernetes, поддерживающем устройство Azure Stack погранично Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 920f7912a1289fe92618d893b94943784e4a9a3a
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102520733"
---
# <a name="deploy-a-php-guestbook-stateless-application-with-redis-on-arc-enabled-kubernetes-cluster-on-azure-stack-edge-pro-gpu"></a>Развертывание приложения без `Guestbook` отслеживания состояния PHP с помощью Redis на ARC с включенным кластером Kubernetes на Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье показано, как создать и развернуть простое многоуровневое веб-приложение с помощью Kubernetes и дуги Azure. Этот пример состоит из следующих компонентов:

- Один экземпляр Redis Master для хранения `guestbook` записей
- Несколько реплицируемых экземпляров Redis для обслуживания операций чтения
- Несколько экземпляров веб-интерфейсов

Развертывание выполняется с помощью Гитопс в кластере Kubernetes с поддержкой Arc на устройстве Azure Stack ребра Pro. 

Эта процедура предназначена для пользователей, которые проверили [рабочие нагрузки Kubernetes на устройстве с Azure Stack пограничным Pro](azure-stack-edge-gpu-kubernetes-workload-management.md) , и знакомы с концепциями того, [что такое служба "Дуга Azure" Kubernetes (Предварительная версия)](../azure-arc/kubernetes/overview.md).

> [!NOTE]
> Эта статья содержит ссылки на термин slave (ведомый). Корпорация Майкрософт больше не использует его. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

## <a name="prerequisites"></a>Предварительные условия

Перед развертыванием приложения без отслеживания состояния убедитесь, что выполнены следующие необходимые условия на устройстве и клиент, который будет использоваться для доступа к устройству:

> [!NOTE]
> Эта статья содержит ссылки на термин slave (ведомый). Корпорация Майкрософт больше не использует его. Когда этот термин будет удален из программного обеспечения, мы удалим его из статьи.

### <a name="for-device"></a>Для устройств

1. У вас есть учетные данные для входа на 1 узел Azure Stack пограничным устройством Pro.
    1. Устройство активировано. См. раздел [Активация устройства](azure-stack-edge-gpu-deploy-activate.md).
    1. Устройство имеет роль вычислений, настроенную через портал Azure и имеющую кластер Kubernetes. См. раздел [Настройка вычислений](azure-stack-edge-gpu-deploy-configure-compute.md).

1. Вы включили дугу Azure в существующем кластере Kubernetes на устройстве, и у вас есть соответствующий ресурс Arc Azure в портал Azure. Подробные инструкции см. [в статье Включение дуги Azure на устройстве Azure Stack ребра Pro](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md).

### <a name="for-client-accessing-the-device"></a>Для клиента, обращающегося к устройству

1. У вас есть клиентская система Windows, которая будет использоваться для доступа к устройству Azure Stack погранично Pro.
  
    - Клиент работает под управлением Windows PowerShell 5,0 или более поздней версии. Чтобы скачать последнюю версию Windows PowerShell, перейдите к разделу [Установка Windows PowerShell](/powershell/scripting/install/installing-windows-powershell).
    
    - Также можно использовать любой другой клиент с [поддерживаемой операционной системой](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device) . В этой статье описывается процедура использования клиента Windows. 
    
1. Вы выполнили процедуру, описанную в статье [доступ к кластеру Kubernetes на устройстве с Azure Stack ребр Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Вы выполнили следующие задачи:
    
    - Установлен `kubectl` на клиенте. <!--and saved the `kubeconfig` file with the user configuration to C:\\Users\\&lt;username&gt;\\.kube. -->
    
    - Убедитесь, что `kubectl` версия клиента отклонена, но не имеет более одной версии из главной версии Kubernetes, работающей на устройстве Azure Stack погранично Pro. 
      - Используйте `kubectl version` для проверки версии kubectl, работающей на клиенте. Запишите полную версию.
      - В локальном пользовательском интерфейсе устройства Azure Stack ребра Pro перейдите к **обзору** и запишите номер Kubernetes Software. 
      - Проверьте эти две версии на совместимость с сопоставлением, предоставленным в поддерживаемой версии Kubernetes. <!--insert link-->

1. У вас есть [Конфигурация гитопс, которую можно использовать для запуска развертывания Azure Arc](https://github.com/kagoyal/dbehaikudemo). В этом примере вы будете использовать следующие `yaml` файлы для развертывания на устройстве Azure Stack ребра Pro.

    - `frontend-deployment.yaml`<!-- - The guestbook application has a web frontend serving the HTTP requests written in PHP. It is configured to connect to the redis-master Service for write requests and the redis-slave service for Read requests. This file describes a deployment that runs the frontend of the guestbook application.-->
    - `frontend-service.yaml` <!-- - This allows you to configure an externally visible frontend Service that can be accessed from outside the Kubernetes cluster on your device.-->
    - `redis-master-deployment.yaml` <!-- - This deployment file runs a single replica Redis master Pod. The guestbook application uses Redis to store its data. It writes its data to a Redis master instance and reads data from multiple Redis slave instances.-->
    - `redis-master-service.yaml` <!-- - The guestbook application needs to communicate to the Redis master to write its data. You need to apply a Service to proxy the traffic to the Redis master Pod. A Service defines a policy to access the Pods.-->
    - `redis-slave-deployment.yaml` <!-- - Although the Redis master is a single pod, you can make it highly available to meet traffic demands by adding replica Redis slaves. The Redis Slave Deployment specifies two replicas.-->
    - `redis-slave-service.yaml` <!-- - The guestbook application needs to communicate to Redis slaves to read data. To make the Redis slaves discoverable, you need to set up a Service. A Service provides transparent load balancing to a set of Pods.-->

<!-- az cli version requirements-->


## <a name="deploy-configuration"></a>Развертывание конфигурации

Выполните следующие действия, чтобы настроить ресурс дуги Azure для развертывания конфигурации Гитопс с помощью портал Azure: 

1. В портал Azure перейдите к ресурсу Arc Azure, созданному при включении дуги Azure в кластере Kubernetes на устройстве. 

    ![Переход к ресурсу Arc Azure](media/azure-stack-edge-gpu-connect-powershell-interface/verify-azure-arc-enabled-1.png)

1. Перейдите в раздел **конфигурации** и выберите **+ Добавить конфигурацию**.

    ![Снимок экрана с включенным кластером Kubernetes в службе "Дуга Azure" с выбранной конфигурацией "Добавить конфигурацию"](media/azure-stack-edge-gpu-connect-powershell-interface/select-configurations-1.png)

1. В поле **Добавить конфигурацию** введите соответствующие значения для полей, а затем нажмите кнопку **Применить**.

    |Параметр  |Описание |
    |---------|---------|
    |Имя конфигурации     | Имя для ресурса конфигурации.        |
    |Имя экземпляра оператора     |Имя экземпляра оператора для задания конкретной конфигурации. Имя — это строка из максимум 253 символов, которая должна быть строчной, буквенно-цифровой, дефисом и точкой.         |
    |Пространство имен оператора     | Задайте значение **демотестгуестбук** для соответствия пространству имен, указанному в развертывании `yaml` . <br> Поле определяет пространство имен, в котором установлен оператор. Имя — это строка из максимум 253 символов, которая должна быть строчной, буквенно-цифровой, дефисом и точкой.         |
    |URL-адрес репозитория     |<br>Путь к репозиторию Git в `http://github.com/username/repo` или `git://github.com/username/repo` формате, где находится конфигурация гитопс.         |
    |Область действия оператора     | Выберите **пространство имен**. <br>Этот параметр определяет область, в которой установлен оператор. Выберите пространство имен, чтобы установить оператор в пространстве имен, указанном в файлах развертывания YAML.       |
    |Тип оператора     | Оставьте значение по умолчанию. <br>Этот параметр задает тип оператора (по умолчанию), установленный как Flux.        |
    |Параметры оператора     | Не заполняйте это поле. <br>Этот параметр содержит параметры, передаваемые оператору Flux.        |
    |Helm     | Установите этот параметр в значение **отключено**. <br>Включите этот параметр, если планируется развертывание на основе диаграмм.        |


    ![Добавление конфигурации](media/azure-stack-edge-gpu-connect-powershell-interface/add-configuration-1.png)


1. Запускается развертывание конфигурации, и **состояние оператора** отображается как **Ожидание**. 

    ![На снимке экрана показан кластер Kubernetes с включенной службой Arc Azure в состоянии ожидания, так как он обновляется.](media/azure-stack-edge-gpu-connect-powershell-interface/view-configurations-1.png)

1. Развертывание займет несколько минут. По завершении развертывания **состояние оператора** отображается как **установлено**.

    ![На снимке экрана показан кластер Kubernetes с включенной службой "Дуга Azure" в установленном состоянии.](media/azure-stack-edge-gpu-connect-powershell-interface/view-configurations-2.png)

## <a name="verify-deployment"></a>Проверка развертывания

При развертывании с помощью конфигурации Гитопс создается `demotestguestbook` пространство имен, указанное в файлах развертывания, `yaml` расположенных в репозитории Git.

1. После применения конфигурации Гитопс [подключитесь к интерфейсу PowerShell устройства](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).
1. Выполните следующую команду, чтобы вывести список модулей Pod, выполняющихся в `demotestguestbook` пространстве имен, соответствующем развертыванию.

    `kubectl get pods -n <your-namespace>`
  
    Ниже приведен пример выходных данных.
  
    ```powershell
    [10.128.44.240]: PS>kubectl get pods -n demotestguestbook
    NAME                            READY   STATUS    RESTARTS   AGE
    aseoperator1-5569658644-cqtb5   1/1     Running   0          91m
    frontend-6cb7f8bd65-4xb4f       1/1     Running   0          91m
    frontend-6cb7f8bd65-q9cxj       1/1     Running   0          91m
    frontend-6cb7f8bd65-xpzs6       1/1     Running   0          91m
    memcached-86bdf9f56b-5l2fq      1/1     Running   0          91m
    redis-master-7db7f6579f-2z29w   1/1     Running   0          91m
    redis-slave-7664787fbc-lgr2n    1/1     Running   0          91m
    redis-slave-7664787fbc-vlvzn    1/1     Running   0          91m
    [10.128.44.240]: PS>
    ```  

1. В этом примере интерфейсная служба была развернута как тип: балансировщик. Для просмотра необходимо найти IP-адрес этой службы `guestbook` . Выполните следующую команду.

    `kubectl get service -n <your-namespace>`
    
    ```powershell
    [10.128.44.240]: PS>kubectl get service -n demotestguestbook
    NAME           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
    frontend       LoadBalancer   10.96.79.38      10.128.44.245   80:31238/TCP   85m
    memcached      ClusterIP      10.102.47.75     <none>          11211/TCP      85m
    redis-master   ClusterIP      10.104.32.99     <none>          6379/TCP       85m
    redis-slave    ClusterIP      10.104.215.146   <none>          6379/TCP       85m
    [10.128.44.240]: PS>
    ```
1. Интерфейсная служба `type:LoadBalancer` имеет внешний IP-адрес. Этот IP-адрес относится к диапазону IP-адресов, указанному для внешних служб при настройке параметров вычислений сети на устройстве. Используйте этот IP-адрес для просмотра `guestbook` URL-адреса по адресу: `https://<external-IP-address>` .

    ![Просмотр гостевой книги](media/azure-stack-edge-gpu-connect-powershell-interface/view-guestbook-1.png)

## <a name="delete-deployment"></a>Удалить развертывание

Чтобы удалить развертывание, можно удалить конфигурацию из портал Azure. Удаление конфигурации приведет к удалению созданных объектов, включая развертывания и службы.

1. В портал Azure перейдите по адресу > конфигурации ресурсов ARC в Azure. 
1. Выберите конфигурацию, которую необходимо удалить. Выберите... чтобы вызвать контекстное меню и выбрать команду **Удалить**.
    ![Удалить конфигурацию](media/azure-stack-edge-gpu-connect-powershell-interface/delete-configuration-1.png)

Удаление конфигурации может занять несколько минут.
 
<!--```powershell
kubectl delete deployment <deployment-name> -n <your-namespace>
kubectl delete service <service-name> -n <your-namespace>
```

Here is a sample output of when you delete the deployments and the services.

```powershell
C:\Users\user>kubectl delete deployment,svc mysql -n userns1
deployment.apps "mysql" deleted
service "mysql" deleted
C:\Users\user>
```-->


## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [использовать панель мониторинга Kubernetes для мониторинга развертываний на устройстве Azure Stack ребра Pro](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md) .