---
title: Как использовать систему диагностики Azure (.NET) с облачными службами (классическая модель) | Документация Майкрософт
description: Сбор данных облачных служб Azure с помощью системы диагностики Azure для отладки, оценки производительности, мониторинга, анализа трафика и многого другого.
ms.topic: article
ms.service: cloud-services
ms.date: 10/14/2020
ms.author: tagore
author: tanmaygore
ms.reviewer: mimckitt
ms.custom: ''
ms.openlocfilehash: 97e68d338580132b6927c4cc8b206db60fe93ba2
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101703513"
---
# <a name="enabling-azure-diagnostics-in-azure-cloud-services-classic"></a>Включение система диагностики Azure в облачных службах Azure (классическая модель)

> [!IMPORTANT]
> [Облачные службы Azure (Расширенная поддержка)](../cloud-services-extended-support/overview.md) — это новая модель развертывания на основе Azure Resource Manager для продукта облачных служб Azure.После этого изменения облачные службы Azure, работающие в модели развертывания на основе Service Manager Azure, были переименованы как облачные службы (классические), и все новые развертывания должны использовать [облачные службы (Расширенная поддержка)](../cloud-services-extended-support/overview.md).

Основные сведения о системе диагностики Azure см. в [обзоре системы диагностики Azure](../azure-monitor/agents/diagnostics-extension-overview.md).

## <a name="how-to-enable-diagnostics-in-a-worker-role"></a>Как включить диагностику в рабочей роли
В этом пошаговом руководстве описывается, как реализовать рабочую роль Azure, которая передает данные телеметрии с помощью класса EventSource .NET. Система диагностики Azure используется для сбора данных телеметрии и хранения их в учетной записи хранения Azure. При создании рабочей роли Visual Studio автоматически включает систему диагностики 1.0 как часть решения в пакетах SDK Azure для .NET версии 2.4 или более поздней. В следующих указаниях описывается процесс создания рабочей роли, отключение системы диагностики 1.0 в решении и развертывание системы диагностики 1.2 или 1.3 в рабочей роли.

### <a name="prerequisites"></a>Предварительные условия
В данной статье предполагается, что у вас есть подписка Azure и вы используете Visual Studio с пакетом SDK для Azure. Если у вас нет подписки Azure, можно зарегистрироваться для получения [бесплатной пробной версии][Free Trial]. Следует обязательно [установить и настроить Azure PowerShell версии 0.8.7 или более поздней][Install and configure Azure PowerShell version 0.8.7 or later].

### <a name="step-1-create-a-worker-role"></a>Шаг 1. Создание рабочей роли
1. Запустите **Visual Studio**.
2. Создайте проект **облачной службы Azure** из шаблона **Облако**, предназначенного для .NET Framework 4.5.  Назовите проект WadExample и нажмите кнопку «ОК».
3. Выберите **рабочую роль** и нажмите кнопку «ОК». После этих действий будет создан новый проект.
4. В **обозревателе решений** дважды щелкните файл свойств **WorkerRole1**.
5. На вкладке **Настройка** снимите флажок **Включить диагностику**, чтобы отключить систему диагностики Diagnostics 1.0 (пакет Azure SDK 2.4 и предыдущих версий).
6. Создайте свое решение, чтобы проверить, что в нем нет ошибок.

### <a name="step-2-instrument-your-code"></a>Шаг 2. Инструментирование кода
Замените содержимое файла WorkerRole.cs следующим кодом. Класс SampleEventSourceWriter, наследуемый от [класса EventSource][EventSource Class], реализует четыре метода ведения журнала: **SendEnums**, **MessageMethod**, **SetOther** и **HighFreq**. Первый параметр для метода **WriteEvent** определяет идентификатор соответствующего события. Метод Run реализует бесконечный цикл, который вызывает каждый из методов ведения журнала, реализованных в классе **SampleEventSourceWriter** , каждые 10 секунд.

