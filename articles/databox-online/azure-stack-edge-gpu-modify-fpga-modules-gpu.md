---
title: Изменение модулей IoT Edge на устройстве FPGA для запуска на устройстве Azure Stack с графическим процессором версии Pro
description: Описывает, какие изменения необходимы для существующих модулей IoT Edge на существующих устройствах FPGA для запуска на устройстве Azure Stack с помощью GPU Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 1276a242efb1917a0c4a24aa73c3e0d11f81e158
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105559168"
---
# <a name="run-existing-iot-edge-modules-from-azure-stack-edge-pro-fpga-devices-on-azure-stack-edge-pro-gpu-device"></a>Запуск существующих модулей IoT Edge из Azure Stack пограничных устройств с пограничными устройствами FPGA на устройстве с Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-sku.md)]

В этой статье подробно описаны изменения, необходимые для модуля IoT Edge DOCKER, который работает на Azure Stack пограничной Pro (FPGA), поэтому он может работать на платформе IoT Edge на основе Kubernetes на устройстве Azure Stack с ГРАФИЧЕСКИм интерфейсом Pro. 

## <a name="about-iot-edge-implementation"></a>О реализации IoT Edge 

Реализация IoT Edge различается на устройствах Azure Stack с пограничными устройствами FPGA и на устройствах с графическим процессором Azure Stack. Для устройств с графическим процессором Kubernetes используется в качестве платформы размещения для IoT Edge. IoT Edge на устройствах FPGA использует платформу на основе DOCKER. Модель приложений на основе IoT Edge DOCKER автоматически преобразуется в модель собственного приложения Kubernetes. Тем не менее, некоторые изменения могут потребоваться, так как поддерживается только небольшое подмножество модели приложения Kubernetes.

При переносе рабочих нагрузок с устройства FPGA на устройство GPU необходимо внести изменения в существующие модули IoT Edge, чтобы они успешно выполнялись на платформе Kubernetes. Может потребоваться по-разному указать требования к хранилищу, сети, использованию ресурсов и веб-прокси. 

## <a name="storage"></a>Память

При указании хранилища для модулей IoT Edge учитывайте следующие сведения.

- Хранилище для контейнеров в Kubernetes указывается с помощью подключений к томам.
- Развертывание в Kubernetes не может иметь привязки для связывания постоянного хранилища или путей к узлам.
    - Для постоянного хранения используйте `Mounts` с типом `volume` .
    - Для путей к узлам используйте `Mounts` параметр с типом `bind` .
- Для IoT Edge в Kubernetes выполните привязку с помощью `Mounts` Works только для каталога, а не для файла.

#### <a name="example---storage-via-volume-mounts"></a>Пример. хранилище через подключения томов 

Для IoT Edge в DOCKER путь к узлу привязки используются для связывания общих папок на устройстве с путями внутри контейнера. Ниже приведены параметры создания контейнера, используемые на устройствах FPGA:

```
{
  "HostConfig": 
  {
   "Binds": 
    [
     "<Host storage path for Edge local share>:<Module storage path>"
    ]
   }
}
```
<!-- is this how it will look on GPU device?-->
Для путей к узлам для IoT Edge на Kubernetes показан пример использования `Mounts` with Type `bind` :
```
{
    "HostConfig": {
        "Mounts": [
            {
                "Target": "<Module storage path>",
                "Source": "<Host storage path>",
                "Type": "bind"
            }
        ]
    }
}
```


<!--following example is for persistent storage where we use mounts w/ type volume-->

Для устройств GPU, работающих IoT Edge в Kubernetes, для указания хранилища используются подключения к томам. Чтобы подготовить хранилище с помощью общих папок, значением `Mounts.Source` будет имя общего ресурса SMB или NFS, подготовленного на устройстве GPU. `/home/input`— Это путь, по которому том доступен в контейнере. Ниже приведены параметры создания контейнера, используемые на устройствах GPU:

