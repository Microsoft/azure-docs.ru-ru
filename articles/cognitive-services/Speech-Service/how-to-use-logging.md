---
title: Ведение журнала пакета речевого SDK — служба речи
titleSuffix: Azure Cognitive Services
description: Узнайте, как включить ведение журнала в речевом пакете SDK (C++, C#, Python, цель-C, Java).
services: cognitive-services
author: amitkumarshukla
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: amishu
ms.custom: devx-track-js, devx-track-csharp
ms.openlocfilehash: f044596cf084e68b6c9ca10e3da4fbdf4c8c2062
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026528"
---
# <a name="enable-logging-in-the-speech-sdk"></a>Включение ведения журнала в пакете SDK для распознавания речи

Ведение журнала в файл является необязательным компонентом для речевого пакета SDK. Во время ведения журнала разработки предоставляет дополнительные сведения и средства диагностики из основных компонентов SDK для распознавания речи. Его можно включить, задав свойству `Speech_LogFilename` в объекте конфигурации речи расположение и имя файла журнала. Ведение журнала будет активировано глобально после создания распознавателя из этой конфигурации и не может быть отключено позже. Нельзя изменить имя файла журнала во время выполнения сеанса ведения журнала.

> [!NOTE]
> Ведение журнала доступно с момента использования пакета SDK для распознавания речи версии 1.4.0 во всех поддерживаемых языках программирования SDK для обработки речи, за исключением JavaScript.

## <a name="sample"></a>Образец

Имя файла журнала указано в объекте конфигурации. Используя в `SpeechConfig` качестве примера, предположим, что вы создали экземпляр с именем `config` :

```csharp
config.SetProperty(PropertyId.Speech_LogFilename, "LogfilePathAndName");
```

```java
config.setProperty(PropertyId.Speech_LogFilename, "LogfilePathAndName");
```

```C++
config->SetProperty(PropertyId::Speech_LogFilename, "LogfilePathAndName");
```

```Python
config.set_property(speechsdk.PropertyId.Speech_LogFilename, "LogfilePathAndName")
```

```objc
[config setPropertyTo:@"LogfilePathAndName" byId:SPXSpeechLogFilename];
```

Вы можете создать распознаватель из объекта config. Это позволит включить ведение журнала для всех распознавателей.

> [!NOTE]
> Если создать объект `SpeechSynthesizer` из объекта конфигурации, ведение журнала не будет включено. Если ведение журнала включено, то будут также получены диагностические сообщения из `SpeechSynthesizer` .

## <a name="create-a-log-file-on-different-platforms"></a>Создание файла журнала на разных платформах

Для Windows или Linux файл журнала может находиться в любом пути, для которого у пользователя есть разрешение на запись. Разрешения на запись в расположения файловой системы в других операционных системах могут быть ограничены или ограничены по умолчанию.

### <a name="universal-windows-platform-uwp"></a>Универсальная платформа Windows (UWP)

Приложения UWP должны поместить файлы журнала в одном из расположений данных приложений (локальных, перемещаемых или временных). Файл журнала можно создать в папке локального приложения:

```csharp
StorageFolder storageFolder = ApplicationData.Current.LocalFolder;
StorageFile logFile = await storageFolder.CreateFileAsync("logfile.txt", CreationCollisionOption.ReplaceExisting);
config.SetProperty(PropertyId.Speech_LogFilename, logFile.Path);
```

Дополнительные сведения о разрешении на доступ к файлам для приложений UWP можно найти [здесь](/windows/uwp/files/file-access-permissions).

### <a name="android"></a>Android

Файл журнала можно сохранить во внутреннем хранилище, во внешнем хранилище или в каталоге кэша. Файлы, созданные во внутреннем хранилище или в каталоге кэша, являются частными для приложения. Лучше создать файл журнала во внешнем хранилище.

```java
File dir = context.getExternalFilesDir(null);
File logFile = new File(dir, "logfile.txt");
config.setProperty(PropertyId.Speech_LogFilename, logFile.getAbsolutePath());
```

Приведенный выше код сохранит файл журнала во внешнем хранилище в корне каталога конкретного приложения. Пользователь может получить доступ к файлу с помощью диспетчера файлов (обычно в `Android/data/ApplicationName/logfile.txt` ). Файл будет удален при удалении приложения.

Кроме того, необходимо запросить `WRITE_EXTERNAL_STORAGE` разрешение в файле манифеста:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="...">
  ...
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  ...
</manifest>
```

Дополнительные сведения о данных и хранилище файлов для приложений Android доступны [здесь](https://developer.android.com/guide/topics/data/data-storage.html).

#### <a name="ios"></a>iOS

Доступны только каталоги в изолированной среде приложения. Файлы можно создавать в каталогах Documents, Library и TEMP. Файлы в каталоге документов можно сделать доступными для пользователя. В следующем фрагменте кода показано создание файла журнала в каталоге документа приложения:

```objc
NSString *filePath = [
  [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject]
    stringByAppendingPathComponent:@"logfile.txt"];
[speechConfig setPropertyTo:filePath byId:SPXSpeechLogFilename];
```

Чтобы получить доступ к созданному файлу, добавьте приведенные ниже свойства в `Info.plist` список свойств приложения.

```xml
<key>UIFileSharingEnabled</key>
<true/>
<key>LSSupportsOpeningDocumentsInPlace</key>
<true/>
```

Дополнительные сведения о файловой системе iOS можно найти [здесь](https://developer.apple.com/library/archive/documentation/FileManagement/Conceptual/FileSystemProgrammingGuide/FileSystemOverview/FileSystemOverview.html).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Примеры на сайте GitHub](https://aka.ms/csspeech/samples)