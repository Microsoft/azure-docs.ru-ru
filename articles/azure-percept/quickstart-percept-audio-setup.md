---
title: Начало работы с Azure Percept Audio
description: Сведения о том, как подключить устройство Azure Percept Audio к Azure Percept DK.
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: quickstart
ms.date: 02/18/2021
ms.custom: template-quickstart
ms.openlocfilehash: 588ebde85b6012ddbfb88ca8305fc735b7a0ba41
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102097998"
---
# <a name="azure-percept-audio-setup"></a>Настройка Azure Percept Audio

Azure Percept Audio поставляется полностью готовым к работе с Azure Percept DK. Никаких специальных настроек не требуется.

## <a name="prerequisites"></a>Предварительные требования

- Azure Percept DK (DevKit).
- Azure Percept Audio
- [Подписка Azure.](https://azure.microsoft.com/free/)
- [Выполненная настройка Azure Percept DK](./quickstart-percept-dk-set-up.md). Предполагается, что вы уже подключили DevKit к сети Wi-Fi, создали центр Интернета вещей и подключили к нему DevKit.
- Динамик или наушники, которые можно подключить к аудиовходу 3,5 мм (необязательно)

## <a name="connecting-your-devices"></a>Подключение устройств

1. Подключите устройство Azure Percept Audio к плате-носителю Azure Percept DK с помощью предоставленного кабеля micro-USB (тип A). Подключите разъем micro-USB этого кабеля к доске высветки (разработчика), а разъем Type-A — к плате-носителю Azure Percept DK.
1. (Необязательно.) Подключите динамик или наушники к устройству Azure Percept Audio через аудиовход, обозначенный как "Line Out" (Линейный выход). Это позволит вам слышать ответы голосового помощника. Если вы не подключите динамики или наушники, то сможете видеть текстовые ответы в окне демоверсии. 

1. Включите DevKit. Светодиодный индикатор L02 на доске высветки начнет мигать белым, указывая на то, что устройство включено, а Audio SoM проходит проверку подлинности.

1. Дождитесь завершения процесса проверки подлинности — это может занять до 3 минут.

1. Вы можете приступать к созданию прототипов, дождавшись любого из следующих событий:

    - Индикатор L02 загорится белым. Это означает, что аутентификация успешно завершена, а для DevKit еще не настроено ключевое слово.
    - Все три индикатора загорятся синим цветом. Это означает, что аутентификация успешно завершена, и для DevKit настроено ключевое слово.

## <a name="next-steps"></a>Дальнейшие действия

Создайте [решение по работе с речью без кода](./tutorial-no-code-speech.md).
