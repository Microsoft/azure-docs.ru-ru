---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: fd635d4c0563c35979f8d85c33dfbde35f05f9e6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91401071"
---
Прослушиватель группы доступности — это IP-адрес и сетевое имя, с которых ожидается передача данных в группе доступности SQL Server. Чтобы создать прослушиватель группы доступности, сделайте следующее:

1. <a name="getnet"></a>Получите имя сетевого ресурса для кластера.

    а. Подключитесь по протоколу RDP к виртуальной машине Azure, на которой размещена основная реплика. 

    b. Откройте диспетчер отказоустойчивости кластеров.

    c. Выберите узел **Сети** узел и запишите сетевое имя кластера. Это имя нужно использовать в переменной `$ClusterNetworkName` в сценарии PowerShell. На следующем изображении сетевое имя кластера — **Cluster Network 1**:

   ![Сетевое имя кластера](./media/virtual-machines-ag-listener-configure/90-clusternetworkname.png)

1. <a name="addcap"></a>Добавьте точку доступа клиента.  
    Точка доступа клиента — это сетевое имя, которое используют приложения для подключения к базам данных в группе доступности. Создайте точку доступа клиента в диспетчере отказоустойчивости кластеров.

    а. Разверните имя кластера и щелкните пункт **Роли**.

    b. На панели **Роли** щелкните правой кнопкой мыши имя группы доступности и выберите **Добавить ресурс** > **Точка доступа клиента**.

   ![Снимок экрана: пункт меню "Точка доступа клиента".](./media/virtual-machines-ag-listener-configure/92-addclientaccesspoint.png)

    c. В поле **Имя** создайте имя для нового прослушивателя. 
   Имя для нового прослушивателя — это сетевое имя, которое используют приложения для подключения к базам данных в группе доступности SQL Server.

    d. Чтобы завершить создание прослушивателя, нажмите кнопку **Далее** дважды, а затем нажмите кнопку **Готово**. Не подключайте прослушивателя или ресурс на этом этапе.

1. Отключите группу доступности роли кластера. В **диспетчере отказоустойчивых кластеров** в разделе **Роли** щелкните правой кнопкой мыши роль и выберите **Остановить роль**.

1. <a name="congroup"></a>Настройте ресурс IP-адреса для группы доступности.

    а. Щелкните вкладку **Ресурсы** и разверните созданную точку доступа клиента.  
    Точка доступа клиента не подключена к сети.

   ![Точка доступа клиента](./media/virtual-machines-ag-listener-configure/94-newclientaccesspoint.png) 

    b. Щелкните правой кнопкой мыши ресурс IP-адреса и выберите пункт "Свойства". Запишите IP-адрес и используйте его в переменной `$IPResourceName` в скрипте PowerShell.

    c. В разделе **IP-адрес** выберите параметр **Статический IP-адрес**. Задайте тот же IP-адрес, который использовался на портале Azure при настройке адреса подсистемы балансировки нагрузки.

   ![Снимок экрана, на котором показано, где нужно указать IP-адрес.](./media/virtual-machines-ag-listener-configure/96-ipresource.png) 

    <!-----------------------I don't see this option on server 2016
    1. Disable NetBIOS for this address and click **OK**. Repeat this step for each IP resource if your solution spans multiple Azure VNets. 
    ------------------------->

1. <a name = "dependencyGroup"></a>Сделайте так, чтобы ресурс группы доступности SQL Server зависел от точки доступа клиента.

    а. В диспетчере отказоустойчивости кластеров щелкните **Роли** и выберите группу доступности.

    b. На вкладке **Ресурсы** в разделе **Другие ресурсы** щелкните правой кнопкой мыши группу ресурсов и выберите **Свойства**. 

    c. На вкладке "Зависимости" добавьте имя ресурса точки доступа клиента (прослушивателя).

   ![Снимок экрана, на котором показано, где на вкладке "Зависимости" нужно добавить имя.](./media/virtual-machines-ag-listener-configure/97-propertiesdependencies.png) 

    d. Нажмите кнопку **ОК**.

1. <a name="listname"></a>Сделайте так, чтобы ресурс точки доступа клиента зависел от IP-адреса.

    а. В диспетчере отказоустойчивости кластеров щелкните **Роли** и выберите группу доступности. 

    b. На вкладке **Ресурсы** в разделе **Имя сервера** щелкните ресурс точки доступа клиента правой кнопкой мыши и выберите **Свойства**. 

   ![Снимок экрана: пункт меню "Свойства" для имени сервера.](./media/virtual-machines-ag-listener-configure/98-dependencies.png) 

    c. Перейдите на вкладку **Зависимости** . Убедитесь, что IP-адрес — это зависимость. В противном случае укажите IP-адрес в качестве зависимости. Если в списке несколько ресурсов, убедитесь, что IP-адреса имеют зависимости OR, а не AND. Нажмите кнопку **ОК**. 

   ![Ресурс IP-адреса](./media/virtual-machines-ag-listener-configure/98-propertiesdependencies.png) 

    >[!TIP]
    >Вы можете проверить правильность настройки зависимостей. В диспетчере отказоустойчивости кластеров перейдите к разделу "Роли", щелкните группу доступности правой кнопкой мыши, затем щелкните **Дополнительные действия** и **Показать отчет о зависимостях**. Когда зависимости настроены правильно, группа доступности зависит от имени сети, которое, в свою очередь, зависит от IP-адреса. 


