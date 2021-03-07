---
title: Развертывание контроллера данных ARC в Azure на устройстве Azure Stack ребра Pro GPU | Документация Майкрософт
description: В этой статье описывается, как развернуть контроллер данных Azure Arc и службы данных Azure на устройстве Azure Stack с помощью GPU Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/05/2021
ms.author: alkohli
ms.openlocfilehash: 3ff07c773a2976a296d13510a3ddd7b41217aaa2
ms.sourcegitcommit: 5bbc00673bd5b86b1ab2b7a31a4b4b066087e8ed
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102437627"
---
# <a name="deploy-azure-data-services-on-your-azure-stack-edge-pro-gpu-device"></a>Развертывание служб данных Azure на устройстве с Azure Stack ребра Pro GPU

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этой статье описывается процесс создания контроллера данных ARC в Azure и последующее развертывание служб данных Azure на устройстве Azure Stack с помощью GPU Pro. 

Контроллер данных Arc Azure — это локальная плоскость управления, которая позволяет выполнять службы данных Azure в управляемых пользователем средах. После создания контроллера данных ARC в Azure в кластере Kubernetes, работающем на устройстве Pro Azure Stack, можно развернуть службы данных Azure, такие как SQL Управляемый экземпляр (Предварительная версия), на этом контроллере данных.

Процедура создания контроллера данных и последующего развертывания Управляемый экземпляр SQL включает в себя использование PowerShell и `kubectl` собственного средства, предоставляющего доступ из командной строки к кластеру Kubernetes на устройстве.


## <a name="prerequisites"></a>Предварительные требования

Перед тем как начать, убедитесь в следующем.

1. Вы получили доступ к устройству с Azure Stack пограничным Pro и активировали устройство, как описано в разделе [активация Azure Stack ребра Pro](azure-stack-edge-gpu-deploy-activate.md).

1. Вы включили роль вычислений на устройстве. Кластер Kubernetes также был создан на устройстве при настройке вычислений на устройстве в соответствии с инструкциями, приведенными в разделе [Настройка вычислений на устройстве Azure Stack пограничной Pro](azure-stack-edge-gpu-deploy-configure-compute.md).

