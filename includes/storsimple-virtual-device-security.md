---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: cb160a140b5c0cb184a5172da10ade0de37c4fed
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185331"
---
<!--v-sharos 10/13/2105 virtual device security-->

При использовании виртуального устройства StorSimple учитывайте следующие рекомендации по безопасности.

* Защита виртуального устройства обеспечивается в рамках подписки Microsoft Azure. Это означает, что, если при использовании виртуального устройства подписка Azure будет скомпрометирована, данные, сохраненные на виртуальном устройстве, также будут уязвимы.
* Открытый ключ сертификата, используемый для шифрования данных, сохраненных в Azure StorSimple, безопасно предоставляется на классическом портале Azure, а закрытый ключ хранится на устройстве StorSimple. На виртуальном устройстве StorSimple и открытый, и закрытый ключи хранятся в Azure.
* Виртуальное устройство размещено в центре обработки данных Microsoft Azure.

