---
title: Основные сведения о репозитории моделей устройств | Документация Майкрософт
description: Как разработчику решения или ИТ-специалисту вы узнаете об основных понятиях репозитория моделей устройств.
author: rido-min
ms.author: rmpablos
ms.date: 11/17/2020
ms.topic: conceptual
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 33ff96b4e51dbf80bfdb924bc37786a344cdfdc6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104582652"
---
# <a name="device-models-repository"></a>Репозиторий моделей устройств

Репозиторий моделей устройств (ДМР) позволяет построителям устройств управлять моделями устройств IoT самонастраивающийся и предоставлять к ним общий доступ. Модели устройств — это документы JSON LD, определенные с помощью [языка двойников моделирования Digital (дтдл)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).

ДМР определяет шаблон для хранения интерфейсов ДТДЛ в структуре папок на основе идентификатора модели двойникаа устройства (ДТМИ). Для поиска интерфейса в ДМР можно преобразовать ДТМИ в относительный путь. Например, `dtmi:com:example:Thermostat;1` дтми преобразуется в `/dtmi/com/example/thermostat-1.json` .

## <a name="public-device-models-repository"></a>Репозиторий общих моделей устройств

Корпорация Майкрософт размещает общедоступный ДМР со следующими характеристиками:

- Проверенные модели. Корпорация Майкрософт изучает и утверждает все доступные интерфейсы с помощью рабочего процесса проверки запроса на включение внесенных изменений (PR) GitHub.
- Неизменяемости.  После публикации интерфейс не может быть обновлен.
- Масштабирование на основе Hyper-Scale. Корпорация Майкрософт предоставляет необходимую инфраструктуру для создания безопасной масштабируемой конечной точки, в которой можно публиковать и использовать модели устройств.

## <a name="custom-device-models-repository"></a>Хранилище настраиваемых моделей устройств

Используйте тот же шаблон ДМР, чтобы создать пользовательский ДМР на любом носителе, например в локальной файловой системе или на пользовательских веб-серверах HTTP. Вы можете получить модели устройств из пользовательской ДМР так же, как из общедоступной ДМР, изменив базовый URL-адрес, используемый для доступа к ДМР.

> [!NOTE]
> Корпорация Майкрософт предоставляет средства для проверки моделей устройств в общедоступной ДМР. Эти средства можно повторно использовать в пользовательских репозиториях.

## <a name="public-models"></a>Открытые модели

Модели общедоступных устройств, хранящиеся в репозитории моделей, доступны всем пользователям для использования и интеграции в своих приложениях. Модели общедоступных устройств позволяют разработчикам устройств и сборщикам решений совместно использовать и повторно использовать свои модели устройств IoT самонастраивающийся.

