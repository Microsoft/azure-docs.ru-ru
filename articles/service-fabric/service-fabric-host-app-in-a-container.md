---
title: Развертывание приложения-контейнера .NET в Azure Service Fabric
description: Узнайте, как поместить имеющееся приложение .NET в контейнер с помощью Visual Studio и выполнять отладку контейнеров в Service Fabric локально. Помещенное в контейнер приложение отправляется в реестр контейнеров Azure и развертывается в кластере Service Fabric. При развертывании в Azure приложение использует базу данных SQL Azure для хранения данных.
ms.topic: tutorial
ms.date: 07/08/2019
ms.openlocfilehash: 7930651a0faa5f37336c15557e2a0f068d613011
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98936716"
---
# <a name="tutorial-deploy-a-net-application-in-a-windows-container-to-azure-service-fabric"></a>Руководство по Развертывание приложения .NET в контейнере Windows в Azure Service Fabric

В этом руководстве рассматривается, как поместить имеющееся приложение ASP.NET в контейнер и упаковать его в качестве приложения Service Fabric.  Запустите контейнеры локально в кластере разработки Service Fabric, а затем разверните приложение в Azure.  Приложение сохраняет данные в [базе данных SQL Azure](../azure-sql/database/sql-database-paas-overview.md).

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Помещение имеющегося приложения в контейнер с использованием Visual Studio.
> * Создание базы данных в службе "База данных SQL Azure"
> * Создание реестра контейнеров Azure.
> * Развертывание приложения Service Fabric в Azure.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

1. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
2. Включите компоненты Windows: **Hyper-V** и **Контейнеры**.
3. Установите [Docker Desktop для Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description), чтобы иметь возможность запускать контейнеры в Windows 10.
4. Установите [среду выполнения Service Fabric версии 6.2 или более поздней](service-fabric-get-started.md) и [пакет SDK Service Fabric версии 3.1](service-fabric-get-started.md) или более поздней.
5. Установите [Visual Studio 2019 версии 16.1](https://www.visualstudio.com/) (или выше), а также рабочие нагрузки **Разработка для Azure** и **ASP.NET и разработка веб-приложений**.
6. Установите [Azure PowerShell][link-azure-powershell-install].

## <a name="download-and-run-fabrikam-fiber-callcenter"></a>Загрузка и запуск Fabrikam Fiber CallCenter

1. Скачайте пример приложения [Fabrikam Fiber CallCenter][link-fabrikam-github].  Щелкните ссылку **скачивания архива**.  Из каталога *sourceCode* в файле *fabrikam.zip* извлеките файл *sourceCode.zip*, а затем извлеките каталог *VS2015* на компьютер.

2. Убедитесь, что приложение Fabrikam Fiber CallCenter создается и выполняется без ошибок.  Запустите Visual Studio с правами **администратора** и откройте файл [FabrikamFiber.CallCenter.sln][link-fabrikam-github].  Нажмите клавишу F5, чтобы отладить и запустить приложение.

   ![Снимок экрана: домашняя страница приложения Fabrikam Fiber CallCenter, работающего на локальном узле. На странице отображается панель мониторинга со списком обращений в службу поддержки.][fabrikam-web-page]

## <a name="create-an-azure-sql-db"></a>Создание базы данных SQL Azure

При выполнении приложения Fabrikam Fiber CallCenter в рабочей среде данные должны сохраняться в базе данных. Пока нет способа гарантировать сохранение данных в контейнере, поэтому хранить рабочие данные в SQL Server в контейнере нельзя.

Мы рекомендуем [Базу данных SQL Azure](../azure-sql/database/powershell-script-content-guide.md). Для установки и запуска управляемой базы данных SQL Server в Azure выполните следующий скрипт.  При необходимости измените переменные скрипта. *clientIP* — это IP-адрес компьютера разработчика. Запишите имя сервера, выводимое скриптом.

```powershell
$subscriptionID="<subscription ID>"

# Sign in to your Azure account and select your subscription.
Login-AzAccount -SubscriptionId $subscriptionID

# The data center and resource name for your resources.
$dbresourcegroupname = "fabrikam-fiber-db-group"
$location = "southcentralus"

# The server name: Use a random value or replace with your own value (do not capitalize).
$servername = "fab-fiber-$(Get-Random)"

# Set an admin login and password for your database.
# The login information for the server.
$adminlogin = "ServerAdmin"
$password = "Password@123"

# The IP address of your development computer that accesses the SQL DB.
$clientIP = "<client IP>"

# The database name.
$databasename = "call-center-db"

# Create a new resource group for your deployment and give it a name and a location.
New-AzResourceGroup -Name $dbresourcegroupname -Location $location

# Create the SQL server.
New-AzSqlServer -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -Location $location `
    -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))