1. У вас есть конечная точка API Kubernetes со страницы **устройства** локального веб-интерфейса. Дополнительные сведения см. в инструкциях в статье [Получение KUBERNETES API в конечной точке](azure-stack-edge-gpu-deploy-configure-compute.md#get-kubernetes-endpoints).

1. У вас есть доступ к клиенту, который будет подключаться к устройству. 
    1. В этой статье для доступа к устройству используется клиентская система Windows с PowerShell 5,0 или более поздней версии. Можно использовать любой другой клиент с [поддерживаемой операционной системой](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). 
    1. Установите `kubectl` на клиенте. Для версии клиента:
        1. Определяет версию сервера Kubernetes, установленную на устройстве. В локальном пользовательском интерфейсе устройства перейдите на страницу **обновления программного обеспечения** . Обратите внимание на **версию сервера Kubernetes** на этой странице.
        1. Загрузите клиент, версия которого отличается от версии главного узла не более чем на единицу. Версия клиента, но может привести к тому, что основная версия будет вынимать до одной дополнительной версии. Например, мастер v 1.3 должен работать с узлами v 1.1, v 1.2 и v 1.3 и работать с клиентами версии 1.2, v 1.3 и v 1.4. Дополнительные сведения о версии клиента Kubernetes см. в разделе [Политика поддержки отклонения версий и версий Kubernetes](https://kubernetes.io/docs/setup/release/version-skew-policy/#supported-version-skew).
    
1. При необходимости [установите клиентские средства для развертывания и управления службами данных, поддерживающими службу "Дуга Azure](../azure-arc/data/install-client-tools.md)". Эти средства не являются обязательными, но рекомендуемыми.  
1. Убедитесь, что на вашем устройстве имеется достаточно ресурсов для подготовки контроллера данных и одного Управляемый экземпляр SQL. Для контроллера данных и одного Управляемый экземпляр SQL потребуется как минимум 16 ГБ ОЗУ и 4 ядра ЦП. Подробные инструкции см. в статье [минимальные требования к развертыванию служб данных с поддержкой службы "Дуга Azure](../azure-arc/data/sizing-guidance.md#minimum-deployment-requirements)".


## <a name="configure-kubernetes-external-service-ips"></a>Настройка IP-адресов внешней службы Kubernetes

1. Перейдите к локальному веб-ИНТЕРФЕЙСу устройства и перейдите к разделу **вычислений**.
1. Выберите сеть, для которой включена среда выполнения вычислений. 

    ![Страницы "Вычисления" в локальном пользовательском веб-интерфейсе, часть 2](./media/azure-stack-edge-gpu-deploy-arc-data-controller/compute-network-1.png)

1. Убедитесь, что предоставлены три дополнительных IP-адреса внешней службы Kubernetes (в дополнение к IP-адресам, которые уже настроены для других внешних служб или контейнеров). При создании Управляемый экземпляр SQL в контроллере данных будут использоваться два IP-адреса службы, а третий — IP. Для каждой дополнительной службы данных, которую вы будете развертывать, потребуется один IP-адрес. 

    ![Страница "Вычисления" в локальном пользовательском веб-интерфейсе, часть 3](./media/azure-stack-edge-gpu-deploy-arc-data-controller/compute-network-2.png)

1. Примените параметры, и эти новые IP-адреса немедленно вступят в силу в уже существующем кластере Kubernetes. 


## <a name="deploy-azure-arc-data-controller"></a>Развертывание контроллера данных Azure Arc

Перед развертыванием контроллера данных необходимо создать пространство имен.

### <a name="create-namespace"></a>Создание пространства имен 

Создайте новое выделенное пространство имен, в котором будет развернут контроллер данных. Вы также создадите пользователя, а затем предоставите ему доступ к созданному пространству имен. 

> [!NOTE]
> Для пространства имен и имен пользователей применяются [соглашения об именовании поддоменов DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names) .

1. [Подключитесь к интерфейсу PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).
1. Создание пространства имен. Тип:

    `New-HcsKubernetesNamespace -Namespace <Name of namespace>`

1. Создайте пользователя. Тип: 

    `New-HcsKubernetesUser -UserName <User name>`

1. Файл конфигурации отображается в виде обычного текста. Скопируйте этот файл и сохраните его как файл *конфигурации* . 

    > [!IMPORTANT]
    > Не сохраняйте файл конфигурации в формате *txt* , сохраните файл без расширения.

1. Файл конфигурации должен находиться в `.kube` папке профиля пользователя на локальном компьютере. Скопируйте файл в эту папку в профиле пользователя.

    ![Расположение файла конфигурации на клиенте](media/azure-stack-edge-j-series-create-kubernetes-cluster/location-config-file.png)
1. Предоставьте пользователю доступ к созданному пространству имен. Тип: 

    `Grant-HcsKubernetesNamespaceAccess -Namespace <Name of namespace> -UserName <User name>`

    Ниже приведен пример выходных данных предыдущих команд. В этом примере мы создадим `myadstest` пространство имен, `myadsuser` пользователя и предоставили ему доступ к пространству имен.
    
    ```powershell
    [10.100.10.10]: PS>New-HcsKubernetesNamespace -Namespace myadstest
    [10.100.10.10]: PS>New-HcsKubernetesUser -UserName myadsuser
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBD=======//snipped//=======VSVElGSUNBVEUtLS0tLQo=
        server: https://compute.myasegpudev.wdshcsso.com:6443
      name: kubernetes
    contexts:
    - context:
        cluster: kubernetes
        user: myadsuser
      name: myadsuser@kubernetes
    current-context: myadsuser@kubernetes
    kind: Config
    preferences: {}
    users:
    - name: myadsuser
      user:
        client-certificate-data: LS0tLS1CRUdJTiBDRV=========//snipped//=====EE9PQotLS0kFURSBLRVktLS0tLQo=
    
    [10.100.10.10]: PS>Grant-HcsKubernetesNamespaceAccess -Namespace myadstest -UserName myadsuser
    [10.100.10.10]: PS>Set-HcsKubernetesAzureArcDataController -SubscriptionId db4e2fdb-6d80-4e6e-b7cd-736098270664 -ResourceGroupName myasegpurg -Location "EastUS" -UserName myadsuser -Password "Password1" -DataControllerName "arctestcontroller" -Namespace myadstest
    [10.100.10.10]: PS>
    ```
1. Добавьте запись DNS в файл hosts в системе. 

    1. Запустите Блокнот от имени администратора и откройте `hosts` файл, расположенный по адресу `C:\windows\system32\drivers\etc\hosts` . 
    2. Используйте сведения, сохраненные на странице **устройства** в локальном пользовательском интерфейсе (предварительные требования), чтобы создать запись в файле hosts. 

        Например, скопируйте эту конечную точку, `https://compute.myasegpudev.microsoftdatabox.com/[10.100.10.10]` чтобы создать следующую запись с IP-адресом устройства и DNS-доменом: 

        `10.100.10.10 compute.myasegpudev.microsoftdatabox.com`

1. Чтобы убедиться, что можно подключиться к Kubernetes Pod, запустите командную строку или сеанс PowerShell. Тип:
    
    ```powershell
    PS C:\WINDOWS\system32> kubectl get pods -n "myadstest"
    No resources found.
    PS C:\WINDOWS\system32>
    ```
Теперь вы можете развернуть контроллеры данных и приложения служб данных в пространстве имен, а затем просмотреть приложения и их журналы.

### <a name="create-data-controller"></a>Создание контроллера данных

Контроллер данных — это коллекция модулей Pod, развернутых в кластере Kubernetes для предоставления API, службы контроллера, загрузчика, а также баз данных и панелей мониторинга. Выполните следующие действия, чтобы создать контроллер данных в кластере Kubernetes, который существует на Azure Stack пограничном устройстве в пространстве имен, созданном ранее.   

1. Соберите следующие сведения, необходимые для создания контроллера данных.

    
    |Column1  |Column2  |
    |---------|---------|
    |Имя контроллера данных     |Описательное имя для контроллера данных. Например, `arctestdatacontroller`.         |
    |Имя пользователя контроллера данных     |Любое имя пользователя администратора контроллера данных. Имя пользователя и пароль контроллера данных используются для проверки подлинности API контроллера данных для выполнения административных функций.          |
    |Пароль контроллера данных     |Пароль для пользователя администратора контроллера данных. Выберите безопасный пароль и предоставьте к нему доступ только тем пользователям, которым требуются права администратора кластера.         |
    |Имя пространства имен Kubernetes     |Имя пространства имен Kubernetes, в котором необходимо создать контроллер данных.         |
    |идентификатор подписки Azure;     |Идентификатор GUID подписки Azure для того места, где требуется создать ресурс контроллера данных в Azure.         |
    |Имя группы ресурсов Azure     |Имя группы ресурсов, в которой должен быть создан ресурс контроллера данных в Azure.         |
    |Расположение Azure.     |Расположение Azure, в котором будут храниться метаданные ресурсов контроллера данных в Azure. Список доступных регионов см. в статье Глобальная инфраструктура и продукты Azure по регионам.|


1. Подключитесь к интерфейсу PowerShell. Чтобы создать контроллер данных, введите: 

    ```powershell
    Set-HcsKubernetesAzureArcDataController -SubscriptionId <Subscription ID> -ResourceGroupName <Resource group name> -Location <Location without spaces> -UserName <User you created> -Password <Password to authenticate to Data Controller> -DataControllerName <Data Controller Name> -Namespace <Namespace you created>    
    ```
    Ниже приведен пример выходных данных предыдущих команд.

    ```powershell
    [10.100.10.10]: PS>Set-HcsKubernetesAzureArcDataController -SubscriptionId db4e2fdb-6d80-4e6e-b7cd-736098270664 -ResourceGroupName myasegpurg -Location "EastUS" -UserName myadsuser -Password "Password1" -DataControllerName "arctestcontroller" -Namespace myadstest
    [10.100.10.10]: PS> 
    ```
    
    Для завершения развертывания может потребоваться около 5 минут.

    > [!NOTE]
    > Контроллер данных, созданный в кластере Kubernetes на устройстве Azure Stack ребра Pro, работает только в отключенном режиме в текущем выпуске.

### <a name="monitor-data-creation-status"></a>Мониторинг состояния создания данных

1. Откройте другое окно PowerShell.
1. Используйте следующую `kubectl` команду для мониторинга состояния создания контроллера данных. 

    ```powershell
    kubectl get datacontroller/<Data controller name> --namespace <Name of your namespace>
    ```
    При создании контроллера его состояние должно быть `Ready` .
    Ниже приведен пример выходных данных предыдущей команды:

    ```powershell
    PS C:\WINDOWS\system32> kubectl get datacontroller/arctestcontroller --namespace myadstest
    NAME                STATE
    arctestcontroller   Ready
    PS C:\WINDOWS\system32>
    ```
1. Чтобы указать IP-адреса, назначенные внешним службам, выполняемым на контроллере данных, используйте `kubectl get svc -n <namespace>` команду. Пример выходных данных:

    ```powershell
    PS C:\WINDOWS\system32> kubectl get svc -n myadstest
    NAME                      TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                                       AGE
    controldb-svc             ClusterIP      172.28.157.130   <none>        1433/TCP,8311/TCP,8411/TCP                    3d21h
    controller-svc            ClusterIP      172.28.123.251   <none>        443/TCP,8311/TCP,8301/TCP,8411/TCP,8401/TCP   3d21h
    controller-svc-external   LoadBalancer   172.28.154.30    10.57.48.63   30080:31090/TCP                               3d21h
    logsdb-svc                ClusterIP      172.28.52.196    <none>        9200/TCP,8300/TCP,8400/TCP                    3d20h
    logsui-svc                ClusterIP      172.28.85.97     <none>        5601/TCP,8300/TCP,8400/TCP                    3d20h
    metricsdb-svc             ClusterIP      172.28.255.103   <none>        8086/TCP,8300/TCP,8400/TCP                    3d20h
    metricsdc-svc             ClusterIP      172.28.208.191   <none>        8300/TCP,8400/TCP                             3d20h
    metricsui-svc             ClusterIP      172.28.158.163   <none>        3000/TCP,8300/TCP,8400/TCP                    3d20h
    mgmtproxy-svc             ClusterIP      172.28.228.229   <none>        443/TCP,8300/TCP,8311/TCP,8400/TCP,8411/TCP   3d20h
    mgmtproxy-svc-external    LoadBalancer   172.28.166.214   10.57.48.64   30777:30621/TCP                               3d20h
    sqlex-svc                 ClusterIP      None             <none>        1433/TCP                                      3d20h
    PS C:\WINDOWS\system32>
    ```

## <a name="deploy-sql-managed-instance"></a>Развертывание управляемого экземпляра SQL

После успешного создания контроллера данных можно использовать шаблон для развертывания Управляемый экземпляр SQL на контроллере данных.

### <a name="deployment-template"></a>Шаблон развертывания

Используйте следующий шаблон развертывания для развертывания Управляемый экземпляр SQL на контроллере данных на устройстве.

```yml
apiVersion: v1
data:
    password: UGFzc3dvcmQx
    username: bXlhZHN1c2Vy
kind: Secret
metadata:
    name: sqlex-login-secret
type: Opaque
---
apiVersion: sql.arcdata.microsoft.com/v1alpha1
kind: sqlmanagedinstance
metadata:
    name: sqlex
spec:
    limits:
    memory: 4Gi
    vcores: "4"
    requests:
    memory: 2Gi
    vcores: "1"
    service:
    type: LoadBalancer
    storage:
    backups:
        className: ase-node-local
        size: 5Gi
    data:
        className: ase-node-local
        size: 5Gi
    datalogs:
        className: ase-node-local
        size: 5Gi
    logs:
        className: ase-node-local
        size: 1Gi
```


#### <a name="metadata-name"></a>Имя метаданных
    
Имя метаданных — это имя Управляемый экземпляр SQL. Связанный модуль в предыдущем `deployment.yaml` случае будет иметь имя `sqlex-n` ( `n` — число модулей Pod, связанных с приложением). 
    
#### <a name="password-and-username-data"></a>Данные пароля и имени пользователя

Имя пользователя и пароль контроллера данных используются для проверки подлинности API контроллера данных для выполнения административных функций. Секрет Kubernetes для имени пользователя и пароля контроллера данных в шаблоне развертывания — это строки в кодировке Base64. 

Вы можете использовать онлайн-инструмент для кодирования требуемого имени пользователя и пароля в Base64 или использовать встроенные средства CLI в зависимости от платформы. При использовании средства кодирования Base64 в Интернете укажите имя пользователя и строки пароля (введенные при создании контроллера данных) в средстве, и средство создаст соответствующие строки в кодировке Base64.
    
#### <a name="service-type"></a>Service type (Тип службы)

Для типа службы необходимо задать значение `LoadBalancer` .
    
#### <a name="storage-class-name"></a>Имя класса хранилища

Вы можете указать класс хранения на Azure Stack пограничном устройстве, которое будет использоваться развертыванием для данных, резервных копий, журналов данных и журналов. Используйте  `kubectl get storageclass` команду, чтобы получить класс хранения, развернутый на устройстве.

```powershell
PS C:\WINDOWS\system32> kubectl get storageclass
NAME             PROVISIONER      RECLAIMPOLICY  VOLUMEBINDINGMODE     ALLOWVOLUMEEXPANSION   AGE
ase-node-local   rancher.io/local-path   Delete  WaitForFirstConsumer  false                  5d23h
PS C:\WINDOWS\system32>
```
В предыдущем примере выходных данных класс хранения `ase-node-local` на устройстве должен быть указан в шаблоне.
 
#### <a name="spec"></a>Спецификаци

Чтобы создать Управляемый экземпляр SQL на устройстве Azure Stack пограничных устройств, можно указать требования к памяти и ЦП в разделе спецификаций `deployment.yaml` . Каждый управляемый экземпляр SQL должен запрашивать минимум 2 ГБ памяти и 1 ядра ЦП, как показано в следующем примере. 

```yml
spec:
    limits:
    memory: 4Gi
    vcores: "4"
    requests:
    memory: 2Gi
    vcores: "1"
```  

Подробные инструкции по выбору размера для контроллера данных и 1 Управляемый экземпляр SQL см. в статье [сведения о размере управляемого экземпляра SQL](../azure-arc/data/sizing-guidance.md#sql-managed-instance-sizing-details).

### <a name="run-deployment-template"></a>Запуск шаблона развертывания

Выполните `deployment.yaml` команду с помощью следующей команды:

```powershell
kubectl create -n <Name of namespace that you created> -f <Path to the deployment yaml> 
```

Ниже приведен пример выходных данных следующей команды:

```powershell
PS C:\WINDOWS\system32> kubectl get pods -n "myadstest"
No resources found.
PS C:\WINDOWS\system32> 
PS C:\WINDOWS\system32> kubectl create -n myadstest -f C:\azure-arc-data-services\sqlex.yaml  secret/sqlex-login-secret created
sqlmanagedinstance.sql.arcdata.microsoft.com/sqlex created
PS C:\WINDOWS\system32> kubectl get pods --namespace myadstest
NAME                 READY   STATUS    RESTARTS   AGE
bootstrapper-mv2cd   1/1     Running   0          83m
control-w9s9l        2/2     Running   0          78m
controldb-0          2/2     Running   0          78m
controlwd-4bmc5      1/1     Running   0          64m
logsdb-0             1/1     Running   0          64m
logsui-wpmw2         1/1     Running   0          64m
metricsdb-0          1/1     Running   0          64m
metricsdc-fb5r5      1/1     Running   0          64m
metricsui-56qzs      1/1     Running   0          64m
mgmtproxy-2ckl7      2/2     Running   0          64m
sqlex-0              3/3     Running   0          13m
PS C:\WINDOWS\system32>
```

В `sqlex-0` примере выходных данных Pod указывается состояние SQL управляемый экземпляр.

## <a name="remove-data-controller"></a>Удаление контроллера данных

Чтобы удалить контроллер данных, удалите выделенное пространство имен, в котором он развернут.


```powershell
kubectl delete ns <Name of your namespace>
```


## <a name="next-steps"></a>Дальнейшие действия

- [Развертывание приложения без отслеживания состояния на Azure Stack пограничной Pro](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md).
