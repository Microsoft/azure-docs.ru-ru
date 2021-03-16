---
title: Развертывание IoT Edge рабочей нагрузки с помощью совместного использования GPU на устройстве Azure Stack ребра Pro GPU
description: В этой статье описывается, как развернуть общую рабочую нагрузку GPU с помощью IoT Edge на устройстве с графическим процессором Azure Stack.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/12/2021
ms.author: alkohli
ms.openlocfilehash: b52d1e834772a2a6e0e000b3df15d8aa0fa866a9
ms.sourcegitcommit: 18a91f7fe1432ee09efafd5bd29a181e038cee05
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103565404"
---
# <a name="deploy-an-iot-edge-workload-using-gpu-sharing-on-your-azure-stack-edge-pro"></a>Развертывание IoT Edge рабочей нагрузки с помощью совместного использования GPU на Azure Stack крае Pro

В этой статье описывается, как контейнерные рабочие нагрузки могут совместно использовать GPU на устройстве Azure Stack ребра Pro GPU. Этот подход включает в себя включение многопроцессной службы (MPS) и указание рабочих нагрузок GPU с помощью IoT Edgeного развертывания. 

## <a name="prerequisites"></a>Предварительные требования

Перед тем как начать, убедитесь в следующем.

1. У вас есть доступ к подключенному устройству GPU Azure Stack с пограничными устройствами, которое было [активировано](azure-stack-edge-gpu-deploy-activate.md) и [настроено для вычислений](azure-stack-edge-gpu-deploy-configure-compute.md). У вас есть [Конечная точка API Kubernetes](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints) , и вы добавили эту конечную точку в `hosts` файл на клиенте, который будет получать доступ к устройству.

1. У вас есть доступ к клиентской системе с [поддерживаемой операционной системой](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). При использовании клиента Windows система должна запустить PowerShell 5,0 или более поздней версии для доступа к устройству.