# Create the firewall rule to allow your development computer to access the server.
New-AzSqlServerFirewallRule -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowClient" -StartIpAddress $clientIP -EndIpAddress $clientIP

# Create the database in the server.
New-AzSqlDatabase  -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -DatabaseName $databasename `
    -RequestedServiceObjectiveName "S0"

Write-Host "Server name is $servername"
```

> [!TIP]
> При наличии корпоративного брандмауэра IP-адрес компьютера разработчика не может быть IP-адресом, которому отказано в доступе к Интернету. Чтобы убедиться, что в правиле брандмауэра указан правильный IP-адрес для базы данных, откройте [портал Azure](https://portal.azure.com) и найдите свою базу данных в разделе "Базы данных SQL". Щелкните имя базы данных и в разделе "Обзор" выберите "Настройка брандмауэра для сервера". "IP-адрес клиента" — это IP-адрес компьютера, используемого для разработки. Убедитесь, что он совпадает с IP-адресом в правиле AllowClient.

## <a name="update-the-web-config"></a>Обновление веб-конфигурации

Вернитесь в проект **FabrikamFiber.Web**, обновите строку подключения в файле **web.config**, чтобы она указывала на SQL Server в контейнере.  Обновите часть строки подключения, содержащую данные о *сервере*, включив имя сервера, созданное предыдущим скриптом. Строка должна выглядеть примерно так: fab-fiber-751718376.database.windows.net. В следующем XML необходимо обновить только атрибут `connectionString`. Атрибуты `providerName` и `name` изменять не нужно.

```xml
<add name="FabrikamFiber-Express" connectionString="Server=<server name>,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
<add name="FabrikamFiber-DataWarehouse" connectionString="Server=<server name>,1433;Initial Catalog=call-center-db;Persist Security Info=False;User ID=ServerAdmin;Password=Password@123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" providerName="System.Data.SqlClient" />
  
```

>[!NOTE]
>Вы можете использовать любой SQL Server для локальной отладки, который доступен с вашего узла. Тем не менее **localdb** не поддерживает взаимодействие типа `container -> host`. Если для создания конечной сборки веб-приложения вы хотите использовать другую базу данных SQL, добавьте еще одну строку подключения в файле *web.release.config*.

## <a name="containerize-the-application"></a>Помещение приложения в контейнер

1. Щелкните правой кнопкой мыши проект **FabrikamFiber.Web** > **Добавить** > **Container Orchestrator Support** (Поддержка оркестратора контейнеров).  Выберите **Service Fabric** в качестве оркестратора контейнеров и нажмите кнопку **ОК**.

2. При появлении запроса нажмите кнопку **Да** для перехода из Docker к контейнерам Windows.

   В решении создается новый проект приложения Service Fabric **FabrikamFiber.CallCenterApplication**.  Dockerfile добавляется в имеющийся проект **FabrikamFiber.Web**.  Каталог **PackageRoot** также добавляется в проект **FabrikamFiber.Web**, который содержит манифест служб и параметры для новой службы FabrikamFiber.Web.

   Контейнер готов к сборке и упаковке в приложение Service Fabric. После создания образа контейнера на компьютере можно отправить его в любой реестр контейнеров и развернуть на любом узле, чтобы запустить.

## <a name="run-the-containerized-application-locally"></a>Выполнение контейнеризованного приложения локально

Нажмите клавишу **F5**, чтобы запустить и отладить приложение в контейнере в локальном кластере разработки Service Fabric. Нажмите кнопку **Да**, если появится окно сообщения с запросом на предоставление группе ServiceFabricAllowedUsers разрешений на чтение и выполнение в каталоге проекта Visual Studio.

Если при нажатии клавиши F5 возникает исключение, приведенное ниже, то в брандмауэре базы данных Azure не добавлен правильный IP-адрес.

```text
System.Data.SqlClient.SqlException
HResult=0x80131904
Message=Cannot open server 'fab-fiber-751718376' requested by the login. Client with IP address '123.456.789.012' is not allowed to access the server.  To enable access, use the Windows Azure Management Portal or run sp_set_firewall_rule on the master database to create a firewall rule for this IP address or address range.  It may take up to five minutes for this change to take effect.
Source=.Net SqlClient Data Provider
StackTrace:
<Cannot evaluate the exception stack trace>
```

Чтобы добавить соответствующий IP-адрес в брандмауэр базы данных Azure, выполните следующую команду.

```powershell
# The IP address of your development computer that accesses the SQL DB.
$clientIPNew = "<client IP from the Error Message>"

# Create the firewall rule to allow your development computer to access the server.
New-AzSqlServerFirewallRule -ResourceGroupName $dbresourcegroupname `
    -ServerName $servername `
    -FirewallRuleName "AllowClientNew" -StartIpAddress $clientIPNew -EndIpAddress $clientIPNew
```

## <a name="create-a-container-registry"></a>Создание реестра контейнеров

Теперь, когда приложение выполняется локально, запустите подготовку к развертыванию в Azure.  Образы контейнеров должны храниться в реестре контейнеров.  Создайте [реестр контейнеров Azure](../container-registry/container-registry-intro.md) с помощью следующего скрипта. Имя реестра контейнеров отображается другим подпискам Azure, поэтому оно должно быть уникальным.
Перед развертыванием приложения в Azure отправьте образ контейнера в этот реестр.  Если приложение развертывается в кластере в Azure, образ контейнера извлекается из этого реестра.

```powershell
# Variables
$acrresourcegroupname = "fabrikam-acr-group"
$location = "southcentralus"
$registryname="fabrikamregistry$(Get-Random)"

New-AzResourceGroup -Name $acrresourcegroupname -Location $location

$registry = New-AzContainerRegistry -ResourceGroupName $acrresourcegroupname -Name $registryname -EnableAdminUser -Sku Basic
```

## <a name="create-a-service-fabric-cluster-on-azure"></a>Создание кластера Service Fabric в Azure

Приложения Service Fabric работают в кластере, наборе виртуальных или физических машин, подключенных к сети.  Перед развертыванием приложения в Azure создайте кластер Service Fabric в Azure.

Вы можете:

* Создать тестовый кластер с помощью Visual Studio. Этот вариант позволяет создать безопасный кластер непосредственно из Visual Studio с нужными конфигурациями.
* [Создать безопасный кластер из шаблона](service-fabric-tutorial-create-vnet-and-windows-cluster.md).

В этом руководстве создается кластер из Visual Studio, который идеально подходит для сценариев тестирования. При создании кластера другим способом или использовании имеющегося кластера можно скопировать и вставить конечную точку подключения или выбрать ее из подписки.

Перед этим в Обозревателе решений откройте файл FabrikamFiber.Web -> PackageRoot -> ServiceManifest.xml. Запишите номер порта для веб-интерфейса, указанный в разделе **Конечная точка**.

Создание кластера

1. Щелкните правой кнопкой мыши проект приложения **FabrikamFiber.CallCenterApplication** в обозревателе решений, а затем выберите **Опубликовать**.
2. Войдите в систему, используя свою учетную запись Azure, чтобы получить доступ к своим подпискам.
3. Под раскрывающимся списком для **конечной точки подключения** выберите вариант **Создать новый кластер...**
4. В диалоговом окне **Создание кластера** измените следующие параметры:

    а. Укажите имя кластера в поле **Имя кластера**, а также подписку и расположение, которые нужно использовать. Запишите имя группы ресурсов кластера.

    b. Необязательно: можно изменить количество узлов. По умолчанию у вас есть три узла — минимальное количество, необходимое для тестирования сценариев Service Fabric.

    c. Выберите вкладку **Сертификат**. На этой вкладке введите пароль, используемый для защиты сертификата кластера. С помощью этого сертификата вы защитите кластер. Также можно изменить путь, используемый для сохранения сертификата. Visual Studio также может импортировать сертификат автоматически, так как это необходимый шаг для публикации приложения в кластер.

    >[!NOTE]
    >Запишите путь к папке, в которую импортируется этот сертификат. Следующим шагом после создания кластера является импорт этого сертификата.

    d. Перейдите на вкладку **Сведения о виртуальной машине**. Укажите пароль, который вы хотите использовать для виртуальных машин, входящих в состав кластера. Имя пользователя и пароль можно использовать для удаленного подключения к виртуальным машинам. Также необходимо выбрать размер виртуальной машины и при необходимости изменить ее образ.

    > [!IMPORTANT]
    > Выберите номер SKU, который поддерживает работающие контейнеры. ОС Windows Server на узлах вашего кластера должна быть совместима с ОС Windows Server вашего контейнера. Дополнительные сведения см. в разделе [ОС контейнера Windows Server и совместимость ОС узлов](service-fabric-get-started-containers.md#windows-server-container-os-and-host-os-compatibility). По умолчанию в данном руководстве создается образ Docker на основе Windows Server 2016 LTSC. Контейнеры на основе этого образа будут запускаться в кластерах, созданных с помощью Windows Server Datacenter 2016 с Контейнерами. Тем не менее при создании кластера или при использовании существующего кластера на основе разных версий Windows Server необходимо изменить образ ОС, на котором основан контейнер. Откройте файл **dockerfile** в проекте **FabrikamFiber.Web**, раскомментируйте имеющиеся инструкции `FROM`, основанные на предыдущей версии Windows Server, а затем добавьте инструкцию `FROM` на основе тега нужной версии на [странице DockerHub Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore). Дополнительные сведения о выпусках Windows Server Core, сроках поддержки и управлении версиями см. на странице со [сведениями о выпусках Windows Server Core](/windows-server/get-started/windows-server-release-info). 

    д) На вкладке **Дополнительно** укажите порты приложения, которые нужно открыть в подсистеме балансировки нагрузки при развертывании кластера. Это порт, номер которого вы записали перед созданием кластера. Вы также можете добавить имеющийся ключ Application Insights, который будет использоваться для направления файлов журнала приложений.

    е) После внесения всех необходимых изменений в параметры нажмите кнопку **Создать**.

