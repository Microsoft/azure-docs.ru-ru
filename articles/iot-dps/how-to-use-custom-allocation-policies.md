---
title: Пользовательские политики выделения с помощью службы подготовки устройств для центра Интернета вещей Azure
description: Как использовать пользовательские политики распределения с помощью службы подготовки устройств для центра Интернета вещей Azure (DPS)
author: wesmc7777
ms.author: wesmc
ms.date: 01/26/2021
ms.topic: conceptual
ms.service: iot-dps
services: iot-dps
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 14a405dbab0460f841a5e9104dbfeff101568f44
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98919230"
---
# <a name="how-to-use-custom-allocation-policies"></a>Как использовать пользовательские политики выделения

Пользовательская политика выделения обеспечивает больший контроль над назначением устройств в Центре Интернета вещей. Это достигается с помощью пользовательского кода в [функции Azure](../azure-functions/functions-overview.md), используемого для назначения устройств Центру Интернета вещей. Служба подготовки устройств вызывает код функции Azure, предоставляя всю необходимую информацию об устройстве и регистрации. Код функции выполняется и возвращает данные Центра Интернета вещей, используемые при подготовке устройства.

С помощью функции пользовательских политик выделения можно определить собственные политики выделения, если политики, предоставленные службой подготовки устройств, не соответствуют требованиям вашего сценария.

Например, вам может понадобиться проверить сертификат, который устройство использует во время подготовки, и назначить устройство в центр Интернета вещей на основе свойства сертификата. Кроме того, сведения об устройствах могут храниться в базе данных, и чтобы определить, какому центру Интернета вещей следует назначить устройство, требуется отправить запрос к этой базе данных.

В этой статье показана работа с пользовательскими политиками с помощью кода функции Azure на языке C#. Создаются два новых Центра Интернета вещей, представляющие *отдел тостеров Contoso* и *отдел тепловых насосов Contoso*. Устройства, запрашивающие подготовку, должны иметь идентификатор регистрации с одним из следующих суффиксов, чтобы они были приняты для подготовки:

* **-contoso-tstrsd-007**: отдел тостеров Contoso;
* **-contoso-hpsd-088**: отдел тепловых насосов Contoso.

Устройства будут подготовлены в соответствии с этими обязательными суффиксами в идентификаторе регистрации. Эти устройства будут имитироваться с помощью примера для подготовки из [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c).

Выполните следующие действия в этой статье:

* Использование Azure CLI для создания центров Интернета вещей для двух отделов Contoso (**отдела тостеров Contoso** и **отдела тепловых насосов Contoso**).
* Создание группы регистрации с помощью функции Azure для пользовательской политики выделения.
* Создание ключей устройства для имитации двух устройств.
* Подготовка среды разработки для пакета SDK Azure IoT для C.
* Имитируйте устройства и убедитесь, что они подготовлены в соответствии с примером кода в пользовательской политике выделения.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Приведенные ниже предварительные требования касаются среды разработки Windows. При использовании Linux или macOS ознакомьтесь с соответствующим разделом в статье [Подготовка среды разработки](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) из документации к пакету SDK.

- [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) с включенной рабочей нагрузкой [Разработка классических приложений на C++](/cpp/ide/using-the-visual-studio-ide-for-cpp-desktop-development). Visual Studio 2015 или Visual Studio 2017 также поддерживаются.

