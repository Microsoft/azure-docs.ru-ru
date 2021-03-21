---
title: Изменение номера SKU для доменных служб Azure AD | Документация Майкрософт
description: Узнайте, как изменить уровень SKU для управляемого домена доменных служб Azure AD в случае изменения бизнес-требований.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: 320bd87aa78d26cee44c48f27365febd1dd426ff
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96620296"
---
# <a name="change-the-sku-for-an-existing-azure-active-directory-domain-services-managed-domain"></a>Изменение номера SKU для существующего управляемого домена доменных служб Azure Active Directory

В Azure Active Directory доменных службах (Azure AD DS) доступные характеристики и производительность основаны на типе SKU. Эти различия функций включают в себя частоту резервного копирования или максимальное число односторонних исходящих доверий лесов.

Номер SKU выбирается при создании управляемого домена. После развертывания управляемого домена можно переключать номера SKU вверх или вниз по мере изменения бизнес-потребностей. Изменения в бизнес-требованиях могут включать в себя потребность в более частом резервном копировании или создании дополнительных доверий лесов. Дополнительные сведения об ограничениях и ценах на различные номера SKU см. в статье [Основные понятия azure AD DS SKU][concepts-sku] и [цены на AD DS Azure][pricing] .

В этой статье показано, как изменить номер SKU для существующего управляемого домена AD DS Azure с помощью портал Azure.

## <a name="before-you-begin"></a>Перед началом

Для работы с этой статьей требуются следующие ресурсы и разрешения:

* Активная подписка Azure.
    * Если у вас еще нет подписки Azure, [создайте учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Связанный с вашей подпиской клиент Azure Active Directory, синхронизированный с локальным или облачным каталогом.
    * Если потребуется, [создайте клиент Azure Active Directory][create-azure-ad-tenant] или [свяжите подписку Azure со своей учетной записью][associate-azure-ad-tenant].
* Управляемый домен доменных служб Azure Active Directory, включенный и настроенный в клиенте Azure AD.
    * При необходимости выполните инструкции из руководства по [созданию и настройке управляемого домена][create-azure-ad-ds-instance].

## <a name="sku-change-limitations"></a>Ограничения на изменение номера SKU

После развертывания управляемого домена можно изменить номера SKU. Однако если вы используете лес ресурсов и создали односторонние отношения доверия исходящего трафика из Azure AD DS в локальную среду AD DS, существуют некоторые ограничения для операции изменения номера SKU. SKU " *премиум* " и " *Корпоративный* " определяют ограничение на число создаваемых доверий. Вы не можете перейти на номер SKU с более низким пределом, чем настроено в настоящее время.

Пример:

* Если вы создали два доверия лесов в номере SKU уровня " *премиум* ", вы не сможете перейти к *стандартному* SKU. SKU " *стандартный* " не поддерживает доверия лесов.
* Если вы создали семь доверий для SKU уровня " *премиум* ", вы не сможете перейти на *Корпоративный* SKU. SKU *Enterprise* поддерживает не более пяти доверий.

Дополнительные сведения об этих ограничениях см. в статье [функции и ограничения Azure AD DS SKU][concepts-sku].

## <a name="select-a-new-sku"></a>Выберите новый номер SKU

Чтобы изменить номер SKU для управляемого домена с помощью портал Azure, выполните следующие действия.

1. В верхней части портал Azure найдите и выберите **доменные службы Azure AD**. Выберите управляемый домен из списка, например *aaddscontoso.com*.
1. В меню в левой части страницы AD DS Azure выберите **параметры > SKU**.

    ![Выберите параметр меню SKU для управляемого домена Azure AD DS в портал Azure](media/change-sku/overview-change-sku.png)

1. В раскрывающемся меню выберите номер SKU, который вы хотите указать для управляемого домена. Если у вас есть лес ресурсов, вы не сможете выбрать SKU " *стандартный* ", так как отношения доверия лесов доступны только на *корпоративном* SKU или более высоком уровне.

    Выберите нужный номер SKU в раскрывающемся меню и нажмите кнопку **сохранить**.

    ![Выберите требуемый номер SKU в раскрывающемся меню портал Azure](media/change-sku/change-sku-selection.png)

Изменение типа номера SKU может занять одну или две минуты.

## <a name="next-steps"></a>Дальнейшие действия

Если у вас есть лес ресурсов и вы хотите создать дополнительные отношения доверия после изменения номера SKU, см. статью [Создание исходящего доверия леса для локального домена в AD DS Azure][create-trust].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[concepts-sku]: administration-concepts.md#azure-ad-ds-skus
[create-trust]: tutorial-create-forest-trust.md

<!-- EXTERNAL LINKS -->
[pricing]: https://azure.microsoft.com/pricing/details/active-directory-ds/
