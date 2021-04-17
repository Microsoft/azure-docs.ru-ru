---
author: dominicbetts
ms.author: dobett
ms.service: iot-pnp
ms.topic: include
ms.date: 03/31/2021
ms.openlocfilehash: 0383949dd9b0aeaa185613474d91d0416169ab1f
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491149"
---
## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить действия, описанные в этой статье, необходимы следующие ресурсы:

* Приложение Azure IoT Central, созданное на основе шаблона **Пользовательское приложение**. Дополнительные сведения см. в [кратком руководстве по созданию приложения](../articles/iot-central/core/quick-deploy-iot-central.md). Приложение должно быть создано не ранее 14 июля 2020 года.
* Компьютер для разработки с [Visual Studio (выпуски Community, Professional или Enterprise)](https://visualstudio.microsoft.com/downloads/).
* Локальная копия репозитория [примеров Интернета вещей Microsoft Azure для C# (.NET)](https://github.com/Azure-Samples/azure-iot-samples-csharp) с сайта GitHub, содержащего образец кода. Используйте эту ссылку, чтобы скачать копию репозитория: [Скачать ZIP-файл](https://github.com/Azure-Samples/azure-iot-samples-csharp/archive/master.zip). Затем распакуйте файл в подходящее расположение на локальном компьютере.

## <a name="review-the-code"></a>Просмотр кода

В копии репозитория примеров Интернета вещей Microsoft Azure для C#, которую вы скачали ранее, откройте файл решения *azure-iot-samples-csharp-master\iot-hub\Samples\device\IoTHubDeviceSamples.sln"* в Visual Studio. В **Обозревателе решений** разверните папку *PnpDeviceSamples > TemperatureController* и откройте файлы *Program.cs* и *TemperatureControllerSample.cs*, чтобы просмотреть код для этого примера.

Когда вы запускаете пример для подключения к IoT Central, для регистрации устройства и создания строки подключения используется служба подготовки устройств (DPS). В примере извлекаются необходимые сведения о подключении DPS из среды.

В *Program.cs* метод `Main` вызывает `SetupDeviceClientAsync`, чтобы:

* использовать идентификатор модели `dtmi:com:example:TemperatureController;2` при подготовке устройства с помощью DPS; IoT Central использует идентификатор модели для идентификации или создания шаблона устройства для этого устройства. Дополнительные сведения см. в разделе [Связывание устройства с шаблоном устройства](../articles/iot-central/core/concepts-get-connected.md#associate-a-device-with-a-device-template).
* создать экземпляр **DeviceClient** для подключения к IoT Central.

```csharp
private static async Task<DeviceClient> SetupDeviceClientAsync(Parameters parameters, CancellationToken cancellationToken)
{
  DeviceClient deviceClient;
  switch (parameters.DeviceSecurityType.ToLowerInvariant())
  {
    case "dps":
      DeviceRegistrationResult dpsRegistrationResult = await ProvisionDeviceAsync(parameters, cancellationToken);
      var authMethod = new DeviceAuthenticationWithRegistrySymmetricKey(dpsRegistrationResult.DeviceId, parameters.DeviceSymmetricKey);
      deviceClient = InitializeDeviceClient(dpsRegistrationResult.AssignedHub, authMethod);
      break;

    case "connectionstring":
      // ...

    default:
      // ...
  }
  return deviceClient;
}
```

Затем метод main создает экземпляр класса **TemperatureControllerSample** и вызывает метод `PerformOperationsAsync` для обработки взаимодействия с IoT Central.

Метод `PerformOperationsAsync` в файле *TemperatureControllerSample.cs* выполняет следующие действия:

* Задает обработчик для команды **reboot** в компоненте по умолчанию.
* Задает обработчики для команд **getMaxMinReport** в двух компонентах термостата.
* Задает обработчики для получения обновлений свойств целевой температуры в двух компонентах термостата.
* Отправляет обновления свойств с начальной информацией об устройстве.
* Периодически отправляет данные телеметрии температуры от двух компонентов термостата.
* Периодически отправляет данные телеметрии рабочего набора от компонента по умолчанию.
* Отправляет значение максимальной температуры с момента последней перезагрузки каждый раз при достижении новой максимальной температуры в двух компонентах термостата.

```csharp
public async Task PerformOperationsAsync(CancellationToken cancellationToken)
{
  await _deviceClient.SetMethodHandlerAsync("reboot", HandleRebootCommandAsync, _deviceClient, cancellationToken);

  // For a component-level command, the command name is in the format "<component-name>*<command-name>".
  await _deviceClient.SetMethodHandlerAsync("thermostat1*getMaxMinReport", HandleMaxMinReportCommand, Thermostat1, cancellationToken);
  await _deviceClient.SetMethodHandlerAsync("thermostat2*getMaxMinReport", HandleMaxMinReportCommand, Thermostat2, cancellationToken);

  await _deviceClient.SetDesiredPropertyUpdateCallbackAsync(SetDesiredPropertyUpdateCallback, null, cancellationToken);
  _desiredPropertyUpdateCallbacks.Add(Thermostat1, TargetTemperatureUpdateCallbackAsync);
  _desiredPropertyUpdateCallbacks.Add(Thermostat2, TargetTemperatureUpdateCallbackAsync);

  await UpdateDeviceInformationAsync(cancellationToken);
  await SendDeviceSerialNumberAsync(cancellationToken);

  bool temperatureReset = true;
  _maxTemp[Thermostat1] = 0d;
  _maxTemp[Thermostat2] = 0d;

  while (!cancellationToken.IsCancellationRequested)
  {
    if (temperatureReset)
    {
      // Generate a random value between 5.0°C and 45.0°C for the current temperature reading for each "Thermostat" component.
      _temperature[Thermostat1] = Math.Round(s_random.NextDouble() * 40.0 + 5.0, 1);
      _temperature[Thermostat2] = Math.Round(s_random.NextDouble() * 40.0 + 5.0, 1);
    }

    await SendTemperatureAsync(Thermostat1, cancellationToken);
    await SendTemperatureAsync(Thermostat2, cancellationToken);
    await SendDeviceMemoryAsync(cancellationToken);

    temperatureReset = _temperature[Thermostat1] == 0 && _temperature[Thermostat2] == 0;
    await Task.Delay(5 * 1000);
  }
}
```

Метод `SendTemperatureAsync` показывает, как устройство отправляет данные телеметрии температуры от компонента в IoT Central. Метод `SendTemperatureTelemetryAsync` использует класс `PnpConvention` для создания сообщения:

```csharp
private async Task SendTemperatureAsync(string componentName, CancellationToken cancellationToken)
{
  await SendTemperatureTelemetryAsync(componentName, cancellationToken);

  double maxTemp = _temperatureReadingsDateTimeOffset[componentName].Values.Max<double>();
  if (maxTemp > _maxTemp[componentName])
  {
    _maxTemp[componentName] = maxTemp;
    await UpdateMaxTemperatureSinceLastRebootAsync(componentName, cancellationToken);
  }
}

private async Task SendTemperatureTelemetryAsync(string componentName, CancellationToken cancellationToken)
{
  const string telemetryName = "temperature";
  double currentTemperature = _temperature[componentName];
  using Message msg = PnpConvention.CreateMessage(telemetryName, currentTemperature, componentName);

  await _deviceClient.SendEventAsync(msg, cancellationToken);

  if (_temperatureReadingsDateTimeOffset.ContainsKey(componentName))
  {
    _temperatureReadingsDateTimeOffset[componentName].TryAdd(DateTimeOffset.UtcNow, currentTemperature);
  }
  else
  {
    _temperatureReadingsDateTimeOffset.TryAdd(
      componentName,
      new Dictionary<DateTimeOffset, double>
      {
        { DateTimeOffset.UtcNow, currentTemperature },
      });
  }
}
```

Метод `UpdateMaxTemperatureSinceLastRebootAsync` отправляет обновление свойства `maxTempSinceLastReboot` в IoT Central. Этот метод использует класс `PnpConvention` для создания исправления:

```csharp
private async Task UpdateMaxTemperatureSinceLastRebootAsync(string componentName, CancellationToken cancellationToken)
{
  const string propertyName = "maxTempSinceLastReboot";
  double maxTemp = _maxTemp[componentName];
  TwinCollection reportedProperties = PnpConvention.CreateComponentPropertyPatch(componentName, propertyName, maxTemp);

  await _deviceClient.UpdateReportedPropertiesAsync(reportedProperties, cancellationToken);
}
```

Метод `TargetTemperatureUpdateCallbackAsync` обрабатывает обновление свойства целевой температуры, доступного для записи, от IoT Central: Этот метод использует класс `PnpConvention` для чтения сообщения об обновлении свойства и создания ответа:

```csharp
private async Task TargetTemperatureUpdateCallbackAsync(TwinCollection desiredProperties, object userContext)
{
  const string propertyName = "targetTemperature";
  string componentName = (string)userContext;

  bool targetTempUpdateReceived = PnpConvention.TryGetPropertyFromTwin(
    desiredProperties,
    propertyName,
    out double targetTemperature,
    componentName);
  if (!targetTempUpdateReceived)
  {
      return;
  }

  TwinCollection pendingReportedProperty = PnpConvention.CreateComponentWritablePropertyResponse(
      componentName,
      propertyName,
      targetTemperature,
      (int)StatusCode.InProgress,
      desiredProperties.Version);

  await _deviceClient.UpdateReportedPropertiesAsync(pendingReportedProperty);

  // Update Temperature in 2 steps
  double step = (targetTemperature - _temperature[componentName]) / 2d;
  for (int i = 1; i <= 2; i++)
  {
      _temperature[componentName] = Math.Round(_temperature[componentName] + step, 1);
      await Task.Delay(6 * 1000);
  }

  TwinCollection completedReportedProperty = PnpConvention.CreateComponentWritablePropertyResponse(
      componentName,
      propertyName,
      _temperature[componentName],
      (int)StatusCode.Completed,
      desiredProperties.Version,
      "Successfully updated target temperature");

  await _deviceClient.UpdateReportedPropertiesAsync(completedReportedProperty);
}

```

Метод `HandleMaxMinReportCommand` обрабатывает команды для компонентов, вызываемых из IoT Central:

```csharp
private Task<MethodResponse> HandleMaxMinReportCommand(MethodRequest request, object userContext)
{
    try
    {
        string componentName = (string)userContext;
        DateTime sinceInUtc = JsonConvert.DeserializeObject<DateTime>(request.DataAsJson);
        var sinceInDateTimeOffset = new DateTimeOffset(sinceInUtc);

        if (_temperatureReadingsDateTimeOffset.ContainsKey(componentName))
        {

            Dictionary<DateTimeOffset, double> allReadings = _temperatureReadingsDateTimeOffset[componentName];
            Dictionary<DateTimeOffset, double> filteredReadings = allReadings.Where(i => i.Key > sinceInDateTimeOffset)
                .ToDictionary(i => i.Key, i => i.Value);

            if (filteredReadings != null && filteredReadings.Any())
            {
                var report = new
                {
                    maxTemp = filteredReadings.Values.Max<double>(),
                    minTemp = filteredReadings.Values.Min<double>(),
                    avgTemp = filteredReadings.Values.Average(),
                    startTime = filteredReadings.Keys.Min(),
                    endTime = filteredReadings.Keys.Max(),
                };

                byte[] responsePayload = Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(report));
                return Task.FromResult(new MethodResponse(responsePayload, (int)StatusCode.Completed));
            }

            return Task.FromResult(new MethodResponse((int)StatusCode.NotFound));
        }

        return Task.FromResult(new MethodResponse((int)StatusCode.NotFound));
    }
    catch (JsonReaderException ex)
    {
        // ...
    }
}
```

## <a name="get-connection-information"></a>Получение сведений о подключении

[!INCLUDE [iot-central-connection-configuration](iot-central-connection-configuration.md)]

## <a name="run-the-code"></a>Выполнение кода

Чтобы запустить пример приложения в Visual Studio, выполните следующие действия:

1. В **Обозревателе решений** выберите файл проекта **PnpDeviceSamples > TemperatureController**.

1. Последовательно выберите **"Проект" > "Свойства контроллера температуры" > "Отладка"** . Затем добавьте в проект следующие переменные среды:

    | Имя | Значение |
    | ---- | ----- |
    | IOTHUB_DEVICE_SECURITY_TYPE | DPS |
    | IOTHUB_DEVICE_DPS_ENDPOINT | global.azure-devices-provisioning.net |
    | IOTHUB_DEVICE_DPS_ID_SCOPE | Значение области идентификатора, которое вы записали ранее. |
    | IOTHUB_DEVICE_DPS_DEVICE_ID | sample-device-01 |
    | IOTHUB_DEVICE_DPS_DEVICE_KEY | Созданное значение ключа устройства, которое вы записали ранее. |

Теперь можно запустить и отладить пример в Visual Studio.

В указанных ниже выходных данных показаны регистрация устройства и его подключение к IoT Central. Этот пример кода запускает отправку данных телеметрии:

```output
[03/31/2021 14:43:17]info: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Press Control+C to quit the sample.
[03/31/2021 14:43:17]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Set up the device client.
[03/31/2021 14:43:18]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Initializing via DPS
[03/31/2021 14:43:27]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Set handler for 'reboot' command.
[03/31/2021 14:43:27]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Connection status change registered - status=Connected, reason=Connection_Ok.
[03/31/2021 14:43:28]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Set handler for "getMaxMinReport" command.
[03/31/2021 14:43:28]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Set handler to receive 'targetTemperature' updates.
[03/31/2021 14:43:28]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Property: Update - component = 'deviceInformation', properties update is complete.
[03/31/2021 14:43:28]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Property: Update - { "serialNumber": "SR-123456" } is complete.
[03/31/2021 14:43:29]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Telemetry: Sent - component="thermostat1", { "temperature": 34.2 } in °C.
[03/31/2021 14:43:29]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Property: Update - component="thermostat1", { "maxTempSinceLastReboot": 34.2 } in °C is complete.
[03/31/2021 14:43:29]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Telemetry: Sent - component="thermostat2", { "temperature": 25.1 } in °C.
[03/31/2021 14:43:29]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Property: Update - component="thermostat2", { "maxTempSinceLastReboot": 25.1 } in °C is complete.
[03/31/2021 14:43:29]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Telemetry: Sent - {"workingSet":31412} in KB.
```

[!INCLUDE [iot-central-monitor-thermostat](iot-central-monitor-thermostat.md)]

Вы можете увидеть, как устройство реагирует на команды и обновления свойств:

```output
[03/31/2021 14:47:00]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Command: Received - component="thermostat2", generating max, min and avg temperature report since 31/03/2021 06:00:00.
[03/31/2021 14:47:00]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Command: component="thermostat2", MaxMinReport since 31/03/2021 06:00:00: maxTemp=36.4, minTemp=36.4, avgTemp=36.4, startTime=31/03/2021 14:46:33, endTime=31/03/2021 14:46:55

...

[03/31/2021 14:46:36]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Property: Received - component="thermostat1", { "targetTemperature": 67°C }.
[03/31/2021 14:46:36]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Property: Update - component="thermostat1", {"targetTemperature": 67 } in °C is InProgress.
[03/31/2021 14:46:49]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Property: Update - component="thermostat1", {"targetTemperature": 67 } in °C is Completed
[03/31/2021 14:46:49]dbug: Microsoft.Azure.Devices.Client.Samples.TemperatureControllerSample[0]
      Telemetry: Sent - component="thermostat1", { "temperature": 67 } in °C.
```
