---
title: Использование образов клиентов Windows в Azure
description: Узнайте, как использовать преимущества подписки Visual Studio для развертывания Windows 7, Windows 8 или Windows 10 в Azure в сценариях разработки и тестирования.
author: cynthn
ms.subservice: imaging
ms.service: virtual-machines
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 12/15/2017
ms.author: cynthn
ms.openlocfilehash: 21038a8d1eabfcca21329c093b866607f0343070
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103200000"
---
# <a name="use-windows-client-in-azure-for-devtest-scenarios"></a>Использование клиента Windows в Azure для сценариев разработки и тестирования
В сценариях разработки и тестирования Azure можно использовать Windows 7, Windows 8 или Windows 10 Корпоративная (x64) при условии, что у вас есть соответствующая подписка Visual Studio (прежнее название — MSDN). 

Сведения о запуске Windows 10 в рабочей среде см. в статье [развертывание Windows 10 в Azure с правами на размещение клиентов](windows-desktop-multitenant-hosting-deployment.md).


## <a name="subscription-eligibility"></a>Доступность в зависимости от подписки
Активные подписчики Visual Studio (люди, получившие лицензию на подписку Visual Studio) могут использовать образы клиента Windows в целях разработки и тестирования. Образы клиента Windows можно использовать на своем оборудовании или на виртуальных машинах Azure.

Некоторые образы клиента Windows доступны в Azure Marketplace. Подписчики Visual Studio в любом типе предложения могут также [подготавливать и создавать](prepare-for-upload-vhd-image.md) 64-разрядные образы Windows 7, Windows 8 или Windows 10, а затем [отправлять их в Azure](upload-generalized-managed.md).

## <a name="eligible-offers-and-client-images"></a>Подходящие предложения и образы клиентов
В следующей таблице приведены идентификаторы предложений, которые можно использовать для развертывания образов клиентов Windows с помощью Azure Marketplace. Образы клиента Windows доступны только для следующих предложений. 

| Название предложения | Номер предложения | Доступные образы клиента | 
|:--- |:---:|:---:|
| [Разработка и тестирование с оплатой по мере использования](https://azure.microsoft.com/offers/ms-azr-0023p/) |0023P | Windows 10 Корпоративная N (x64) <br> Windows 8.1 Enterprise N (x64) <br> Windows 7 Корпоративная N с пакетом обновления 1 (SP1) (x64) |
| [Подписчики Visual Studio Enterprise (MPN)](https://azure.microsoft.com/offers/ms-azr-0029p/) |0029P | Windows 10 Корпоративная N (x64) <br> Windows 8.1 Enterprise N (x64) <br> Windows 7 Корпоративная N с пакетом обновления 1 (SP1) (x64) |
| [Подписчики Visual Studio Professional](https://azure.microsoft.com/offers/ms-azr-0059p/) |0059P | Windows 10 Корпоративная N (x64) <br> Windows 8.1 Enterprise N (x64) <br> Windows 7 Корпоративная N с пакетом обновления 1 (SP1) (x64) |
| [Подписчики Visual Studio Test Professional](https://azure.microsoft.com/offers/ms-azr-0060p/) |0060P | Windows 10 Корпоративная N (x64) <br> Windows 8.1 Enterprise N (x64) <br> Windows 7 Корпоративная N с пакетом обновления 1 (SP1) (x64) |
| [Visual Studio Premium с подпиской MSDN (преимущество)](https://azure.microsoft.com/offers/ms-azr-0061p/) |0061P | Windows 10 Корпоративная N (x64) <br> Windows 8.1 Enterprise N (x64) <br> Windows 7 Корпоративная N с пакетом обновления 1 (SP1) (x64) |
| [Подписчики Visual Studio Enterprise](https://azure.microsoft.com/offers/ms-azr-0063p/) |0063P | Windows 10 Корпоративная N (x64) <br> Windows 8.1 Enterprise N (x64) <br> Windows 7 Корпоративная N с пакетом обновления 1 (SP1) (x64) |
| [Подписчики Visual Studio Enterprise (BizSpark)](https://azure.microsoft.com/offers/ms-azr-0064p/) |0064P | Windows 10 Корпоративная N (x64) <br> Windows 8.1 Enterprise N (x64) <br> Windows 7 Корпоративная N с пакетом обновления 1 (SP1) (x64) |
| [Enterprise — разработка и тестирование](https://azure.microsoft.com/offers/ms-azr-0148p/) |0148P | Windows 10 Корпоративная N (x64) <br> Windows 8.1 Enterprise N (x64) <br> Windows 7 Корпоративная N с пакетом обновления 1 (SP1) (x64) |

## <a name="check-your-azure-subscription"></a>Проверка подписки Azure
Если вы не знакомы с ИДЕНТИФИКАТОРом предложения, его можно получить с помощью портал Azure.  
- В окне *подписки* : ![ сведения об идентификаторе предложения из портал Azure](./media/client-images/offer-id-azure-portal.png) 
- Или щелкните **Выставление счетов** и выберите свой идентификатор подписки. Идентификатор предложения отобразится в окне *Выставление счетов*. 
- Идентификатор предложения можно также просмотреть на [вкладке "подписки](https://account.windowsazure.com/Subscriptions) " портала учетных записей Azure: ![ сведения об идентификаторе предложения на портале учетных записей Azure.](./media/client-images/offer-id-azure-account-portal.png) 

## <a name="next-steps"></a>Дальнейшие действия
Теперь вы можете развернуть виртуальные машины с помощью [PowerShell](quick-create-powershell.md), [шаблонов Resource Manager](ps-template.md) или [Visual Studio](../../azure-resource-manager/templates/create-visual-studio-deployment-project.md).
