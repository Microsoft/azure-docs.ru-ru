---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 03/27/2020
ms.author: trbye
ms.openlocfilehash: 4a4705647b90d29f47e37b88531f3432c6a2f448
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102434556"
---
При разработке для macOS доступно три пакета SDK для распознавания речи.

- Пакет SDK цели-C для распознавания речи доступен в виде пакета Кокоапод.
- Пакет SDK для .NET для распознавания речи можно использовать с **Xamarin. Mac** , так как он реализует .NET Standard 2,0
- Пакет SDK для распознавания речи Python доступен в виде модуля PyPI

> [!TIP]
> Дополнительные сведения об использовании речевого пакета "объектив-C" с помощью SWIFT см. в разделе <a href="https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift" target="_blank">Импорт цели-c в SWIFT </a>.

### <a name="system-requirements"></a>Требования к системе

- MacOS версии 10,13 или более поздней.

# <a name="xcode"></a>[Xcode](#tab/mac-xcode)

:::row:::
    :::column span="3":::
        Пакет macOS Кокоапод доступен для загрузки и использования в интегрированной среде разработки (IDE) <a href="https://apps.apple.com/us/app/xcode/id497799835" target="_blank">Xcode 9.4.1 (или более поздней версии) </a> . Сначала <a href="https://aka.ms/csspeech/macosbinary" target="_blank">Скачайте двоичный кокоапод </a>. Извлеките модуль Pod в том же каталоге, в котором он использовался, создайте *Podfile* и перечислите его `pod` как `target` .
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

target 'MyApp' do
  pod 'MicrosoftCognitiveServicesSpeech', '~> 1.15.0'
end
```

# <a name="xamarinmac"></a>[Xamarin.Mac](#tab/mac-xamarin)

:::row:::
    :::column span="3":::
        Xamarin. Mac предоставляет полный пакет SDK для macOS для разработчиков .NET, позволяющий создавать собственные приложения Mac на C#. Дополнительные сведения см. в разделе <a href="https://docs.microsoft.com/xamarin/mac/" target="_blank">Xamarin. Mac </a>.
    :::column-end:::
    :::column:::
        <br>
        <div class="icon is-large">
            <img alt="Xamarin" src="https://docs.microsoft.com/media/logos/logo_xamarin.svg" width="60px">
        </div>
    :::column-end:::
:::row-end:::

[!INCLUDE [Get .NET Speech SDK](get-speech-sdk-dotnet.md)]

# <a name="python"></a>[Python](#tab/mac-python)

[!INCLUDE [Get Python Speech SDK](get-speech-sdk-python.md)]

---

#### <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/objectivec/macos" target="_blank">Краткий пример пакета SDK для macOS для распознавания речи — исходный код на языке C </a>
- <a href="https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/swift/macos" target="_blank">исходный код руководства по пакету SDK для macOS для распознавания речи </a>