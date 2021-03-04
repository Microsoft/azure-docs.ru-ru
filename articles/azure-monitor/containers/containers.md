---
title: Решение мониторинга контейнеров в Azure Monitor | Документация Майкрософт
description: Решение для мониторинга контейнеров в Azure Monitor помогает просматривать узлы контейнеров DOCKER и Windows и управлять ими в одном расположении.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/06/2020
ms.openlocfilehash: 59f81103e72be6c0f1b629981fdb53c8467000a9
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102037427"
---
# <a name="container-monitoring-solution-in-azure-monitor"></a>Решение для мониторинга контейнеров в Azure Monitor

![Символ решения "Контейнеры"](./media/containers/containers-symbol.png)

В этой статье описывается, как настроить и использовать решение для мониторинга контейнеров в Azure Monitor, которое помогает просматривать узлы контейнеров DOCKER и Windows и управлять ими в одном расположении. Docker — это система виртуализации программного обеспечения, с помощью которой можно создавать контейнеры, автоматизирующие развертывание программного обеспечения в соответствующей ИТ-инфраструктуре.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

Решение показывает, какие контейнеры запущены, какой образ контейнера запущен и где выполняются контейнеры. Также можно просматривать подробные сведения аудита, в том числе команды, используемые в контейнерах. Кроме того, вы можете устранять неполадки контейнеров, просматривая централизованные журналы и выполняя в них поиск, без необходимости удаленного просмотра узлов Docker или Windows. Решение позволяет находить контейнеры, содержащие ошибки и использующие слишком много ресурсов на узле. Также у вас есть возможность централизованно просматривать сведения об использовании и производительности ЦП, памяти, хранилища и сети по контейнерам. На компьютерах под управлением Windows можно централизовать и сравнивать журналы из Windows Server, Hyper-V и контейнеров Docker. Решение поддерживает следующие оркестраторы контейнеров:

- Docker Swarm
- DC/OS
- Kubernetes
- Service Fabric
- Red Hat OpenShift.

Если вы развернули контейнеры в [Azure Service Fabric](../../service-fabric/service-fabric-overview.md), рекомендуется включить [решение Service Fabric](../../service-fabric/service-fabric-diagnostics-oms-setup.md) и это решение, включив мониторинг событий кластера. Перед включением решения Service Fabric ознакомьтесь [с разрешениями Service Fabric](../../service-fabric/service-fabric-diagnostics-event-analysis-oms.md) , чтобы понять, что оно предоставляет и как его использовать.

Если вы заинтересованы в мониторинге производительности рабочих нагрузок, развернутых в средах Kubernetes, которые размещены в Службе Azure Kubernetes (AKS), см. статью [Обзор службы "Azure Monitor для контейнеров"](../containers/container-insights-overview.md). Решение для мониторинга контейнеров не поддерживает мониторинг этой платформы.  

На следующей схеме показаны связи между различными узлами контейнеров и агентами с Azure Monitor.

![Схема контейнеров](./media/containers/containers-diagram.png)

## <a name="system-requirements-and-supported-platforms"></a>Требования к системе и поддерживаемые платформы

Сначала необходимо выполнить следующие требования.

### <a name="container-monitoring-solution-support-for-docker-orchestrator-and-os-platform"></a>Поддержка решений для мониторинга контейнеров: оркестратор Docker и платформа ОС

В следующей таблице показано, как система управления DOCKER и отслеживание операционной системы поддерживают инвентаризацию контейнеров, производительность и журналы с Azure Monitor.   

|Оркестрации DOCKER | ACS | Linux | Windows | Контейнер<br>Inventory (Товары) | Образ —<br>Inventory (Товары) | Узел<br>Inventory (Товары) | Контейнер<br>Производительность | Контейнер<br>Событие | Событие<br>Журнал | Контейнер<br>Журнал |
|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
| Kubernetes | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; |
| Mesosphere<br>DC/OS | &#8226; | &#8226; | | &#8226; | &#8226; | &#8226; | &#8226;| &#8226; | &#8226; | &#8226; |
| Docker<br>Swarm | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |
| Служба<br>Fabric | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; |
| Red Hat Open<br>Сдвиг | | &#8226; | | &#8226; | &#8226;| &#8226; | &#8226; | &#8226; | | &#8226; |
| Windows Server<br>(изолированный) | | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |
| Linux Server<br>(изолированный) | | &#8226; | | &#8226; | &#8226; | &#8226; | &#8226; | &#8226; | | &#8226; |

### <a name="docker-versions-supported-on-linux"></a>Версии Docker, поддерживаемые в Linux

- Docker 1.11–1.13
- Docker CE и EE v17.06

### <a name="x64-linux-distributions-supported-as-container-hosts"></a>Дистрибутивы Linux x64, поддерживаемые в качестве узлов контейнера

- Ubuntu 14.04 LTS и 16.04 LTS
- CoreOS (стабильный выпуск);
- Amazon Linux 2016.09.0;
- openSUSE 13.2
- openSUSE LEAP 42.2
- CentOS 7.2 и 7.3
- SLES 12
- RHEL 7.2 и 7.3
- Платформа контейнеров Red Hat OpenShift (OCP) 3.4 и 3.5
- ACS Mesosphere DC/OS 1.7.3–1.8.8
- ACS Kubernetes 1.4.5–1.6
    - События Kubernetes, список Kubernetes и процессы контейнера поддерживаются только в 1.4.1-45 и более поздних версиях агента Log Analytics для Linux.
