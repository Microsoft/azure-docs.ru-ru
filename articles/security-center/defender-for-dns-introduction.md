---
title: Преимущества и функции Azure Defender для DNS
description: Сведения о преимуществах и функциях Azure Defender для DNS
author: memildin
ms.author: memildin
ms.date: 12/07/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: dffb505719e6778adfdd8e99f62790df9ebd615a
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102100701"
---
# <a name="introduction-to-azure-defender-for-dns"></a>Общие сведения об Azure Defender для DNS

[Azure DNS](../dns/dns-overview.md) является службой размещения доменов DNS, осуществляющей разрешение имен на базе инфраструктуры Microsoft Azure. Размещая домены в Azure, вы можете управлять своими записями DNS с помощью тех же учетных данных, API и инструментов и оплачивать использование, как и другие службы Azure.

Azure Defender для DNS обеспечивает дополнительный уровень защиты облачных ресурсов благодаря:

- постоянному мониторингу всех запросов DNS, поступающих от ресурсов Azure;
- запуску расширенной аналитики безопасности для оповещения о подозрительных действиях.

## <a name="availability"></a>Доступность

|Аспект|Сведения|
|----|:----|
|Состояние выпуска:|Предварительный просмотр<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)] |
|Цены:|Плата за использование **Azure Defender для DNS** начисляется по тарифам, приведенным на странице [Цены на Центр безопасности](https://azure.microsoft.com/pricing/details/security-center/).|
|Облако.|![Да](./media/icons/yes-icon.png) Коммерческие облака<br>![Нет](./media/icons/no-icon.png) Национальные и независимые (US Gov, China Gov, другие правительственные облака)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-dns"></a>Каковы преимущества Azure Defender для DNS?

Azure Defender для DNS защищает от проблем, в том числе:

- кражи данных из ресурсов Azure с помощью туннелирования DNS;
- взаимодействие вредоносной программы с сервером контроля и управления;
- взаимодействия с вредоносными доменами, используемыми для фишинга и майнинга криптовалют;
- атаки на DNS — взаимодействия с вредоносными сопоставителями DNS. 

Полный список оповещений, предоставляемых Azure Defender для DNS, приведен на [странице справки по оповещениям](alerts-reference.md#alerts-dns).

## <a name="dependencies"></a>Зависимости

В Azure Defender для DNS не используются агенты. 

Чтобы защитить свой уровень DNS, включите Azure Defender для DNS для каждой из подписок, как описано в разделе о [включении Azure Defender](enable-azure-defender.md).


## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали об Azure Defender для DNS. Связанные материалы см. в следующих статьях. 

- Оповещения системы безопасности могут создаваться самим центром безопасности или приниматься центром безопасности от различных продуктов по обеспечению безопасности. Чтобы экспортировать все эти оповещения в Azure Sentinel, любое стороннее средство SIEM или другие внешние инструменты, следуйте инструкциям в статье [Экспорт оповещений о безопасности и рекомендаций](continuous-export.md).

- > [!div class="nextstepaction"]
    > [Включение Azure Defender](enable-azure-defender.md)