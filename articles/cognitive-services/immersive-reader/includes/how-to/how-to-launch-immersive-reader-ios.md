---
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 03/04/2021
ms.author: erhopf
ms.openlocfilehash: a3345ff9a6cc1d5f8e2fbc78c2a76ba6f183aeb6
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102620201"
---
## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Ресурс "Иммерсивное средство чтения", настроенный для проверки подлинности Azure Active Directory. Инструкции по настройке см. [здесь](../../how-to-create-immersive-reader.md).  Вам потребуются некоторые значения, созданные здесь при настройке свойств среды. Сохраните результаты своего сеанса в текстовом файле для использования в будущем.
* [macOS](https://www.apple.com/macos).
* [Git](https://git-scm.com/).
* [Пакет SDK иммерсивного средства чтения.](https://github.com/microsoft/immersive-reader-sdk)
* [Xcode.](https://apps.apple.com/us/app/xcode/id497799835?mt=12)

## <a name="configure-authentication-credentials"></a>Определение учетных данных для проверки подлинности

1. В Xcode откройте **immersive-reader-sdk/js/samples/ios/quickstart-swift/quickstart-swift.xcodeproj**.
1. В верхнем меню выберите **Product (Продукт)**  > **Scheme (Схема)**  > **Edit Scheme (Изменить схему)** .
1. В представлении **Run** (Запуск) откройте вкладку **Arguments** (Аргументы).
1. В разделе **Environment Variables** (Переменные среды) добавьте следующие имена и значения. Укажите значения, заданные при создании ресурса "Иммерсивное средство чтения".

    ```text
    TENANT_ID=<YOUR_TENANT_ID>
    CLIENT_ID=<YOUR_CLIENT_ID>
    CLIENT_SECRET<YOUR_CLIENT_SECRET>
    SUBDOMAIN=<YOUR_SUBDOMAIN>
    ```

Не фиксируйте это изменение в системе управления версиями, так как оно содержит секреты, не предназначенные для публикации.

## <a name="start-the-immersive-reader-with-sample-content"></a>Запуск иммерсивного средства чтения с примером содержимого

В Xcode нажмите клавиши **CTRL+R**, чтобы запустить проект.

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со статьей о [пакете SDK для иммерсивного средства чтения](https://github.com/microsoft/immersive-reader-sdk) и [справкой по этому пакету](../../reference.md).
* Просмотрите примеры кода на сайте [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/).