1. Сохраните следующее развертывание `json` в локальной системе. Вы будете использовать сведения из этого файла для запуска развертывания IoT Edge. Это развертывание основано на [простых контейнерах CUDA](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#running-simple-containers) , которые общедоступны из NVIDIA. 

    ```json
    {
        "modulesContent": {
            "$edgeAgent": {
                "properties.desired": {
                    "modules": {
                        "cuda-sample1": {
                            "settings": {
                                "image": "nvidia/samples:nbody",
                                "createOptions": "{\"Entrypoint\":[\"/bin/sh\"],\"Cmd\":[\"-c\",\"/tmp/nbody -benchmark -i=1000; while true; do echo no-op; sleep 10000;done\"],\"HostConfig\":{\"IpcMode\":\"host\",\"PidMode\":\"host\"}}"
                            },
                            "type": "docker",
                            "version": "1.0",
                            "env": {
                                "NVIDIA_VISIBLE_DEVICES": {
                                    "value": "0"
                                }
                            },
                            "status": "running",
                            "restartPolicy": "never"
                        },
                        "cuda-sample2": {
                            "settings": {
                                "image": "nvidia/samples:nbody",
                                "createOptions": "{\"Entrypoint\":[\"/bin/sh\"],\"Cmd\":[\"-c\",\"/tmp/nbody -benchmark -i=1000; while true; do echo no-op; sleep 10000;done\"],\"HostConfig\":{\"IpcMode\":\"host\",\"PidMode\":\"host\"}}"
                            },
                            "type": "docker",
                            "version": "1.0",
                            "env": {
                                "NVIDIA_VISIBLE_DEVICES": {
                                    "value": "0"
                                }
                            },
                            "status": "running",
                            "restartPolicy": "never"
                        }
                    },
                    "runtime": {
                        "settings": {
                            "minDockerVersion": "v1.25"
                        },
                        "type": "docker"
                    },
                    "schemaVersion": "1.1",
                    "systemModules": {
                        "edgeAgent": {
                            "settings": {
                                "image": "mcr.microsoft.com/azureiotedge-agent:1.0",
                                "createOptions": ""
                            },
                            "type": "docker"
                        },
                        "edgeHub": {
                            "settings": {
                                "image": "mcr.microsoft.com/azureiotedge-hub:1.0",
                                "createOptions": "{\"HostConfig\":{\"PortBindings\":{\"443/tcp\":[{\"HostPort\":\"443\"}],\"5671/tcp\":[{\"HostPort\":\"5671\"}],\"8883/tcp\":[{\"HostPort\":\"8883\"}]}}}"
                            },
                            "type": "docker",
                            "status": "running",
                            "restartPolicy": "always"
                        }
                    }
                }
            },
            "$edgeHub": {
                "properties.desired": {
                    "routes": {
                        "route": "FROM /messages/* INTO $upstream"
                    },
                    "schemaVersion": "1.1",
                    "storeAndForwardConfiguration": {
                        "timeToLiveSecs": 7200
                    }
                }
            },
            "cuda-sample1": {
                "properties.desired": {}
            },
            "cuda-sample2": {
                "properties.desired": {}
            }
        }
    }
    ```

## <a name="verify-gpu-driver-cuda-version"></a>Проверка драйвера GPU, версия CUDA

Первым делом необходимо убедиться, что на устройстве выполняется требуемый драйвер GPU и CUDA версии.

1. [Подключитесь к интерфейсу PowerShell устройства](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).

1. Выполните следующую команду:

    `Get-HcsGpuNvidiaSmi`

1. В выходных данных устройства NVIDIA SMI Запишите версию GPU и версию CUDA на устройстве. Если вы используете программное обеспечение Azure Stack погранично 2102, эта версия будет соответствовать следующим версиям драйвера:

    - Версия драйвера GPU: 460.32.03
    - Версия CUDA: 11,2
    
    Пример выходных данных:

    ```powershell
    [10.100.10.10]: PS>Get-HcsGpuNvidiaSmi
    K8S-1HXQG13CL-1HXQG13:
    
    Tue Feb 23 10:34:01 2021
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            On   | 0000041F:00:00.0 Off |                    0 |
    | N/A   40C    P8    15W /  70W |      0MiB / 15109MiB |      0%      Default |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
    [10.100.10.10]: PS>  
    ```

1. Не закрывайте этот сеанс, так как он будет использоваться для просмотра выходных данных SMI для NVIDIA в этой статье.


## <a name="deploy-without-context-sharing"></a>Развертывание без общего доступа к контексту

Теперь вы можете развернуть приложение на устройстве, если служба многопроцессной обработки не запущена и нет общего доступа к контексту. Развертывание осуществляется через портал Azure в `iotedge` пространстве имен, которое существует на вашем устройстве.

### <a name="create-user-in-iot-edge-namespace"></a>Создание пользователя в IoT Edge пространстве имен

Сначала вы создадите пользователя, который будет подключаться к `iotedge` пространству имен. IoT Edge модули развертываются в пространстве имен iotedge. Дополнительные сведения см. [в разделе пространства имен Kubernetes на устройстве](azure-stack-edge-gpu-kubernetes-rbac.md#namespaces-types).

Выполните следующие действия, чтобы создать пользователя и предоставить пользователю доступ к `iotedge` пространству имен. 

1. [Подключитесь к интерфейсу PowerShell устройства](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).

1. Создайте нового пользователя в `iotedge` пространстве имен. Выполните следующую команду:

    `New-HcsKubernetesUser -UserName <user name>`

    Пример выходных данных:

    ```powershell
    [10.100.10.10]: PS>New-HcsKubernetesUser -UserName iotedgeuser
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: 
    ===========================//snipped //======================// snipped //=============================
        server: https://compute.myasegpudev.wdshcsso.com:6443
      name: kubernetes
    contexts:
    - context:
        cluster: kubernetes
        user: iotedgeuser
      name: iotedgeuser@kubernetes
    current-context: iotedgeuser@kubernetes
    kind: Config
    preferences: {}
    users:
    - name: iotedgeuser
      user:
        client-certificate-data: 
    ===========================//snipped //======================// snipped //=============================
        client-key-data: 
    ===========================//snipped //======================// snipped ============================
    PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=
    ```

1. Скопируйте выходные данные в виде обычного текста. Сохраните выходные данные в виде файла *конфигурации* (без расширения) в `.kube` папке вашего профиля пользователя на локальном компьютере, например `C:\Users\<username>\.kube` . 

1. Предоставьте созданному пользователю доступ к `iotedge` пространству имен. Выполните следующую команду:

    `Grant-HcsKubernetesNamespaceAccess -Namespace iotedge -UserName <user name>`    

    Пример выходных данных:

    ```python
    [10.100.10.10]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace iotedge -UserName iotedgeuser
    [10.100.10.10]: PS>    
    ```
Подробные инструкции см. в статье [Подключение к кластеру Kubernetes и управление им с помощью kubectl на устройстве Azure Stack с графическим](azure-stack-edge-gpu-create-kubernetes-cluster.md#configure-cluster-access-via-kubernetes-rbac)стандартом Pro.

### <a name="deploy-modules-via-portal"></a>Развертывание модулей с помощью портала

Развертывание модулей IoT Edge с помощью портал Azure. Вы развернете общедоступные образцы модулей NVIDIA CUDA, которые выполняют имитацию n-Body. Дополнительные сведения см. в разделе [имитация N-Body](https://physics.princeton.edu//~fpretori/Nbody/intro.htm).

1. Убедитесь, что на вашем устройстве запущена служба IoT Edge.

    ![Служба IoT Edge запущена.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-1.png)

1. Щелкните плитку IoT Edge на правой панели. Перейдите к **IOT Edge свойства >**. На правой панели выберите ресурс центра Интернета вещей, связанный с устройством.

    ![Просматривать свойства.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-2.png)

1. В ресурсе центра Интернета вещей перейдите в раздел **Автоматическое управление устройствами > IOT Edge**. На правой панели выберите устройство IoT Edge, связанное с устройством.

    ![Перейдите в IoT Edge.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-3.png)

1. Щелкните **Set modules** (Настроить модули).

    ![Перейдите к разделу Установка модулей.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-4.png)

1. Выберите **+ Добавить модуль > + IOT Edge**.

    ![Add IoT Edge Module (Azure IoT Edge: добавить модуль IoT Edge).](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-5.png)

1. На вкладке **Параметры модуля** укажите **имя модуля IOT Edge** и **URI образа**. Задайте для **параметра опрашивающей политики образа** значение **On Create**.

    ![Параметры модуля.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-6.png)
1. На вкладке **переменные среды** укажите **NVIDIA_VISIBLE_DEVICES** как **0**.

    ![Переменные среды.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-7.png)

1. На вкладке **Параметры создания контейнера** укажите следующие параметры.

    ```json
    {
        "Entrypoint": [
            "/bin/sh"
        ],
        "Cmd": [
            "-c",
            "/tmp/nbody -benchmark -i=1000; while true; do echo no-op; sleep 10000;done"
        ],
        "HostConfig": {
            "IpcMode": "host",
            "PidMode": "host"
        }
    }    
    ```
    Параметры отображаются следующим образом:

    ![Параметры создания контейнера.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-8.png)

    Щелкните **Добавить**.

1. Добавленный модуль должен отображаться как **выполняемый**. 

    ![Проверьте и создайте развертывание.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-9.png)


1. Повторите все шаги, чтобы добавить модуль, который вы составите при добавлении первого модуля. В этом примере укажите имя модуля в виде `cuda-sample2` . 

    ![Параметры модуля для второго модуля.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-12.png)

    Используйте ту же переменную среды, в которой оба модуля будут совместно использовать один GPU.

    ![Переменная среды для второго модуля.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-13.png)

    Используйте те же параметры создания контейнера, которые вы указали для первого модуля, и нажмите кнопку **Добавить**.

    ![Параметры создания контейнера для 2-го модуля.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-14.png)

1. На странице **Установка модулей** выберите проверить и **создать** , а затем щелкните **создать**. 

    ![Проверьте и создайте второе развертывание.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-15.png)

1. **Состояние среды выполнения** обоих модулей должно отобразиться как **выполняемое**.  

    ![второе состояние развертывания.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/gpu-sharing-deploy-16.png)


### <a name="monitor-workload-deployment"></a>Мониторинг развертывания рабочей нагрузки

1. Запустите новый сеанс PowerShell.

1. Выведите список модулей Pod, выполняющихся в `iotedge` пространстве имен. Выполните следующую команду:

    `kubectl get pods -n iotedge`   

    Пример выходных данных:

    ```powershell
    PS C:\WINDOWS\system32> kubectl get pods -n iotedge --kubeconfig C:\GPU-sharing\kubeconfigs\configiotuser1
    NAME                            READY   STATUS    RESTARTS   AGE
    cuda-sample1-869989578c-ssng8   2/2     Running   0          5s
    cuda-sample2-6db6d98689-d74kb   2/2     Running   0          4s
    edgeagent-79f988968b-7p2tv      2/2     Running   0          6d21h
    edgehub-d6c764847-l8v4m         2/2     Running   0          24h
    iotedged-55fdb7b5c6-l9zn8       1/1     Running   1          6d21h
    PS C:\WINDOWS\system32>   
    ```
    `cuda-sample1-97c494d7f-lnmns`На вашем устройстве работают два модуля (Pod) и `cuda-sample2-d9f6c4688-2rld9` .

1. Хотя оба контейнера работают с имитацией n-Body, просмотрите использование GPU из выходных данных NVIDIA SMI. Перейдите к интерфейсу PowerShell устройства и выполните команду `Get-HcsGpuNvidiaSmi` .

    Ниже приведен пример выходных данных, когда оба контейнера работают с имитацией n-Body:

    ```powershell
    [10.100.10.10]: PS>Get-HcsGpuNvidiaSmi
    K8S-1HXQG13CL-1HXQG13:
    
    Fri Mar  5 13:31:16 2021
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            On   | 00002C74:00:00.0 Off |                    0 |
    | N/A   52C    P0    69W /  70W |    221MiB / 15109MiB |    100%      Default |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |    0   N/A  N/A    188342      C   /tmp/nbody                        109MiB |
    |    0   N/A  N/A    188413      C   /tmp/nbody                        109MiB |
    +-----------------------------------------------------------------------------+
    [10.100.10.10]: PS>
    ```
    Как видите, существует два контейнера, работающих с имитацией n-Body в GPU 0. Можно также просмотреть соответствующее использование памяти.

1. После завершения моделирования выходные данные NVIDIA SMI будут показывать, что на устройстве не запущено ни одного процесса.

    ```powershell
    [10.100.10.10]: PS>Get-HcsGpuNvidiaSmi
    K8S-1HXQG13CL-1HXQG13:
    
    Fri Mar  5 13:54:48 2021
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            On   | 00002C74:00:00.0 Off |                    0 |
    | N/A   34C    P8     9W /  70W |      0MiB / 15109MiB |      0%      Default |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |  No running processes found                                                 |
    +-----------------------------------------------------------------------------+
    [10.100.10.10]: PS>
    ```
 
1. После завершения имитации n-Body просмотрите журналы, чтобы понять сведения о развертывании и время, необходимое для завершения моделирования. 

    Ниже приведен пример выходных данных из первого контейнера:

    ```powershell
    PS C:\WINDOWS\system32> kubectl -n iotedge  --kubeconfig C:\GPU-sharing\kubeconfigs\configiotuser1 logs cuda-sample1-869989578c-ssng8 cuda-sample1
    Run "nbody -benchmark [-numbodies=<numBodies>]" to measure performance.
    ==============// snipped //===================//  snipped  //=============
    > Windowed mode
    > Simulation data stored in video memory
    > Single precision floating point simulation
    > 1 Devices used for simulation
    GPU Device 0: "Turing" with compute capability 7.5
    
    > Compute 7.5 CUDA device: [Tesla T4]
    40960 bodies, total time for 10000 iterations: 170171.531 ms
    = 98.590 billion interactions per second
    = 1971.801 single-precision GFLOP/s at 20 flops per interaction
    no-op
    PS C:\WINDOWS\system32>
    ```
    Ниже приведен пример выходных данных из второго контейнера:

    ```powershell
    PS C:\WINDOWS\system32> kubectl -n iotedge  --kubeconfig C:\GPU-sharing\kubeconfigs\configiotuser1 logs cuda-sample2-6db6d98689-d74kb cuda-sample2
    Run "nbody -benchmark [-numbodies=<numBodies>]" to measure performance.
    ==============// snipped //===================//  snipped  //=============
    > Windowed mode
    > Simulation data stored in video memory
    > Single precision floating point simulation
    > 1 Devices used for simulation
    GPU Device 0: "Turing" with compute capability 7.5
    
    > Compute 7.5 CUDA device: [Tesla T4]
    40960 bodies, total time for 10000 iterations: 170054.969 ms
    = 98.658 billion interactions per second
    = 1973.152 single-precision GFLOP/s at 20 flops per interaction
    no-op
    PS C:\WINDOWS\system32>
    ```

1. Останавливает развертывание модуля. В ресурсе центра Интернета вещей для вашего устройства:
    1. Выберите **Автоматическое развертывание устройства > IOT Edge**. Выберите устройство IoT Edge, соответствующее устройству.

    1. Перейдите к разделу **Настройка модулей** и выберите модуль. 

        ![Выберите задать модуль.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/stop-module-deployment-1.png)

    1. На вкладке **модули** выберите модуль.
    
        ![Выберите модуль.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/stop-module-deployment-2.png)

    1.  На вкладке **Параметры модуля** задайте для параметра **желательное состояние** значение остановлено. Нажмите кнопку **Обновить**.

        ![Измените параметры модуля.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/stop-module-deployment-3.png)

    1. Повторите шаги, чтобы закрыть второй модуль, развернутый на устройстве. Выберите **Просмотр и создание**, а затем щелкните **Создать**. Это должно обновить развертывание.

        ![Проверьте и создайте обновленное развертывание.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/stop-module-deployment-6.png)
 
    1. Страница « **Установка модулей** » несколько раз. пока **состояние выполнения** модуля не будет отображаться как **остановленное**.

        ![Проверьте состояние развертывания.](media/azure-stack-edge-gpu-deploy-iot-edge-gpu-sharing/stop-module-deployment-8.png) 
    

## <a name="deploy-with-context-sharing"></a>Развертывание с совместно используемым контекстом

Теперь можно развернуть модель n-Body в двух контейнерах CUDA при выполнении MPS на устройстве. Во первых, вы включите MPS на устройстве.


1. [Подключитесь к интерфейсу PowerShell устройства](azure-stack-edge-gpu-connect-powershell-interface.md).

1. Чтобы включить MPS на устройстве, выполните `Start-HcsGpuMPS` команду.

    ```powershell
    [10.100.10.10]: PS>Start-HcsGpuMPS
    K8S-1HXQG13CL-1HXQG13:
    Set compute mode to EXCLUSIVE_PROCESS for GPU 0000191E:00:00.0.
    All done.
    Created nvidia-mps.service
    [10.100.10.10]: PS>    
    ```
1. Получение выходных данных NVIDIA SMI из интерфейса PowerShell устройства. На устройстве можно увидеть, что `nvidia-cuda-mps-server` процесс или служба MPS запущена.

    Пример выходных данных:

    ```yml
    [10.100.10.10]: PS>Get-HcsGpuNvidiaSmi
    K8S-1HXQG13CL-1HXQG13:
    
    Thu Mar  4 12:37:39 2021
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            On   | 00002C74:00:00.0 Off |                    0 |
    | N/A   36C    P8     9W /  70W |     28MiB / 15109MiB |      0%   E. Process |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |    0   N/A  N/A    122792      C   nvidia-cuda-mps-server             25MiB |
    +-----------------------------------------------------------------------------+
    [10.100.10.10]: PS>Get-HcsGpuNvidiaSmi
    ```

1. Разверните модули, которые были остановлены ранее. Задайте **требуемое состояние** для выполнения через **набор модулей**.

    Ниже приведен пример выходных данных:

    ```yml
    PS C:\WINDOWS\system32> kubectl get pods -n iotedge --kubeconfig C:\GPU-sharing\kubeconfigs\configiotuser1
    NAME                            READY   STATUS    RESTARTS   AGE
    cuda-sample1-869989578c-2zxh6   2/2     Running   0          44s
    cuda-sample2-6db6d98689-fn7mx   2/2     Running   0          44s
    edgeagent-79f988968b-7p2tv      2/2     Running   0          5d20h
    edgehub-d6c764847-l8v4m         2/2     Running   0          27m
    iotedged-55fdb7b5c6-l9zn8       1/1     Running   1          5d20h
    PS C:\WINDOWS\system32>
    ```
    Вы видите, что модули развернуты и выполняются на устройстве.

1. При развертывании модулей Эмуляция n-Body также начинает работать в обоих контейнерах. Ниже приведен пример выходных данных при завершении имитации в первом контейнере:

    ```powershell
    PS C:\WINDOWS\system32> kubectl -n iotedge logs cuda-sample1-869989578c-2zxh6 cuda-sample1
    Run "nbody -benchmark [-numbodies=<numBodies>]" to measure performance.
    ==============// snipped //===================//  snipped  //=============
    
    > Windowed mode
    > Simulation data stored in video memory
    > Single precision floating point simulation
    > 1 Devices used for simulation
    GPU Device 0: "Turing" with compute capability 7.5
    
    > Compute 7.5 CUDA device: [Tesla T4]
    40960 bodies, total time for 10000 iterations: 155256.062 ms
    = 108.062 billion interactions per second
    = 2161.232 single-precision GFLOP/s at 20 flops per interaction
    no-op
    PS C:\WINDOWS\system32> 
    ```
    Ниже приведен пример выходных данных по завершении имитации во втором контейнере:

    ```powershell
    PS C:\WINDOWS\system32> kubectl -n iotedge  --kubeconfig C:\GPU-sharing\kubeconfigs\configiotuser1 logs cuda-sample2-6db6d98689-fn7mx cuda-sample2
    Run "nbody -benchmark [-numbodies=<numBodies>]" to measure performance.
    ==============// snipped //===================//  snipped  //=============
    
    > Windowed mode
    > Simulation data stored in video memory
    > Single precision floating point simulation
    > 1 Devices used for simulation
    GPU Device 0: "Turing" with compute capability 7.5
    
    > Compute 7.5 CUDA device: [Tesla T4]
    40960 bodies, total time for 10000 iterations: 155366.359 ms
    = 107.985 billion interactions per second
    = 2159.697 single-precision GFLOP/s at 20 flops per interaction
    no-op
    PS C:\WINDOWS\system32>    
    ```      

1. Получите выходные данные NVIDIA SMI из интерфейса PowerShell устройства, когда оба контейнера работают с имитацией n-Body. Ниже приведен пример выходных данных. Существует три процесса: `nvidia-cuda-mps-server` процесс (тип C) соответствует службе MPS, а `/tmp/nbody` процессы (тип M + C) соответствуют рабочим нагрузкам n-Body, развернутым модулями. 

    ```powershell
    [10.100.10.10]: PS>Get-HcsGpuNvidiaSmi
    K8S-1HXQG13CL-1HXQG13:
    
    Thu Mar  4 12:59:44 2021
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 460.32.03    Driver Version: 460.32.03    CUDA Version: 11.2     |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |                               |                      |               MIG M. |
    |===============================+======================+======================|
    |   0  Tesla T4            On   | 00002C74:00:00.0 Off |                    0 |
    | N/A   54C    P0    69W /  70W |    242MiB / 15109MiB |    100%   E. Process |
    |                               |                      |                  N/A |
    +-------------------------------+----------------------+----------------------+
    
    +-----------------------------------------------------------------------------+
    | Processes:                                                                  |
    |  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
    |        ID   ID                                                   Usage      |
    |=============================================================================|
    |    0   N/A  N/A     56832    M+C   /tmp/nbody                        107MiB |
    |    0   N/A  N/A     56900    M+C   /tmp/nbody                        107MiB |
    |    0   N/A  N/A    122792      C   nvidia-cuda-mps-server             25MiB |
    +-----------------------------------------------------------------------------+
    [10.100.10.10]: PS>Get-HcsGpuNvidiaSmi
    ```
    
## <a name="next-steps"></a>Дальнейшие действия

- [Развертывание общей рабочей нагрузки KUBERNETES GPU на Azure Stack крае Pro](azure-stack-edge-gpu-deploy-kubernetes-gpu-sharing.md).