5. Процесс создания займет несколько минут. Когда кластер будет полностью создан, в окне вывода отобразится соответствующее уведомление.

## <a name="install-the-imported-certificate"></a>Установка импортированного сертификата

Установите сертификат, импортированный в процессе создания кластера, в расположение хранилища **Текущего пользователя** и введите пароль закрытого ключа, указанный ранее.

Установку можно подтвердить, открыв раздел **Управление сертификатами пользователей** на панели управления и убедившись, что сертификат установлен в разделе **Сертификаты — текущий пользователь** -> **Личные** -> **Сертификаты**. Сертификат должен иметь формат *[Имя кластера]* . *[Расположение кластера]* .cloudapp.azure.com, например *fabrikamfibercallcenter.southcentralus.cloudapp.azure.com*. 

## <a name="allow-your-application-running-in-azure-to-access-sql-database"></a>Настройка доступа приложения, выполняющегося в Azure, к Базе данных SQL

Ранее было создано правило брандмауэра SQL для предоставления доступа к выполняющемуся локально приложению.  Теперь необходимо разрешить приложению, выполняющемуся в Azure, получать доступ к базе данных SQL.  Создайте [конечную точку службы для виртуальной сети](../azure-sql/database/vnet-service-endpoint-rule-overview.md) для кластера Service Fabric, а затем создайте правило, чтобы разрешить этой конечной точке получать доступ к базе данных SQL. Не забудьте указать переменную группы ресурсов кластера, которую вы записали при создании кластера.

