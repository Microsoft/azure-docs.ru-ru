---
title: Учебник по созданию приложения для iOS, которое создает фотографию и запускает ее в иммерсивном средстве чтения (Swift)
titleSuffix: Azure Cognitive Services
description: В этом руководстве вы создадите приложение для iOS с нуля и добавите изображение в функциональность Иммерсивного средства чтения.
services: cognitive-services
author: metanMSFT
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: tutorial
ms.date: 01/14/2020
ms.author: metang
ms.openlocfilehash: 23e9fd586882d25aed646b984e17c5885145ccff
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/03/2021
ms.locfileid: "106284640"
---
# <a name="tutorial-create-an-ios-app-that-launches-the-immersive-reader-with-content-from-a-photo-swift"></a>Учебник по созданию приложения для iOS, которое запускает иммерсивное средство чтения с содержимым из фотографии (Swift)

[Иммерсивное средство чтения](https://www.onenote.com/learningtools) — это включительно разработанное решение, в котором реализованы проверенные методы, улучшающие понимание при чтении.

[API чтение службы Cognitive Service "Компьютерное зрение"](../computer-vision/overview-ocr.md) обнаруживает текстовое содержимое в изображении, используя новейшие модели распознавания Майкрософт и преобразует обнаруженный текст в поток символов, пригодный для машинного чтения.

В этом руководстве вы создадите приложение для iOS с нуля, интегрируете API чтения, а также Иммерсивное средство чтения с помощью пакета SDK Ииммерсивного средства чтения. Полностью рабочий пример этого руководства доступен [здесь](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/ios).

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services/), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* [Xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12)
* Ресурс "Иммерсивное средство чтения", настроенный для проверки подлинности Azure Active Directory. Инструкции по настройке см. [здесь](./how-to-create-immersive-reader.md). Некоторые из этих созданных значений потребуются вам при настройке свойств примера проекта. Сохраните результаты своего сеанса в текстовом файле для использования в будущем.
* Для использования этого примера требуется подписка Azure на службу Cognitive Service "Компьютерное зрение". [Создайте ресурс "Компьютерное зрение" Cognitive Service на портале Azure](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision).

## <a name="create-an-xcode-project"></a>Создание проекта Xcode

Создайте проект в Xcode.

![Создать проект](./media/ios/xcode-create-project.png)

Выберите **Приложение одного представления**.

![Новое приложение одного представления](./media/ios/xcode-single-view-app.png)

## <a name="get-the-sdk-cocoapod"></a>Получение пакета SDK CocoaPod
Самый простой способ использовать пакет SDK для Иммерсивного средства чтения — через CocoaPods. Чтобы установить его с помощью Cocoapods, выполните следующее.
1. [Установка CocoaPods](http://guides.cocoapods.org/using/getting-started.html) по инструкциям из руководства по началу работы.
2. Создайте Podfile, запустив `pod init` в корневом каталоге проекта Xcode.
3.  Добавьте CocoaPod в Podfile, добавив `pod 'immersive-reader-sdk', :path => 'https://github.com/microsoft/immersive-reader-sdk/tree/master/iOS/immersive-reader-sdk'`. Ваш Podfile должен выглядеть следующим образом, при этом имя вашего целевого объекта заменяется рисунком "picture-to-immersive-reader-swift":
 ```ruby
  platform :ios, '9.0'

  target 'picture-to-immersive-reader-swift' do
  use_frameworks!
  # Pods for picture-to-immersive-reader-swift
  pod 'immersive-reader-sdk', :git => 'https://github.com/microsoft/immersive-reader-sdk.git'
  end
```
4. В окне терминала в каталоге проекта Xcode запустите команду `pod install`, чтобы установить pod пакета SDK для иммерсивного средства чтения
5. Добавьте `import immersive_reader_sdk` ко всем файлам, которые должны ссылаться на этот пакет SDK.
6. Убедитесь, что проект открыт, открыв файл `.xcworkspace`, а не `.xcodeproj`.

## <a name="acquire-an-azure-ad-authentication-token"></a>Получение маркера проверки подлинности Azure AD

Для этой части потребуются некоторые значения из описанного выше предварительного требования конфигурации проверки подлинности Azure AD. Вернитесь к текстовому файлу, который вы сохранили в этом сеансе.

````text
TenantId     => Azure subscription TenantId
ClientId     => Azure AD ApplicationId
ClientSecret => Azure AD Application Service Principal password
Subdomain    => Immersive Reader resource subdomain (resource 'Name' if the resource was created in the Azure portal, or 'CustomSubDomain' option if the resource was created with Azure CLI Powershell. Check the Azure portal for the subdomain on the Endpoint in the resource Overview page, for example, 'https://[SUBDOMAIN].cognitiveservices.azure.com/')
````

В главной папке проекта, которая содержит файл ViewController.swift, создайте файл для класса Swift с именем Constants.swift. Замените этот класс следующим кодом, добавив в него нужные реальные значения. Сохраните этот файл как локальный, существующий только на вашем компьютере, и убедитесь, что этот файл не передан в систему управления версиями, так как он содержит секреты, которые не следует публиковать. Мы рекомендуем не сохранять секреты в коде приложения. Вместо этого лучше использовать внутреннюю службу для получения токена, что позволяет хранить секреты вне приложения и вне устройства. Конечная точка внутреннего-API должна быть защищена какой-либо формой аутентификации (например, [OAuth](https://oauth.net/2/)), чтобы предотвратить получение неавторизованными пользователями токенов для использования в службе "Иммерсивное средство чтения" и выставлении счетов. Эти действия не входят в данное руководство.

## <a name="set-up-the-app-to-run-without-a-storyboard"></a>Настройка приложения для запуска без раскадровки

Откройте AppDelegate.swift и замените его содержимое следующим кодом.

## <a name="add-functionality-for-taking-and-uploading-photos"></a>Добавление функций для создания и отправки фотографий

Переименуйте ViewController.swift в PictureLaunchViewController.swift и замените файл следующим кодом.

## <a name="build-and-run-the-app"></a>Создание и запуск приложения

Настройте схему архивации в Xcode, выбрав симулятор или цель устройства.
![Схема архива](./media/ios/xcode-archive-scheme.png)<br/>
![Выбор цели](./media/ios/xcode-select-target.png)

В Xcode нажмите сочетание клавиш CTRL+R или нажмите кнопку "Воспроизведение", чтобы запустить проект. Приложение должно запуститься в указанном симуляторе или устройстве.

В приложении вы должны увидеть:

![Пример приложения](./media/ios/picture-to-immersive-reader-ipad-app.png)

В приложении сделайте или загрузите фотографию текста, нажав кнопку "Сделать фотографию" или "Выбрать фотографию из библиотеки", после чего Иммерсивное средство чтения запустит отображение текста с фотографии.

![Иммерсивное средство чтения](./media/ios/picture-to-immersive-reader-ipad.png)

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со справочной документацией по [пакету SDK для иммерсивного средства чтения](./reference.md).