1. <a name="setparam"></a>Настройте параметры кластера в PowerShell.

   а. Скопируйте следующий скрипт PowerShell на один из экземпляров SQL Server. Обновите переменные для среды.

   - `$ListenerILBIP` — IP-адрес создан в подсистеме балансировки нагрузки Azure для прослушивателя группы доступности.
    
   - `$ListenerProbePort` — порт, который вы настроили в подсистеме балансировки нагрузки Azure для прослушивателя группы доступности.

   ```powershell
   $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
   $IPResourceName = "<IPResourceName>" # the IP Address resource name
   $ListenerILBIP = "<n.n.n.n>" # the IP Address of the Internal Load Balancer (ILB). This is the static IP address for the load balancer you configured in the Azure portal.
   [int]$ListenerProbePort = <nnnnn>
  
   Import-Module FailoverClusters

   Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ListenerILBIP";"ProbePort"=$ListenerProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   ```

   b. Задайте параметры кластера, выполнив скрипт PowerShell на одном из узлов кластера.  

   > [!NOTE]
   > Если экземпляры SQL Server находятся в разных регионах, необходимо дважды запустить сценарий PowerShell. При первом запуске используйте значения `$ListenerILBIP` и `$ListenerProbePort` из первого региона. При втором запуске используйте значения `$ListenerILBIP` и `$ListenerProbePort` из второго региона. Имя сети кластера и имя ресурса IP-кластера разные для каждого региона.

1. Включите группу доступности роли кластера. В **диспетчере отказоустойчивых кластеров** в разделе **Роли** щелкните правой кнопкой мыши роль и выберите **Запуск роли**.

При необходимости повторите предыдущие действия, чтобы задать параметры для IP-адреса кластера WSFC.

1. Получите имя IP-адреса для кластера WSFC. В **диспетчере отказоустойчивости кластеров** в разделе **Ресурсы ядра кластера** найдите **имя сервера**.

1. Щелкните правой кнопкой мыши **IP-адрес** и выберите пункт **Свойства**.

1. Запишите **IP-адрес**. Может иметь значение `Cluster IP Address`. 

1. <a name="setwsfcparam"></a>Настройте параметры кластера в PowerShell.
  
   а. Скопируйте следующий скрипт PowerShell на один из экземпляров SQL Server. Обновите переменные для среды.

   - `$ClusterCoreIP` — IP-адрес создан в подсистеме балансировки нагрузки Azure для ядра кластерного ресурса WSFC. Он отличается от IP-адреса для прослушивателя группы доступности.

   - `$ClusterProbePort` — порт, который вы настроили в подсистеме балансировки нагрузки Azure для пробы работоспособности WSFC. Он отличается от пробы для прослушивателя группы доступности.

   ```powershell
   $ClusterNetworkName = "<MyClusterNetworkName>" # the cluster network name (Use Get-ClusterNetwork on Windows Server 2012 of higher to find the name)
   $IPResourceName = "<ClusterIPResourceName>" # the IP Address resource name
   $ClusterCoreIP = "<n.n.n.n>" # the IP Address of the Cluster IP resource. This is the static IP address for the load balancer you configured in the Azure portal.
   [int]$ClusterProbePort = <nnnnn> # The probe port from the WSFCEndPointprobe in the Azure portal. This port must be different from the probe port for the availability group listener probe port.
  
   Import-Module FailoverClusters
  
   Get-ClusterResource $IPResourceName | Set-ClusterParameter -Multiple @{"Address"="$ClusterCoreIP";"ProbePort"=$ClusterProbePort;"SubnetMask"="255.255.255.255";"Network"="$ClusterNetworkName";"EnableDhcp"=0}
   ```

   b. Задайте параметры кластера, выполнив скрипт PowerShell на одном из узлов кластера.  

>[!WARNING]
>Порт пробы работоспособности прослушивателя группы доступности должен отличаться от порта пробы работоспособности ядра кластера IP-адреса. В этих примерах порт прослушивателя — 59999, а порт для пробы работоспособности IP-адреса ядра кластера — 58888. Оба порта требуют правило разрешения брандмауэра для входящих подключений.