```
{
    "HostConfig": {
        "Mounts": [
        {
            "Target": "/home/input",
            "Source": "<nfs-or-smb-share-name-here>",
            "Type": "volume"
        },
        {
            "Target": "/home/output",
            "Source": "<nfs-or-smb-share-name-here>",
            "Type": "volume"
        }]
    }
}
```



## <a name="network"></a>Сеть

При указании сети для модулей IoT Edge учитывайте следующие сведения. 

- `HostPort` Спецификация необходима для предоставления службы как внутри, так и за пределами кластера.
    - K8sExperimental параметры для ограничения раскрытия службы только для кластера.
- Для взаимодействия между модулями требуется `HostPort` Указание спецификации и подключение с помощью сопоставленного порта (без использования порта, доступного для контейнера).
- Сеть узла работает с `dnsPolicy = ClusterFirstWithHostNet` , и все контейнеры (особенно `edgeHub` ) не обязательно должны находиться в сети узла. <!--Need further clarifications on this one-->
- Добавление сопоставлений портов TCP, UDP в одном и том же запросе не работает.

#### <a name="example---external-access-to-modules"></a>Пример. внешний доступ к модулям 

Для всех модулей IoT Edge, которые задают привязки портов, IP-адрес назначается с помощью диапазона внешних IP-адресов внешней службы Kubernetes, указанного в локальном пользовательском интерфейсе устройства. В контейнере отсутствуют изменения параметров CREATE для IoT Edge в DOCKER и IoT Edge в Kubernetes, как показано в следующем примере.  

```json 
{
    "HostConfig": {
        "PortBindings": {
            "5000/tcp": [
                {
                    "HostPort": "5000"
                }
            ]
        }
    }
}
```

