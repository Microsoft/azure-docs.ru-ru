---
author: baanders
description: Включение файла для работы с Azure Digital Twins — предварительные требования для настройки экземпляра
ms.service: digital-twins
ms.topic: include
ms.date: 10/29/2020
ms.author: baanders
ms.openlocfilehash: 2be13256ff0fcc93aa40db3bcb2f38aa0cf58cbc
ms.sourcegitcommit: afb9e9d0b0c7e37166b9d1de6b71cd0e2fb9abf5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/14/2021
ms.locfileid: "103463901"
---
Для работы с Azure Digital Twins, как показано в этой статье, сначала нужно настроить **экземпляр Azure Digital Twins** и необходимые разрешения для его использования. Если у вас уже есть настроенный экземпляр Azure Digital Twins после предыдущих действий, можно использовать его.

В противном случае следуйте инструкциям в статье [*Практическое руководство. Настройка экземпляра Azure Digital двойников и проверки подлинности (с помощью сценария)*](../articles/digital-twins/how-to-set-up-instance-portal.md). Кроме того, в этих инструкциях указано, как проверить, успешно ли завершен каждый шаг и готов ли новый экземпляр к работе.

После настройки экземпляра Azure Digital Twins запишите следующие значения, которые потребуются вам позднее при подключении к экземпляру:
* **_Имя узла_** экземпляра. Это значение можно найти на портале Azure ([инструкции](../articles/digital-twins/how-to-set-up-instance-portal.md#verify-success-and-collect-important-values)).
* **Подписка Azure**, которую вы использовали для создания экземпляра (можно использовать как имя, так и идентификатор). Узнать, в какой подписке находится экземпляр Azure Digital Twins, можно на той же странице *обзора* странице для экземпляра на [портале Azure](https://portal.azure.com).
