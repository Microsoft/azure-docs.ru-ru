---
author: nikuklic
ms.service: azure-communication-services
ms.topic: include
ms.date: 03/10/2021
ms.author: nikuklic
ms.openlocfilehash: 43e3463a3284f57825073888146b38fa14cbf5d3
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105109048"
---
[!INCLUDE [Emergency Calling Notice](../../../includes/emergency-calling-notice-include.md)]
## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) бесплатно. 
- Развернутый ресурс Служб коммуникации. [Создайте ресурс Служб коммуникации.](../../create-communication-resource.md)
- Номер телефона, полученный из ресурса Служб коммуникации. [Как получить номер телефона.](../../telephony-sms/get-phone-number.md)
- `User Access Token` для включения клиента вызова. Дополнительные сведения о том, [как получить `User Access Token`](../../access-tokens.md)
- Выполните инструкции из краткого руководства [Начало работы путем добавления вызова в приложение](../getting-started-with-calling.md).

### <a name="prerequisite-check"></a>Проверка предварительных условий

- Чтобы просмотреть номера телефонов, связанные с ресурсом Служб коммуникации, войдите на [портал Azure](https://portal.azure.com/), перейдите к ресурсу Служб коммуникации и откройте вкладку с **номерами телефонов** в области навигации слева.
- Вы можете создать и запустить приложение с помощью пакета SDK для вызовов Служб коммуникации Azure для iOS:

## <a name="setting-up"></a>Настройка

## <a name="start-a-call-to-phone"></a>Вызов телефонного номера

Укажите номер телефона, полученный в ресурсе Служб коммуникации, который будет использоваться для осуществления вызова:
> [!WARNING]
> Обратите внимание, что номера телефонов должны быть указаны в формате международного стандарта E.164, например, +12223334444.

Измените обработчик событий `startCall`, который будет запускаться при нажатии кнопки *Начать вызов*:

```swift
func startCall() {
    // Ask permissions
    AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
        if granted {
            let startCallOptions = ACSStartCallOptions()
            startCallOptions!.alternateCallerID = PhoneNumber(phoneNumber: "+12223334444")
            self.call = self.callAgent!.startCall([PhoneNumber(phoneNumber: self.callee)], options: startCallOptions)
            self.callDelegate = CallDelegate(self)
            self.call!.delegate = self.callDelegate
        }
    }
}
```

## <a name="run-the-code"></a>Выполнение кода

Вы можете создать и запустить свое приложение в симуляторе iOS, выбрав **Product** > **Run** (Продукт > Выполнить) или с помощью клавиш (&#8984;-R).

![Окончательный вид приложения из этого краткого руководства](../media/ios/quick-start-make-call.png)

Вы можете позвонить на телефон, указав номер телефона в добавленном текстовом поле и нажав кнопку **Начать вызов**.
> [!WARNING]
> Обратите внимание, что номера телефонов должны быть указаны в формате международного стандарта E.164, например, +12223334444.

> [!NOTE]
> При первом вызове в системе отобразится запрос на получение доступа к микрофону. В приложении в рабочей среде используйте [проверку состояния разрешения](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/requesting_access_to_protected_resources) API `AVAudioSession` и корректно обновите поведение приложения, если разрешение не предоставлено.