Однако для запроса IP-адреса, назначенного вашему модулю, можно использовать панель мониторинга Kubernetes, как описано в разделе [получение IP-адреса для служб или модулей](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#get-ip-address-for-services-or-modules). 

Кроме того, можно [подключиться к интерфейсу PowerShell устройства](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) и использовать `iotedge` команду list для вывода списка всех модулей, выполняющихся на устройстве. [Выходные данные команды](azure-stack-edge-gpu-connect-powershell-interface.md#debug-kubernetes-issues-related-to-iot-edge) также будут указывать на внешние IP-адреса, связанные с модулем.


## <a name="resource-usage"></a>Использование ресурсов 

При настройке IoT Edge на основе Kubernetes на устройствах GPU ресурсы, такие как аппаратное ускорение, память и требования к ЦП, указаны не так, как на устройствах FPGA. 

#### <a name="compute-acceleration-usage"></a>Использование ускорения вычислений

Чтобы развернуть модули в FPGA, используйте параметры создания контейнера. <!--with Device Bindings--> как показано в следующей конфигурации: <!--not sure where are device bindings in this config--> 

```json
{
    "HostConfig": {
    "Privileged": true,
    "PortBindings": {
        "50051/tcp": [
        {
            "HostPort": "50051"
        }
        ]
    }
    },
    "k8s-experimental": {
    "resources": {
        "limits": {
        "microsoft.com/fpga_catapult": 2
        },
        "requests": {
        "microsoft.com/fpga_catapult": 2
        }
    }
    },
    "Env": [
    "WIRESERVER_ADDRESS=10.139.218.1"
    ]
}
``` 

    
<!--Note: The IP address assigned to your FPGA module's service can be used to send inferencing requests from outside the cluster OR your ML module can be used along with DBE Simple Module Flow by passing files to the module using an input share.-->
    
Для GPU используйте спецификации запросов ресурсов вместо привязок устройств, как показано в следующей минимальной конфигурации. Вместо взлететь вы запрашиваете ресурсы NVIDIA, а не указываете `wireserver` . 

```json     
{
    "HostConfig": {
    "Privileged": true,
    "PortBindings": {
        "50051/tcp": [
        {
            "HostPort": "50051"
        }
        ]
    }
    },
    "k8s-experimental": {
    "resources": {
        "limits": {
        "nvidia.com/gpu": 2
        }    
    }
}
```

#### <a name="memory-and-cpu-usage"></a>Использование памяти и ЦП
 
Чтобы задать использование памяти и ЦП, используйте ограничения процессора для модулей в `k8s-experimental` разделе. <!--can we verify if this is how we set limits of memory and CPU-->

```json
    "k8s-experimental": {
    "resources": {
        "limits": {
            "memory": "128Mi",
            "cpu": "500m",
            "nvidia.com/gpu": 2
        },
        "requests": {
            "nvidia.com/gpu": 2
        }
}
```
Спецификация памяти и ЦП не является обязательной, но обычно рекомендуется. Если параметр `requests` не указан, то значения, заданные в ограничениях, используются в качестве минимально необходимого. 

Использование общей памяти для модулей также требует другого способа. Например, можно использовать режим IPC узла для доступа к общей памяти между решениями Live Video Analytics и выводом, как описано в статье [развертывание Live Video Analytics на Azure Stack](../media-services/live-video-analytics-edge/deploy-azure-stack-edge-how-to.md#deploy-live-video-analytics-edge-module-using-azure-portal).


## <a name="web-proxy"></a>Веб-прокси 

При настройке веб-прокси учитывайте следующие сведения.

Если в вашей сети настроен веб-прокси, настройте следующие переменные среды для `edgeHub` развертывания в IOT Edge установки на основе DOCKER на устройствах FPGA:

- `https_proxy : <proxy URL>`
- `UpstreamProtocol : AmqpWs` (если веб-прокси не разрешает `Amqp` трафик)

Для настроек IoT Edge на основе Kubernetes на устройствах GPU необходимо настроить эту дополнительную переменную во время развертывания:

- `no_proxy`: localhost

- IoT Edge прокси-сервер на платформе Kubernetes использует порты 35000 и 35001. Убедитесь, что модуль не запущен на этих портах, или это может привести к конфликтам портов. 

## <a name="other-differences"></a>Другие различия

- **Стратегия развертывания**. может потребоваться изменить поведение развертывания для любых обновлений модуля. Поведение по умолчанию для модулей IoT Edge — это пошаговое обновление. Такое поведение предотвращает перезагрузку обновленного модуля, если модуль использует такие ресурсы, как аппаратное ускорение или сетевые порты. Такое поведение может привести к непредвиденным последствиям, специально при работе с постоянными томами на платформе Kubernetes для устройств GPU. Чтобы переопределить это поведение по умолчанию, можно указать `Recreate` в `k8s-experimental` разделе модуля.

    ```    
    {
      "k8s-experimental": {
        "strategy": {
          "type": "Recreate"
        }
      }
    }
    ```

- **Имена модулей**: имена модулей должны соответствовать соглашениям об именовании Kubernetes. При перемещении этих модулей в IoT Edge с помощью Kubernetes может потребоваться переименовать модули, выполняющиеся на IoT Edge с помощью DOCKER. Дополнительные сведения об именовании см. в разделе [Kubernetes Naming Conventions](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/).
- **Другие параметры**: 
    - Некоторые параметры создания DOCKER, которые работали на устройствах FPGA, не будут работать в среде Kubernetes на устройствах GPU. Например:, Like – EntryPoint.<!--can we confirm what exactly is required here-->
    - Переменные среды, такие как, `:` необходимо заменить на `__` .
    - Состояние **создания контейнера** для Kubernetes Pod **приводит к** отправке состояния для модуля в ресурсе центра Интернета вещей. Хотя существует ряд причин, по которым Pod может находиться в этом состоянии, но распространенной причиной является то, что большой образ контейнера передается через медленное подключение с низкой пропускной способностью сети. Когда модуль Pod находится в этом состоянии, состояние модуля отображается как " **отход** " в центре Интернета вещей при запуске модуля.


## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [настройке GPU для использования модуля](./azure-stack-edge-gpu-configure-gpu-modules.md).