```csharp
using Microsoft.WindowsAzure.ServiceRuntime;
using System;
using System.Diagnostics;
using System.Diagnostics.Tracing;
using System.Net;
using System.Threading;

namespace WorkerRole1
{
    sealed class SampleEventSourceWriter : EventSource
    {
        public static SampleEventSourceWriter Log = new SampleEventSourceWriter();
        public void SendEnums(MyColor color, MyFlags flags) { if (IsEnabled())  WriteEvent(1, (int)color, (int)flags); }// Cast enums to int for efficient logging.
        public void MessageMethod(string Message) { if (IsEnabled())  WriteEvent(2, Message); }
        public void SetOther(bool flag, int myInt) { if (IsEnabled())  WriteEvent(3, flag, myInt); }
        public void HighFreq(int value) { if (IsEnabled()) WriteEvent(4, value); }

    }

    enum MyColor
    {
        Red,
        Blue,
        Green
    }

    [Flags]
    enum MyFlags
    {
        Flag1 = 1,
        Flag2 = 2,
        Flag3 = 4
    }

    public class WorkerRole : RoleEntryPoint
    {
        public override void Run()
        {
            // This is a sample worker implementation. Replace with your logic.
            Trace.TraceInformation("WorkerRole1 entry point called");

            int value = 0;

            while (true)
            {
                Thread.Sleep(10000);
                Trace.TraceInformation("Working");

                // Emit several events every time we go through the loop
                for (int i = 0; i < 6; i++)
                {
                    SampleEventSourceWriter.Log.SendEnums(MyColor.Blue, MyFlags.Flag2 | MyFlags.Flag3);
                }

                for (int i = 0; i < 3; i++)
                {
                    SampleEventSourceWriter.Log.MessageMethod("This is a message.");
                    SampleEventSourceWriter.Log.SetOther(true, 123456789);
                }

                if (value == int.MaxValue) value = 0;
                SampleEventSourceWriter.Log.HighFreq(value++);
            }
        }

        public override bool OnStart()
        {
            // Set the maximum number of concurrent connections
            ServicePointManager.DefaultConnectionLimit = 12;

            // For information on handling configuration changes
            // see the MSDN topic at https://go.microsoft.com/fwlink/?LinkId=166357.

            return base.OnStart();
        }
    }
}
```


### <a name="step-3-deploy-your-worker-role"></a>Шаг 3. Развертывание рабочей роли

[!INCLUDE [cloud-services-wad-warning](../../includes/cloud-services-wad-warning.md)]

1. Разверните свою рабочую роль в Azure из Visual Studio, выбрав проект **WadExample** в обозревателе решений, а затем выбрав команду **Опубликовать** в меню **Сборка**.
2. Выберите свою подписку.
3. В диалоговом окне **Microsoft Azure параметры публикации** выберите **создать...**.
4. В диалоговом окне **Создание облачной службы и учетной записи хранилища** введите **имя** (например, WadExample) и выберите регион или территориальную группу.
5. Выберите для параметра **Среда** значение **Промежуточная среда**.
6. Измените любые другие **Параметры** по необходимости и щелкните **Опубликовать**.
7. После завершения развертывания на портале Azure должно быть указано, что облачная служба находится в состоянии **Выполняется**.

### <a name="step-4-create-your-diagnostics-configuration-file-and-install-the-extension"></a>Шаг 4. Создание файла конфигурации системы диагностики и установка расширения
1. Скачайте общедоступное определение схемы файла конфигурации, выполнив следующую команду PowerShell:

    ```powershell
    (Get-AzureServiceAvailableExtension -ExtensionName 'PaaSDiagnostics' -ProviderNamespace 'Microsoft.Azure.Diagnostics').PublicConfigurationSchema | Out-File -Encoding utf8 -FilePath 'WadConfig.xsd'
    ```
2. Добавьте XML-файл в свой проект **WorkerRole1**, щелкнув правой кнопкой мыши проект **WorkerRole1** и выбрав **Добавить** -> **Создать элемент…**. -> **Элементы**  ->  Visual C# **Данные**  ->  **XML-файл**. Назовите файл «WadExample.xml».

   ![CloudServices_diag_add_xml](./media/cloud-services-dotnet-diagnostics/AddXmlFile.png)
3. Свяжите файл WadConfig.xsd с файлом конфигурации. Убедитесь, что окно редактора WadExample.xml активно. Нажмите клавишу **F4** , чтобы открыть окно " **свойства** ". Щелкните свойство **Схемы** в окне **Свойства**. Нажмите кнопку **...** in the **Schemas** . Нажмите кнопку **Добавить...** , перейдите в расположение, где сохранен XSD-файл, и выберите файл WadConfig.xsd. Нажмите кнопку **OK**.

4. Замените содержимое файла настройки WadExample.xml приведенным кодом XML и сохраните файл. Этот файл конфигурации определяет пару счетчиков производительности: один — для использование ЦП, и один — для использования памяти. Затем конфигурация определяет четыре события, соответствующие методам в классе SampleEventSourceWriter.

