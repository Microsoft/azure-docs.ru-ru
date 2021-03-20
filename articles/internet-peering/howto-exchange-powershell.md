---
title: Создание или изменение пиринга Exchange с помощью PowerShell
titleSuffix: Azure
description: Создание или изменение пиринга Exchange с помощью PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 59d098f1ff0f78e37601544619c603b776e0cf22
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89071760"
---
# <a name="create-or-modify-an-exchange-peering-by-using-powershell"></a>Создание или изменение пиринга Exchange с помощью PowerShell

В этой статье описывается, как создать пиринг Microsoft Exchange с помощью командлетов PowerShell и модели развертывания диспетчер ресурсов. В этой статье также показано, как проверить состояние ресурса, обновить его или удалить и отменить его предоставление.

При желании вы можете выполнить это пошаговое руководством с помощью [портала](howto-exchange-portal.md)Azure.

## <a name="before-you-begin"></a>Перед началом
* Перед началом конфигурации изучите [предварительные требования](prerequisites.md) и [пошаговое руководство по пирингу Exchange](walkthrough-exchange-all.md).
* Если у вас уже есть пиринг Exchange с Майкрософт, которые не преобразованы в ресурсы Azure, см. статью [Преобразование устаревшего пиринга Exchange в ресурс Azure с помощью PowerShell](howto-legacy-exchange-powershell.md).

### <a name="work-with-azure-powershell"></a>Работа с Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="create-and-provision-an-exchange-peering"></a>Создание и подготовка пиринга Exchange

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку.
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="get-the-list-of-supported-peering-locations-for-exchange-peering"></a><a name=exchange-location></a>Получение списка поддерживаемых расположений пиринга для пиринга Exchange
[!INCLUDE [exchange-location](./includes/exchange-powershell-create-location.md)]

### <a name="create-an-exchange-peering"></a><a name=create></a>Создание пиринга Exchange
[!INCLUDE [exchange-peering](./includes/exchange-powershell-create-connection.md)]

### <a name="get-exchange-peering"></a><a name=get></a>Получение пиринга Exchange
[!INCLUDE [peering-exchange-get](./includes/exchange-powershell-get.md)]

## <a name="modify-an-exchange-peering"></a><a name="modify"></a>Изменение пиринга Exchange
[!INCLUDE [peering-exchange-modify](./includes/exchange-powershell-modify.md)]

## <a name="deprovision-an-exchange-peering"></a><a name=delete></a>Отзыв пиринга Exchange

[!INCLUDE [peering-exchange-delete](./includes/delete.md)]

## <a name="next-steps"></a>Дальнейшие действия

* [Создание или изменение прямого пиринга с помощью PowerShell](howto-direct-powershell.md)
* [Преобразование устаревшего прямого пиринга в ресурс Azure с помощью PowerShell](howto-legacy-direct-powershell.md)

## <a name="additional-resources"></a>Дополнительные ресурсы
Подробное описание всех параметров можно получить, выполнив следующую команду.

```powershell
Get-Help Get-AzPeering -detailed
```

Дополнительные сведения см. в статье [Часто задаваемые вопросы об интернет-пиринге](faqs.md).