- ACS Docker Swarm

[!INCLUDE [log-analytics-agent-note.md](../../../includes/log-analytics-agent-note.md)] 

### <a name="supported-windows-operating-system"></a>Поддерживаемая операционная система Windows

- Windows Server 2016
- Юбилейный выпуск Windows 10 (профессиональный или корпоративный)

### <a name="docker-versions-supported-on-windows"></a>Версии Docker, поддерживаемые в Windows

- Docker 1.12 и 1.13
- Docker 17.03.0 и более поздних версий

## <a name="installing-and-configuring-the-solution"></a>Установка и настройка решения

Для установки и настройки решений используйте указанные ниже данные.

1. Добавьте решение для мониторинга контейнеров в рабочую область Log Analytics из [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.ContainersOMS?tab=Overview) или с помощью процесса, описанного в разделе [Добавление решений мониторинга из коллекция решений](../insights/solutions.md).

2. Установите и используйте Docker с агентом Log Analytics. В зависимости от операционной системы и оркестратора Docker можно использовать следующие методы настройки агента.
   - Для автономных узлов:
     - В поддерживаемых операционных системах Linux установите и запустите Docker, а затем установите и настройте [агент Log Analytics для Linux](../vm/quick-collect-linux-computer.md).  
     - В CoreOS невозможно запустить агент Log Analytics для Linux. Вместо этого можно запустить контейнерную версию агента Log Analytics для Linux. Если вы работаете с контейнерами в облаке "Azure для государственных организаций", то см. раздел "Для всех узлов контейнера Linux, включая CoreOS" или "Для всех узлов контейнера Linux Azure для государственных организаций, включая CoreOS".
     - В Windows Server 2016 и Windows 10 Установите подсистему DOCKER и клиент, а затем подключите агент для сбора информации и отправки его в Azure Monitor. Если вы используете среду Windows, см. сведения в разделе [Установка и настройка узлов контейнера Windows](#install-and-configure-windows-container-hosts).
   - Для многоузловой оркестрации Docker:
     - Если вы используете среду Red Hat OpenShift, ознакомьтесь с разделом по настройке агента Log Analytics для Red Hat OpenShift.
     - Если вы используете кластер Kubernetes с помощью Службы контейнеров Azure (AKS), см. следующие разделы:
       - Ознакомьтесь с разделом [Настройка агента Log Analytics в Linux для Kubernetes](#configure-a-log-analytics-linux-agent-for-kubernetes).
       - См. дополнительные сведения о [настройке агента Log Analytics в Windows для Kubernetes](#configure-a-log-analytics-windows-agent-for-kubernetes).
       - Ознакомьтесь с разделом "Использование Helm для развертывания агента Log Analytics в Kubernetes для Linux".
     - Если у вас есть кластер DC/OS службы контейнеров Azure, дополнительные сведения см. в статье [мониторинг кластера DC/OS службы контейнеров Azure с Azure Monitor](/previous-versions/azure/container-service/dcos-swarm/container-service-monitoring-oms).
     - При наличии среды режима Docker Swarm ознакомьтесь с разделом "Настройка агента Log Analytics для Docker Swarm".
     - При наличии кластера Service Fabric см. Дополнительные сведения в [контейнерах Monitor с Azure Monitor](../../service-fabric/service-fabric-diagnostics-oms-containers.md).

Дополнительные сведения о том, как установить и настроить модули Docker на компьютерах под управлением Windows, см. в [этой статье](/virtualization/windowscontainers/manage-docker/configure-docker-daemon).

> [!IMPORTANT]
> Docker необходимо запустить **перед** установкой [агента Log Analytics для Linux](../vm/quick-collect-linux-computer.md) на узлах контейнера. Если вы уже установили агент перед установкой Docker, то необходимо переустановить агент Log Analytics для Linux. Дополнительные сведения о Docker см. на [веб-сайте Docker](https://www.docker.com).

### <a name="install-and-configure-linux-container-hosts"></a>Установка и настройка узлов контейнера Linux

После установки Docker используйте приведенные ниже параметры узла контейнера, чтобы настроить агент для использования с Docker. Сначала необходимо получить идентификатор и ключ рабочей области Log Analytics, которые можно найти на портале Azure. В рабочей области щелкните **быстрое начало**  >  **Компьютеры** , чтобы просмотреть **идентификатор рабочей области** и **первичный ключ**.  Скопируйте их и вставьте в любой удобный для вас редактор.

**Для всех узлов контейнера Linux, за исключением CoreOS:**

- См. дополнительные сведения об [установке агента Log Analytics для Linux](../agents/log-analytics-agent.md).

**Для всех узлов контейнера Linux, включая CoreOS:**

Запустите контейнер, который вы хотите отслеживать. Используйте следующий пример, внеся в него необходимые изменения:

```
sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -v /var/lib/docker/containers:/var/lib/docker/containers -e WSID="your workspace id" -e KEY="your key" -h=`hostname` -p 127.0.0.1:25225:25225 --name="omsagent" --restart=always microsoft/oms
```

**Для всех узлов контейнеров Linux в Azure для государственных организаций, включая CoreOS:**

Запустите контейнер, который вы хотите отслеживать. Используйте следующий пример, внеся в него необходимые изменения:

```
sudo docker run --privileged -d -v /var/run/docker.sock:/var/run/docker.sock -v /var/log:/var/log -v /var/lib/docker/containers:/var/lib/docker/containers -e WSID="your workspace id" -e KEY="your key" -e DOMAIN="opinsights.azure.us" -p 127.0.0.1:25225:25225 -p 127.0.0.1:25224:25224/udp --name="omsagent" -h=`hostname` --restart=always microsoft/oms
```

**Переход от использования установленного агента Linux к использованию агента в контейнере**

Если ранее вы использовали установленный напрямую агент и теперь вместо него хотите использовать агент, работающий в контейнере, сначала необходимо удалить агент Log Analytics для Linux. Сведения об удалении агента Log Analytics для Linux см. в [этой статье](../vm/quick-collect-linux-computer.md).  

#### <a name="configure-a-log-analytics-agent-for-docker-swarm"></a>Настройка агента Log Analytics для Docker Swarm

Агент Log Analytics можно запускать в качестве глобальной службы в Docker Swarm. Используйте следующие сведения для создания службы агента Log Analytics. Необходимо будет предоставить идентификатор и первичный ключ рабочей области Log Analytics.

- Выполните следующую команду на главном узле.

    ```
    sudo docker service create  --name omsagent --mode global  --mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock --mount type=bind,source=/var/lib/docker/containers,destination=/var/lib/docker/containers -e WSID="<WORKSPACE ID>" -e KEY="<PRIMARY KEY>" -p 25225:25225 -p 25224:25224/udp  --restart-condition=on-failure microsoft/oms
    ```

##### <a name="secure-secrets-for-docker-swarm"></a>Защита секретов для Docker Swarm

Если вы используете Docker Swarm, после создания секрета для идентификатора рабочей области и первичного ключа создайте секретные данные, используя приведенные ниже сведения.

1. Выполните следующую команду на главном узле.

    ```
    echo "WSID" | docker secret create WSID -
    echo "KEY" | docker secret create KEY -
    ```

2. Убедитесь, что секретные данные созданы надлежащим образом.

    ```
    keiko@swarmm-master-13957614-0:/run# sudo docker secret ls
    ```

    ```
    ID                          NAME                CREATED             UPDATED
    j2fj153zxy91j8zbcitnjxjiv   WSID                43 minutes ago      43 minutes ago
    l9rh3n987g9c45zffuxdxetd9   KEY                 38 minutes ago      38 minutes ago
    ```

3. Запустите следующую команду, чтобы подключить секреты к контейнерному агенту Log Analytics.

    ```
    sudo docker service create  --name omsagent --mode global  --mount type=bind,source=/var/run/docker.sock,destination=/var/run/docker.sock --mount type=bind,source=/var/lib/docker/containers,destination=/var/lib/docker/containers --secret source=WSID,target=WSID --secret source=KEY,target=KEY  -p 25225:25225 -p 25224:25224/udp --restart-condition=on-failure microsoft/oms
    ```

#### <a name="configure-a-log-analytics-agent-for-red-hat-openshift"></a>Настройка агента Log Analytics для Red Hat OpenShift

Существует три способа добавления агента Log Analytics в Red Hat OpenShift, чтобы начать сбор данных мониторинга контейнера:

* [Установка агента log Analytics для Linux](../vm/quick-collect-linux-computer.md) непосредственно на каждом узле OpenShift  
* [включить расширение виртуальной машины Log Analytics](../vm/quick-collect-azurevm.md) на каждом узле OpenShift, размещенном в Azure;  
* Установка агента Log Analytics как набора управляющих программ OpenShift  

В этом разделе описаны действия, которые необходимо выполнить для установки агента Log Analytics как набора daemon-set для OpenShift.  

1. Войдите на главный узел OpenShift и скопируйте YAML-файл [ocp-omsagent.yaml](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-omsagent.yaml) с портала GitHub на свой главный узел. При этом замените значения идентификатора рабочей области Log Analytics и первичного ключа своими значениями.
2. Выполните следующие команды, чтобы создать проект для Azure Monitor и задать учетную запись пользователя.

    ```
    oc adm new-project omslogging --node-selector='zone=default'
    oc project omslogging  
    oc create serviceaccount omsagent  
    oc adm policy add-cluster-role-to-user cluster-reader   system:serviceaccount:omslogging:omsagent  
    oc adm policy add-scc-to-user privileged system:serviceaccount:omslogging:omsagent  
    ```

3. Чтобы развернуть набор daemon-set, выполните следующую команду:

    `oc create -f ocp-omsagent.yaml`

4. Чтобы проверить правильность его настроек и работоспособность, введите следующую команду:

    `oc describe daemonset omsagent`  

    Выходные данные должны выглядеть примерно так:

    ```
    [ocpadmin@khm-0 ~]$ oc describe ds oms  
    Name:           oms  
    Image(s):       microsoft/oms  
    Selector:       name=omsagent  
    Node-Selector:  zone=default  
    Labels:         agentVersion=1.4.0-12  
                    dockerProviderVersion=10.0.0-25  
                    name=omsagent  
    Desired Number of Nodes Scheduled: 3  
    Current Number of Nodes Scheduled: 3  
    Number of Nodes Misscheduled: 0  
    Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed  
    No events.  
    ```

Чтобы применить секреты для защиты идентификатора рабочей области Log Analytics и первичного ключа, когда используется YAML-файл набора daemon-set агента Log Analytics, выполните следующие действия.

1. Войдите на главный узел OpenShift и скопируйте YAML-файл [ocp-ds-omsagent.yaml](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-ds-omsagent.yaml) и сценарий создания секретов [ocp secretgen.sh](https://github.com/Microsoft/OMS-docker/blob/master/OpenShift/ocp-secretgen.sh) с портала GitHub.  Этот сценарий создаст YAML-файл секретов для идентификатора рабочей области Log Analytics и первичного ключа, чтобы защитить ваши секретные сведения.  
2. Выполните следующие команды, чтобы создать проект для Azure Monitor и задать учетную запись пользователя. Сценарий создания секретов запросит ввести идентификатор рабочей области Log Analytics `<WSID>` и первичный ключ `<KEY>`, после чего создаст файл ocp-secret.yaml.  

    ```
    oc adm new-project omslogging --node-selector='zone=default'  
    oc project omslogging  
    oc create serviceaccount omsagent  
    oc adm policy add-cluster-role-to-user cluster-reader   system:serviceaccount:omslogging:omsagent  
    oc adm policy add-scc-to-user privileged system:serviceaccount:omslogging:omsagent  
    ```

3. Разверните файл секретов, выполнив следующую команду:

    `oc create -f ocp-secret.yaml`

4. Проверьте развертывание, выполнив следующую команду:

    `oc describe secret omsagent-secret`  

    Выходные данные должны выглядеть примерно так:  

    ```
    [ocpadmin@khocp-master-0 ~]$ oc describe secret omsagent-secret  
    Name:           omsagent-secret  
    Namespace:      omslogging  
    Labels:         <none>  
    Annotations:    <none>  

    Type:   Opaque  

    Data  
    ====  
    KEY:    89 bytes  
    WSID:   37 bytes  
    ```

5. Разверните YAML-файл набора daemon-set агента Log Analytics, выполнив следующую команду:

    `oc create -f ocp-ds-omsagent.yaml`  

6. Проверьте развертывание, выполнив следующую команду:

    `oc describe ds oms`

    Выходные данные должны выглядеть примерно так:

    ```
    [ocpadmin@khocp-master-0 ~]$ oc describe ds oms  
    Name:           oms  
    Image(s):       microsoft/oms  
    Selector:       name=omsagent  
    Node-Selector:  zone=default  
    Labels:         agentVersion=1.4.0-12  
                    dockerProviderVersion=10.0.0-25  
                    name=omsagent  
    Desired Number of Nodes Scheduled: 3  
    Current Number of Nodes Scheduled: 3  
    Number of Nodes Misscheduled: 0  
    Pods Status:    3 Running / 0 Waiting / 0 Succeeded / 0 Failed  
    No events.  
    ```

#### <a name="configure-a-log-analytics-linux-agent-for-kubernetes"></a>Настройка агента Log Analytics в Linux для Kubernetes

Чтобы установить агент Log Analytics в Linux для Kubernetes, используйте сценарий, позволяющий создать YAML-файл секретов для идентификатора рабочей области и первичного ключа. На странице [Log Analytics Docker Kubernetes в GitHub](https://github.com/Microsoft/OMS-docker/tree/master/Kubernetes) есть файлы, которые можно использовать с секретными данными или без них.

- Стандартный файл DaemonSet агента Log Analytics для Linux не поддерживает секретные сведения (omsagent.yaml).
- YAML-файл DaemonSet агента Log Analytics для Linux использует секретные сведения (omsagent-ds-secrets.yaml) со сценариями формирования секретов, чтобы создать YAML-файл секретов (omsagentsecret.yaml).

Вы можете создать DaemonSet агента OMS с секретными данными или без них.

**Стандартный файл YAML DaemonSet агента OMS без секретов**

- Для стандартного файла YAML DaemonSet агента Log Analytics замените `<WSID>` и `<KEY>` на ваши WSID и ключ. Скопируйте файл на главный узел и выполните следующую команду:

    ```
    sudo kubectl create -f omsagent.yaml
    ```

**Стандартный файл YAML DaemonSet агента OMS с секретами**

1. Чтобы использовать DaemonSet агента Log Analytics с секретными сведениями, сначала создайте секреты.
    1. Скопируйте сценарий и файл шаблона секретов в один и тот же каталог.
        - Сценарий создания секретов — secret-gen.sh
        - Шаблон секретов — secret-template.yaml
    2. Выполните сценарий, как показано в следующем примере. Сценарий запрашивает идентификатор рабочей области Log Analytics и первичный ключ, а после их ввода создает YAML-файл секретов, который можно запустить.   

        ```
        #> sudo bash ./secret-gen.sh
        ```

    3. Создайте pod секретов, выполнив следующую команду:
        ```
        sudo kubectl create -f omsagentsecret.yaml
        ```

    4. Чтобы проверить, выполните следующую команду:

        ```
        keiko@ubuntu16-13db:~# sudo kubectl get secrets
        ```

        Выходные данные должны выглядеть примерно так:

        ```
        NAME                  TYPE                                  DATA      AGE
        default-token-gvl91   kubernetes.io/service-account-token   3         50d
        omsagent-secret       Opaque                                2         1d
        ```

        ```
        keiko@ubuntu16-13db:~# sudo kubectl describe secrets omsagent-secret
        ```

        Выходные данные должны выглядеть примерно так:

        ```
        Name:           omsagent-secret
        Namespace:      default
        Labels:         <none>
        Annotations:    <none>

        Type:   Opaque

        Data
        ====
        WSID:   36 bytes
        KEY:    88 bytes
        ```

    5. Создание свой набор daemon-set omsagent, выполнив команду ```sudo kubectl create -f omsagent-ds-secrets.yaml```

2. Проверьте, работает ли DaemonSet агента Log Analytics, что будет выглядеть примерно так:

    ```
    keiko@ubuntu16-13db:~# sudo kubectl get ds omsagent
    ```

    ```
    NAME       DESIRED   CURRENT   NODE-SELECTOR   AGE
    omsagent   3         3         <none>          1h
    ```

Для агента Log Analytics в Linux для Kubernetes используйте сценарий, позволяющий создать YAML-файл секретов для идентификатора рабочей области и первичного ключа. Используйте сведения из следующего примера с [файлом YAML omsagent](https://github.com/Microsoft/OMS-docker/blob/master/Kubernetes/omsagent.yaml), чтобы защитить свои секретные сведения.

```
keiko@ubuntu16-13db:~# sudo kubectl describe secrets omsagent-secret
Name:           omsagent-secret
Namespace:      default
Labels:         <none>
Annotations:    <none>

Type:   Opaque

Data
====
WSID:   36 bytes
KEY:    88 bytes
```

#### <a name="configure-a-log-analytics-windows-agent-for-kubernetes"></a>Настройка агента Log Analytics в Windows для Kubernetes

Чтобы установить агент Log Analytics в Windows для Kubernetes, используйте сценарий, позволяющий создать YAML-файл секретов для идентификатора рабочей области и первичного ключа. На странице [Log Analytics Docker Kubernetes в GitHub](https://github.com/Microsoft/OMS-docker/tree/master/Kubernetes/windows) есть файлы, которые можно использовать с секретными сведениями.  Необходимо установить агент Log Analytics отдельно для главного узла и узла агента.  

1. Чтобы использовать DaemonSet агента Log Analytics с секретными сведениями на главном узле, сначала выполните вход и создайте секреты.
    1. Скопируйте сценарий и файл шаблона секретов в один и тот же каталог.
        - Сценарий создания секретов — secret-gen.sh
        - Шаблон секретов — secret-template.yaml

    2. Выполните сценарий, как показано в следующем примере. Сценарий запрашивает идентификатор рабочей области Log Analytics и первичный ключ, а после их ввода создает YAML-файл секретов, который можно запустить.

        ```
        #> sudo bash ./secret-gen.sh
        ```
    3. Создание свой набор daemon-set omsagent, выполнив команду ```kubectl create -f omsagentsecret.yaml```
    4. Чтобы проверить, выполните следующую команду:

        ```
        root@ubuntu16-13db:~# kubectl get secrets
        ```

        Выходные данные должны выглядеть примерно так:

        ```
        NAME                  TYPE                                  DATA      AGE
        default-token-gvl91   kubernetes.io/service-account-token   3         50d
        omsagent-secret       Opaque                                2         1d
        root@ubuntu16-13db:~# kubectl describe secrets omsagent-secret
        Name:           omsagent-secret
        Namespace:      default
        Labels:         <none>
        Annotations:    <none>

        Type:   Opaque

        Data
        ====
        WSID:   36 bytes
        KEY:    88 bytes
        ```

    5. Создание свой набор daemon-set omsagent, выполнив команду ```kubectl create -f ws-omsagent-de-secrets.yaml```

2. Проверьте, работает ли DaemonSet агента Log Analytics, что будет выглядеть примерно так:

    ```
    root@ubuntu16-13db:~# kubectl get deployment omsagent
    NAME       DESIRED   CURRENT   NODE-SELECTOR   AGE
    omsagent   1         1         <none>          1h
    ```

3. Чтобы установить агент на рабочем узле под управлением Windows, следуйте указаниям в разделе по [установке и настройке узлов контейнера Windows](#install-and-configure-windows-container-hosts).

#### <a name="use-helm-to-deploy-log-analytics-agent-on-linux-kubernetes"></a>Использование Helm для развертывания агента Log Analytics в Kubernetes для Linux

Чтобы использовать Helm для развертывания агента Log Analytics в Kubernetes в среде Linux, сделайте следующее.

1. Создание свой набор daemon-set omsagent, выполнив команду ```helm install --name omsagent --set omsagent.secret.wsid=<WSID>,omsagent.secret.key=<KEY> stable/msoms```
2. Результаты будут выглядеть следующим образом.

    ```
    NAME:   omsagent
    LAST DEPLOYED: Tue Sep 19 20:37:46 2017
    NAMESPACE: default
    STATUS: DEPLOYED

    RESOURCES:
    ==> v1/Secret
    NAME            TYPE    DATA  AGE
    omsagent-msoms  Opaque  3     3s

    ==> v1beta1/DaemonSet
    NAME            DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
    omsagent-msoms  3        3        3      3           3          <none>         3s
    ```

3. Вы можете проверить состояние omsagent, запустив ```helm status "omsagent"```. Выходные данные будут выглядеть, как показано ниже:

    ```
    keiko@k8s-master-3814F33-0:~$ helm status omsagent
    LAST DEPLOYED: Tue Sep 19 20:37:46 2017
    NAMESPACE: default
    STATUS: DEPLOYED
 
    RESOURCES:
    ==> v1/Secret
    NAME            TYPE    DATA  AGE
    omsagent-msoms  Opaque  3     17m
 
    ==> v1beta1/DaemonSet
    NAME            DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE-SELECTOR  AGE
    omsagent-msoms  3        3        3      3           3          <none>         17m
    ```
   
    Дополнительные сведения см. на странице [KubeApps](https://aka.ms/omscontainerhelm).

### <a name="install-and-configure-windows-container-hosts"></a>Установка и настройка узлов контейнера Windows

Сведения в этом разделе помогут вам установить и настроить узлы контейнера Windows.

#### <a name="preparation-before-installing-windows-agents"></a>Подготовка к установке агентов Windows

Настройте службу Docker, прежде чем устанавливать агенты на компьютерах под управлением Windows. Конфигурация позволяет агенту Windows или расширению виртуальной машины Azure Monitor использовать сокет TCP DOCKER, чтобы агенты могли удаленно получать доступ к управляющей программе DOCKER и записывать данные для мониторинга.

##### <a name="to-configure-the-docker-service"></a>Настройка службы DOCKER  

Чтобы включить TCP-канал и именованный канал для Windows Server, выполните следующие команды PowerShell:

```
Stop-Service docker
dockerd --unregister-service
dockerd --register-service -H npipe:// -H 0.0.0.0:2375  
Start-Service docker
```

Дополнительные сведения о настройке управляющей программы Docker, используемой в контейнерах Windows, см. в статье [Подсистема Docker в Windows](/virtualization/windowscontainers/manage-docker/configure-docker-daemon).

#### <a name="install-windows-agents"></a>Установка агентов Windows

Чтобы включить мониторинг контейнеров Windows и Hyper-V, установите Microsoft Monitoring Agent (MMA) на компьютерах Windows, которые являются узлами контейнера. Для компьютеров под управлением Windows в локальной среде см. раздел [Подключение компьютеров Windows к Azure Monitor](../agents/agent-windows.md). Для виртуальных машин, работающих в Azure, подключите их к Azure Monitor с помощью [расширения виртуальной машины](../vm/quick-collect-azurevm.md).

Вы можете отслеживать контейнеры Windows, запущенные в Service Fabric. Однако сейчас для Service Fabric поддерживаются только [виртуальные машины, работающие в Azure](../vm/quick-collect-azurevm.md), и [компьютеры под управлением Windows в локальной среде](../agents/agent-windows.md).

Убедитесь, что решение для мониторинга контейнеров правильно установлено в Windows. Чтобы проверить, был ли пакет управления скачан должным образом, найдите файл *ContainerManagement.xxx*. Файлы должны находиться в папке, расположенной по адресу C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs.

## <a name="solution-components"></a>Компоненты решения

На портале Azure перейдите к *коллекции решений* и добавьте **решение для мониторинга контейнера**. Если вы используете агенты Windows, то при добавлении этого решения на каждом компьютере с агентом устанавливается следующий пакет управления. Для этих пакетов управления не требуются настройка или обслуживание.

- *ContainerManagement.xxx* устанавливается в папке, расположенной по адресу C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs.

## <a name="container-data-collection-details"></a>Сведения о сборе данных из контейнеров

Решение для мониторинга контейнеров собирает различные метрики производительности и данные журналов из узлов контейнеров и контейнеров, в которых включен агент.

Данные собираются каждые три минуты следующими типами агентов.

- [Агент Log Analytics для Linux](../vm/quick-collect-linux-computer.md)
- [Агент Windows](../agents/agent-windows.md)
- [Расширение виртуальной машины Log Analytics](../vm/quick-collect-azurevm.md)

### <a name="container-records"></a>Записи контейнеров

В таблице ниже приведены примеры записей, собранных решением для мониторинга контейнеров, и типов данных, которые отображаются при поиске по журналам и в результатах.

| Тип данных | Тип данных, используемый для поиска в журналах | Поля |
| --- | --- | --- |
| Производительность узлов и контейнеров | `Perf` | Computer, ObjectName, CounterName &#40;%Processor Time, Disk Reads MB, Disk Writes MB, Memory Usage MB, Network Receive Bytes, Network Send Bytes, Processor Usage sec, Network&#41;, CounterValue,TimeGenerated, CounterPath, SourceSystem |
| Список контейнеров | `ContainerInventory` | TimeGenerated, Computer, container name, ContainerHostname, Image, ImageTag, ContainerState, ExitCode, EnvironmentVar, Command, CreatedTime, StartedTime, FinishedTime, SourceSystem, ContainerID, ImageID |
| Список образов контейнеров | `ContainerImageInventory` | TimeGenerated, Computer, Image, ImageTag, ImageSize, VirtualSize, Running, Paused, Stopped, Failed, SourceSystem, ImageID, TotalContainer |
| Журнал контейнеров | `ContainerLog` | TimeGenerated, Computer, image ID, container name, LogEntrySource, LogEntry, SourceSystem, ContainerID |
| Журнал службы контейнеров | `ContainerServiceLog`  | TimeGenerated, Computer, TimeOfCommand, Image, Command, SourceSystem, ContainerID |
| Список узлов контейнеров | `ContainerNodeInventory_CL`| TimeGenerated, Computer, ClassName_s, DockerVersion_s, OperatingSystem_s, Volume_s, Network_s, NodeRole_s, OrchestratorType_s, InstanceID_g, SourceSystem|
| Список Kubernetes | `KubePodInventory_CL` | TimeGenerated, Computer, PodLabel_deployment_s, PodLabel_deploymentconfig_s, PodLabel_docker_registry_s, Name_s, Namespace_s, PodStatus_s, PodIp_s, PodUid_g, PodCreationTimeStamp_t, SourceSystem |
| Процесс контейнера | `ContainerProcess_CL` | TimeGenerated, Computer, Pod_s, Namespace_s, ClassName_s, InstanceID_s, Uid_s, PID_s, PPID_s, C_s, STIME_s, Tty_s, TIME_s, Cmd_s, Id_s, Name_s, SourceSystem |
| События Kubernetes | `KubeEvents_CL` | TimeGenerated, Computer, Name_s, ObjectKind_s, Namespace_s, Reason_s, Type_s, SourceComponent_s, SourceSystem, Message |

Метки, добавленные в типы данных *PodLabel* — это ваши метки. Например, приведенные в таблице метки PodLabel. Таким образом, `PodLabel_deployment_s`, `PodLabel_deploymentconfig_s`, `PodLabel_docker_registry_s` будут отличаться в наборе данных вашей среды и должны выглядеть примерно так: `PodLabel_yourlabel_s`.

## <a name="monitor-containers"></a>Мониторинг контейнеров
После включения решения в портал Azure на плитке **контейнеры** отображаются сводные сведения о узлах контейнеров и контейнерах, запущенных на узлах.

![Плитка "Контейнеры"](./media/containers/containers-title.png)

Плитка показывает число контейнеров в среде, число запущенных и остановленных контейнеров, а также число контейнеров, работа которых завершилась сбоем.

### <a name="using-the-containers-dashboard"></a>Использование панели мониторинга "Контейнеры"

Щелкните плитку **Контейнеры**. Отсюда можно открыть представления, упорядоченные по:

- **Событиям контейнера.** Здесь отображается состояние контейнера и компьютеры, содержащие контейнеры со сбоями.
- **Журналам контейнера.** Здесь отображается диаграмма файлов журналов контейнеров, созданных за определенный период, и список компьютеров с наибольшим количеством файлов журналов.
- **Событиям Kubernetes.** Здесь отображается диаграмма событий Kubernetes, созданных за определенный период, и список причин, по которым модули pod создали события. *Этот набор данных используется только в средах Linux.*
- **Списку пространств имен Kubernetes.** Здесь отображается количество пространств имен и модулей pod, а также их иерархия. *Этот набор данных используется только в средах Linux.*
- **Списку узлов контейнеров.** Здесь отображается количество типов оркестрации, которые используются на узлах контейнеров. Узлы компьютеров также перечислены по количеству контейнеров. *Этот набор данных используется только в средах Linux.*
- **Списку образов контейнеров.** Здесь отображается общее количество используемых образов контейнеров и количество типов образов. Количество образов также указано по тегам образов.
- **Состоянию контейнеров.** Здесь отображается общее количество компьютеров узлов контейнера с запущенными контейнерами. Компьютеры также перечислены по количеству запущенных узлов.
- **Процессу контейнера.** Здесь отображается график процессов контейнера, выполняемых за период времени. Контейнеры также перечислены по выполняемым в них командам или процессам. *Этот набор данных используется только в средах Linux.*
- **Производительности ЦП контейнера.** Здесь отображается график среднего использования ЦП по времени для узлов компьютеров. В нем также перечисляются узлы компьютеров по среднему использованию ЦП.
- **Производительности памяти контейнера.** Здесь отображается график использования памяти по времени, а также использование памяти компьютера по имени экземпляра.
- **Производительности компьютера.** Здесь отображаются графики с процентами производительности ЦП, использования памяти и свободным дисковым пространством в мегабайтах по времени. Наведите указатель на любую линию в диаграмме, чтобы просмотреть дополнительные сведения.

Каждая область на панели мониторинга — это визуальное представление поиска по собранным данным.

![Снимок экрана, на котором показана панель мониторинга для просмотра собранных данных. ](./media/containers/containers-dash01.png)

![Панель мониторинга "Контейнеры"](./media/containers/containers-dash02.png)

Перейдите в область **Container Status** (Состояния контейнера) и щелкните верхнюю область, как показано ниже.

![Состояние контейнера](./media/containers/containers-status.png)

Откроется Log Analytics, в котором отображаются сведения о состоянии контейнеров.

![Log Analytics для контейнеров](./media/containers/containers-log-search.png)

Здесь можно изменить поисковый запрос таким образом, чтобы найти нужную вам информацию. Дополнительные сведения о запросах журналов см. [в разделе запросы журналов в Azure Monitor](../logs/log-query-overview.md).

## <a name="troubleshoot-by-finding-a-failed-container"></a>Устранение неполадок с помощью поиска контейнера со сбоем

Log Analytics добавляет к контейнеру пометку **Сбой**, если такой контейнер завершил работу с ненулевым кодом выхода. Обзор ошибок и сбоев в среде можно просмотреть в области **Failed Containers** (Контейнеры со сбоями).

### <a name="to-find-failed-containers"></a>Поиск контейнеров со сбоями

1. Щелкните область **Container Status** (Состояние контейнера).  
   ![состояние контейнера](./media/containers/containers-status.png)
2. Log Analytics откроется и отобразит состояние контейнеров, как показано ниже.  
   ![Состояние контейнеров](./media/containers/containers-log-search.png)
3. Разверните строку Failed (ошибка) и нажмите кнопку +, чтобы добавить критерии в запрос. Затем закомментируйте строку сводки в запросе.
   ![Снимок экрана, на котором показана строка, которая должна быть снабжена комментариями.](./media/containers/containers-state-failed-select.png)  
1. Выполните запрос, а затем разверните строку в результатах, чтобы просмотреть идентификатор изображения.  
   ![Снимок экрана, показывающий, как просмотреть идентификатор изображения.](./media/containers/containers-state-failed.png)  
1. Введите следующую команду в запрос журнала. `ContainerImageInventory | where ImageID == <ImageID>` для просмотра сведений об образе, таких как размер образа, количество остановленных образов, а также образы со сбоями.  
   ![Контейнеры со сбоями](./media/containers/containers-failed04.png)

## <a name="query-logs-for-container-data"></a>Журналы запросов для данных контейнера

При устранении определенной ошибки эта функция помогает узнать, где именно в среде возникает эта ошибка. С помощью журналов следующих типов можно создавать запросы для получения нужных вам сведений.

- **ContainerImageInventory** — этот тип используется для поиска сведений, упорядоченных по образам, и для просмотра информации об образах, включая идентификаторы или размеры образов.
- **ContainerInventory** — этот тип используется для получения сведений о расположении контейнеров, их именах и образах, которые в них выполняются.
- **ContainerLog** — этот тип используется, если требуется найти определенные сведения или записи в журнале ошибок.
- **ContainerNodeInventory_CL** — этот тип используется для получения сведений об узле, где находятся контейнеры. С его помощью можно получить сведения о версии Docker, типе оркестрации, хранилище и сведения о сети.
- **ContainerProcess_CL** — этот тип используется для быстрого просмотра процессов, запущенных в контейнере.
- **ContainerServiceLog** — этот тип используется для поиска в журнале аудита данных об управляющей программе Docker, включая команды для запуска, остановки и извлечения.
- **KubeEvents_CL** — этот тип используется для просмотра событий Kubernetes.
- **KubePodInventory_CL** — этот тип используется, если необходимо получить сведения об иерархии кластера.

### <a name="to-query-logs-for-container-data"></a>Запрос журналов для данных контейнера

* Выберите образ, в котором недавно произошел сбой, и найдите журнал ошибок для этого образа. Сначала найдите имя контейнера, в котором выполняется этот образ, выполнив поиск в журнале **ContainerInventory**. Например, выполните поиск по запросу `ContainerInventory | where Image == "ubuntu" and ContainerState == "Failed"`  
    ![Поиск контейнеров Ubuntu](./media/containers/search-ubuntu.png)

  Разверните любую строку в результатах, чтобы просмотреть подробные сведения для этого контейнера.

## <a name="example-log-queries"></a>Примеры запросов к журналам

При создании запросов часто бывает полезно начать с одного-двух примеров, внося затем в них изменения в соответствии с конкретной средой. В качестве отправной точки можно поэкспериментировать с областью **примеры запросов** в правой части страницы решения, чтобы упростить создание более сложных запросов.

![Запросы по контейнерам](./media/containers/containers-queries.png)

## <a name="saving-log-queries"></a>Сохранение запросов журнала

Сохранение запросов — это стандартная функция в Azure Monitor. Она позволяет сохранять полезные запросы для использования в будущем.

Создав запрос, который вы считаете полезным, сохраните его, щелкнув **Избранное** в верхней части страницы поиска по журналам. Позднее вы сможете легко открыть его на странице **Моя панель мониторинга**.

## <a name="next-steps"></a>Дальнейшие действия

[Журналы запросов](../logs/log-query-overview.md) для просмотра подробных записей данных контейнера.

