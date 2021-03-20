---
title: Использование записи пакетов для упреждающего мониторинга сети с помощью оповещений — функции Azure
titleSuffix: Azure Network Watcher
description: В этой статье описывается, как создавать активируемую с использованием оповещений запись пакетов в службе наблюдения за сетями Azure
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: 75e6e7c4-b3ba-4173-8815-b00d7d824e11
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 8e2af73be0fc887b132f523133159472ce1d1f98
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100570974"
---
# <a name="use-packet-capture-for-proactive-network-monitoring-with-alerts-and-azure-functions"></a>Использование записи пакетов для упреждающего мониторинга сети с помощью оповещений и функций Azure

Функция записи пакетов службы наблюдения за сетями ("Наблюдатель за сетями") отслеживает входящий и исходящий трафик виртуальных машин. Файл записи можно создать с использованием фильтра, чтобы отслеживать только нужный трафик. Эти данные затем сохраняются в хранилище BLOB-объектов или локально на гостевом компьютере.

Эта возможность допускает удаленный запуск из других сценариев службы автоматизации, таких как функции Azure. Возможность записи пакетов позволяет создавать упреждающие записи пакетов, активируемые при обнаружении определенных сетевых аномалий. Они также помогают выполнять сбор сетевой статистики, получать сведения о сетевых вторжениях, выполнять отладку передачи данных между клиентом и сервером и другие операции.

Развернутые в Azure ресурсы работают круглосуточно и без выходных. Вам и вашим сотрудникам вряд ли удастся эффективно отслеживать состояние всех ресурсов без перерывов. К примеру, что вы будете делать, если сбой произойдет в два часа ночи?

С помощью службы "Наблюдатель за сетями", оповещений и функций экосистемы Azure вы можете заранее подготовить данные и средства для устранения проблем в сети.

![На схеме показано расширение наблюдателя за сетями на виртуальной машине, которая перемещается в сегменты T C P, отправленные > ошибке 100, которая передается в функции Azure, которые передаются в службу "наблюдатель за сетями", которая передается в расширение наблюдателя за сетями.][scenario]


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные условия

* Последняя версия [Azure PowerShell](/powershell/azure/install-Az-ps).
* Существующий экземпляр службы "Наблюдатель за сетями". Если у вас нет экземпляра этой службы, [создайте его](network-watcher-create.md).
* Существующая виртуальная машина в том же регионе, что и служба "Наблюдатель за сетями", с [расширением виртуальных машин для Windows](../virtual-machines/extensions/network-watcher-windows.md) или [Linux](../virtual-machines/extensions/network-watcher-linux.md).

## <a name="scenario"></a>Сценарий

В этом примере мы рассмотрим виртуальную машину, которая отправляет больше TCP-сегментов, чем обычно. Предположим, вы хотите получать оповещения о такой ситуации. TCP-сегменты используются только в качестве примера. Вы можете использовать любое условие оповещения.

Когда создается оповещение, вам нужно получить данные на уровне пакетов, чтобы найти причину необычной активности. Это позволит принять меры для восстановления на виртуальной машине обычного режима обмена данными.

В этом сценарии предполагается, что у вас есть экземпляр службы "Наблюдатель за сетями", а также группа ресурсов с допустимой виртуальной машиной.

Ниже приведен обзор соответствующего рабочего процесса.

1. На виртуальной машине активируется оповещение.
1. Это оповещение вызывает функцию Azure через webhook.
1. Функция Azure обрабатывает оповещение и запускает сеанс записи пакетов в наблюдателе за сетями.
1. На виртуальной машине выполняется запись пакетов и собираются данные о трафике.
1. Файл записи пакетов передается в учетную запись хранения для проверки и диагностики.

Чтобы автоматизировать этот процесс, мы создадим и подключим на виртуальной машине оповещение, которое будет срабатывать при возникновении проблемы. Также мы создадим функцию для вызова службы "Наблюдатель за сетями".

Этот сценарий выполняет следующее:

* создает функцию Azure, которая запускает запись пакетов;
* создает правило генерации оповещений на виртуальной машине и настраивает его для вызова функции Azure.

## <a name="create-an-azure-function"></a>Создание функции Azure

Первым делом нам нужно создать функцию Azure для обработки оповещения и создания записи пакетов.

