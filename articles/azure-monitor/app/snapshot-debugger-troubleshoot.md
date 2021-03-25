---
title: Устранение неполадок Application Insights Azure Snapshot Debugger
description: В этой статье представлены действия по устранению неполадок и сведения, помогающие разработчикам включать и использовать Application Insights Snapshot Debugger.
ms.topic: conceptual
author: cweining
ms.author: cweining
ms.date: 03/07/2019
ms.reviewer: mbullwin
ms.openlocfilehash: bd83367ae073e03f03188cdf62cb60faaad7ac97
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105026459"
---
# <a name="troubleshoot-problems-enabling-application-insights-snapshot-debugger-or-viewing-snapshots"></a><a id="troubleshooting"></a> Устранение неполадок, связанных с включением Application Insights Snapshot Debugger или просмотром моментальных снимков
Если вы включили Application Insights Snapshot Debugger для приложения, но не видите моментальные снимки для исключений, эти инструкции можно использовать для устранения неполадок.

Создание моментальных снимков может быть вызвано множеством разных причин. Можно начать с запуска проверки работоспособности моментального снимка, чтобы определить некоторые из возможных распространенных причин.

## <a name="make-sure-youre-using-the-appropriate-snapshot-debugger-endpoint"></a>Убедитесь, что вы используете соответствующую конечную точку Snapshot Debugger

