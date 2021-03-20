---
title: Принцип работы входного аудиопотока в пакете SDK для службы "Речь"
titleSuffix: Azure Cognitive Services
description: Обзор возможностей API входного аудиопотока в пакете SDK для службы "Речь".
services: cognitive-services
author: fmegen
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: fmegen
ms.custom: devx-track-csharp
ms.openlocfilehash: 87fa97dafe9de4a23f5eaadfd4083cd1ca517cde
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95026596"
---
# <a name="about-the-speech-sdk-audio-input-stream-api"></a>Об API входного аудиопотока в пакете SDK для службы "Речь"

API **потока входных данных** РЕЧЕВОГО пакета SDK предоставляет способ потоковой передачи в распознаватели вместо использования микрофона или API-интерфейсов входного файла.

При использовании входных аудиопотоков выполняется следующая процедура.

- Определите формат аудиопотока. Формат должен поддерживаться пакетом SDK для службы "Речь" и службой "Речь". Сейчас поддерживается только приведенная ниже конфигурация.

  Звуковые примеры относятся к формату PCM, по одному каналу, 16 битам на выборку, 8000 или 16000 в секунду (16000 или 32000 байт в секунду), два блока (16 бит, включая заполнение для примера).

  Соответствующий код в пакете SDK для создания звукового формата выглядит следующим образом.

  ```csharp
  byte channels = 1;
  byte bitsPerSample = 16;
  int samplesPerSecond = 16000; // or 8000
  var audioFormat = AudioStreamFormat.GetWaveFormatPCM(samplesPerSecond, bitsPerSample, channels);
  ```

- Убедитесь, что код предоставляет необработанные аудио данные в соответствии с этими спецификациями. Также убедитесь, что 16-разрядные примеры поступают в формате с прямым порядком байтов. Также поддерживаются подписанные примеры. Если источник аудиоданных не соответствует поддерживаемым форматам, звук необходимо перекодировать в требуемый формат.

- Создайте собственный класс входного аудиопотока, производный от `PullAudioInputStreamCallback`. Реализуйте члены `Read()` и `Close()`. Точная сигнатура функции зависит от языка, однако код будет выглядеть аналогично этому примеру кода.

  ```csharp
   public class ContosoAudioStream : PullAudioInputStreamCallback {
      ContosoConfig config;

      public ContosoAudioStream(const ContosoConfig& config) {
          this.config = config;
      }

      public int Read(byte[] buffer, uint size) {
          // returns audio data to the caller.
          // e.g. return read(config.YYY, buffer, size);
      }

      public void Close() {
          // close and cleanup resources.
      }
   };
  ```

- Создайте конфигурацию звука, основанную на формате звука и входном аудиопотоке. Передайте конфигурацию для обычной речи и конфигурацию ввода звука при создании распознавателя. Пример:

  ```csharp
  var audioConfig = AudioConfig.FromStreamInput(new ContosoAudioStream(config), audioFormat);

  var speechConfig = SpeechConfig.FromSubscription(...);
  var recognizer = new SpeechRecognizer(speechConfig, audioConfig);

  // run stream through recognizer
  var result = await recognizer.RecognizeOnceAsync();

  var text = result.GetText();
  ```

## <a name="next-steps"></a>Дальнейшие действия

- [Создание бесплатной учетной записи Azure.](https://azure.microsoft.com/free/cognitive-services/)
- [Распознавание речи в C#](./get-started-speech-to-text.md?pivots=programming-language-csharp&tabs=dotnet)