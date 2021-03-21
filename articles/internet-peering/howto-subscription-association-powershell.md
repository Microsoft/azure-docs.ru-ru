---
title: Связывание однорангового ASN с подпиской Azure с помощью PowerShell
titleSuffix: Azure
description: Связывание однорангового ASN с подпиской Azure с помощью PowerShell
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 12/15/2020
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3b463293899dc2586404d68145943caff3105e89
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97590194"
---
# <a name="associate-peer-asn-to-azure-subscription-using-powershell"></a>Связывание однорангового ASN с подпиской Azure с помощью PowerShell

Прежде чем отправлять запрос пиринга, сначала нужно связать ASN с подпиской Azure, выполнив описанные ниже действия.

При желании вы можете выполнить инструкции с помощью [портала](howto-subscription-association-portal.md).

### <a name="working-with-azure-powershell"></a>Работа с Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="create-peerasn-to-associate-your-asn-with-azure-subscription"></a>Создание PeerASN для связывания ASN с подпиской Azure

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Войдите в учетную запись Azure и выберите подписку.
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="register-for-peering-resource-provider"></a>Регистрация в качестве поставщика ресурсов пиринга
Зарегистрируйтесь для использования поставщика ресурсов пиринга в подписке с помощью приведенной ниже команды. Если вы не выполните эту процедуру, ресурсы Azure, необходимые для настройки пиринга, будут недоступны.

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Peering
```

Проверить состояние регистрации можно с помощью следующих команд:
```powershell
Get-AzResourceProvider -ProviderNamespace Microsoft.Peering
```

> [!IMPORTANT]
> Перед продолжением дождитесь, пока *RegistrationState* зарегистрируется "Registered". Выполнение команды может занять от 5 до 30 минут.

### <a name="update-the-peer-information-associated-with-this-subscription"></a>Обновление сведений о одноранговом узле, связанных с этой подпиской

Ниже приведен пример обновления сведений об одноранговом узле.

```powershell
New-AzPeerAsn `
    -Name "Contoso_1234" `
    -PeerName "Contoso" `
    -PeerAsn 1234 `
    -Email noc@contoso.com, support@contoso.com `
    -Phone "+1 (555) 555-5555"
```

> [!NOTE]
> — Имя соответствует имени ресурса и может быть любым выбранным. Однако параметр-однорангово соответствует названию вашей компании и должен быть максимально близко к профилю Пирингдб. Обратите внимание, что параметр-одноранговое значение поддерживает только символы a – z, A – Z и пробел.

Подписка может иметь несколько ASN. Обновите сведения об пиринга для каждого ASN. Убедитесь, что "имя" уникально для каждого ASN.

Предполагается, что узлы имеют полный и актуальный профиль на [пирингдб](https://www.peeringdb.com). Мы используем эти сведения во время регистрации, чтобы проверить сведения об одноранговом узле, такие как сведения о центре Интернета, технические контактные данные и их присутствие на узлах и т. д.

Обратите внимание, что вместо **{SubscriptionId}** в приведенных выше выходных данных будет отображен фактический идентификатор подписки.

## <a name="view-status-of-a-peerasn"></a>Просмотр состояния PeerASN

Проверьте состояние проверки ASN с помощью следующей команды:

```powershell
Get-AzPeerAsn
```

Ниже приведен пример ответа.
```powershell
PeerContactInfo : Microsoft.Azure.PowerShell.Cmdlets.Peering.Models.PSContactInfo
PeerName        : Contoso
ValidationState : Approved
PeerAsnProperty : 1234
Name            : Contoso_1234
Id              : /subscriptions/{subscriptionId}/providers/Microsoft.Peering/peerAsns/Contoso_1234
Type            : Microsoft.Peering/peerAsns
```

> [!IMPORTANT]
> Дождитесь, пока состояние проверки не изменится на "Утверждено", а потом отправьте запрос пиринга. Утверждение может занять до 12 часов.

## <a name="modify-peerasn"></a>Изменение PeerAsn
Вы можете изменить контактные данные в любое время.

Ниже приведен пример:

```powershell
Set-PeerAsn -Name Contoso_1234 -Email "newemail@test.com" -Phone "1800-000-0000"
```

## <a name="delete-peerasn"></a>Удаление PeerAsn
Удаление PeerASN в настоящее время не поддерживается. Если вам нужно удалить PeerASN, обратитесь в службу [пиринга Майкрософт](mailto:peering@microsoft.com).

## <a name="next-steps"></a>Дальнейшие действия

* [Создание или изменение прямого пиринга](howto-direct-powershell.md)
* [Преобразование устаревшего прямого пиринга в ресурс Azure](howto-legacy-direct-powershell.md)
* [Создание или изменение пиринга Exchange](howto-exchange-powershell.md)
* [Преобразование устаревшего пиринга через точку обмена в ресурс Azure](howto-legacy-exchange-powershell.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

Дополнительные сведения см. в разделе [Часто задаваемые вопросы об интернет-пиринге](faqs.md).
