---
title: Чтение входных данных в любом формате с помощью пользовательских десериализаторов .NET в Azure Stream Analytics
description: В этой статье описывается формат сериализации и интерфейсы, определяющие пользовательские десериализаторы .NET для облака Azure Stream Analytics облачных и пограничных заданий.
author: sidramadoss
ms.author: sidram
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 1/28/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 140a836882ad3abe048047120e4fe1ebc0a3067c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98018162"
---
# <a name="read-input-in-any-format-using-net-custom-deserializers"></a>Считывание входных данных любого формата с помощью пользовательских десериализаторов .NET

Пользовательские десериализаторы .NET позволяют заданию Azure Stream Analytics считывать данные из форматов, не входящих в три [встроенных формата данных](stream-analytics-parsing-json.md). В этой статье описывается формат сериализации и интерфейсы, определяющие пользовательские десериализаторы .NET для облака Azure Stream Analytics облачных и пограничных заданий. Также приведены примеры десериализаторов для буфера протокола и формата CSV.

## <a name="net-custom-deserializer"></a>Создание пользовательского десериализатора .NET

Следующие примеры кода являются интерфейсами, определяющими пользовательский десериализатор и реализующими `StreamDeserializer<T>`.

`UserDefinedOperator` — базовый класс для всех операторов настраиваемой потоковой передачи. Он инициализирует `StreamingContext`, который предоставляет контекст, включающий механизм для публикации диагностики, для которого необходимо выполнить отладку любых проблем с десериализатором.

```csharp
    public abstract class UserDefinedOperator
    {
        public abstract void Initialize(StreamingContext streamingContext);
    }
```

В следующем фрагменте кода показана десериализация для потоковой передачи данных. 

Ошибки, которые можно пропустить, следует выдавать с помощью `IStreamingDiagnostics`, передаваемого методом инициализации `UserDefinedOperator`. Все исключения будут рассматриваться как ошибки, и десериализатор будет создан повторно. После определенного числа ошибок задание перейдет в состояние сбоя.

`StreamDeserializer<T>` десериализует поток в объект типа `T`. Должны быть выполнены следующие условия.

1. T является классом или структурой.
1. Все открытые поля в T являются либо
    1. одним из полей [sbyte, byte, short, ushort, int, uint, long, DateTime, string, float, double], либо их эквивалентами, допускающими значение NULL.
    1. Другая структура или класс, следующие тем же правилам.
    1. Массив типа `T2`, который следует тем же правилам.
    1. IList`T2`, где T2 следует тем же правилам.
    1. Не имеет рекурсивных типов.

Параметр `stream` — это поток, содержащий сериализованный объект. `Deserialize` возвращает коллекцию экземпляров `T`.

```csharp
    public abstract class StreamDeserializer<T> : UserDefinedOperator
    {
        public abstract IEnumerable<T> Deserialize(Stream stream);
    }
```

`StreamingContext` предоставляет контекст, который включает в себя механизм публикации диагностики для оператора пользователя.

```csharp
    public abstract class StreamingContext
    {
        public abstract StreamingDiagnostics Diagnostics { get; }
    }
```

`StreamingDiagnostics` — это диагностика для определяемых пользователем операторов, включающая сериализатор, десериализатор и определяемые пользователем функции.

`WriteError` записывает сообщение об ошибке в журналы ресурсов и отправляет ошибку диагностике.

`briefMessage` — краткое сообщение об ошибке. Это сообщение отображается в диагностике и используется группой разработки продукта для отладки. Не включайте в него конфиденциальную информацию. Сообщение должно быть короче 200 символов.

`detailedMessage` — подробное сообщение об ошибке, которое добавляется только в журналы ресурсов в хранилище. Это сообщение должно быть короче 2000 символов.

```csharp
    public abstract class StreamingDiagnostics
    {
        public abstract void WriteError(string briefMessage, string detailedMessage);
    }
```

## <a name="deserializer-examples"></a>Примеры десериализатора

