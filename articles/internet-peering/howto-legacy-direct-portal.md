---
title: Преобразование устаревшего прямого пиринга в ресурс Azure с помощью портала Azure
titleSuffix: Azure
description: Преобразование устаревшего прямого пиринга в ресурс Azure с помощью портала Azure
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 496774f034c861131624b552b1acefca9f399ea8
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "84700269"
---
# <a name="convert-a-legacy-direct-peering-to-an-azure-resource-by-using-the-azure-portal"></a>Преобразование устаревшего прямого пиринга в ресурс Azure с помощью портала Azure

В этой статье описывается, как преобразовать существующий прямой пиринг в ресурс Azure с помощью портала Azure.

При необходимости инструкции из этого руководства можно выполнить с помощью [PowerShell](howto-legacy-direct-powershell.md).

## <a name="before-you-begin"></a>Перед началом
* Изучите [предварительные требования](prerequisites.md) и [пошаговое руководство по прямому пирингу](walkthrough-direct-all.md).


## <a name="convert-a-legacy-direct-peering-to-an-azure-resource"></a>Преобразование устаревшего прямого пиринга в ресурс Azure

### <a name="sign-in-to-the-portal-and-select-your-subscription"></a>Вход на портал и выбор подписки
[!INCLUDE [Account](./includes/account-portal.md)]

### <a name="convert-a-legacy-direct-peering"></a><a name=create></a>Преобразование устаревшего прямого пиринга

Являясь поставщиком услуг Интернета, вы можете преобразовать устаревшие прямые пиринговые подключения на странице [Создание пиринга]( https://go.microsoft.com/fwlink/?linkid=2129593).

1. На странице **Создание пиринга** на вкладке **Основные** заполните поля, как показано ниже:

    > [!div class="mx-imgBorder"] 
    > ![Регистрация Службы пиринга](./media/setup-basics-tab.png)

*    Выберите свою подписку Azure.

* Для параметра "Группа ресурсов" можно выбрать имеющуюся группу ресурсов из раскрывающегося списка или создать новую группу, нажав "Создать". В этом примере мы создадим группу ресурсов.

* Имя соответствует имени ресурса и может быть любым из выбранных.

* Если выбрана существующая группа ресурсов, то регион выбирается автоматически. Если вы решили создать новую группу ресурсов, необходимо также выбрать регион Azure, где будет размещаться этот ресурс.

>[!NOTE]
>Регион, в котором находится группа ресурсов, не зависит от расположения, в котором вы хотите создать пиринг с Майкрософт. Ресурсы пиринга рекомендуется упорядочивать в группах ресурсов, которые находятся в ближайших регионах Azure. Например, для пиринга в городе Эшберн можно создать группу ресурсов в восточной части США или восточной США 2.

* Выберите значение ASN в поле **PeerASN**.

>[!IMPORTANT] 
>Перед отправкой запроса на пиринг можно выбрать только ASN с параметром ValidationState, которому присвоено значение "Утверждено". Если вы только что отправили запрос PeerAsn, обратите внимание на то, что утверждение запроса может занять до 12 часов. Если выбранный ASN ожидает проверки, отобразится сообщение об ошибке. Если вы не видите ASN, который требуется выбрать, убедитесь, что выбрана правильная подписка. Если это так, убедитесь, что вы уже создали одноранговый узел ASN с помощью функции **[Связать одноранговый узел ASN с подпиской Azure](https://go.microsoft.com/fwlink/?linkid=2129592)** .

#### <a name="launch-the-resource-and-configure-basic-settings"></a>Запуск ресурса и настройка основных параметров
[!INCLUDE [direct-peering-basic](./includes/direct-portal-basic.md)]

#### <a name="configure-connections-and-submit"></a>Настройка подключений и отправка
[!INCLUDE [direct-peering-configuration](./includes/direct-portal-configuration-legacy.md)]

### <a name="verify-direct-peering"></a><a name=get></a>Проверка прямого пиринга
[!INCLUDE [peering-direct-get-portal](./includes/direct-portal-get.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

Дополнительные сведения см. в разделе [Часто задаваемые вопросы об интернет-пиринге](faqs.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Создание или изменение прямого пиринга с помощью портала](howto-direct-portal.md)
