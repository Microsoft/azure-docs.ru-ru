---
title: Включение пограничных параметров реестра контейнеров на устройстве Azure Stack ребра Pro GPU
description: Описывает, как включить локальный реестр контейнеров граничных границ на устройстве Azure Stack ребра Pro GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 02/22/2021
ms.author: alkohli
ms.openlocfilehash: 56b691b2755b5e248b16e338f8fd82864f5bf218
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105560341"
---
# <a name="enable-edge-container-registry-on-your-azure-stack-edge-pro-gpu-device"></a>Включение реестра граничных контейнеров на устройстве с Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается, как включить пограничные реестр контейнеров и использовать его из кластера Kubernetes на устройстве Azure Stack. Pro. В примере, приведенном в статье, описывается, как отправить образ из исходного реестра (в данном случае, реестра контейнеров Майкрософт) в реестр на Azure Stack пограничном устройстве, в реестре контейнеров ребра.

### <a name="about-edge-container-registry"></a>Сведения о реестре контейнеров граничных устройств

Контейнерные приложения вычислений выполняются на образах контейнеров, и эти образы хранятся в реестре. Реестры могут быть общедоступными, например, в качестве центра DOCKER, частного или облачного поставщика, управляемого, например реестра контейнеров Azure. Дополнительные сведения см. в разделе [о реестрах, репозиториях и образах](../container-registry/container-registry-concepts.md).

Контейнерный реестр контейнеров предоставляет репозиторий на пограничном устройстве на Azure Stack пограничной Pro. Этот реестр можно использовать для хранения частных образов контейнеров и управления ими.

В среде с несколькими узлами образы контейнеров могут быть скачаны и помещены в реестр контейнера граничных контейнеров один раз. Все пограничные приложения могут использовать для последующих развертываний реестр контейнеров граничных приложений.


## <a name="prerequisites"></a>Предварительные требования

Перед тем как начать, убедитесь в следующем.

1. У вас есть доступ к устройству Azure Stack пограничной Pro.

1. Устройство Azure Stack Edge Pro активировано, как описано в статье [Активация Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).

1. Вы включили на устройстве роль вычислений. Кластер Kubernetes также был создан на устройстве при настройке вычислений на устройстве в соответствии с инструкциями, приведенными в разделе [Настройка вычислений на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-configure-compute.md).

1. У вас есть конечная точка API Kubernetes со страницы **устройства** локального веб-интерфейса. Дополнительные сведения см. в инструкциях в статье [Получение KUBERNETES API в конечной точке](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints).

