---
title: Общие сведения о средстве синтаксического анализа модели Digital двойников | Документация Майкрософт
description: Как разработчику, Узнайте, как использовать средство синтаксического анализа ДТДЛ для проверки моделей.
author: rido-min
ms.author: rmpablos
ms.date: 10/21/2020
ms.topic: conceptual
ms.custom: mvc
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: d68abe8548dac3306228683e4b6ce8935a248ebc
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92331793"
---
# <a name="understand-the-digital-twins-model-parser"></a>Общие сведения о средстве синтаксического анализа модели цифровых двойников

Язык определения цифровых двойников (ДТДЛ) описан в [спецификации дтдл](https://github.com/Azure/opendigitaltwins-dtdl). Пользователи могут использовать пакет NuGet _средства синтаксического анализа цифровых двойников Model_ для проверки и запроса модели, определенной в нескольких файлах.

## <a name="install-the-dtdl-model-parser"></a>Установка средства синтаксического анализа модели ДТДЛ

Средство синтаксического анализа доступно в NuGet.org с ИДЕНТИФИКАТОРом: [Microsoft. Azure. дигиталтвинс. Parser](https://www.nuget.org/packages/Microsoft.Azure.DigitalTwins.Parser). Чтобы установить средство синтаксического анализа, используйте любой совместимый диспетчер пакетов NuGet, такой как в Visual Studio или в интерфейсе `dotnet` командной строки.

```bash
dotnet add package Microsoft.Azure.DigitalTwins.Parser
```

> [!NOTE]
> На момент написания статьи версия средства синтаксического анализа имеет значение `3.12.5` .

## <a name="use-the-parser-to-validate-a-model"></a>Проверка модели с помощью средства синтаксического анализа

Модель может состоять из одного или нескольких интерфейсов, описанных в JSON-файлах. Вы можете использовать средство синтаксического анализа для загрузки всех файлов в заданной папке и использовать средство синтаксического анализа для проверки всех файлов в целом, включая ссылки между файлами:

1. Создать `IEnumerable<string>` со списком всех содержимого модели:

    ```csharp
    using System.IO;

    string folder = @"c:\myModels\";
    string filespec = "*.json";

    List<string> modelJson = new List<string>();
    foreach (string filename in Directory.GetFiles(folder, filespec))
    {
        using StreamReader modelReader = new StreamReader(filename);
        modelJson.Add(modelReader.ReadToEnd());
    }
    ```

1. Создайте экземпляр `ModelParser` вызова и `ParseAsync` :

    ```csharp
    using Microsoft.Azure.DigitalTwins.Parser;

    ModelParser modelParser = new ModelParser();
    IReadOnlyDictionary<Dtmi, DTEntityInfo> parseResult = await modelParser.ParseAsync(modelJson);
    ```

1. Проверка на наличие ошибок проверки. Если средство синтаксического анализа обнаруживает ошибки, оно создает исключение `ParsingException` со списком ошибок:

    ```csharp
    try
    {
        IReadOnlyDictionary<Dtmi, DTEntityInfo> parseResult = await modelParser.ParseAsync(modelJson);
    }
    catch (ParsingException pex)
    {
        Console.WriteLine(pex.Message);
        foreach (var err in pex.Errors)
        {
            Console.WriteLine(err.PrimaryID);
            Console.WriteLine(err.Message);
        }
    }
    ```

1. Проверьте `Model` . Если проверка прошла, можно использовать API анализатора модели для проверки модели. В следующем фрагменте кода показано, как выполнить итерацию всех проанализированных моделей и отобразить существующие свойства.

    ```csharp
    foreach (var item in parseResult)
    {
        Console.WriteLine($"\t{item.Key}");
        Console.WriteLine($"\t{item.Value.DisplayName?.Values.FirstOrDefault()}");
    }
    ```

## <a name="next-steps"></a>Дальнейшие действия

API анализатора модели, рассмотренный в этой статье, включает множество сценариев для автоматизации или проверки задач, зависящих от моделей ДТДЛ. Например, можно динамически создать пользовательский интерфейс на основе информации в модели.
