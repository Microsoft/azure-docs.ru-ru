---
title: Создание или изменение пиринга Exchange с помощью портала Azure
titleSuffix: Azure
description: Создание или изменение пиринга Exchange с помощью портала Azure
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: how-to
ms.date: 5/2/2020
ms.author: derekol
ms.openlocfilehash: 69201c97882846fb929b3b6f9a90be6647603bcc
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84700486"
---
# <a name="create-or-modify-an-exchange-peering-by-using-the-azure-portal"></a>Создание или изменение пиринга Exchange с помощью портала Azure

В этой статье описывается создание пиринга Microsoft Exchange с помощью портала Azure. Также здесь показано, как проверить состояние ресурса, обновить его или удалить и отозвать.

При необходимости инструкции из этого руководства можно выполнить с помощью [PowerShell](howto-exchange-powershell.md).

## <a name="before-you-begin"></a>Перед началом
* Изучите [предварительные требования](prerequisites.md) и [пошаговое руководство по пирингу Exchange](walkthrough-exchange-all.md).
* Если у вас уже есть подключения пиринга Microsoft Exchange, не преобразованные в ресурсы Azure, см. статью [Преобразование устаревшего пиринга Exchange в ресурс Azure с помощью портала](howto-legacy-exchange-portal.md).

## <a name="create-and-provision-an-exchange-peering"></a>Создание и подготовка пиринга Exchange

### <a name="sign-in-to-the-portal-and-select-your-subscription"></a>Вход на портал и выбор подписки
[!INCLUDE [Account](./includes/account-portal.md)]

### <a name="create-an-exchange-peering"></a><a name=create></a>Создание пиринга Exchange


Поставщик услуг Интернета может создать запрос на новое пиринговое подключение Exchange, как описано в [этой статье]( https://go.microsoft.com/fwlink/?linkid=2129593).

1. На странице **Create a Peering** (Создание пиринга) заполните поля на вкладке **Основные**, как показано ниже.

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

* По завершении выберите **Next: Конфигурация**, чтобы продолжить.

#### <a name="configure-connections-and-submit"></a>Настройка подключений и отправка
[!INCLUDE [exchange-peering-configuration](./includes/exchange-portal-configuration.md)]

### <a name="verify-an-exchange-peering"></a><a name=get></a>Проверка пиринга Exchange
[!INCLUDE [peering-exchange-get-portal](./includes/exchange-portal-get.md)]

## <a name="modify-an-exchange-peering"></a><a name="modify"></a>Изменение пиринга Exchange
[!INCLUDE [peering-exchange-modify-portal](./includes/exchange-portal-modify.md)]

## <a name="deprovision-an-exchange-peering"></a><a name="delete"></a>Отзыв пиринга Exchange
[!INCLUDE [peering-exchange-delete-portal](./includes/delete.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Создание или изменение прямого пиринга с помощью портала](howto-direct-portal.md)
* [Преобразование устаревшего прямого пиринга в ресурс Azure с помощью портала Azure](howto-legacy-direct-portal.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

Дополнительные сведения см. в статье [Часто задаваемые вопросы об интернет-пиринге](faqs.md).