1. На [портале Azure](https://portal.azure.com) последовательно выберите **Создать ресурс** > **Вычисление** > **Приложение-функция**.

    ![Создание приложения-функции][1-1]

2. В колонке **Приложение-функция** введите следующие значения и нажмите кнопку **ОК** для создания приложения:

    |**Параметр** | **Значение** | **Сведения** |
    |---|---|---|
    |**Имя приложения**|PacketCaptureExample|Имя приложения-функции.|
    |**Подписка**|[Ваша подписка] Подписка, в которой нужно создать приложение-функцию.||
    |**Группа ресурсов**|PacketCaptureRG|Группа ресурсов, куда будет помещено приложение-функция.|
    |**План размещения**|План потребления| Тип плана, который использует приложение-функция. Можно выбрать план потребления или план службы приложений Azure. |
    |**Расположение**|Центральная часть США| Регион для создания приложения-функции.|
    |**Учетная запись хранения**|{autogenerated}| Это учетная запись хранения, необходимая функциям Azure для хранилища общего назначения.|

3. В колонке **приложения-функции PacketCaptureExample** последовательно выберите **Функции** > **Пользовательская функция** >**+**.

4. Выберите **HttpTrigger-Powershell** и введите остальные сведения. Наконец, чтобы создать функцию, выберите **Создать**.

    |**Параметр** | **Значение** | **Сведения** |
    |---|---|---|
    |**Сценарий**|Экспериментальный|Тип сценария|
    |**Имя функции**|AlertPacketCapturePowerShell|Имя функции|
    |**Уровень авторизации**|Функция|Уровень авторизации для функции|

![Пример функций][functions1]

> [!NOTE]
> Шаблон PowerShell является экспериментальным и не обеспечивает полную поддержку.

Для этого примера требуются настройки, описанные ниже.

### <a name="add-modules"></a>Добавление модулей

Чтобы использовать командлеты PowerShell для службы "Наблюдатель за сетями", передайте в приложение-функцию последнюю версию модуля PowerShell.

1. На локальном компьютере, на котором установлена последняя версия модулей Azure PowerShell, выполните следующую команду PowerShell.

    ```powershell
    (Get-Module Az.Network).Path
    ```

    Этот пример отображает локальный путь к модулям Azure PowerShell. Мы используем эти папки позже. Ниже перечислены модули, используемые в этом сценарии.

   * Az.Network

   * Az.Accounts

   * Az.Resources

     ![Папки PowerShell][functions5]

1. Выберите **Параметры приложения функции**  >  **Переход к редактор службы приложений**.

    ![Параметры приложения-функции][functions2]

1. Щелкните правой кнопкой мыши папку **AlertPacketCapturePowershell** и создайте папку **azuremodules**. 

4. Создайте вложенную папку для каждого требуемого модуля.

    ![Папки и вложенные папки][functions3]

    * Az.Network

    * Az.Accounts

    * Az.Resources

1. Щелкните подпапку **AZ. Network** правой кнопкой мыши и выберите команду **отправить файлы**. 

6. Перейдите к модулям Azure. В локальной папке **AZ. Network** выберите все файлы в папке. Нажмите кнопку **ОК**. 

7. Повторите эти шаги для команды **AZ. Accounts** и **AZ. Resources**.

    ![Отправка файлов][functions6]

1. По завершении в каждой папке должны находиться файлы модулей PowerShell с локального компьютера.

    ![Файлы PowerShell][functions7]

### <a name="authentication"></a>Аутентификация

Чтобы использовать командлеты PowerShell, нужно пройти аутентификацию. Настройте проверку подлинности для приложения-функции. Для этого задайте переменные среды и передайте зашифрованный файл ключа в приложение-функцию.

> [!NOTE]
> В этом сценарии приводится всего один пример того, как реализовать проверку подлинности с использованием функций Azure. Существуют и другие способы сделать это.

#### <a name="encrypted-credentials"></a>Зашифрованные учетные данные

С помощью указанного ниже скрипта PowerShell создается файл ключа с именем **PassEncryptKey.key**, а также предоставляется зашифрованная версия пароля. Это тот же пароль, который определен для приложения Azure Active Directory и используется для проверки подлинности.

```powershell
#Variables
$keypath = "C:\temp\PassEncryptKey.key"
$AESKey = New-Object Byte[] 32
$Password = "<insert a password here>"

#Keys
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey) 
Set-Content $keypath $AESKey

#Get encrypted password
$secPw = ConvertTo-SecureString -AsPlainText $Password -Force
$AESKey = Get-content $KeyPath
$Encryptedpassword = $secPw | ConvertFrom-SecureString -Key $AESKey
$Encryptedpassword
```

В редакторе службы приложений для приложения-функции создайте папку **keys** в папке **AlertPacketCapturePowerShell**. Затем передайте в нее файл **PassEncryptKey.key**, который вы создали в предыдущем примере PowerShell.

![Ключ для функций][functions8]

### <a name="retrieve-values-for-environment-variables"></a>Извлечение значений переменных среды

В завершение необходимо задать переменные среды, которые требуются для доступа к значениям для проверки подлинности. Ниже приведен список создаваемых переменных среды.

* AzureClientID

* AzureTenant

* AzureCredPassword


#### <a name="azureclientid"></a>AzureClientID

Идентификатор клиента — это идентификатор приложения в Azure Active Directory.

1. Если используемого приложения еще нет, выполните приведенный ниже пример, чтобы создать приложение.

    ```powershell
    $app = New-AzADApplication -DisplayName "ExampleAutomationAccount_MF" -HomePage "https://exampleapp.com" -IdentifierUris "https://exampleapp1.com/ExampleFunctionsAccount" -Password "<same password as defined earlier>"
    New-AzADServicePrincipal -ApplicationId $app.ApplicationId
    Start-Sleep 15
    New-AzRoleAssignment -RoleDefinitionName Contributor -ServicePrincipalName $app.ApplicationId
    ```

   > [!NOTE]
   > Пароль, который вы использовали при создании приложения, должен совпадать с паролем, созданным ранее при сохранении файла ключа.

1. На портале Azure выберите **Подписки**. Выберите подписку, которую нужно использовать, а затем — пункт **Управление доступом (IAM)**.

    ![Функции (IAM)][functions9]

1. Выберите учетную запись, а затем — пункт **Свойства**. Скопируйте идентификатор приложения.

    ![Идентификатор приложений-функций][functions10]

#### <a name="azuretenant"></a>AzureTenant

Получите код клиента, выполнив следующий пример PowerShell:

```powershell
(Get-AzSubscription -SubscriptionName "<subscriptionName>").TenantId
```

#### <a name="azurecredpassword"></a>AzureCredPassword

Значением переменной среды AzureCredPassword является значение, получаемое с помощью примера PowerShell ниже. Это пример, который был показан в предыдущем разделе **Зашифрованные учетные данные**. Требуемым значением являются выходные данные переменной `$Encryptedpassword`.  Это пароль субъекта-службы, который вы зашифровали с помощью скрипта PowerShell.

```powershell
#Variables
$keypath = "C:\temp\PassEncryptKey.key"
$AESKey = New-Object Byte[] 32
$Password = "<insert a password here>"

#Keys
[Security.Cryptography.RNGCryptoServiceProvider]::Create().GetBytes($AESKey) 
Set-Content $keypath $AESKey

#Get encrypted password
$secPw = ConvertTo-SecureString -AsPlainText $Password -Force
$AESKey = Get-content $KeyPath
$Encryptedpassword = $secPw | ConvertFrom-SecureString -Key $AESKey
$Encryptedpassword
```

### <a name="store-the-environment-variables"></a>Хранение переменных среды

1. Перейдите к приложению-функции. Затем выберите **Параметры приложения функции**  >  **настроить параметры приложения**.

    ![Настройка параметров приложения][functions11]

1. Добавьте переменные среды и их значения к параметрам приложения и нажмите кнопку **Сохранить**.

    ![Параметры приложения][functions12]

### <a name="add-powershell-to-the-function"></a>Добавление PowerShell в функцию

Пришло время вызвать службу "Наблюдатель за сетями" из функции Azure. Реализация этой функции будет зависеть от конкретных требований. Но в целом поток кода будет выглядеть примерно так:

1. Обработка входных параметров.
2. Запрос существующих записей пакетов для проверки ограничений и разрешения конфликтов имен.
3. Создание записи пакетов с необходимыми параметрами.
4. Периодический опрос процесса записи пакетов вплоть до его завершения.
5. Уведомление пользователя о завершении сеанса записи пакетов.

В функции можно использовать указанный ниже пример кода PowerShell. Необходимо заменить значения **subscriptionId**, **resourceGroupName** и **storageAccountName**.

```powershell
            #Import Azure PowerShell modules required to make calls to Network Watcher
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Accounts\Az.Accounts.psd1" -Global
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Network\Az.Network.psd1" -Global
            Import-Module "D:\home\site\wwwroot\AlertPacketCapturePowerShell\azuremodules\Az.Resources\Az.Resources.psd1" -Global

            #Process alert request body
            $requestBody = Get-Content $req -Raw | ConvertFrom-Json

            #Storage account ID to save captures in
            $storageaccountid = "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}"

            #Packet capture vars
            $packetcapturename = "PSAzureFunction"
            $packetCaptureLimit = 10
            $packetCaptureDuration = 10

            #Credentials
            $tenant = $env:AzureTenant
            $pw = $env:AzureCredPassword
            $clientid = $env:AzureClientId
            $keypath = "D:\home\site\wwwroot\AlertPacketCapturePowerShell\keys\PassEncryptKey.key"

            #Authentication
            $secpassword = $pw | ConvertTo-SecureString -Key (Get-Content $keypath)
            $credential = New-Object System.Management.Automation.PSCredential ($clientid, $secpassword)
            Connect-AzAccount -ServicePrincipal -Tenant $tenant -Credential $credential #-WarningAction SilentlyContinue | out-null


            #Get the VM that fired the alert
            if($requestBody.context.resourceType -eq "Microsoft.Compute/virtualMachines")
            {
                Write-Output ("Subscription ID: {0}" -f $requestBody.context.subscriptionId)
                Write-Output ("Resource Group:  {0}" -f $requestBody.context.resourceGroupName)
                Write-Output ("Resource Name:  {0}" -f $requestBody.context.resourceName)
                Write-Output ("Resource Type:  {0}" -f $requestBody.context.resourceType)

                #Get the Network Watcher in the VM's region
                $networkWatcher = Get-AzResource | Where {$_.ResourceType -eq "Microsoft.Network/networkWatchers" -and $_.Location -eq $requestBody.context.resourceRegion}

                #Get existing packetCaptures
                $packetCaptures = Get-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher

                #Remove existing packet capture created by the function (if it exists)
                $packetCaptures | %{if($_.Name -eq $packetCaptureName)
                { 
                    Remove-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -PacketCaptureName $packetCaptureName
                }}

                #Initiate packet capture on the VM that fired the alert
                if ((Get-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher).Count -lt $packetCaptureLimit){
                    echo "Initiating Packet Capture"
                    New-AzNetworkWatcherPacketCapture -NetworkWatcher $networkWatcher -TargetVirtualMachineId $requestBody.context.resourceId -PacketCaptureName $packetCaptureName -StorageAccountId $storageaccountid -TimeLimitInSeconds $packetCaptureDuration
                    Out-File -Encoding Ascii -FilePath $res -inputObject "Packet Capture created on ${requestBody.context.resourceID}"
                }
            } 
 ``` 
#### <a name="retrieve-the-function-url"></a>Извлечение URL-адреса функции 
1. Когда функция будет готова, настройте вызов связанного с ней URL-адреса в оповещении. Чтобы получить нужное значение, скопируйте URL-адрес функции из приложения-функции.

    ![Поиск URL-адреса функции][functions13]

2. Скопируйте URL-адрес функции для приложения-функции.

    ![Копирование URL-адреса функции][2]

Если вы хотите передавать пользовательские свойства в полезных данных запроса POST, передаваемого в веб-перехватчике, см. статью [Настройка объектов webhook для оповещений на основе метрик Azure](../azure-monitor/alerts/alerts-webhooks.md).

## <a name="configure-an-alert-on-a-vm"></a>Настройка оповещения на виртуальной машине

Оповещения можно настроить так, чтобы они уведомляли определенных пользователей о превышении порогового значения для определенных метрик. В нашем примере оповещение создается по числу отправленных TCP-сегментов, но можно использовать и другие метрики. Наше оповещение обращается к webhook для вызова функции.

### <a name="create-the-alert-rule"></a>Создание правила для оповещения

Перейдите к существующей виртуальной машине и добавьте правило генерации оповещений. Дополнительные сведения о настройке оповещений см. в статье [Создание оповещений в Azure Monitor для служб Azure с помощью портала Azure](../azure-monitor/alerts/alerts-classic-portal.md). Введите указанные ниже значения в колонке **Правило генерации оповещений** и нажмите кнопку **ОК**.

  |**Параметр** | **Значение** | **Сведения** |
  |---|---|---|
  |**имя**;|TCP_Segments_Sent_Exceeded|Имя правила генерации оповещений.|
  |**Описание**|Число отправленных TCP-сегментов превысило пороговое значение|Описание для правила генерации оповещений.|
  |**Метрика**|Отправлено TCP-сегментов| Метрика, используемая для активации оповещения. |
  |**Condition**|Больше чем| Условие, используемое при вычислении метрики.|
  |**Пороговое значение**.|100| Это значение метрики, при достижении которого запускается оповещение. Оно должно быть допустимым для вашей среды.|
  |**Периода**|За последние пять минут| Определяет период, за который проверяется пороговое значение для метрики.|
  |**Веб-перехватчик**|[URL-адрес веб-перехватчика из приложения-функции]| Это URL-адрес веб-перехватчика из приложения-функции, созданного на предыдущих шагах.|

> [!NOTE]
> Метрика сегментов TCP не включена по умолчанию. Подробные сведения о том, как включить дополнительные метрики, см. в статье [Приступая к работе с Azure Monitor](../azure-monitor/overview.md).

## <a name="review-the-results"></a>Просмотр результатов

После задания условий для триггеров оповещений создается запись пакетов. Перейдите к службе "Наблюдатель за сетями" и выберите **Запись пакетов**. На этой странице вы можете воспользоваться ссылкой на файл записи пакетов, чтобы скачать его.

![Просмотр записи пакетов][functions14]

Если же файл записи хранится локально, вы можете войти на виртуальную машину, чтобы извлечь его.

Инструкции по скачиванию файлов из учетных записей хранения Azure см. в статье [Приступая к работе с хранилищем BLOB-объектов Azure с помощью .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md). Кроме того, можно использовать такое средство, как [обозреватель хранилищ](https://storageexplorer.com/).

Когда вы скачаете нужную запись, ее можно просмотреть в любом средстве, поддерживающем формат **CAP**. Ниже приведены ссылки на два таких средства.

- [Анализатор сообщений Майкрософт](/message-analyzer/microsoft-message-analyzer-operating-guide)
- [WireShark](https://www.wireshark.org/)

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как просмотреть запись пакетов, прочитав статью [Packet capture analysis with Wireshark](network-watcher-deep-packet-inspection.md) (Анализ записи пакетов с помощью Wireshark).


[1]: ./media/network-watcher-alert-triggered-packet-capture/figure1.png
[1-1]: ./media/network-watcher-alert-triggered-packet-capture/figure1-1.png
[2]: ./media/network-watcher-alert-triggered-packet-capture/figure2.png
[3]: ./media/network-watcher-alert-triggered-packet-capture/figure3.png
[functions1]:./media/network-watcher-alert-triggered-packet-capture/functions1.png
[functions2]:./media/network-watcher-alert-triggered-packet-capture/functions2.png
[functions3]:./media/network-watcher-alert-triggered-packet-capture/functions3.png
[functions4]:./media/network-watcher-alert-triggered-packet-capture/functions4.png
[functions5]:./media/network-watcher-alert-triggered-packet-capture/functions5.png
[functions6]:./media/network-watcher-alert-triggered-packet-capture/functions6.png
[functions7]:./media/network-watcher-alert-triggered-packet-capture/functions7.png
[functions8]:./media/network-watcher-alert-triggered-packet-capture/functions8.png
[functions9]:./media/network-watcher-alert-triggered-packet-capture/functions9.png
[functions10]:./media/network-watcher-alert-triggered-packet-capture/functions10.png
[functions11]:./media/network-watcher-alert-triggered-packet-capture/functions11.png
[functions12]:./media/network-watcher-alert-triggered-packet-capture/functions12.png
[functions13]:./media/network-watcher-alert-triggered-packet-capture/functions13.png
[functions14]:./media/network-watcher-alert-triggered-packet-capture/functions14.png
[scenario]:./media/network-watcher-alert-triggered-packet-capture/scenario.png