Инструкции по публикации модели в репозитории моделей, чтобы сделать ее общедоступной, см. в разделе [Публикация модели](#publish-a-model) .

Пользователи могут просматривать, искать и просматривать открытые интерфейсы из официального [репозитория GitHub](https://github.com/Azure/iot-plugandplay-models).

Все интерфейсы в `dtmi` папках также доступны из общедоступной конечной точки. [https://devicemodels.azure.com](https://devicemodels.azure.com)

### <a name="resolve-models"></a>Разрешение моделей

Для программного доступа к этим интерфейсам можно использовать `ModelsRepositoryClient` Доступные в пакете NuGet [Azure. IOT. моделсрепоситори](https://www.nuget.org/packages/Azure.IoT.ModelsRepository). Этот клиент настроен по умолчанию для запроса общедоступных ДМР, доступных по адресу [devicemodels.Azure.com](https://devicemodels.azure.com/) , и может быть настроен для любого пользовательского репозитория.

Клиент принимает `DTMI` входные данные и возвращает словарь со всеми необходимыми интерфейсами:

```cs
using Azure.IoT.ModelsRepository;

var client = new ModelsRepositoryClient();
IDictionary<string, string> models = client.GetModels("dtmi:com:example:TemperatureController;1");
models.Keys.ToList().ForEach(k => Console.WriteLine(k));
```

Ожидаемый результат должен отображать `DTMI` список из трех интерфейсов, найденных в цепочке зависимостей:

```txt
dtmi:com:example:TemperatureController;1
dtmi:com:example:Thermostat;1
dtmi:azure:DeviceManagement:DeviceInformation;1
```

`ModelsRepositoryClient`Можно настроить для запроса репозитория настраиваемой модели, доступного через HTTP (s), и задать разрешение зависимостей с помощью любого из доступных `ModelDependencyResolution` :

- Отключена. Возвращает только указанный интерфейс без зависимости.
- Включен. Возвращает все интерфейсы в цепочке зависимостей
- Трифромекспандед. Использование `.expanded.json` файла для получения предварительно вычисленных зависимостей 

> [!Tip] 
> Пользовательские репозитории могут не предоставлять `.expanded.json` файл, если он недоступен, клиент будет выполнять обработку каждой зависимости локально.

В следующем примере кода показано, как инициализировать с `ModelsRepositoryClient` помощью базового URL-адреса пользовательского репозитория, в данном случае с использованием `raw` URL-адресов из API GitHub без использования `expanded` формы, так как она недоступна в `raw` конечной точке. `AzureEventSourceListener`Инициализируется для проверки HTTP-запроса, выполняемого клиентом:

```cs
using AzureEventSourceListener listener = AzureEventSourceListener.CreateConsoleLogger();

var client = new ModelsRepositoryClient(
    new Uri("https://raw.githubusercontent.com/Azure/iot-plugandplay-models/main"),
    new ModelsRepositoryClientOptions(dependencyResolution: ModelDependencyResolution.Enabled));

IDictionary<string, string> models = client.GetModels("dtmi:com:example:TemperatureController;1");

models.Keys.ToList().ForEach(k => Console.WriteLine(k));
```

В исходном коде в репозитории Azure SDK GitHub доступны дополнительные примеры: [Azure. IOT. моделсрепоситори/Samples](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/modelsrepository/Azure.IoT.ModelsRepository/samples) .

## <a name="publish-a-model"></a>Публикация модели

> [!Important]
> Для отправки моделей в общедоступную ДМР необходимо иметь учетную запись GitHub.

1. Разветвление общедоступного репозитория GitHub: [https://github.com/Azure/iot-plugandplay-models](https://github.com/Azure/iot-plugandplay-models) .
1. Клонировать разветвленный репозиторий. При необходимости создайте новую ветвь, чтобы изменения были изолированы от `main` ветви.
1. Добавьте новые интерфейсы в папку, `dtmi` используя соглашение о папках и именах файлов. Дополнительные сведения см. в разделе [Импорт модели в `dtmi/` папку](#import-a-model-to-the-dtmi-folder).
1. Проверьте модели локально с помощью `dmr-client` средства. Дополнительные сведения см. в разделе [Проверка моделей](#validate-models).
1. Зафиксируйте изменения локально и отправьте их в вилку.
1. В вилке создайте запрос на вытягивание, предназначенный для `main` ветви. См. статью создание документации по [вопросу или запросу на вытягивание](https://docs.github.com/free-pro-team@latest/desktop/contributing-and-collaborating-using-github-desktop/creating-an-issue-or-pull-request) .
1. Ознакомьтесь с [требованиями к запросу на включение внесенных изменений](https://github.com/Azure/iot-plugandplay-models/blob/main/pr-reqs.md).

Запрос на вытягивание активирует набор действий GitHub, проверяющих отправленные интерфейсы, и гарантирует, что ваш запрос на вытягивание удовлетворяет всем требованиям.

Корпорация Майкрософт будет отвечать на запросы на вытягивание всех проверок в течение трех рабочих дней.

### <a name="dmr-client-tools"></a>`dmr-client` инструментари

Средства, используемые для проверки моделей во время проверки запроса на вытягивание, также можно использовать для добавления и проверки интерфейсов ДТДЛ в локальной среде.

> [!NOTE]
> Для этого средства требуется [пакет SDK для .NET](https://dotnet.microsoft.com/download) версии 3,1 или более поздней.

### <a name="install-dmr-client"></a>установить `dmr-client`;

```bash
curl -L https://aka.ms/install-dmr-client-linux | bash
```

```powershell
iwr https://aka.ms/install-dmr-client-windows -UseBasicParsing | iex
```

### <a name="import-a-model-to-the-dtmi-folder"></a>Импорт модели в `dtmi/` папку

Если модель уже хранится в файлах JSON, можно использовать `dmr-client import` команду, чтобы добавить их в `dtmi/` папку с правильными именами файлов:

```bash
# from the local repo root folder
dmr-client import --model-file "MyThermostat.json"
```

> [!TIP]
> Аргумент можно использовать `--local-repo` для указания корневой папки локального репозитория.

### <a name="validate-models"></a>Проверка моделей

Проверить модели можно с помощью `dmr-client validate` команды:

```bash
dmr-client validate --model-file ./my/model/file.json
```

> [!NOTE]
> Проверка использует последнюю версию средства синтаксического анализа ДТДЛ, чтобы обеспечить совместимость всех интерфейсов с спецификацией языка ДТДЛ.

Чтобы проверить внешние зависимости, они должны существовать в локальном репозитории. Для проверки моделей используйте параметр, `--repo` чтобы указать `local` `remote` папку или для разрешения зависимостей.

```bash
# from the repo root folder
dmr-client validate --model-file ./my/model/file.json --repo .
```

### <a name="strict-validation"></a>Жесткое подтверждение

ДМР включает дополнительные [требования](https://github.com/Azure/iot-plugandplay-models/blob/main/pr-reqs.md). Используйте флаг, `stict` чтобы проверить модель на соответствие следующим требованиям:

```bash
dmr-client validate --model-file ./my/model/file.json --repo . --strict true
```

Проверьте выходные данные консоли на наличие сообщений об ошибках.

### <a name="export-models"></a>Экспорт моделей

Модели можно экспортировать из заданного репозитория (локального или удаленного) в один файл с помощью массива JSON:

```bash
dmr-client export --dtmi "dtmi:com:example:TemperatureController;1" -o TemperatureController.expanded.json
```

## <a name="next-steps"></a>Дальнейшие действия

Следующим шагом является проверка [архитектуры Самонастраивающийся IOT](concepts-architecture.md).
