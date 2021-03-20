---
title: Включение службы пиринга Azure на прямом одноранговом узле с помощью портал Azure
titleSuffix: Azure
description: Включение службы пиринга Azure на прямом одноранговом узле с помощью портал Azure
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: how-to
ms.date: 3/18/2020
ms.author: derekol
ms.openlocfilehash: a52e6038b622c004dc0d133394cd4f53600b2935
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84700048"
---
# <a name="enable-azure-peering-service-on-a-direct-peering-by-using-the-azure-portal"></a>Включение службы пиринга Azure на прямом одноранговом узле с помощью портал Azure

В этой статье описывается, как включить [службу пиринга](overview-peering-service.md) Azure на непосредственном одноранговом пиринге с помощью портал Azure.

При необходимости инструкции из этого руководства можно выполнить с помощью [PowerShell](howto-peering-service-powershell.md).

## <a name="before-you-begin"></a>Перед началом
* Прежде чем приступить к настройке, ознакомьтесь с [предварительными требованиями](prerequisites.md) .
* Выберите прямой пиринг в подписке, для которой требуется включить службу пиринга. Если у вас ее нет, преобразуйте устаревший прямой пиринг или создайте новый прямой пиринг:
    * Чтобы преобразовать устаревший прямой пиринг, следуйте инструкциям в статье [Преобразование устаревшего прямого пиринга в ресурс Azure с помощью портала](howto-legacy-direct-portal.md).
    * Чтобы создать новый прямой пиринг, следуйте инструкциям в статье [Создание или изменение прямого пиринга с помощью портала](howto-direct-portal.md).

## <a name="enable-peering-service-on-a-direct-peering"></a>Включение прямого пиринга в Службе пиринга

### <a name="view-direct-peering"></a><a name= get></a>Просмотр прямого пиринга
[!INCLUDE [peering-direct-get-portal](./includes/direct-portal-get.md)]

### <a name="enable-the-direct-peering-for-peering-service"></a><a name= get></a>Включение прямого пиринга для службы пиринга

После открытия прямого пиринга на предыдущем шаге включите его для службы пиринга.
[!INCLUDE [peering-direct-modify](./includes/peering-service-direct-portal.md)]

## <a name="modify-a-direct-peering-connection"></a>Изменение соединения с прямым пирингом

Сведения об изменении параметров подключения см. в разделе "изменение прямого пиринга" статьи [Создание или изменение прямого пиринга с помощью портала](howto-direct-portal.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Создание или изменение пиринга по обмену интернет-трафика с помощью портала](howto-exchange-portal.md)
* [Преобразование устаревшего пиринга по обмену интернет-трафика в ресурс Azure с помощью портала](howto-legacy-exchange-portal.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

Часто задаваемые вопросы см. в статье [часто ЗАдаваемые вопросы о службе пиринга](service-faqs.md).