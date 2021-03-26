---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: 31634abe2768ec47ee2aa66051a7a363f83c6009
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105043227"
---
:::row:::
    :::column span="3":::
        При разработке для iOS доступно два пакета SDK для распознавания речи. Пакет SDK для "цель-C" доступен в виде пакета iOS Кокоапод. Кроме того, пакет SDK для .NET для распознавания речи можно использовать с Xamarin. iOS, так как он реализует .NET Standard 2,0.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="iOS" src="https://docs.microsoft.com/media/logos/logo_ios.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

> [!TIP]
> Дополнительные сведения об использовании речевого пакета "объектив-C" с помощью SWIFT см. в разделе <a href="https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift" target="_blank">Импорт цели-c в SWIFT </a>.

### <a name="system-requirements"></a>Требования к системе

- MacOS версии 10,3 или более поздней.
- Целевой сервер iOS 9,3 или более поздней версии

# <a name="xcode"></a>[Xcode](#tab/ios-xcode)

:::row:::
    :::column span="3":::
        Пакет iOS Кокоапод доступен для загрузки и использования с интегрированной средой разработки <a href="https://apps.apple.com/us/app/xcode/id497799835" target="_blank">Xcode 9.4.1 (или более поздней версии) </a> . Сначала <a href="https://aka.ms/csspeech/iosbinary" target="_blank">Скачайте двоичный кокоапод </a>. Извлеките модуль Pod в том же каталоге, в котором он использовался, создайте *Podfile* и перечислите его `pod` как `target` .
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xcode" src="https://docs.microsoft.com/media/logos/logo_xcode.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

```
platform :ios, '9.3'
use_frameworks!

target 'AppName' do
  pod 'MicrosoftCognitiveServicesSpeech', '~> 1.10.0'
end
```

# <a name="xamarinios"></a>[Xamarin.iOS](#tab/ios-xamarin)

:::row:::
    :::column span="3":::
        Xamarin.iOS предоставляет полный пакет SDK для iOS для разработчиков .NET. Создавайте собственные приложения iOS с помощью C# и F# в Visual Studio. Дополнительные сведения см. в разделе <a href="/xamarin/ios/" target="_blank">Xamarin. iOS </a>.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xamarin" src="https://docs.microsoft.com/media/logos/logo_xamarin.svg" width="60px">
            &nbsp;❤️ &nbsp;        <img alt="iOS" src="https://docs.microsoft.com/media/logos/logo_ios.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

[!INCLUDE [Get .NET Speech SDK](get-speech-sdk-dotnet.md)]

---

#### <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/objectivec/ios" target="_blank">Краткое руководство по пакету SDK для iOS Speech — исходный код на языке C </a>
- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/swift/ios" target="_blank">Краткое руководство по использованию пакета SDK для распознавания речи iOS в исходном коде </a>