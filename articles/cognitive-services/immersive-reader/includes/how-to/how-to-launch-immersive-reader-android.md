---
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 03/04/2021
ms.author: erhopf
ms.openlocfilehash: d5a483cb239893d04d2c2581fb378f411878f4ba
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102620177"
---
## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Ресурс "Иммерсивное средство чтения", настроенный для проверки подлинности Azure Active Directory. Инструкции по настройке см. [здесь](../../how-to-create-immersive-reader.md).  Вам потребуются некоторые значения, созданные здесь при настройке свойств среды. Сохраните результаты своего сеанса в текстовом файле для использования в будущем.
* [Git](https://git-scm.com/).
* [Пакет SDK иммерсивного средства чтения.](https://github.com/microsoft/immersive-reader-sdk)
* [Android Studio](https://developer.android.com/studio).

## <a name="configure-authentication-credentials"></a>Определение учетных данных для проверки подлинности

1. Запустите Android Studio и откройте проект в каталоге **immersive-reader-sdk/js/samples/quickstart-java-android** (Java) или **immersive-reader-sdk/js/samples/quickstart-kotlin** (Kotlin).

1. Создайте файл **env** в папке **/assets**. Добавьте следующие имена и значения, используя данные о вашей среде. Не фиксируйте этот файл в системе управления версиями, так как он содержит секреты, не предназначенные для публикации.
    
    ```text
    TENANT_ID=<YOUR_TENANT_ID>
    CLIENT_ID=<YOUR_CLIENT_ID>
    CLIENT_SECRET=<YOUR_CLIENT_SECRET>
    SUBDOMAIN=<YOUR_SUBDOMAIN>
    ```

## <a name="start-the-immersive-reader-with-sample-content"></a>Запуск иммерсивного средства чтения с примером содержимого

Выберите эмулятор устройства в диспетчере AVD и запустите проект.

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со статьей о [пакете SDK для иммерсивного средства чтения](https://github.com/microsoft/immersive-reader-sdk) и [справкой по этому пакету](../../reference.md).
* Просмотрите примеры кода на сайте [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/).