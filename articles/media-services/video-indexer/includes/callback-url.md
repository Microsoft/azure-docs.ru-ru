---
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 11/13/2020
ms.author: juliako
ms.openlocfilehash: fbc77b960cac0db2d077345c74d64485bd7ad8bb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95994505"
---
URL-адрес, используемый для уведомления клиента (с помощью запроса POST) о текущих событиях:

- Изменение состояния индексирования: 
    - Свойства    
    
        |Имя|Описание|
        |---|---|
        |идентификатор|Идентификатор видео|
        |Состояние|Состояние видео|  
    - Пример: https: \/ /test.com/notifyme?projectName=MyProject&ID = 1234abcd&State = обработано
- Человек, идентифицированный на видео:
  - Свойства
    
      |Имя|Описание|
      |---|---|
      |идентификатор| Идентификатор видео|
      |faceId|Идентификатор лица, которое отображается в индексе видео|
      |knownPersonId|Идентификатор человека, который имеет уникальную модель лица|
      |personName|Имя человека|
        
    - Пример: https: \/ /test.com/notifyme?projectName=MyProject&ID = 1234abcd&FaceId = 12&кновнперсонид = CCA84350-89B7-4262-861C-3CAC796542A5&персоннаме = Inigo_Montoya 