```powershell
# Create a virtual network service endpoint
$clusterresourcegroup = "<cluster resource group>"
$resource = Get-AzResource -ResourceGroupName $clusterresourcegroup -ResourceType Microsoft.Network/virtualNetworks | Select-Object -first 1
$vnetName = $resource.Name

Write-Host 'Virtual network name: ' $vnetName

# Get the virtual network by name.
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName $clusterresourcegroup `
  -Name              $vnetName

Write-Host "Get the subnet in the virtual network:"

# Get the subnet, assume the first subnet contains the Service Fabric cluster.
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet | Select-Object -first 1

$subnetName = $subnet.Name
$subnetID = $subnet.Id
$addressPrefix = $subnet.AddressPrefix

Write-Host "Subnet name: " $subnetName " Address prefix: " $addressPrefix " ID: " $subnetID

# Assign a Virtual Service endpoint 'Microsoft.Sql' to the subnet.
$vnet = Set-AzVirtualNetworkSubnetConfig `
  -Name            $subnetName `
  -AddressPrefix   $addressPrefix `
  -VirtualNetwork  $vnet `
  -ServiceEndpoint Microsoft.Sql | Set-AzVirtualNetwork

$vnet.Subnets[0].ServiceEndpoints;  # Display the first endpoint.

# Add a SQL DB firewall rule for the virtual network service endpoint
$subnet = Get-AzVirtualNetworkSubnetConfig `
  -Name           $subnetName `
  -VirtualNetwork $vnet;

$VNetRuleName="ServiceFabricClusterVNetRule"
$vnetRuleObject1 = New-AzSqlServerVirtualNetworkRule `
  -ResourceGroupName      $dbresourcegroupname `
  -ServerName             $servername `
  -VirtualNetworkRuleName $VNetRuleName `
  -VirtualNetworkSubnetId $subnetID;
