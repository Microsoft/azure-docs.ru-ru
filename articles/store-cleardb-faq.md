---
title: Часто задаваемые вопросы о базах данных ClearDB MySql в службе приложений Azure
description: Ответы на распространенные вопросы об использовании баз данных ClearDB MySQL со службой приложений Azure.
documentationcenter: php
services: mysql
author: sunbuild
manager: yochayk
tags: mysql
ms.service: multiple
ms.topic: article
ms.date: 10/27/2016
ms.author: sumuth
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: references_regions
ms.openlocfilehash: d9d4ae746c192e8b4afd1781c1e072fc0dc70422
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98599792"
---
# <a name="faq-for-cleardb-mysql-databases-with-azure-app-service"></a>Часто задаваемые вопросы о базах данных ClearDB MySql в службе приложений Azure
Ответы на часто задаваемые вопросы об использовании и покупке баз данных ClearDB MySql для службы веб-приложений Azure.

> [!IMPORTANT]
> По состоянию на 13 июня 2018 г. клиенты ClearDB на базе Azure, для которых сейчас выставляются счета корпорацией Майкрософт, переведены на модель прямого выставления счетов организацией ClearDB. Информация в этой статье теперь является устаревшей. Вы больше не сможете создавать базы данных ClearDB в Azure или обновлять базы данных ClearDB, созданные в Azure.
>
> Дополнительные сведения и описание дальнейших действий см. в статье об [изменениях в планах служб ClearDB](https://w2.cleardb.net/important-change-of-billing-notice-for-all-azure-cleardb-service-plans/).

## <a name="what-options-do-i-have-for-mysql-on-azure"></a>Какие параметры баз данных MySQL доступны в Azure?
Последние сведения об этой службе см. на странице [ClearDB](https://w2.cleardb.net/). ClearDB является службой размещения MySQL и управляет инфраструктурой MySQL. 

У вас есть несколько вариантов размещения MySQL в Azure.
* [База данных Azure для MySQL](https://azure.microsoft.com/services/mysql/)
* [Кластер MySQL на виртуальной машине Azure](https://github.com/azure/azure-quickstart-templates/tree/master/mysql-replication)
* [Отдельный экземпляр MySQL на виртуальной машине Azure](/previous-versions/azure/virtual-machines/windows/classic/mysql-2008r2?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json)


## <a name="do-i-need-a-credit-card-for-the-web-app--mysql-template-in-the-azure-marketplace"></a>Нужна ли кредитная карта для загрузки шаблона "Веб-приложение + MySQL" из Azure Marketplace?
Это зависит от типа вашей подписки. Чаще всего используются следующие типы подписки:

* [Оплата по мере использования:](https://azure.microsoft.com/offers/ms-azr-0003p/) требуется кредитная карта. При покупке платной базы данных MySQL ее стоимость будет списана с вашей кредитной карты.
* [Бесплатная пробная версия:](https://azure.microsoft.com/pricing/free-trial/) включает кредиты на использование со службами Microsoft Azure, но не позволяет приобретение ресурсов сторонних производителей. Для покупки служб сторонних производителей или платных баз данных MySQL требуется подписка с активной кредитной картой. Для веб-приложений можно создать БЕСПЛАТНУЮ базу данных MySQL ClearDB.
* [Подписка MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits/) и **MSDN для разработки и тестирования с оплатой по мере использования**: как и в случае с бесплатной пробной версией, для покупки платного решения MySQL у ClearDB требуется кредитная карта.
* [Соглашение Enterprise (EA):](https://azure.microsoft.com/pricing/enterprise-agreement/) клиенты с Соглашением EA ежеквартально получают отдельный консолидированный счет на все совершенные ими покупки решений сторонних разработчиков в Azure Marketplace. Счет выставляется за пределами предоплаты Azure (ранее назывались денежными обязательствами) для покупок из Marketplace. Обратите внимание на то, что в настоящее время хранилище Azure недоступно для клиентов из Азербайджана, Хорватии, Норвегии и Пуэрто-Рико. 

## <a name="why-was-i-charged-350-for-a-web-app--mysql-from-the-azure-marketplace"></a>Почему с меня списали 3,50 доллара за веб-приложение + MySQL в Azure Marketplace?
По умолчанию используется база данных Titan, которая стоит 3,50 доллара. Эта сумма не отображается при создании базы данных, поэтому вы могли приобрести ее по ошибке. Мы пытаемся найти способ решения этой проблемы, а до тех пор рекомендуем проверять выбранные ценовые категории веб-приложения и базы данных, прежде чем щелкнуть **Готово** и развернуть ресурсы.

## <a name="i-am-running-mysql-on-my-own-azure-virtual-machine-can-i-connect-my-azure-web-app-to-my-database"></a>Я использую MySQL на собственной виртуальной машине Azure. Могу ли я подключить к этой базе данных веб-приложение Azure?
Да. Веб-приложение можно подключить к базе данных, если у виртуальной машины Azure есть удаленный доступ к этому веб-приложению. Дополнительные сведения см. в статье об [установке MySQL на виртуальной машине](/previous-versions/azure/virtual-machines/windows/classic/mysql-2008r2?toc=%2fazure%2fvirtual-machines%2fwindows%2fclassic%2ftoc.json).

## <a name="in-which-countriesregions-are-cleardb-premium-mysql-clusters-supported"></a>В каких странах или регионах поддерживаются кластеры ClearDB Premium MySQL?
Кластеры ClearDB MySQL Premium доступны во всех регионах Azure, кроме Индии, Австралии, Южной Бразилии и Китая.

## <a name="can-i-create-a-new-cluster-prior-to-creating-a-database-with-cleardb-premium-cluster-solution"></a>Можно ли создать новый кластер до создания базы данных с использованием кластера ClearDB Premium?
Нет, создание пустых кластеров ClearDB не поддерживается. Портал Azure позволяет создавать базы данных в кластере, которые могут сразу провести эту процедуру.

## <a name="will-i-be-warned-if-i-try-to-delete-a-cleardb-mysql-database-that-is-in-use-by-one-of-my-applications"></a>Отображается ли предупреждение при попытке удалить базу данных ClearDB MySQL, которая используется одним из моих приложений?
Нет, Azure не предупреждает пользователя об удалении покупки из Azure Marketplace, от которой зависит приложение.

## <a name="which-regions-can-i-create-cleardb-databases-in"></a>В каких регионах можно создавать базы данных ClearDB?
Azure Marketplace недоступен для клиентов из Азербайджана, Хорватии, Норвегии и Пуэрто-Рико. В этих регионах ClearDB недоступна.

## <a name="what-pricing-tier-should-i-choose-for-a-production-web-app-and-database"></a>Какую ценовую категорию лучше выбрать для рабочего веб-приложения и базы данных?
Для веб-приложений выбирайте базовую или более высокую ценовую категорию. Для ClearDB рекомендуется план "Сатурн" или "Юпитер". Изучите возможности и ограничения каждой ценовой категории для [веб-приложений](https://azure.microsoft.com/pricing/details/app-service/) и [баз данных ClearDB MySQL](https://w2.cleardb.net/important-change-of-billing-notice-for-all-azure-cleardb-service-plans/). Выберите наиболее подходящий для вас вариант.

## <a name="how-do-i-upgrade-my-cleardb-database-from-one-plan-to-another"></a>Как перевести базу данных ClearDB с одного плана на другой?
На [портале Azure](https://portal.azure.com) можно увеличить масштаб общей базы данных размещения ClearDB. Дополнительные сведения см. в [этой статье](https://blogs.msdn.microsoft.com/appserviceteam/2016/10/06/upgrade-your-cleardb-mysql-database-in-azure-portal/). В настоящее время обновление для кластеров ClearDB Premium на портале Azure не поддерживается.

## <a name="i-cant-see-my-cleardb-database-in-azure-portal"></a>Я не вижу свою базу данных ClearDB на портале Azure!
Если вы создали базу данных ClearDB в классическом виде, вы не сможете увидеть базу данных в [портал Azure](https://portal.azure.com). Решения этой проблемы нет.

## <a name="who-do-i-contact-for-support-when-my-database-is-down"></a>Куда обращаться за помощью в случае отказа базы данных?
По всем вопросам, связанным с базами данных, обращайтесь в [службу поддержки ClearDB](https://www.cleardb.com/developers/help/support) . Вам нужно будет сообщить данные вашей подписки Azure.

## <a name="can-i-create-additional-users-for-my-cleardb-mysql-database-cluster-solution"></a>Можно ли создавать дополнительных пользователей для кластера баз данных ClearDB MySQL?
Нет. Создавать дополнительных пользователей в кластере баз данных ClearDB нельзя, но можно создавать дополнительные базы данных.  

## <a name="can-basicpro-series-databases-be-upgraded-in-place-similar-to-planetary-plans-today-on-cleardb-portal"></a>Можно ли обновить базы данных серий Basic и Pro на месте, как в случае с текущими планами Planetary, на портале ClearDB?
Да, базы данных серии Basic (Basic 60 – Basic 500) можно обновить на месте. Вы можете обновить на месте базы данных серии Pro (Pro 125–1000), за исключением Pro 60. В настоящее время обновление базы данных Pro 60 не поддерживается. 

## <a name="when-i-migrate-my-resources-from-one-subscription-to-another-does-my-cleardb-mysql-database-get-migrated-as-well"></a>Переносится ли база данных ClearDB MySQL при переносе ресурсов из одной подписки в другую?
При переносе ресурсов из одной подписки в другую действуют некоторые [ограничения](azure-resource-manager/management/move-support-resources.md) . База данных ClearDB MySQL — это сторонняя служба, в связи с чем она не перемещается при переносе подписки Azure. Перенос базы данных MySQL необходимо выполнить до переноса ресурсов Azure, иначе базы данных ClearDB MySQL могут быть отключены. Сначала вручную перенесите свои базы данных, а затем измените подписку веб-приложения. 

## <a name="i-hit-the-spending-limit-on-my-subscription-i-removed-the-limit-and-my-app-service-is-online-however-the-database-is-not-accessible-how-do-i-re-enable-the-cleardb-database"></a>В моей подписке достигнута предельная сумма расходов. Предел был снят, и служба приложений работает, однако база данных недоступна. Как повторно включить базу данных ClearDB?
Чтобы повторно включить базу данных, обратитесь в [службу поддержки ClearDB](https://www.cleardb.com/developers/help/support). Сообщите данные вашей подписки Azure и имя базы данных.

## <a name="can-i-transfer-a-cleardb-database-from-a-credit-card-subscription-to-an-ea-subscription"></a>Можно ли перенести базу данных ClearDB из подписки с кредитной карты в подписку EA?
Для существующих баз данных ClearDB используется кредитная карта, связанная с действующими подписками. Чтобы использовать подписку EA, необходимо перенести данные в новую базу данных:

* Приобретите новую базу данных ClearDB для своей подписки EA.
* Перенесите данные в новую базу данных.
* Обновите приложение для использования новой базы данных.
* Удалите старую базу данных ClearDB.

При создании веб-приложения с базой данных MySQL (ClearDB) или базы данных MySQL (ClearDB) метод оплаты этой услуги зависит от выбранной подписки. Подписка EA позволяет покупать службы сторонних производителей, например ClearDB, на портале Azure. Счета на подписку EA выставляются за пределами предоплаты Azure и выставляются ежеквартально и в прошедший период. Для использования подписки EA необходимо выбрать способ оплаты служб сторонних разработчиков, например через кредитную карту.

## <a name="where-can-i-see-the-charges-for-cleardb-resources-in-an-ea-subscription"></a>Где узнать размер платы за ресурсы ClearDB в подписке EA?
Для пользователей подписки Direct EA стоимость ресурсов из Azure Marketplace отображается на портале Enterprise. Обратите внимание, что все покупки и потребление в Marketplace выставляются за пределами предоплаты Azure и выставляются ежеквартально и в прошедший период. Пользователи EA платят непосредственно поставщикам сторонних служб и могут оплачивать их с помощью кредитной карты, указанной в их учетной записи EA.

Клиенты, которые пользуются подпиской EA через посредников, могут увидеть свои подписки в Azure Marketplace на странице **Управление подписками** портала Enterprise, однако цены будут скрыты. За информацией о сборах Azure Marketplace клиентам следует обращаться к поставщикам решений по лицензированию.

Доступом к Azure Marketplace для покупки служб сторонних производителей могут управлять администраторы регистрации EA Azure. Они могут включать и отключать доступ к покупкам служб сторонних производителей в магазине в областях "Управление учетными записями" и "Подписка" раздела "Учетные записи" на портале Enterprise.

## <a name="who-do-i-contact-for-questions-about-my-bill-for-cleardb-services-in-my-ea-subscription"></a>К кому обращаться с вопросами по счету за службы ClearDB по подписке EA?
По вопросам оплаты подписки EA обращайтесь в [службу поддержки клиентов Enterprise](https://aka.ms/AzureEntSupport) . Сотрудники службы поддержки на портале EA ответят на ваши вопросы и помогут решить проблемы.

## <a name="more-information"></a>Дополнительные сведения
[Часто задаваемые вопросы об Azure Marketplace](https://azure.microsoft.com/marketplace/faq/)