---
title: Преобразование устаревшего пиринга Exchange в ресурс Azure с помощью портала Azure
titleSuffix: Azure
description: Преобразование устаревшего пиринга Exchange в ресурс Azure с помощью портала Azure
services: internet-peering
author: derekolo
ms.service: internet-peering
ms.topic: how-to
ms.date: 5/21/2020
ms.author: derekol
ms.openlocfilehash: 6249d68486af754f2d2980aaed9d5fd8287dcb5e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84700167"
---
# <a name="convert-a-legacy-exchange-peering-to-an-azure-resource-by-using-the-azure-portal"></a>Преобразование устаревшего пиринга Exchange в ресурс Azure с помощью портала Azure

В этой статье описывается, как преобразовать существующий пиринг Exchange в ресурс Azure с помощью портала Azure.

При необходимости инструкции из этого руководства можно выполнить с помощью [PowerShell](howto-legacy-exchange-powershell.md).

## <a name="before-you-begin"></a>Перед началом
* Перед началом конфигурации изучите [предварительные требования](prerequisites.md) и [пошаговое руководство по пирингу Exchange](walkthrough-exchange-all.md).

## <a name="convert-a-legacy-exchange-peering-to-an-azure-resource"></a>Преобразование устаревшего пиринга Exchange в ресурс Azure

Поставщик услуг Интернета может создать запрос на новое пиринговое подключение Exchange, как описано в [этой статье]( https://go.microsoft.com/fwlink/?linkid=2129593).

1. На странице **Create a Peering** (Создание пиринга) заполните поля на вкладке **Основные**, как показано ниже.

   ![Регистрация в Службе пиринга](./media/setup-basics-tab.png)

* Выберите свою подписку Azure.

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
[!INCLUDE [exchange-peering-configuration](./includes/exchange-portal-configuration-legacy.md)]

### <a name="verify-exchange-peering"></a><a name=get></a>Проверка пиринга Exchange
[!INCLUDE [peering-exchange-get-portal](./includes/exchange-portal-get.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

Дополнительные сведения см. в разделе [Часто задаваемые вопросы об интернет-пиринге](faqs.md).

## <a name="next-steps"></a>Дальнейшие действия

* [Создание или изменение пиринга Exchange с помощью портала](howto-exchange-portal.md)
