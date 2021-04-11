---
title: Настройка Azure Percept Audio
description: Сведения о том, как подключить устройство Azure Percept Audio к Azure Percept DK.
author: mimcco
ms.author: mimcco
ms.service: azure-percept
ms.topic: quickstart
ms.date: 03/25/2021
ms.openlocfilehash: fa3dad8cdd38e6db621d8194cc9472430c7c5008
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105605796"
---
# <a name="azure-percept-audio-setup"></a>Настройка Azure Percept Audio

Azure Percept Audio поставляется полностью готовым к работе с Azure Percept DK. Никаких специальных настроек не требуется.

## <a name="prerequisites"></a>Предварительные требования

- Azure Percept DK (DevKit).
- Azure Percept Audio
- [Подписка Azure.](https://azure.microsoft.com/free/)
- [Выполненная настройка Azure Percept DK](./quickstart-percept-dk-set-up.md). Предполагается, что вы уже подключили DevKit к сети Wi-Fi, создали центр Интернета вещей и подключили к нему DevKit.
- Динамик или наушники, которые можно подключить к аудиовходу 3,5 мм (необязательно).

## <a name="connecting-your-devices"></a>Подключение устройств

1. Подключите устройство Azure Percept Audio к плате-носителю Azure Percept DK с помощью предоставленного кабеля micro-USB (тип A). Подключите разъем micro-USB этого кабеля к плате аудио-интерпозера (разработчика), а разъем Type-A — к плате-носителю Azure Percept DK.

1. (Необязательно.) Подключите динамик или наушники к устройству Azure Percept Audio через аудиовход, обозначенный как "Line Out" (линейный выход). Это позволит вам прослушивать аудиоответы.

1. Включите DevKit. Светодиодный индикатор L02 на плате аудио-интерпозера начнет мигать белым цветом, указывая на то, что устройство включено, а аудио-плата (SOM) проходит аутентификацию.

1. Дождитесь завершения процесса проверки подлинности — это может занять до 3 минут.

1. Вы можете приступать к созданию прототипов, дождавшись любого из следующих событий:

    - Светодиодный индикатор L02 загорится белым цветом. Это означает, что аутентификация успешно завершена, а для DevKit еще не настроено ключевое слово.
    - Все три светодиодных индикатор загорятся синим цветом. Это означает, что аутентификация успешно завершена и для DevKit настроено ключевое слово.

## <a name="next-steps"></a>Дальнейшие действия

Создайте [решение для использования речевых функций без написания кода](./tutorial-no-code-speech.md) в [Azure Percept Studio](https://go.microsoft.com/fwlink/?linkid=2135819).
