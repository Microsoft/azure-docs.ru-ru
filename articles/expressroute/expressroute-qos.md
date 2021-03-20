---
title: 'Azure ExpressRoute: требования к качеству обслуживания'
description: На этой странице подробно описаны требования по настройке качества обслуживания и управлению им. В этой статье также рассматривается Skype для бизнеса и голосовые службы.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: duau
ms.openlocfilehash: 7f14f43fa341df40ecd35340b7311e1acd18004c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92204713"
---
# <a name="expressroute-qos-requirements"></a>Требования к качеству обслуживания для ExpressRoute
В Skype для бизнеса имеются различные рабочие нагрузки, требующие дифференцированного подхода к качеству обслуживания. Если вы планируете использовать голосовые службы с помощью ExpressRoute, то необходимо соблюдать указанные ниже требования.

![Схема, на которой показаны службы Voice, идущие через ExpressRoute.](./media/expressroute-qos/expressroute-qos.png)

> [!NOTE]
> Требования к качеству обслуживания применяются только к пирингу Майкрософт. Значения DSCP в сетевом трафике, полученном через общедоступный или частный пиринг Azure, будут сброшены до "0". 
> 
> 

В следующей таблице приведен список меток DSCP, используемых Microsoft Teams и Skype для бизнеса. Дополнительные сведения см. в статье [Управление качеством обслуживания для Skype для бизнеса](/SkypeForBusiness/manage/network-management/qos/managing-quality-of-service-QoS).

| **Класс трафика** | **Обработка (пометки DSCP)** | **Рабочие нагрузки Microsoft Teams и Skype для бизнеса** |
| --- | --- | --- |
| **Голосовая связь** |EF (46) |Skype, Microsoft Teams или Lync Voice |
| **Интерактивный** |AF41 (34) |Видео, VBSS |
| |AF21 (18) |Совместный доступ к приложениям | 
| **По умолчанию** |AF11 (10) |Передача файлов |
| |CS0 (0) |Другие варианты |

* Вам необходимо классифицировать рабочие нагрузки и пометить правильные значения DSCP. Следуйте указанным [здесь](/SkypeForBusiness/manage/network-management/qos/configuring-port-ranges-for-your-skype-clients#configure-quality-of-service-policies-for-clients-running-on-windows-10) инструкциям по настройке пометок DSCP в сети.
* Вам необходимо настроить и поддерживать несколько очередей качества обслуживания в сети. Voice должен быть отдельным классом и принимать обработку EF, указанную в [RFC 3246](https://www.ietf.org/rfc/rfc3246.txt). 
* Вы можете решить, какие механизм очередей, политику обнаружения перегрузки и пропускную способность использовать для каждого класса трафика. Тем не менее необходимо сохранить пометки DSCP для рабочих нагрузок Skype для бизнеса. Если вы используете пометки DSCP, которых нет в списке выше, например AF31 (26), то перед отправкой пакета в корпорацию Майкрософт необходимо перезаписать это значение DSCP и сделать его равным 0. Корпорация Майкрософт отправляет только те пакеты, которые помечены значениями DSCP, указанными в таблице выше. 

## <a name="next-steps"></a>Дальнейшие действия
* См. сведения о требованиях для [маршрутизации](expressroute-routing.md) и [преобразования сетевых адресов (NAT)](expressroute-nat.md).
* Чтобы настроить подключение ExpressRoute, см. следующие статьи:
  
  * [Создание канала ExpressRoute](expressroute-howto-circuit-classic.md)
  * [Настройка маршрутизации](expressroute-howto-routing-classic.md)
  * [Связывание виртуальной сети с каналом ExpressRoute](expressroute-howto-linkvnet-classic.md)