```xml
<?xml version="1.0" encoding="utf-8"?>
<PublicConfig xmlns="http://schemas.microsoft.com/ServiceHosting/2010/10/DiagnosticsConfiguration">
  <WadCfg>
    <DiagnosticMonitorConfiguration overallQuotaInMB="25000">
      <PerformanceCounters scheduledTransferPeriod="PT1M">
        <PerformanceCounterConfiguration counterSpecifier="\Processor(_Total)\% Processor Time" sampleRate="PT1M" unit="percent" />
        <PerformanceCounterConfiguration counterSpecifier="\Memory\Committed Bytes" sampleRate="PT1M" unit="bytes"/>
      </PerformanceCounters>
      <EtwProviders>
        <EtwEventSourceProviderConfiguration provider="SampleEventSourceWriter" scheduledTransferPeriod="PT5M">
          <Event id="1" eventDestination="EnumsTable"/>
          <Event id="2" eventDestination="MessageTable"/>
          <Event id="3" eventDestination="SetOtherTable"/>
          <Event id="4" eventDestination="HighFreqTable"/>
          <DefaultEvents eventDestination="DefaultTable" />
        </EtwEventSourceProviderConfiguration>
      </EtwProviders>
    </DiagnosticMonitorConfiguration>
  </WadCfg>
</PublicConfig>
```

### <a name="step-5-install-diagnostics-on-your-worker-role"></a>Шаг 5. Установка системы диагностики в рабочей роли
Командлеты PowerShell для управления диагностикой в веб-роли или в рабочей роли: Set-AzureServiceDiagnosticsExtension, Get-AzureServiceDiagnosticsExtension и Remove-AzureServiceDiagnosticsExtension.

1. Откройте Azure PowerShell.
2. Выполните сценарий для установки системы диагностики в рабочую роль (замените *StorageAccountKey* ключом для своей учетной записи хранения wadexample, а *config_path* — путем к файлу *WadExample.xml*).

```powershell
$storage_name = "wadexample"
$key = "<StorageAccountKey>"
$config_path="c:\users\<user>\documents\visual studio 2013\Projects\WadExample\WorkerRole1\WadExample.xml"
$service_name="wadexample"
$storageContext = New-AzureStorageContext -StorageAccountName $storage_name -StorageAccountKey $key
Set-AzureServiceDiagnosticsExtension -StorageContext $storageContext -DiagnosticsConfigurationPath $config_path -ServiceName $service_name -Slot Staging -Role WorkerRole1
```

### <a name="step-6-look-at-your-telemetry-data"></a>Шаг 6. Просмотр данных телеметрии
В **обозревателе решений** Visual Studio перейдите к учетной записи хранения wadexample. После того как облачная служба проработает около пяти (5) минут, следует просмотреть таблицы **WADEnumsTable**, **WADHighFreqTable**, **WADMessageTable**, **WADPerformanceCountersTable** и **WADSetOtherTable**. Дважды щелкните одну из таблиц, чтобы просмотреть собранные данные телеметрии.

![CloudServices_diag_tables](./media/cloud-services-dotnet-diagnostics/WadExampleTables.png)

## <a name="configuration-file-schema"></a>Схема файла конфигурации
Файл конфигурации системы диагностики определяет значения, которые используются для инициализации параметров конфигурации диагностики, когда запускается агент диагностики. Допустимые значения и примеры см. в [последнем справочнике по схеме](../azure-monitor/agents/diagnostics-extension-versions.md).

## <a name="troubleshooting"></a>Устранение неполадок
Если возникли проблемы, см. сведения в статье [Устранение неполадок с помощью системы диагностики Azure](../azure-monitor/agents/diagnostics-extension-troubleshooting.md). Это поможет вам устранить распространенные проблемы.

## <a name="next-steps"></a>Дальнейшие действия
[См. перечень статей о системе диагностики виртуальных машин Azure](../azure-monitor/agents/diagnostics-extension-overview.md), чтобы изменить данные, которые вы собираете, устранить неполадки или больше узнать о диагностике в целом.

[EventSource Class]: /dotnet/api/system.diagnostics.tracing.eventsource

[Debugging an Azure Application]: https://msdn.microsoft.com/library/windowsazure/ee405479.aspx   
[Collect Logging Data by Using Azure Diagnostics]: /previous-versions/azure/gg433048(v=azure.100)
[Free Trial]: https://azure.microsoft.com/pricing/free-trial/
[Install and configure Azure PowerShell version 0.8.7 or later]: /powershell/azure/