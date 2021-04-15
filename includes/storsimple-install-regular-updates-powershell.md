---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: dc50f94ae9b207961a71480c2fc172e88db79cf4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185320"
---
#### <a name="to-install-regular-updates-via-windows-powershell-for-storsimple"></a>Установка обычных обновлений с помощью Windows PowerShell для StorSimple
1. Откройте последовательную консоль устройства и выберите вариант 1 **Войти с полным доступом**. Задайте пароль. Пароль по умолчанию — *Password1*. 
2. В командной строке введите:
   
     `Get-HcsUpdateAvailability`
   
    Вы получите уведомление о том, что обновления доступны, и о том, являются они критическими или некритическими.
3. В командной строке введите:
   
     `Start-HcsUpdate`
   
    Начнется процесс обновления.

> [!IMPORTANT]
> * Эта команда применяется только для обычных обновлений. Эта команда выполняется только на одном контроллере, но обновляться будут оба контроллера. 
> * В процессе обновления может выполняться отработка отказа контроллера, но это не повлияет на доступность или работу системы.
> 
> 