В этом разделе показано, как создавать пользовательские десериализаторы для Protobuf и CSV. Дополнительные примеры, например формат AVRO для записи в концентратор событий, можно найти в [Azure Stream Analytics на сайте GitHub](https://github.com/Azure/azure-stream-analytics/tree/master/CustomDeserializers).

### <a name="protocol-buffer-protobuf-format"></a>Формат буфера протокола (Protobuf)

Это пример использования формата буфера протокола.

Предположим следующее определение буфера протокола.

```proto
syntax = "proto3";
// protoc.exe from nuget "Google.Protobuf.Tools" is used to generate .cs file from this schema definition.
// Run below command to generate the csharp class
// protoc.exe --csharp_out=. MessageBodyProto.proto

package SimulatedTemperatureSensor;
message MessageBodyProto {
    message Ambient {
      double temperature = 1;
      int64 humidity = 2;
    }

    message Machine {
      double temperature = 1;
      double pressure = 2;
    }

    Machine machine = 1;
    Ambient ambient = 2;
    string timeCreated = 3;
}
```

Запуск `protoc.exe` из **Google.Protobuf.Tools** NuGet создает CS-файл с определением. Созданный файл здесь не показан. Необходимо убедиться, что версия Protobuf NuGet, используемая в проекте Stream Analytics, соответствует версии Protobuf, которая использовалась для создания входных данных. 

Следующий фрагмент кода является реализацией десериализатора при условии, что созданный файл включен в проект. Эта реализация является просто тонкой оболочкой для созданного файла.

```csharp
    public class MessageBodyDeserializer : StreamDeserializer<SimulatedTemperatureSensor.MessageBodyProto>
    {
        public override IEnumerable<SimulatedTemperatureSensor.MessageBodyProto> Deserialize(Stream stream)
        {
            while (stream.Position < stream.Length)
            {
                yield return SimulatedTemperatureSensor.MessageBodyProto.Parser.ParseDelimitedFrom(stream);
            }
        }

        public override void Initialize(StreamingContext streamingContext)
        {
        }
    }
```

### <a name="csv"></a>CSV

Следующий фрагмент кода является простым десериализатором CSV, который также демонстрирует распространение ошибок.

```csharp
using System.Collections.Generic;
using System.IO;

using Microsoft.Azure.StreamAnalytics;
using Microsoft.Azure.StreamAnalytics.Serialization;

namespace ExampleCustomCode.Serialization
{
    public class CustomCsvDeserializer : StreamDeserializer<CustomEvent>
    {
        private StreamingDiagnostics streamingDiagnostics;

        public override IEnumerable<CustomEvent> Deserialize(Stream stream)
        {
            using (var sr = new StreamReader(stream))
            {
                string line = sr.ReadLine();
                while (line != null)
                {
                    if (line.Length > 0 && !string.IsNullOrWhiteSpace(line))
                    {
                        string[] parts = line.Split(',');
                        if (parts.Length != 3)
                        {
                            streamingDiagnostics.WriteError("Did not get expected number of columns", $"Invalid line: {line}");
                        }
                        else
                        {
                            yield return new CustomEvent()
                            {
                                Column1 = parts[0],
                                Column2 = parts[1],
                                Column3 = parts[2]
                            };
                        }
                    }

                    line = sr.ReadLine();
                }
            }
        }

        public override void Initialize(StreamingContext streamingContext)
        {
            this.streamingDiagnostics = streamingContext.Diagnostics;
        }
    }

    public class CustomEvent
    {
        public string Column1 { get; set; }

        public string Column2 { get; set; }

        public string Column3 { get; set; }
    }
}

```

## <a name="serialization-format-for-rest-apis"></a>Формат сериализации для REST API

Все входные данные Stream Analytics имеют **формат сериализации**. Дополнительные сведения о параметрах ввода см. в документации по [REST API ввода](/rest/api/streamanalytics/2016-03-01/inputs).

Следующий код JavaScript является примером формата сериализации десериализатора .NET при использовании REST API:

```javascript
{    
   "properties":{    
      "type":"stream",  
      "serialization":{    
         "type":"CustomCLR",  
         "properties":{    
            "serializationDllPath":"<path to the dll inside UserCustomCode\CLR\ folder>", 
            "serializationClassName":"<Full name of the deserializer class name>" 
         }  
      }
   }  
}  
```

`serializationClassName` должен быть классом, реализующим `StreamDeserializer<T>`. Это описывается в следующем разделе.

## <a name="region-support"></a>Поддержка регионов

Эта функция доступна в следующих регионах:

* центрально-западная часть США
* Северная Европа
* Восточная часть США
* западная часть США
* восточная часть США 2
* Западная Европа

Вы можете [запросить поддержку](https://aka.ms/ccodereqregion) для дополнительных регионов.

## <a name="frequently-asked-questions"></a>Часто задаваемые вопросы

### <a name="when-will-this-feature-be-available-in-all-azure-regions"></a>Когда эта функция станет доступной во всех регионах Azure?

Указанная функция доступна в [6 регионах](#region-support). Если вы заинтересованы в использовании этой функции в другом регионе, можно [отправить запрос](https://aka.ms/ccodereqregion). Поддержка всех регионов Azure включена в план.

### <a name="can-i-access-metadatapropertyvalue-from-my-inputs-similar-to-getmetadatapropertyvalue-function"></a>Можно ли получить доступ к MetadataPropertyValue из входных данных, похожих на функцию GetMetadataPropertyValue?

Эта функциональность не поддерживается. Если вам нужна эта возможность, вы можете проголосовать за этот запрос на [UserVoice](https://feedback.azure.com/forums/270577-stream-analytics/suggestions/38779801-accessing-input-metadata-properties-in-custom-dese).

### <a name="can-i-share-my-deserializer-implementation-with-the-community-so-that-others-can-benefit"></a>Можно ли поделиться реализацией десериализатора с сообществом, чтобы другие пользователи могли воспользоваться ею?

После реализации десериализатора вы можете помочь другим пользователям, поделившись им с сообществом. Отправьте код в [репозиторий GitHub Azure Stream Analytics](https://github.com/Azure/azure-stream-analytics/tree/master/CustomDeserializers).

### <a name="what-are-the-other-limitations-of-using-custom-deserializers-in-stream-analytics"></a>Каковы другие ограничения использования пользовательских десериализаторов в Stream Analytics?

Если входные данные имеют формат protobuf с схемой `MapField` , содержащей тип, вы не сможете реализовать пользовательский десериализатор. Кроме того, пользовательские десериализаторы не поддерживают образцы данных или данные для предварительного просмотра. 

## <a name="next-steps"></a>Next Steps

* [Настраиваемые десериализаторы .NET для облачных заданий Azure Stream Analytics](custom-deserializer.md)