- Установите последнюю версию [Git](https://git-scm.com/download/).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-the-provisioning-service-and-two-divisional-iot-hubs"></a>Создание службы подготовки и двух центров выпусков IoT

В этом разделе описано, как использовать Azure Cloud Shell для создания службы подготовки и двух центров Интернета вещей, представляющих **подразделение** и подразделение " **теплообменники** Contoso".

> [!TIP]
> Команды, используемые в этой статье, создают службу подготовки и другие ресурсы в расположении "Западная часть США". Рекомендуется создавать ресурсы в ближайшем для вас регионе, поддерживающем службу подготовки устройств. Чтобы просмотреть список доступных расположений, выполните команду `az provider show --namespace Microsoft.Devices --query "resourceTypes[?resourceType=='ProvisioningServices'].locations | [0]" --out table`, или перейдите на страницу [Состояние Azure](https://azure.microsoft.com/status/) и введите в строке поиска "Служба подготовки устройств". В командах расположения можно указать, используя одно слово или несколько. Например: westus, West US, WEST US и т. д. Обратите внимание, что регистр не учитывается. Если для указания расположения вы используете несколько слов, укажите значение поиска в кавычках, например `-- location "West US"`.
>

1. В Azure Cloud Shell создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az-group-create). Группа ресурсов Azure является логическим контейнером, в котором происходит развертывание ресурсов Azure и управление ими.

    В следующем примере создается группа ресурсов с именем *contoso-US-Resource-Group* в регионе *westus* . Рекомендуется использовать эту группу для всех ресурсов, созданных в рамках этой статьи. Этот подход упростит очистку после завершения работы.

    ```azurecli-interactive 
    az group create --name contoso-us-resource-group --location westus
    ```

2. Используйте Azure Cloud Shell, чтобы создать службу подготовки устройств (DPS) с помощью команды [AZ IOT DP Create](/cli/azure/iot/dps#az-iot-dps-create) . Служба подготовки будет добавлена в *contoso-US-Resource-Group*.

    В следующем примере создается служба подготовки с именем *contoso-подготовка-Service-1098* в расположении *westus* . Необходимо использовать уникальное имя службы. Создайте собственный суффикс в имени службы вместо **1098**.

    ```azurecli-interactive 
    az iot dps create --name contoso-provisioning-service-1098 --resource-group contoso-us-resource-group --location westus
    ```

    Выполнение этой команды может занять несколько минут.

3. В Azure Cloud Shell создайте Центр Интернета вещей **отдела тостеров Contoso** с помощью команды [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create). Этот Центр Интернета вещей будет добавлен в группу *contoso-us-resource-group*.

    В следующем примере создается центр Интернета вещей с именем *contoso-Names-Hub-1098* в расположении *westus* . Необходимо использовать уникальное имя концентратора. Придумайте собственный суффикс в имени центра вместо **1098**. 

    > [!CAUTION]
    > В примере кода функции Azure для настраиваемой политики распределения требуется подстрока `-toasters-` в имени концентратора. Убедитесь, что используется имя, содержащее подстроку обязательных всплывающих уведомлений.
    
    ```azurecli-interactive 
    az iot hub create --name contoso-toasters-hub-1098 --resource-group contoso-us-resource-group --location westus --sku S1
    ```

    Выполнение этой команды может занять несколько минут.

4. В Azure Cloud Shell создайте Центр Интернета вещей **отдела тепловых насосов Contoso** с помощью команды [az iot hub create](/cli/azure/iot/hub#az-iot-hub-create). Этот Центр Интернета вещей также будет добавлен в группу *contoso-us-resource-group*.

    В следующем примере создается центр Интернета вещей с именем *contoso-хеатпумпс-Hub-1098* в расположении *westus* . Необходимо использовать уникальное имя концентратора. Придумайте собственный суффикс в имени центра вместо **1098**. 

    > [!CAUTION]
    > В примере кода функции Azure для настраиваемой политики распределения требуется подстрока `-heatpumps-` в имени концентратора. Убедитесь, что используется имя, содержащее необходимую подстроку хеатпумпс.

    ```azurecli-interactive 
    az iot hub create --name contoso-heatpumps-hub-1098 --resource-group contoso-us-resource-group --location westus --sku S1
    ```

    Выполнение этой команды может занять несколько минут.

5. Центры Интернета вещей должны быть связаны с ресурсом DPS. 

    Выполните следующие две команды, чтобы получить строки подключения для только что созданного концентратора. Замените имена ресурсов концентратора именами, выбранными в каждой команде.

    ```azurecli-interactive 
    hubToastersConnectionString=$(az iot hub connection-string show --hub-name contoso-toasters-hub-1098 --key primary --query connectionString -o tsv)
    hubHeatpumpsConnectionString=$(az iot hub connection-string show --hub-name contoso-heatpumps-hub-1098 --key primary --query connectionString -o tsv)
    ```

    Выполните следующие команды, чтобы связать концентраторы с ресурсом DPS. Замените имя ресурса DPS именем, выбранным в каждой команде:

    ```azurecli-interactive 
    az iot dps linked-hub create --dps-name contoso-provisioning-service-1098 --resource-group contoso-us-resource-group --connection-string $hubToastersConnectionString --location westus
    az iot dps linked-hub create --dps-name contoso-provisioning-service-1098 --resource-group contoso-us-resource-group --connection-string $hubHeatpumpsConnectionString --location westus
    ```




## <a name="create-the-custom-allocation-function"></a>Создание пользовательской функции выделения

В этом разделе вы создадите функцию Azure, которая реализует пользовательскую политику выделения. Эта функция определяет, какой центр Интернета вещей должен быть зарегистрирован, в зависимости от того, содержит ли его идентификатор регистрации строку **-contoso-тстрсд-007** или **-contoso-хпсд-088**. Он также задает начальное состояние двойникаа устройства в зависимости от того, является ли устройство тостерным или теплоотводом.

1. Войдите на [портал Azure](https://portal.azure.com). На домашней странице портала Azure выберите **+ Создать ресурс**.

2. В поле *Поиск в Marketplace* введите "Приложение-функция". В раскрывающемся списке выберите **Приложение-функция**, а затем — **Создать**.

3. На открывшейся странице создания **приложения-функции** на вкладке **Основы** введите приведенные ниже параметры новой функции и нажмите **Просмотр и создание**.

    **Группа ресурсов**. Выберите **contoso-US-Resource-Group** , чтобы все ресурсы, созданные в этой статье, были объединены.

    **Имя приложения-функции**. Введите уникальное имя приложения-функции. В этом примере используется **contoso-Function-App-1098**.

    **Publish**: Убедитесь, что выбран параметр **Код**.

    **Стек среды выполнения**. В раскрывающемся меню выберите **.NET Core**.

    **Версия**: выберите **3,1** из раскрывающегося списка.

    **Регион**. Выберите тот же регион, что и для группы ресурсов. В этом примере используется регион **западная часть США**.

    > [!NOTE]
    > По умолчанию служба Application Insights активирована. Она необязательна для этой статьи, но может помочь понять и исследовать проблемы, возникающие при работе с пользовательским выделением. При желании Application Insights можно отключить. Для этого на вкладке **Мониторинг** выберите **Нет** для параметра **Включить Application Insights**.

    ![Создание приложения-функции Azure для размещения пользовательской функции выделения](./media/how-to-use-custom-allocation-policies/create-function-app.png)

4. На странице **Сводка** щелкните **Создать**, чтобы создать приложение-функцию. Развертывание может занять несколько минут. По завершении выберите **Перейдите к ресурсу**.

5. В левой области страницы **Обзор** приложения-функции щелкните **функции** , а затем **+ добавить** , чтобы добавить новую функцию.

6. На странице **Добавление функции** щелкните **триггер HTTP**, а затем нажмите кнопку **добавить** .

7. На следующей странице щелкните **код + тест**. Это позволяет изменить код функции с именем **HttpTrigger1**. Файл кода **Run. CSX** должен быть открыт для редактирования.

8. Требуется ссылка на пакеты NuGet. Чтобы создать первоначальное двойника устройства, пользовательская функция выделения использует классы, определенные в двух пакетах NuGet, которые должны быть загружены в среду размещения. С помощью функций Azure вы указываете на пакеты NuGet, используя файл *Function. proj* . На этом шаге вы сохраняете и отправляете файл *Function. proj* для необходимых сборок.  Дополнительные сведения см. в статье [Использование пакетов NuGet с функциями Azure](../azure-functions/functions-reference-csharp.md#using-nuget-packages).

    1. Скопируйте следующие строки в свой любимый редактор и сохраните файл на своем компьютере как *Function. proj*.

        ```xml
        <Project Sdk="Microsoft.NET.Sdk">  
            <PropertyGroup>  
                <TargetFramework>netstandard2.0</TargetFramework>  
            </PropertyGroup>  
            <ItemGroup>  
                <PackageReference Include="Microsoft.Azure.Devices.Provisioning.Service" Version="1.16.3" />
                <PackageReference Include="Microsoft.Azure.Devices.Shared" Version="1.27.0" />
            </ItemGroup>  
        </Project>
        ```

    2. Нажмите кнопку **Отправить** , расположенную над редактором кода, чтобы отправить файл *Function. proj* . После отправки выберите файл в редакторе кода, используя раскрывающийся список, чтобы проверить содержимое.

9. Убедитесь, что в редакторе кода выбран параметр *Run. CSX* для **HttpTrigger1** . Замените код функции **HttpTrigger1** следующим кодом и выберите **сохранить**:

    ```csharp
    #r "Newtonsoft.Json"

    using System.Net;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    using Microsoft.Azure.Devices.Shared;               // For TwinCollection
    using Microsoft.Azure.Devices.Provisioning.Service; // For TwinState

    public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        // Get request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);

        log.LogInformation("Request.Body:...");
        log.LogInformation(requestBody);

        // Get registration ID of the device
        string regId = data?.deviceRuntimeContext?.registrationId;

        string message = "Uncaught error";
        bool fail = false;
        ResponseObj obj = new ResponseObj();

        if (regId == null)
        {
            message = "Registration ID not provided for the device.";
            log.LogInformation("Registration ID : NULL");
            fail = true;
        }
        else
        {
            string[] hubs = data?.linkedHubs.ToObject<string[]>();

            // Must have hubs selected on the enrollment
            if (hubs == null)
            {
                message = "No hub group defined for the enrollment.";
                log.LogInformation("linkedHubs : NULL");
                fail = true;
            }
            else
            {
                // This is a Contoso Toaster Model 007
                if (regId.Contains("-contoso-tstrsd-007"))
                {
                    //Find the "-toasters-" IoT hub configured on the enrollment
                    foreach(string hubString in hubs)
                    {
                        if (hubString.Contains("-toasters-"))
                            obj.iotHubHostName = hubString;
                    }

                    if (obj.iotHubHostName == null)
                    {
                        message = "No toasters hub found for the enrollment.";
                        log.LogInformation(message);
                        fail = true;
                    }
                    else
                    {
                        // Specify the initial tags for the device.
                        TwinCollection tags = new TwinCollection();
                        tags["deviceType"] = "toaster";

                        // Specify the initial desired properties for the device.
                        TwinCollection properties = new TwinCollection();
                        properties["state"] = "ready";
                        properties["darknessSetting"] = "medium";

                        // Add the initial twin state to the response.
                        TwinState twinState = new TwinState(tags, properties);
                        obj.initialTwin = twinState;
                    }
                }
                // This is a Contoso Heat pump Model 008
                else if (regId.Contains("-contoso-hpsd-088"))
                {
                    //Find the "-heatpumps-" IoT hub configured on the enrollment
                    foreach(string hubString in hubs)
                    {
                        if (hubString.Contains("-heatpumps-"))
                            obj.iotHubHostName = hubString;
                    }

                    if (obj.iotHubHostName == null)
                    {
                        message = "No heat pumps hub found for the enrollment.";
                        log.LogInformation(message);
                        fail = true;
                    }
                    else
                    {
                        // Specify the initial tags for the device.
                        TwinCollection tags = new TwinCollection();
                        tags["deviceType"] = "heatpump";

                        // Specify the initial desired properties for the device.
                        TwinCollection properties = new TwinCollection();
                        properties["state"] = "on";
                        properties["temperatureSetting"] = "65";

                        // Add the initial twin state to the response.
                        TwinState twinState = new TwinState(tags, properties);
                        obj.initialTwin = twinState;
                    }
                }
                // Unrecognized device.
                else
                {
                    fail = true;
                    message = "Unrecognized device registration.";
                    log.LogInformation("Unknown device registration");
                }
            }
        }

        log.LogInformation("\nResponse");
        log.LogInformation((obj.iotHubHostName != null) ? JsonConvert.SerializeObject(obj) : message);

        return (fail)
            ? new BadRequestObjectResult(message) 
            : (ActionResult)new OkObjectResult(obj);
    }

    public class ResponseObj
    {
        public string iotHubHostName {get; set;}
        public TwinState initialTwin {get; set;}
    }
    ```

## <a name="create-the-enrollment"></a>Создание регистрации

В этом разделе вы узнаете, как создать группу регистрации, использующую пользовательскую политику выделения. Для удобства в этой статье используется [аттестация симметричного ключа](concepts-symmetric-key-attestation.md) с регистрацией. В качестве более безопасного решения рекомендуется использовать [аттестацию сертификатов X.509](concepts-x509-attestation.md) с цепочкой доверия.

1. На [портале Azure](https://portal.azure.com) откройте службу подготовки.

2. В области слева выберите вкладку **Управление регистрациями**, а затем нажмите кнопку **Добавить группу регистрации** в верхней части страницы.

3. На странице **Добавление группы регистрации** введите следующие сведения и нажмите кнопку **сохранить** .

    **Имя группы**: введите **contoso-custom-allocated-devices**.

    **Тип аттестации**: выберите **Симметричный ключ**.

    **Автоматически создавать ключи**: этот флажок должен быть уже установлен.

    **Выберите способ назначения устройств для Центров**: выберите **Custom (Use Azure Function)** (Пользовательский (функция Azure)).

    **Подписка**. Выберите подписку, в которой была создана функция Azure.

    **Приложение-функция**. Выберите свое приложение функции по имени. в этом примере использовался **contoso-Function-App-1098** .

    **Функция**. Выберите функцию **HttpTrigger1** .

    ![Добавление группы регистрации пользовательского выделения для аттестации симметричного ключа](./media/how-to-use-custom-allocation-policies/create-custom-allocation-enrollment.png)

4. После сохранения регистрации снова откройте ее и запомните или запишите **первичный ключ**. Для формирования ключей необходимо сначала сохранить регистрацию. Этот ключ будет позднее использоваться для создания уникальных ключей устройства для имитированных устройств.

## <a name="derive-unique-device-keys"></a>Получение производных уникальных ключей

В этом разделе описано создание двух уникальных ключей устройств. Один ключ будет использоваться для имитированного устройства toaster. Другой ключ будет использоваться для имитированного устройства heat pump.

Для создания ключа устройства используется **первичный ключ** , записанный ранее, для вычисления [HMAC-SHA256](https://wikipedia.org/wiki/HMAC) идентификатора регистрации устройства для каждого устройства и преобразования результата в формат Base64. Дополнительные сведения о создании производных ключей устройств с группами регистраций см. в разделе о регистрации групп в статье [Аттестация симметричного ключа](concepts-symmetric-key-attestation.md).

Для примера в этой статье используйте следующие два идентификатора регистрации устройств и вычислите ключи для обоих устройств. Оба идентификатора регистрации содержат допустимый суффикс для работы с примером кода для пользовательской политики выделения:

* **breakroom499-contoso-tstrsd-007**
* **mainbuilding167-contoso-hpsd-088**


# <a name="windows"></a>[Windows](#tab/windows)

Если вы используете рабочую станцию для Windows, можно использовать PowerShell для формирования производного ключа устройства, как показано в следующем примере.

Замените значение **KEY** значением **первичного ключа**, записанным ранее.

```powershell
$KEY='oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA=='

$REG_ID1='breakroom499-contoso-tstrsd-007'
$REG_ID2='mainbuilding167-contoso-hpsd-088'

$hmacsha256 = New-Object System.Security.Cryptography.HMACSHA256
$hmacsha256.key = [Convert]::FromBase64String($KEY)
$sig1 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID1))
$sig2 = $hmacsha256.ComputeHash([Text.Encoding]::ASCII.GetBytes($REG_ID2))
$derivedkey1 = [Convert]::ToBase64String($sig1)
$derivedkey2 = [Convert]::ToBase64String($sig2)

echo "`n`n$REG_ID1 : $derivedkey1`n$REG_ID2 : $derivedkey2`n`n"
```

```powershell
breakroom499-contoso-tstrsd-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
mainbuilding167-contoso-hpsd-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
```

# <a name="linux"></a>[Linux](#tab/linux)

Если вы используете рабочую станцию Linux, вы можете использовать OpenSSL для создания ключей производных устройств, как показано в следующем примере.

Замените значение **KEY** значением **первичного ключа**, записанным ранее.

```bash
KEY=oiK77Oy7rBw8YB6IS6ukRChAw+Yq6GC61RMrPLSTiOOtdI+XDu0LmLuNm11p+qv2I+adqGUdZHm46zXAQdZoOA==

REG_ID1=breakroom499-contoso-tstrsd-007
REG_ID2=mainbuilding167-contoso-hpsd-088

keybytes=$(echo $KEY | base64 --decode | xxd -p -u -c 1000)
devkey1=$(echo -n $REG_ID1 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)
devkey2=$(echo -n $REG_ID2 | openssl sha256 -mac HMAC -macopt hexkey:$keybytes -binary | base64)

echo -e $"\n\n$REG_ID1 : $devkey1\n$REG_ID2 : $devkey2\n\n"
```

```bash
breakroom499-contoso-tstrsd-007 : JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=
mainbuilding167-contoso-hpsd-088 : 6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=
```

---

Имитированные устройства будут использовать производные ключи устройств с соответствующим идентификатором регистрации для выполнения аттестации симметричного ключа.

## <a name="prepare-an-azure-iot-c-sdk-development-environment"></a>Подготовка среды разработки для пакета SDK Azure IoT для C

В этом разделе описано, как подготовить среду разработки, которая используется для сборки [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c). В пакет SDK входит пример кода для имитированного устройства. Для этого имитированного устройства будет выполнена попытка подготовки во время последовательности загрузки.

В этом разделе описывается использование рабочей станции Windows. Пример для Linux приведен в инструкциях по настройке виртуальных машин в статье [Подготовка к мультитенантности](how-to-provision-multitenant.md).

1. Скачайте [систему сборки CMake](https://cmake.org/download/).

    **Перед** установкой `CMake` очень важно установить на компьютер необходимые компоненты Visual Studio (Visual Studio с рабочей нагрузкой "Разработка классических приложений на C++"). После установки компонентов и проверки загрузки установите систему сборки CMake.

2. Найдите имя тега для [последнего выпуска](https://github.com/Azure/azure-iot-sdk-c/releases/latest) пакета SDK.

3. Откройте командную строку или оболочку Git Bash. Выполните приведенные ниже команды, чтобы клонировать репозиторий GitHub с последним выпуском [пакета SDK Azure IoT для C](https://github.com/Azure/azure-iot-sdk-c). Используйте найденный тег в качестве значения для параметра `-b`:

    ```cmd/sh
    git clone -b <release-tag> https://github.com/Azure/azure-iot-sdk-c.git
    cd azure-iot-sdk-c
    git submodule update --init
    ```

    Выполнение этой операции может занять несколько минут.

4. Создайте подкаталог `cmake` в корневом каталоге репозитория Git и перейдите в эту папку. В каталоге `azure-iot-sdk-c` выполните следующие команды:

    ```cmd/sh
    mkdir cmake
    cd cmake
    ```

5. Выполните приведенную ниже команду, чтобы создать версию пакета SDK для используемой клиентской платформы разработки. Эта команда также создает решение Visual Studio для имитированного устройства в каталоге `cmake`. 

    ```cmd
    cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    ```

    Если `cmake` не удается найти компилятор C++, во время выполнения команды могут возникнуть ошибки сборки. В этом случае попробуйте, выполнить эту команду в [командной строке Visual Studio](/dotnet/framework/tools/developer-command-prompt-for-vs).

    После успешного создания последние несколько строк выходных данных будут выглядеть следующим образом:

    ```cmd/sh
    $ cmake -Dhsm_type_symm_key:BOOL=ON -Duse_prov_client:BOOL=ON  ..
    -- Building for: Visual Studio 15 2017
    -- Selecting Windows SDK version 10.0.16299.0 to target Windows 10.0.17134.
    -- The C compiler identification is MSVC 19.12.25835.0
    -- The CXX compiler identification is MSVC 19.12.25835.0

    ...

    -- Configuring done
    -- Generating done
    -- Build files have been written to: E:/IoT Testing/azure-iot-sdk-c/cmake
    ```

## <a name="simulate-the-devices"></a>Имитация устройств

В этом разделе описано, как обновить пример для подготовки **prov\_dev\_client\_sample**, который размещен в пакете SDK Azure IoT для C, настроенном ранее.

Пример кода имитирует последовательность загрузки устройства, которое отправляет запрос на подготовку в экземпляр службы подготовки устройств. При выполнении последовательности загрузки устройство toaster будет распознано и назначено Центру Интернета вещей с помощью пользовательской политики выделения.

1. На портале Azure выберите вкладку **Обзор** службы подготовки устройств и запишите значение **_области идентификатора_**.

    ![Извлеките сведения о конечной точке службы подготовки устройств из колонки на портале](./media/quick-create-simulated-device-x509/extract-dps-endpoints.png) 

2. В Visual Studio откройте файл решения **azure_iot_sdks.sln**, который был создан ранее в результате запуска CMake. Файл решения должен находиться в следующем расположении:

    ```
    azure-iot-sdk-c\cmake\azure_iot_sdks.sln
    ```

3. В окне *Обозреватель решений* Visual Studio перейдите в папку **Provision\_Samples**. Разверните пример проекта с именем **prov\_dev\_client\_sample**. Разверните **исходные файлы** и откройте **prov\_dev\_client\_sample.c**.

4. Найдите константу `id_scope` и замените ее значение ранее скопированным значением **области идентификатора**. 

    ```c
    static const char* id_scope = "0ne00002193";
    ```

5. Найдите определение функции `main()` в том же файле. Проверьте, чтобы переменной `hsm_type` было задано значение `SECURE_DEVICE_TYPE_SYMMETRIC_KEY`, как показано ниже.

    ```c
    SECURE_DEVICE_TYPE hsm_type;
    //hsm_type = SECURE_DEVICE_TYPE_TPM;
    //hsm_type = SECURE_DEVICE_TYPE_X509;
    hsm_type = SECURE_DEVICE_TYPE_SYMMETRIC_KEY;
    ```

6. Щелкните проект **prov\_dev\_client\_sample** правой кнопкой мыши и выберите пункт **Назначить запускаемым проектом**.

### <a name="simulate-the-contoso-toaster-device"></a>Имитация устройства toaster Contoso

1. Для имитации устройства тостера найдите закомментированный вызов `prov_dev_set_symmetric_key_info()` в **prov\_dev\_client\_sample.c**.

    ```c
    // Set the symmetric key if using they auth type
    //prov_dev_set_symmetric_key_info("<symm_registration_id>", "<symmetric_Key>");
    ```

    Раскомментируйте вызов функции и замените значения заполнителей (включая угловые скобки) идентификатором регистрации тостера и производным ключом устройства, который был создан ранее. Значение ключа **JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=**, показанное ниже, приведено только в качестве примера.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("breakroom499-contoso-tstrsd-007", "JC8F96eayuQwwz+PkE7IzjH2lIAjCUnAa61tDigBnSs=");
    ```

    Сохраните файл.

2. В меню Visual Studio выберите **Отладка** > **Запуск без отладки**, чтобы запустить решение. При появлении запроса перестроить проект щелкните **Да**, чтобы перестроить его перед запуском.

    Ниже приведен пример, в котором виртуальное устройство Тостер успешно загрузилось и подключается к экземпляру службы подготовки, чтобы он был назначен для центра Интернета вещей с помощью настраиваемой политики выделения.

    ```cmd
    Provisioning API Version: 1.3.6

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-toasters-hub-1098.azure-devices.net, deviceId: breakroom499-contoso-tstrsd-007

    Press enter key to exit:
    ```

### <a name="simulate-the-contoso-heat-pump-device"></a>Имитация устройства heat pump Contoso

1. Для имитации устройства теплового насоса обновите вызов `prov_dev_set_symmetric_key_info()` в **prov\_dev\_client\_sample.c**, указав идентификатор регистрации теплового насоса и производный ключ устройства, созданный ранее. Значение ключа **6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=**, показанное ниже, приведено только в качестве примера.

    ```c
    // Set the symmetric key if using they auth type
    prov_dev_set_symmetric_key_info("mainbuilding167-contoso-hpsd-088", "6uejA9PfkQgmYylj8Zerp3kcbeVrGZ172YLa7VSnJzg=");
    ```

    Сохраните файл.

2. В меню Visual Studio выберите **Отладка** > **Запуск без отладки**, чтобы запустить решение. При появлении запроса перестроить проект щелкните **Да**, чтобы перестроить его перед запуском.

    Ниже приведен пример, в котором виртуальное устройство теплоотвода успешно загружается и подключается к экземпляру службы подготовки, который назначается настраиваемой политикой распределения для центра Интернета вещей "тепло-насоса".

    ```cmd
    Provisioning API Version: 1.3.6

    Registering Device

    Provisioning Status: PROV_DEVICE_REG_STATUS_CONNECTED
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING
    Provisioning Status: PROV_DEVICE_REG_STATUS_ASSIGNING

    Registration Information received from service: contoso-heatpumps-hub-1098.azure-devices.net, deviceId: mainbuilding167-contoso-hpsd-088

    Press enter key to exit:
    ```

## <a name="troubleshooting-custom-allocation-policies"></a>Устранение неполадок пользовательских политик выделения

В следующей таблице показаны ожидаемые сценарии и коды ошибок, которые могут быть получены. Используйте эту таблицу при устранении неполадок в работе пользовательских политик выделения с функциями Azure.

| Сценарий | Результат регистрации из службы подготовки | Результаты пакета SDK для подготовки |
| -------- | --------------------------------------------- | ------------------------ |
| Веб-перехватчик возвращает ответ "200 ОК" со свойством iotHubHostName, которому присвоено допустимое имя узла Центра Интернета вещей | Состояние результата: "Назначено"  | Пакет SDK возвращает PROV_DEVICE_RESULT_OK, а также сведения о центре |
| Веб-перехватчик возвращает ответ "200 ОК" со свойством iotHubHostName, но его значение является пустой строкой или NULL | Состояние результата: "Сбой"<br><br> Код ошибки: CustomAllocationIotHubNotSpecified (400208) | Пакет SDK возвращает PROV_DEVICE_RESULT_HUB_NOT_SPECIFIED |
| Веб-перехватчик возвращает ответ "401 — недостаточно прав" | Состояние результата: "Сбой"<br><br>Код ошибки: CustomAllocationUnauthorizedAccess (400209) | Пакет SDK возвращает PROV_DEVICE_RESULT_UNAUTHORIZED |
| Была создана индивидуальная регистрация для отключения устройства | Состояние результата: "Отключено" | Пакет SDK возвращает PROV_DEVICE_RESULT_DISABLED |
| Веб-перехватчик возвращает код ошибки больше или равный 429 | Попытка повторной оркестрации DPS будет выполнена несколько раз. Текущее состояние политики повтора:<br><br>&nbsp;&nbsp; счетчик попыток: 10;<br>&nbsp;&nbsp; исходный интервал: 1 с;<br>&nbsp;&nbsp; шаг увеличения: 9 с. | Пакет SDK пропустит ошибку и отправит еще одно сообщение для получения состояния в указанное время. |
| Веб-перехватчик возвращает любой другой код состояния | Состояние результата: "Сбой"<br><br>Код ошибки: CustomAllocationFailed (400207) | Пакет SDK возвращает PROV_DEVICE_RESULT_DEV_AUTH_ERROR |

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы планируете продолжить работу с ресурсами, созданными при работе с этой статьей, не удаляйте их. В противном случае выполните следующие действия, чтобы удалить все созданные ресурсы, за которые может взиматься плата.

Здесь предполагается, что вы создали все используемые в этой статье ресурсы, как было указано, в одной группе ресурсов, **contoso-us-resource-group**.

> [!IMPORTANT]
> Удаление группы ресурсов — процесс необратимый. Группа ресурсов и все содержащиеся в ней ресурсы удаляются без возможности восстановления. Будьте внимательны, чтобы случайно не удалить не ту группу ресурсов или не те ресурсы. Если вы создали Центр Интернета вещей в группе ресурсов, содержащей ресурсы, которые нужно сохранить, удалите только ресурс Центра Интернета вещей, не удаляя всю группу ресурсов.
>

Удаление группы ресурсов по имени.

1. Войдите в портал [Azure](https://portal.azure.com) и выберите **Группы ресурсов**.

2. Введите в текстовое поле **Фильтровать по имени...** имя вашей группы ресурсов: **contoso-us-resource-group**. 

3. Справа от своей группы ресурсов в списке результатов щелкните **...**, а затем выберите **Удалить группу ресурсов**.

4. Вам будет предложено подтвердить операцию удаления группы ресурсов. Снова введите имя группы ресурсов, которую необходимо удалить, и щелкните **Удалить**. Через некоторое время группа ресурсов и все ее ресурсы будут удалены.

## <a name="next-steps"></a>Дальнейшие действия

* Дополнительные сведения о повторной подготовке см. в статье [Основные понятия повторной инициализации устройств центра Интернета вещей](concepts-device-reprovision.md) . 
* Дополнительные сведения о повторной подготовке см. в статье [как отменить предварительную инициализацию устройств, которые были настроены ранее](how-to-unprovision-devices.md) .