```

## <a name="deploy-the-application-to-azure"></a>Развертывание приложения в Azure

Теперь, когда приложение готово, можно развернуть его в кластер в Azure напрямую из Visual Studio.  В обозревателе решений щелкните правой кнопкой мыши проект приложения **FabrikamFiber.CallCenterApplication**, а затем выберите **Опубликовать**.  В разделе **конечной точки подключения** выберите конечную точку кластера, которая была создана ранее.  В **реестре контейнеров Azure** выберите реестр контейнеров, который был создан ранее.  Щелкните **Опубликовать**, чтобы развернуть приложение в кластере Azure.

![публикации приложения][publish-app]

Отслеживайте ход развертывания в окне вывода. После развертывания приложения откройте браузер и введите адрес кластера и порт приложения. Например, `http://fabrikamfibercallcenter.southcentralus.cloudapp.azure.com:8659/`.

![Снимок экрана: домашняя страница приложения Fabrikam Fiber CallCenter, запущенного на сайте azure.com. На странице отображается панель мониторинга со списком обращений в службу поддержки.][fabrikam-web-page-deployed]

Если не удается загрузить страницу или возникает ошибка при запросе сертификата, попробуйте открыть путь обозревателя, например `https://fabrikamfibercallcenter.southcentralus.cloudapp.azure.com:19080/Explorer`, и выбрать только что установленный сертификат.

## <a name="set-up-continuous-integration-and-deployment-cicd-with-a-service-fabric-cluster"></a>Настройка непрерывной интеграции и развертывания (CI/CD) с помощью кластера Service Fabric

Дополнительные сведения о том, как использовать Azure DevOps для настройки развертывания приложений с помощью CI/CD в кластере Service Fabric см. в статье [Руководство. Развертывание приложения с непрерывной интеграцией и развертыванием в кластере Service Fabric](service-fabric-tutorial-deploy-app-with-cicd-vsts.md). Процесс, описанный в этом руководстве, такой же и для проекта (FabrikamFiber). Просто пропустите загрузку примера Voting и замените FabrikamFiber именем репозитория вместо Voting.

## <a name="clean-up-resources"></a>Очистка ресурсов

После завершения работы не забудьте удалить все созданные ресурсы.  Простейший способ — это удалить группы ресурсов, которые содержат кластер Service Fabric, базу данных SQL Azure и службу "Реестр контейнеров Azure".

```powershell
$dbresourcegroupname = "fabrikam-fiber-db-group"
$acrresourcegroupname = "fabrikam-acr-group"
$clusterresourcegroupname="fabrikamcallcentergroup"

# Remove the Azure SQL DB
Remove-AzResourceGroup -Name $dbresourcegroupname

# Remove the container registry
Remove-AzResourceGroup -Name $acrresourcegroupname

# Remove the Service Fabric cluster
Remove-AzResourceGroup -Name $clusterresourcegroupname
```

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
>
> * Помещение имеющегося приложения в контейнер с использованием Visual Studio.
> * Создание базы данных в службе "База данных SQL Azure"
> * Создание реестра контейнеров Azure.
> * Развертывание приложения Service Fabric в Azure.

В следующей части этого руководства вы научитесь [развертывать приложение в контейнере с помощью CI/CD в кластере Service Fabric](service-fabric-tutorial-deploy-container-app-with-cicd-vsts.md).

[link-fabrikam-github]: https://aka.ms/fabrikamcontainer
[link-azure-powershell-install]: /powershell/azure/install-Az-ps
[link-servicefabric-create-secure-clusters]: service-fabric-cluster-creation-via-arm.md
[link-visualstudio-cd-extension]: https://aka.ms/cd4vs
[link-servicefabric-containers]: service-fabric-get-started-containers.md
[link-azure-portal]: https://portal.azure.com
[link-sf-clustertemplate]: https://aka.ms/securepreviewonelineclustertemplate
[link-azure-pricing-calculator]: https://azure.microsoft.com/pricing/calculator/
[link-azure-subscription]: https://azure.microsoft.com/free/
[link-vsts-account]: https://www.visualstudio.com/team-services/pricing/
[link-azure-sql]: /azure/sql-database/

[fabrikam-web-page]: media/service-fabric-host-app-in-a-container/fabrikam-web-page.png
[fabrikam-web-page-deployed]: media/service-fabric-host-app-in-a-container/fabrikam-web-page-deployed.png
[publish-app]: media/service-fabric-host-app-in-a-container/publish-app.png