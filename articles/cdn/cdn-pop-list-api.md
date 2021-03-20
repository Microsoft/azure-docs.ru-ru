---
title: Получение списка текущих IP-адресов POP для Azure CDN | Документация Майкрософт
description: Узнайте, как получить серверы POP с помощью REST API. Серверы POP выполняют запросы к исходным серверам, связанным с конечными точками сети доставки содержимого Azure.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/22/2019
ms.author: allensu
ms.custom: ''
ms.openlocfilehash: 4197b1a5f047190872d055dc2ba8ccaa11efbe6c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100376349"
---
# <a name="retrieve-the-current-pop-ip-list-for-azure-cdn"></a>Получение списка текущих IP-адресов POP для Azure CDN

## <a name="retrieve-the-current-verizon-pop-ip-list-for-azure-cdn"></a>Получение списка IP-адресов POP текущего Verizon для Azure CDN

С помощью REST API можно получить набор IP-адресов для серверов точек подключения (POP) Verizon. Эти серверы POP отправляют запросы к серверам-источникам, которые связаны с конечными точками сети доставки содержимого (CDN) Azure в профиле Verizon (**Azure CDN уровня "Стандартный" от Verizon** или **Azure CDN уровня "Премиум" от Verizon**). Обратите внимание, что этот набор IP-адресов отличается от IP-адресов, которые будет видеть клиент при выполнении запросов к точкам POP. 

Синтаксис операции REST API для получения списка точек POP см. в статье [Edge Nodes - List](/rest/api/cdn/cdn/edgenodes/list) (Пограничные узлы — список).

## <a name="retrieve-the-current-microsoft-pop-ip-list-for-azure-cdn"></a>Получение списка IP-адресов Microsoft POP для Azure CDN

Чтобы заблокировать приложение для приема трафика только от Azure CDN корпорации Майкрософт, необходимо настроить списки ACL IP-адресов для серверной части. Вы также можете ограничить набор допустимых значений для заголовка "X-Forwardd-Host", отправленного Azure CDN от Майкрософт. Эти действия описаны ниже.

Настройте IP адресам для серверной части, чтобы принимать трафик от Azure CDN пространства IP-адресов внутренней сети Майкрософт и только служб инфраструктуры Azure. 

* Azure CDN из внутреннего IP-пространства IPv4 Майкрософт: 147.243.0.0/16
* Azure CDN из внутреннего IP-адреса Microsoft IPv6:2a01:111:2050::/44

Чтобы использовать теги службы с Azure CDN Майкрософт, используйте тег передней дверцы Azure. Диапазоны IP-адресов и теги служб для служб Майкрософт можно найти [здесь](https://www.microsoft.com/download/details.aspx?id=56519) .


## <a name="typical-use-case"></a>Типичный вариант использования

В целях безопасности этот список IP-адресов можно использовать для принудительного выполнения запросов к серверу-источнику только с допустимой точки POP Verizon. Например, если кто-то узнает имя узла или IP-адрес сервера-источника конечной точки CDN, он сможет отправлять запросы напрямую на сервер-источник, обходя, таким образом, функции масштабирования и безопасности, предоставляемые Azure CDN. Чтобы избежать такой ситуации, IP-адреса в возвращаемом списке следует задать только как разрешенные на сервере-источнике. Чтобы иметь в распоряжении актуальный список точек POP, получайте его по меньшей мере один раз в день. 

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о REST API см. в статье [Azure CDN REST API](/rest/api/cdn/) (REST API сети доставки содержимого Azure).