1. У вас есть доступ к клиентской системе с [поддерживаемой операционной системой](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). При использовании клиента Windows система должна запустить PowerShell 5,0 или более поздней версии для доступа к устройству.

    1. Если вы хотите извлечь и отправить собственные образы контейнеров, убедитесь, что в системе установлен клиент DOCKER. При использовании клиента Windows [установите DOCKER Desktop в Windows](https://docs.docker.com/docker-for-windows/install/).  


## <a name="enable-container-registry-as-add-on"></a>Включить реестр контейнеров в качестве надстройки

Первым шагом является включение пограничных параметров реестра контейнеров в качестве надстройки.

1. [Подключитесь к интерфейсу PowerShell устройства](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface). 

1. Чтобы включить реестр контейнеров в качестве надстройки, введите: 

    `Set-HcsKubernetesContainerRegistry`
    
    Выполнение этой операции займет несколько минут.

    Ниже приведен пример выходных данных этой команды:  
            
    ```powershell
    [10.128.44.40]: PS>Set-HcsKubernetesContainerRegistry
    Operation completed successfully. Use Get-HcsKubernetesContainerRegistryInfo for credentials    
    ```
            
1. Чтобы получить сведения о реестре контейнеров, введите:

    `Get-HcsKubernetesContainerRegistryInfo`

    Вот пример из этой команды:  
    
    ```powershell
    [10.128.44.40]: PS> Get-HcsKubernetesContainerRegistryInfo
                
    Endpoint                                   IPAddress    Username     Password
    --------                                   ---------    --------     --------
    ecr.dbe-hw6h1t2.microsoftdatabox.com:31001 10.128.44.41 ase-ecr-user i3eTsU4zGYyIgxV
    ``` 

1. Запишите имя пользователя и пароль из выходных данных `Get-HcsKubernetesContainerRegistryInfo` . Эти учетные данные используются для входа в пограничные реестр контейнеров при передаче образов.         


## <a name="manage-container-registry-images"></a>Управление образами реестра контейнеров

Вам может потребоваться доступ к реестру контейнеров извне устройства Azure Stack. Также может потребоваться отправить или извлечь образы в реестре.

Выполните следующие действия, чтобы получить доступ к реестру контейнеров граничных данных:

1. Получите сведения о конечной точке для реестра граничных контейнеров.
    1. В локальном пользовательском интерфейсе устройства перейдите на **устройство**.
    1. Откройте **конечную точку реестра контейнеров граничных контейнеров**.
        ![Конечная точка реестра контейнера ребра на странице устройства](media/azure-stack-edge-gpu-edge-container-registry/get-edge-container-registry-endpoint-1.png) 
    1. Скопируйте эту конечную точку и Создайте соответствующую запись DNS в `C:\Windows\System32\Drivers\etc\hosts` файле клиента, чтобы подключиться к конечной точке реестра контейнера ребра. 

        <IP address of the Kubernetes main node>    <Edge container registry endpoint> 
        
        ![Добавление записи DNS для конечной точки реестра контейнера ребра](media/azure-stack-edge-gpu-edge-container-registry/add-domain-name-service-entry-hosts-1.png)    

1. Скачайте сертификат реестра контейнера ребра из локального пользовательского интерфейса. 
    1. В локальном пользовательском интерфейсе устройства перейдите к разделу **Сертификаты**.
    1. Поиск записи для **сертификата реестра контейнера ребра**. Справа от этой записи выберите **скачивание** , чтобы скачать сертификат реестра граничных контейнеров в клиентской системе, который будет использоваться для доступа к устройству. 

        ![Скачивание сертификата конечной точки реестра контейнера ребра](media/azure-stack-edge-gpu-edge-container-registry/download-edge-container-registry-endpoint-certificate-1.png)  

1. Установите скачанный сертификат на клиенте. При использовании клиента Windows выполните следующие действия. 
    1. Выберите сертификат и в **мастере импорта сертификатов** выберите сохранить расположение как **локальный компьютер**. 

        ![Установка сертификата 1](media/azure-stack-edge-gpu-edge-container-registry/install-certificate-1.png) 
    
    1. Установите сертификат на локальном компьютере в доверенном корневом хранилище. 

        ![Установка сертификата 2](media/azure-stack-edge-gpu-edge-container-registry/install-certificate-2.png) 

1. После установки сертификата перезапустите клиент DOCKER в системе.

1. Войдите в реестр контейнеров ребра. Тип:

    `docker login <Edge container registry endpoint> -u <username> -p <password>`

    Укажите конечную точку реестра контейнера ребра на странице **устройства** , а также имя пользователя и пароль, полученные из выходных данных `Get-HcsKubernetesContainerRegistryInfo` . 

1. Используйте команды DOCKER Push или pull для отправки или извлечения образов контейнеров из реестра контейнеров.
 
    1. Извлечение образа из образа реестра контейнеров Microsoft. Тип:
        
        `docker pull <Full path to the container image in the Microsoft Container Registry>`
       
    1. Создайте псевдоним образа, который был извлечен с полным путем к реестру.

        `docker tag <Path to the image in the Microsoft container registry> <Path to the image in the Edge container registry/Image name with tag>`

    1. Отправьте образ в реестр.
    
        `docker push <Path to the image in the Edge container registry/Image name with tag>`

    1. Запустите образ, отправленный в реестр.
    
        `docker run -it --rm -p 8080:80 <Path to the image in the Edge container registry/Image name with tag>`

        Ниже приведен пример выходных данных команд Pull и Push.

        ```powershell
        PS C:\WINDOWS\system32> docker login ecr.dbe-hw6h1t2.microsoftdatabox.com:31001 -u ase-ecr-user -p 3bbo2sOtDe8FouD
        WARNING! Using --password via the CLI is insecure. Use --password-stdin.
        Login Succeeded
        PS C:\WINDOWS\system32> docker pull mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        1.17.5-alpine: Pulling from oss/nginx/nginx
        Digest: sha256:5466bbc0a989bd1cd283c0ba86d9c2fc133491ccfaea63160089f47b32ae973b
        Status: Image is up to date for mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine
        PS C:\WINDOWS\system32> docker tag mcr.microsoft.com/oss/nginx/nginx:1.17.5-alpine ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        PS C:\WINDOWS\system32> docker push ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        The push refers to repository [ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx]
        bba7d2385bc1: Pushed
        77cae8ab23bf: Pushed
        2.0: digest: sha256:b4c0378c841cd76f0b75bc63454bfc6fe194a5220d4eab0d75963bccdbc327ff size: 739
        PS C:\WINDOWS\system32> docker run -it --rm -p 8080:80 ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        2020/11/10 00:00:49 [error] 6#6: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 172.17.0.1, server: localhost, request: "GET /favicon.ico HTTP/1.1", host: "localhost:8080", referrer: "http://localhost:8080/"
        172.17.0.1 - - [10/Nov/2020:00:00:49 +0000] "GET /favicon.ico HTTP/1.1" 404 555 "http://localhost:8080/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.183 Safari/537.36" "-"
        ^C
        PS C:\WINDOWS\system32>    
        ```
    
1. Перейдите по адресу `http://localhost:8080`, чтобы просмотреть запущенный контейнер. В этом случае вы увидите, что на сервере nginx работает.

    ![Просмотр работающего контейнера](media/azure-stack-edge-gpu-edge-container-registry/view-running-container-1.png)

    Чтобы остановить и удалить контейнер, нажмите `Control+C`.

 

## <a name="use-edge-container-registry-images-via-kubernetes-pods"></a>Использование пограничных образов реестра контейнеров с помощью модулей Kubernetes Pod

Теперь вы можете развернуть образ, отправленный в реестре контейнеров с граничным контейнером, с помощью модулей Kubernetes.

1. Чтобы развернуть образ, необходимо настроить доступ к кластеру через *kubectl*. Создайте пространство имен, пользователя, предоставьте пользователю доступ к пространству имен и получите файл *конфигурации* . Убедитесь, что вы можете подключиться к Kubernetes Pod. 
    
    Выполните все действия, описанные в статье [Подключение к кластеру Kubernetes и управление им с помощью kubectl на устройстве Azure Stack с графическим](azure-stack-edge-gpu-create-kubernetes-cluster.md)стандартом Pro. 

    Ниже приведен пример выходных данных для пространства имен на устройстве, откуда пользователь может получить доступ к кластеру Kubernetes.

    ```powershell
    [10.128.44.40]: PS>New-HcsKubernetesNamespace -Namespace myecr
    [10.128.44.40]: PS>New-HcsKubernetesUser -UserName ecruser
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01URXdOVEF6TkRJek1Gb1hEVE13TVRFd016QXpOREl6TUZvd0ZURVRNQkVnNjOVRLWndCQ042cm1XQms2eXFwcXI1MUx6bApTaXMyTy91UEJ2YXNSSUUzdzgrbmEwdG1aTERZZ2F6MkQwMm42Q29mUmtyUTR2d1lLTnR1MlpzR3pUdz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
        server: https://compute.dbe-hw6h1t2.microsoftdatabox.com:6443
        name: kubernetes
        ===================CUT=========================================CUT==============
        client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUMwRENDQWJpZ0F3SUJBZ0lJYmVWRGJSTzZ3ell3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURFeE1EVXdNelF5TXpCYUZ3MHlNVEV4TURreU16UTRNal
        ===================CUT=========================================CUT==============
        DMVUvN3lFOG5UU3k3b2VPWitUeHdzCjF1UDByMjhDZ1lCdHdRY0ZpcFh1blN5ak16dTNIYjhveFI2V3VWWmZldFFKNElKWEFXOStVWGhKTFhyQ2x4bUcKWHRtbCt4UU5UTzFjQVNKRVZWVDd6Tjg2ay9kSU43S3JIVkdUdUxlUDd4eGVjV2VRcWJrZEVScUsxN0liTXpiVApmbnNxc0dobEdmLzdmM21kTGtyOENrcWs5TU5aM3MvUVIwRlFCdk94ZVpuUlpTeDVGbUR5S1E9PQotLS0tLUVORCBSU0EgUFJJVkFURSBLRVktLS0tLQo=

    [10.128.44.40]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace myecr -UserName ecruser
    [10.128.44.40]: PS>kubectl get pods -n "myecr"
    No resources found.
    PS C:\WINDOWS\system32>
    ```  

2. Секреты извлечения образа уже заданы во всех пространствах имен Kubernetes на устройстве. Секреты можно получить с помощью `get secrets` команды. Пример выходных данных:

    ```powershell
    PS C:\WINDOWS\system32> .\kubectl.exe get secrets -n myecr
    NAME                  TYPE                                  DATA   AGE
    ase-ecr-credentials   kubernetes.io/dockerconfigjson        1      99m
    default-token-c7kww   kubernetes.io/service-account-token   3      107m
    sec-smbcredentials    microsoft.com/smb                     2      99m
    PS C:\WINDOWS\system32>   
    ```    

3. Разверните модуль Pod в пространстве имен с помощью kubectl. Используйте следующие `yaml` . 

    Замените образ: на `<image-name>` образ, отправленный в реестр контейнеров. Используйте секреты в пространствах имен, используя Имажепуллсекретс с именем: `ase-ecr-credentials` .
    
    ```yml
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
      - name: nginx
        image: ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/nginx:2.0
        imagePullPolicy: Always
      imagePullSecrets:
      - name: ase-ecr-credentials
    ```

4. Примените развертывание в пространстве имен, созданном с помощью команды Apply. Убедитесь, что контейнер работает. Пример выходных данных:
   
    ```yml
    PS C:\Windows\System32> .\kubectl.exe apply -f .\deployment.yml -n myecr
    pod/nginx configured
    PS C:\Windows\System32> .\kubectl.exe get pods -n myecr
    NAME    READY   STATUS    RESTARTS   AGE
    nginx   1/1     Running   0          27m
    PS C:\Windows\System32>
    ```

## <a name="delete-container-registry-images"></a>Удаление образов реестра контейнеров

Хранилище реестра пограничных контейнеров размещается в локальном общем ресурсе на устройстве Azure Stack ребра Pro, которое ограничено доступным хранилищем на устройстве. Вы обязаны удалять неиспользуемые образы DOCKER из реестра контейнеров с помощью API DOCKER HTTP v2 ( https://docs.docker.com/registry/spec/api/) .  

Чтобы удалить один или несколько образов контейнеров, выполните следующие действия.

1. Задайте имя образа, который нужно удалить.

    ```powershell
    PS C:\WINDOWS\system32> $imageName="nginx"    
    ```

1. Задайте имя пользователя и пароль для реестра контейнеров как учетные данные PS.

    ```powershell
    PS C:\WINDOWS\system32> $username="ase-ecr-user"
    PS C:\WINDOWS\system32> $password="3bbo2sOtDe8FouD"
    PS C:\WINDOWS\system32> $securePassword = ConvertTo-SecureString $password -AsPlainText -Force
    PS C:\WINDOWS\system32> $credential = New-Object System.Management.Automation.PSCredential ($username, $securePassword)    
    ```

1. Перечисление тегов, связанных с изображением

    ```powershell
    PS C:\WINDOWS\system32> $tags = Invoke-RestMethod -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/nginx/tags/list" | Select-Object -ExpandProperty tags
    PS C:\WINDOWS\system32> $tags
    2.0
    PS C:\WINDOWS\system32> $tags = Invoke-RestMethod -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/tags/list" | Select-Object -ExpandProperty tags
    PS C:\WINDOWS\system32> $tags
    2.0
    PS C:\WINDOWS\system32>    
    ```
  
1. Перечислите хэш-код, связанный с тегом, который вы хотите удалить. В этом случае используется $tags из выходных данных команды выше. Если у вас несколько тегов, выберите один из них и используйте его в следующей команде.

    ```powershell
    PS C:\WINDOWS\system32> $response = Invoke-WebRequest -Method Head -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/manifests/$tags" -Headers @{ 'Accept' = 'application/vnd.docker.distribution.manifest.v2+json' }
    PS C:\WINDOWS\system32> $digest = $response.Headers['Docker-Content-Digest']
    PS C:\WINDOWS\system32> $digest
    sha256:b4c0378c841cd76f0b75bc63454bfc6fe194a5220d4eab0d75963bccdbc327ff
    PS C:\WINDOWS\system32>    
    ```
1. Удаление образа с помощью дайджеста Image: TAG

    ```powershell
    PS C:\WINDOWS\system32> Invoke-WebRequest -Method Delete -Credential $credential -Uri "https://ecr.dbe-hw6h1t2.microsoftdatabox.com:31001/v2/$imageName/manifests/$digest" | Select-Object -ExpandProperty StatusDescription    
    ```

После удаления неиспользуемых образов пространство, связанное с изображениями без ссылок, автоматически освобождается процессом, который выполняется ночью. 

## <a name="next-steps"></a>Дальнейшие действия

- [Развертывание приложения без отслеживания состояния на Azure Stack пограничной Pro](./azure-stack-edge-gpu-deploy-stateless-application-kubernetes.md).