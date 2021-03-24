---
author: laujan
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 12/11/2020
ms.author: lajanuar
ms.openlocfilehash: 8bf78d4c2f703ee10b26b1936620f7cf23ed7c14
ms.sourcegitcommit: 3ea12ce4f6c142c5a1a2f04d6e329e3456d2bda5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/15/2021
ms.locfileid: "103467210"
---
Чтобы получить URL-адрес SAS для обучающих данных пользовательской модели, перейдите к ресурсу хранилища на портале Azure и выберите вкладку **Обозреватель службы хранилища**. Перейдите к контейнеру, щелкните правой кнопкой мыши и выберите элемент **Получить подписанный URL-адрес**. Важно получить SAS для вашего контейнера, а не для самой учетной записи хранения. Убедитесь, что заданы разрешения на **чтение**, **запись**, **удаление** и **перечисление**, а затем щелкните **Создать**. Затем скопируйте во временное расположение значение из раздела **URL-адрес**. Оно должно быть в таком формате: `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
