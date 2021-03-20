---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/15/2020
ms.author: alkohli
ms.openlocfilehash: 8e2bd10081bb344fde239e92bd834b31062efde6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96583073"
---
Для неактивных данных:

- Доступ к данным, хранящимся в общих ресурсах, ограничен.

    - Клиентам SMB, обращающимся к данным общего доступа, требуются учетные данные пользователя, связанные с общей папкой. Эти учетные данные определяются при создании общей папки.
    - При создании общей папки необходимо добавить IP-адреса клиентов NFS, которые обращаются к общей папке.