Сейчас только регионы, требующие внесения изменений в конечную точку, — это [Azure для государственных организаций](../../azure-government/compare-azure-government-global-azure.md#application-insights) и [Azure для Китая](/azure/china/resources-developer-guide).

Для службы приложений и приложений, использующих пакет SDK для Application Insights, необходимо обновить строку подключения, используя поддерживаемые переопределения для Snapshot Debugger, как определено ниже.

|Свойство строки подключения    | Облако для государственных организаций США | Облако для Китая |   
|---------------|---------------------|-------------|
|снапшотендпоинт         | `https://snapshot.monitor.azure.us`    | `https://snapshot.monitor.azure.cn` |

Дополнительные сведения о других переопределениях соединений см. в [документации по Application Insights](./sdk-connection-string.md?tabs=net#connection-string-with-explicit-endpoint-overrides).

Для приложение-функция необходимо обновить `host.json` с использованием поддерживаемых переопределений ниже:

|Свойство    | Облако для государственных организаций США | Облако для Китая |   
|---------------|---------------------|-------------|
|ажентендпоинт         | `https://snapshot.monitor.azure.us`    | `https://snapshot.monitor.azure.cn` |

Ниже приведен пример `host.json` обновленной конечной точки облачного агента для государственных организаций США:
```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingExcludedTypes": "Request",
      "samplingSettings": {
        "isEnabled": true
      },
      "snapshotConfiguration": {
        "isEnabled": true,
        "agentEndpoint": "https://snapshot.monitor.azure.us"
      }
    }
  }
}
```

## <a name="use-the-snapshot-health-check"></a>Использование проверки работоспособности моментальных снимков
Из-за некоторых распространенных проблем окно "Открыть моментальный снимок отладки" может не отображаться. Например, из-за использования устаревшего сборщика моментальных снимков, достижения ежедневного лимита отправки или длительного времени передачи моментального снимка. Для устранения распространенных неполадок можно использовать проверку работоспособности моментальных снимков.

В области исключения в представлении сквозной трассировки есть ссылка, которая позволяет перейти к окну проверки работоспособности моментальных снимков.

![Введите данные для проверки работоспособности моментальных снимков](./media/snapshot-debugger/enter-snapshot-health-check.png)

Интерактивный интерфейс, подобный чату, ищет распространенные проблемы и приводит инструкции по их устранению.

![Проверка работоспособности](./media/snapshot-debugger/healthcheck.png)

Если это не устранит проблему, воспользуйтесь следующими шагами по устранению неполадок вручную.

## <a name="verify-the-instrumentation-key"></a>Проверка ключа инструментирования

Убедитесь, что в опубликованном приложении используется правильный ключ инструментирования. Как правило, ключ инструментирования считывается из файла ApplicationInsights.config. Убедитесь, что его значение такое же, что и у ключа инструментирования для ресурса Application Insights, который отображается на портале.

## <a name="check-tlsssl-client-settings-aspnet"></a><a id="SSL"></a>Проверка параметров клиента TLS/SSL (ASP.NET)

Если у вас есть приложение ASP.NET, размещенное в службе приложений Azure или в службах IIS на виртуальной машине, приложение может не подключиться к службе Snapshot Debugger из-за отсутствия протокола безопасности SSL.

[Для конечной точки snapshot Debugger требуется TLS версии 1,2](snapshot-debugger-upgrade.md?toc=/azure/azure-monitor/toc.json). Набор протоколов безопасности SSL — одна из особенностей, включенных в значение httpRuntime targetFramework в разделе System. Web web.config. Если параметр httpRuntime targetFramework имеет значение 4.5.2 или ниже, то TLS 1,2 не включается по умолчанию.

> [!NOTE]
> Значение httpRuntime targetFramework не зависит от целевой платформы, используемой при сборке приложения.

Чтобы проверить этот параметр, откройте файл web.config и найдите раздел System. Web. Убедитесь, что `targetFramework` для параметра для задано `httpRuntime` значение 4,6 или выше.

   ```xml
   <system.web>
      ...
      <httpRuntime targetFramework="4.7.2" />
      ...
   </system.web>
   ```

> [!NOTE]
> Изменение значения httpRuntime targetFramework изменяет особенности среды выполнения, применяемые к приложению, и может привести к другим незначительным изменениям в работе. После внесения этого изменения обязательно протестируйте приложение тщательно. Полный список изменений совместимости см. на https://docs.microsoft.com/dotnet/framework/migration-guide/application-compatibility#retargeting-changes

> [!NOTE]
> Если targetFramework имеет значение 4,7 или выше, Windows определяет доступные протоколы. В службе приложений Azure доступен TLS-1,2. Однако при использовании собственной виртуальной машины может потребоваться включить TLS 1,2 в ОС.

## <a name="preview-versions-of-net-core"></a>Предварительные версии .NET Core
Если вы используете предварительную версию .NET Core или приложение ссылается на Application Insights SDK, прямо или косвенно с помощью зависимой сборки, следуйте инструкциям по [включению snapshot Debugger для других сред](snapshot-debugger-vm.md?toc=/azure/azure-monitor/toc.json).

## <a name="check-the-diagnostic-services-site-extension-status-page"></a>Проверка страницы "состояние" расширения сайта служб диагностики
Если Snapshot Debugger было включено с помощью [панели Application Insights](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json) на портале, оно было включено расширением веб-сайта служб диагностики.

> [!NOTE]
> Установка Application Insights Snapshot Debugger, не поддерживающая код, соответствует политике поддержки .NET Core.
> Дополнительные сведения о поддерживаемых средах выполнения см. в разделе [Политика поддержки .NET Core](https://dotnet.microsoft.com/platform/support/policy/dotnet-core).

Страницу состояния этого расширения можно проверить, перейдя по следующему URL-адресу: `https://{site-name}.scm.azurewebsites.net/DiagnosticServices`

> [!NOTE]
> Домен ссылки на страницу состояния будет зависеть от облака.
Этот домен будет таким же, как и сайт управления KUDU для службы приложений.

На этой странице состояния указывается состояние установки для агентов профилировщика и Snapshot Collector. Если произошла непредвиденная ошибка, она отобразится и покажет, как ее исправить.

Вы можете использовать сайт управления KUDU для службы приложений, чтобы получить базовый URL-адрес этой страницы состояния:
1. Откройте службу приложений на портале Azure.
2. Выберите **Дополнительные инструменты** или выполните поиск по запросу **KUDU**.
3. Выберите **Перейти**.
4. На сайте управления KUDU в URL-адресе **добавьте следующий текст `/DiagnosticServices` и нажмите клавишу ВВОД**.
 Он завершится следующим образом: `https://<kudu-url>/DiagnosticServices`

Отобразится страница состояния, аналогичная приведенной ниже: ![ страница состояние служб диагностики.](./media/diagnostic-services-site-extension/status-page.png)

## <a name="upgrade-to-the-latest-version-of-the-nuget-package"></a>Обновление пакета NuGet до последней версии
В зависимости от того, как была включена Snapshot Debugger, см. следующие параметры:

* Если Snapshot Debugger был включен с помощью [панели Application Insights на портале](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json), то в приложении должен быть уже установлен последний пакет NuGet.

* Если Snapshot Debugger был включен, включив пакет NuGet [Microsoft. ApplicationInsights. SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) , используйте диспетчер пакетов NuGet Visual Studio, чтобы убедиться, что вы используете последнюю версию Microsoft. ApplicationInsights. SnapshotCollector.

Последние обновления и исправления ошибок см. [в заметках о выпуске](./snapshot-collector-release-notes.md).

## <a name="check-the-uploader-logs"></a>Проверьте журналы отправителя

После создания моментального снимка на диске создается файл минидампа (.dmp). Этот файл минидампа и любые связанные PDB-файлы создаются в отдельном процессе передачи в хранилище Snapshot Debugger Application Insights. После успешной передачи минидамп удаляется с диска. Файлы журнала для процесса передачи сохраняются на диске. В среде службы приложений эти журналы можно найти в `D:\Home\LogFiles`. Используйте веб-сайт управления Kudu для службы приложений, чтобы найти эти файлы журнала.

1. Откройте службу приложений на портале Azure.
2. Выберите **Дополнительные инструменты** или выполните поиск по запросу **KUDU**.
3. Выберите **Перейти**.
4. В раскрывающемся списке **Debug console** (Консоль отладки) выберите **CMD** (Команда).
5. Выберите **файлы журнала**.

Вы увидите хотя бы один файл с именем, начинающимся с `Uploader_` или `SnapshotUploader_`, и расширением `.log`. Выберите соответствующий значок, чтобы скачать все файлы журнала или открыть их в браузере.
Имя файла содержит уникальный суффикс, который идентифицирует экземпляр службы приложений. Если экземпляр службы приложений размещен на нескольких компьютерах, для каждого компьютера существуют отдельные файлы журналов. Когда отправитель обнаруживает новый файл минидампа, он записывается в файл журнала. Ниже приведен пример успешного моментального снимка и отправки:

```
SnapshotUploader.exe Information: 0 : Received Fork request ID 139e411a23934dc0b9ea08a626db16c5 from process 6368 (Low pri)
    DateTime=2018-03-09T01:42:41.8571711Z
SnapshotUploader.exe Information: 0 : Creating minidump from Fork request ID 139e411a23934dc0b9ea08a626db16c5 from process 6368 (Low pri)
    DateTime=2018-03-09T01:42:41.8571711Z
SnapshotUploader.exe Information: 0 : Dump placeholder file created: 139e411a23934dc0b9ea08a626db16c5.dm_
    DateTime=2018-03-09T01:42:41.8728496Z
SnapshotUploader.exe Information: 0 : Dump available 139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:45.7525022Z
SnapshotUploader.exe Information: 0 : Successfully wrote minidump to D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:45.7681360Z
SnapshotUploader.exe Information: 0 : Uploading D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp, 214.42 MB (uncompressed)
    DateTime=2018-03-09T01:42:45.7681360Z
SnapshotUploader.exe Information: 0 : Upload successful. Compressed size 86.56 MB
    DateTime=2018-03-09T01:42:59.6184651Z
SnapshotUploader.exe Information: 0 : Extracting PDB info from D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp.
    DateTime=2018-03-09T01:42:59.6184651Z
SnapshotUploader.exe Information: 0 : Matched 2 PDB(s) with local files.
    DateTime=2018-03-09T01:42:59.6809606Z
SnapshotUploader.exe Information: 0 : Stamp does not want any of our matched PDBs.
    DateTime=2018-03-09T01:42:59.8059929Z
SnapshotUploader.exe Information: 0 : Deleted D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\139e411a23934dc0b9ea08a626db16c5.dmp
    DateTime=2018-03-09T01:42:59.8530649Z
```

> [!NOTE]
> Приведенный выше пример взят из пакета NuGet Microsoft.ApplicationInsights.SnapshotCollector версии 1.2.0. В более ранних версиях процесс передачи называется `MinidumpUploader.exe`, а журнал содержит менее подробные сведения.

В предыдущем примере, ключ инструментирования — это `c12a605e73c44346a984e00000000000`. Это значение должно соответствовать ключу инструментирования для вашего приложения.
Минидамп связан с моментальным снимком с идентификатором `139e411a23934dc0b9ea08a626db16c5`. Этот идентификатор можно использовать позже для нахождение связанной записи об исключении в Application Insights Analytics.

Отправитель проверяет наличие новых PDB-файлов примерно один раз каждые 15 минут. Приведем пример:

```
SnapshotUploader.exe Information: 0 : PDB rescan requested.
    DateTime=2018-03-09T01:47:19.4457768Z
SnapshotUploader.exe Information: 0 : Scanning D:\home\site\wwwroot for local PDBs.
    DateTime=2018-03-09T01:47:19.4457768Z
SnapshotUploader.exe Information: 0 : Local PDB scan complete. Found 2 PDB(s).
    DateTime=2018-03-09T01:47:19.4614027Z
SnapshotUploader.exe Information: 0 : Deleted PDB scan marker : D:\local\Temp\Dumps\c12a605e73c44346a984e00000000000\6368.pdbscan
    DateTime=2018-03-09T01:47:19.4614027Z
```

Для приложений, которые _не_ размещаются в службе приложений, журналы отправителя находятся в той же папке, что и минидампы: `%TEMP%\Dumps\<ikey>` (где `<ikey>` — ваш ключ инструментирования).

## <a name="troubleshooting-cloud-services"></a>Устранение неполадок с облачными службами
В облачных службах временная папка по умолчанию может быть слишком маленькой для хранения файлов минидампа, что приведет к потере моментальных снимков.

Необходимый размер зависит от общего рабочего набора приложения и количества параллельных моментальных снимков.

Как правило, рабочий набор веб-роли 32-разрядного приложения ASP.NET может занимать от 200 МБ до 500 МБ. Предусмотрите место хотя бы для двух параллельных моментальных снимков.

Например, если приложение использует 1 ГБ общего рабочего набора, убедитесь, что для хранения моментальных снимков требуется не менее 2 ГБ дискового пространства.

Выполните следующие действия, чтобы настроить роль облачной службы с выделенным локальным ресурсом для моментальных снимков.

1. Добавьте новый локальный ресурс в облачную службу, изменив файл ее определения (.csdef). В следующем примере определяется ресурс с именем `SnapshotStore` и размером 5 ГБ.
   ```xml
   <LocalResources>
     <LocalStorage name="SnapshotStore" cleanOnRoleRecycle="false" sizeInMB="5120" />
   </LocalResources>
   ```

2. Измените код запуска роли, чтобы добавить переменную среды, указывающую на локальный ресурс `SnapshotStore`. Для рабочих ролей код необходимо добавить в метод `OnStart` нужной роли:
   ```csharp
   public override bool OnStart()
   {
       Environment.SetEnvironmentVariable("SNAPSHOTSTORE", RoleEnvironment.GetLocalResource("SnapshotStore").RootPath);
       return base.OnStart();
   }
   ```
   Для веб-ролей (ASP.NET) код необходимо добавить в метод `Application_Start` веб-приложения:
   ```csharp
   using Microsoft.WindowsAzure.ServiceRuntime;
   using System;

   namespace MyWebRoleApp
   {
       public class MyMvcApplication : System.Web.HttpApplication
       {
          protected void Application_Start()
          {
             Environment.SetEnvironmentVariable("SNAPSHOTSTORE", RoleEnvironment.GetLocalResource("SnapshotStore").RootPath);
             // TODO: The rest of your application startup code
          }
       }
   }
   ```

3. Обновите файл ApplicationInsights.config для роли, чтобы переопределить размещение временной папки, которое используется в `SnapshotCollector`.
   ```xml
   <TelemetryProcessors>
    <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
      <!-- Use the SnapshotStore local resource for snapshots -->
      <TempFolder>%SNAPSHOTSTORE%</TempFolder>
      <!-- Other SnapshotCollector configuration options -->
    </Add>
   </TelemetryProcessors>
   ```

## <a name="overriding-the-shadow-copy-folder"></a>Переопределение папки теневой копии

При запуске сборщик Snapshot Collector пытается найти папку на диске, который подходит для выполнения процесса передачи моментальных снимков. Выбранная папка называется папкой теневой копии.

Snapshot Collector проверяет несколько известных расположений, гарантируя наличие прав на копирование двоичных файлов отправителя моментальных снимков. Используются следующие переменные среды:
- Fabric_Folder_App_Temp
- LOCALAPPDATA
- APPDATA
- TEMP

Если подходящая папка не найдена, Snapshot Collector сообщает об ошибке _"не удалось найти подходящую папку теневых копий"._

Если копирование завершается сбоем, Snapshot Collector выдает ошибку `ShadowCopyFailed`.

Если не удается запустить отправитель, Snapshot Collector выдает ошибку `UploaderCannotStartFromShadowCopy`. Текст сообщения часто содержит исключение `System.UnauthorizedAccessException`. Эта ошибка обычно возникает из-за того, что приложение работает от имени учетной записи с ограниченными разрешениями. У учетной записи есть разрешение на запись в папку теневой копии, но нет разрешения на выполнение кода.

Так как эти ошибки обычно возникают во время запуска, за ними следует ошибка `ExceptionDuringConnect` — _Uploader failed to start_ (Не удалось запустить отправитель).

Чтобы устранить эти ошибки, вы можете указать папку теневой копии вручную с помощью параметра конфигурации `ShadowCopyFolder`. Например, в файле ApplicationInsights.config:

   ```xml
   <TelemetryProcessors>
    <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
      <!-- Override the default shadow copy folder. -->
      <ShadowCopyFolder>D:\SnapshotUploader</ShadowCopyFolder>
      <!-- Other SnapshotCollector configuration options -->
    </Add>
   </TelemetryProcessors>
   ```

Или если вы используете файл appsettings.json с приложением .NET Core:

   ```json
   {
     "ApplicationInsights": {
       "InstrumentationKey": "<your instrumentation key>"
     },
     "SnapshotCollectorConfiguration": {
       "ShadowCopyFolder": "D:\\SnapshotUploader"
     }
   }
   ```

## <a name="use-application-insights-search-to-find-exceptions-with-snapshots"></a>Поиск исключений с моментальными снимками с помощью поиска Application Insights

При создании моментального снимка вызванное исключение обозначается идентификатором моментального снимка. Этот идентификатор моментального снимка включается в качестве пользовательского свойства, когда в сообщение об исключении передается Application Insights. Используя **Поиск** в Application Insights, можно найти все записи с `ai.snapshot.id` пользовательским свойством.

1. Перейдите к ресурсу Application Insights на портале Azure.
2. Выберите **Search** (Поиск).
3. В текстовом поле поиска введите `ai.snapshot.id` и нажмите клавишу ВВОД.

![Поиск телеметрии с помощью идентификатора моментального снимка на портале](./media/snapshot-debugger/search-snapshot-portal.png)

Если поиск не дал результатов, то для Application Insights в выбранном диапазоне времени не было отправлено ни одного моментального снимка.

Чтобы найти определенный идентификатор снимка из журналов отправителя, введите этот идентификатор в поле поиска. Если не удается найти записи для моментального снимка, который вы загрузили, выполните следующие действия.

1. Еще раз проверьте, что вы смотрите на правильный ресурс Application Insights при проверке ключа инструментирования.

2. С помощью метки времени из журнала отправителя, настройте фильтр диапазона времени поиска так, чтобы охватить этот диапазон времени.

Если вы по-прежнему не видите исключение с ИДЕНТИФИКАТОРом моментального снимка, то запись об исключении не передавалась Application Insights. Такая ситуация может возникнуть в случае сбоя приложения после создания моментального снимка, но до получения сообщения об исключении. В этом случае проверьте журналы службы приложений в `Diagnose and solve problems`, чтобы проверить, были ли неожиданные перезагрузки или необработанные исключения.

## <a name="edit-network-proxy-or-firewall-rules"></a>Изменение правил прокси-сервера сети или брандмауэра

Если приложение подключается к Интернету через прокси-сервер или брандмауэр, может потребоваться обновить правила для взаимодействия со службой Snapshot Debugger.

IP-адреса, используемые Snapshot Debugger Application Insights, включены в тег службы Azure Monitor. Дополнительные сведения см. в [документации по тегам служб](../../virtual-network/service-tags-overview.md).