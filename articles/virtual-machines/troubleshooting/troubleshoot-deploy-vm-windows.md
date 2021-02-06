---
title: Устранение неполадок при развертывании виртуальных машин Windows в Azure | Документация Майкрософт
description: Устранение неполадок при развертывании проблем с виртуальными машинами Windows в модели развертывания Azure Resource Manager.
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 4e383427-4aff-4bf3-a0f4-dbff5c6f0c81
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 11/01/2018
ms.author: genli
ms.openlocfilehash: 98a16e0a60ddf149e8f0e1a092051f3e98ea8225
ms.sourcegitcommit: 59cfed657839f41c36ccdf7dc2bee4535c920dd4
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/06/2021
ms.locfileid: "99627068"
---
# <a name="troubleshoot-deploying-windows-virtual-machine-issues-in-azure"></a>Устранение неполадок при развертывании виртуальных машин Windows в Azure

Для устранения неполадок, связанных с развертыванием виртуальных машин в Azure, ознакомьтесь с разделом [Наиболее важные проблемы](#top-issues), где описаны распространенные ошибки и способы их разрешения.

Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/). Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните **Получить поддержку**.

## <a name="top-issues"></a>Наиболее важные проблемы
[!INCLUDE [virtual-machines-windows-troubleshoot-deploy-vm-top](../../../includes/virtual-machines-windows-troubleshoot-deploy-vm-top.md)]

## <a name="the-cluster-cannot-support-the-requested-vm-size"></a>Кластер не поддерживает запрошенный размер виртуальной машины
\<properties
supportTopicIds="123456789"
resourceTags="windows"
productPesIds="1234, 5678"
/>
- Повторите запрос с указанием меньшего размера виртуальной машины.
- Если нельзя изменить размер запрошенной виртуальной машины,
    - остановите все виртуальные машины в группе доступности. Выберите **Группы ресурсов** > имя вашей группы ресурсов > **Ресурсы** > имя вашей группы доступности > **Виртуальные машины** > имя вашей виртуальной машины > **Остановить**.
    - После остановки всех виртуальных машин создайте виртуальную машину необходимого размера.
    - Сначала запустите новую виртуальную машину, а затем выберите каждую из остановленных виртуальных машин и нажмите кнопку "Запустить".


## <a name="the-cluster-does-not-have-free-resources"></a>В кластере нет свободных ресурсов
\<properties
supportTopicIds="123456789"
resourceTags="windows"
productPesIds="1234, 5678"
/>
- Повторите запрос позже.
- Если новая виртуальная машина должна быть частью другой группы доступности:
    - создайте виртуальную машину в другой группе доступности (в том же регионе);
    - добавьте новую виртуальную машину в ту же виртуальную сеть.

## <a name="how-can-i-use-and-deploy-a-windows-client-image-into-azure"></a>Как использовать и развернуть образ клиента Windows в Azure?

В сценариях разработки и тестирования Azure можно использовать Windows 7, Windows 8 или Windows 10, если у вас есть соответствующая подписка Visual Studio (прежнее название — MSDN). В этой [статье](../windows/client-images.md) описываются требования к доступности при запуске клиента Windows в Azure и использовании образов из коллекции Azure.

## <a name="how-can-i-deploy-a-virtual-machine-using-the-hybrid-use-benefit-hub"></a>Как развернуть виртуальную машину с помощью программы преимуществ гибридного использования (HUB)?

Существует несколько различных способов развертывания виртуальных машин Windows с помощью программы преимуществ гибридного использования Azure.

Для подписки с соглашением Enterprise:

•   можно развернуть виртуальные машины на основе специальных образов из Marketplace, предварительно настроенных для применения программы преимуществ гибридного использования Azure.

Для соглашения Enterprise:

•   можно передать настраиваемую виртуальную машину и развернуть ее с помощью шаблона Resource Manager или Azure PowerShell.

Дополнительные сведения см. в следующих ресурсах:

 - [Общие сведения о преимуществах гибридного использования Azure](https://azure.microsoft.com/pricing/hybrid-use-benefit/)

 - [Скачиваемый файл с часто задаваемыми вопросами](https://download.microsoft.com/download/4/2/1/4211AC94-D607-4A45-B472-4B30EDF437DE/Windows_Server_Azure_Hybrid_Use_FAQ_EN_US.pdf)

 - [Преимущества гибридного использования Azure для Windows Server и клиента Windows](../windows/hybrid-use-benefit-licensing.md).

 - [How can I use the Hybrid Use Benefit in Azure](/archive/blogs/azureedu/how-can-i-use-the-hybrid-use-benefit-in-azure) (Как можно применить программу преимуществ гибридного использования в Azure)

## <a name="how-do-i-activate-my-monthly-credit-for-visual-studio-enterprise-bizspark"></a>Как активировать мой ежемесячный кредит для Visual Studio Enterprise (BizSpark)?

Чтобы активировать ежемесячный кредит, см. эту [статью](https://azure.microsoft.com/offers/ms-azr-0064p/).

## <a name="how-to-add-enterprise-devtest-to-my-enterprise-agreement-ea-to-get-access-to-window-client-images"></a>Как мне добавить предложение "Enterprise — разработка и тестирование" в соглашение Enterprise (EA) для получения доступа к образам клиентов Windows?

Возможность создания подписок на основании предложения "Enterprise — разработка и тестирование" предоставляется тем владельцам учетных записей, которым администратор предприятия выдал разрешение на это. Владелец учетной записи создает подписки через портал учетных записей Azure, а затем должен добавить активных подписчиков Visual Studio в качестве соадминистраторов. Тогда они смогут использовать ресурсы, необходимые для разработки и тестирования, а также управлять ими. Дополнительные сведения см. в статье [Разработка и тестирование Enterprise](https://azure.microsoft.com/offers/ms-azr-0148p/).

## <a name="my-drivers-are-missing-for-my-windows-n-series-vm"></a>Для моей виртуальной машины Windows серии N отсутствуют драйверы

Инструкции по установке драйверов для виртуальных машин под управлением Windows находятся [здесь](../sizes-gpu.md#supported-operating-systems-and-drivers).

## <a name="i-cant-find-a-gpu-instance-within-my-n-series-vm"></a>Не удается найти экземпляр графического процессора (GPU) на виртуальной машине серии N

Чтобы воспользоваться преимуществами возможностей GPU виртуальных машин Azure серии N, необходимо установить графические драйверы на каждой виртуальной машине после развертывания. Сведения о настройке драйвера доступны [здесь](../sizes-gpu.md#supported-operating-systems-and-drivers).

## <a name="are-n-series-vms-available-in-my-region"></a>Доступны ли виртуальные машины серии N в моем регионе?

Доступность можно проверить в [таблице "Доступность продуктов по регионам"](https://azure.microsoft.com/regions/services), а цену можно узнать [здесь](https://azure.microsoft.com/pricing/details/virtual-machines/series/#n-series).

## <a name="what-client-images-can-i-use-and-deploy-in-azure-and-how-to-i-get-them"></a>Какие образы клиентов можно использовать и развертывать в Azure и как их получить?

В сценариях разработки и тестирования Azure можно использовать Windows 7, Windows 8 или Windows 10 при условии, что у вас есть соответствующая подписка Visual Studio (прежнее название — MSDN). 

- Образы Windows 10, которые можно использовать для разработки и тестирования, доступны в коллекции Azure. См. раздел [Доступные предложения](../windows/client-images.md). 
- Подписчики Visual Studio с предложением любого типа также смогут [правильно подготавливать и создавать](../windows/prepare-for-upload-vhd-image.md) 64-разрядные образы Windows 7, Windows 8 или Windows 10, а затем [отправлять их в Azure](../windows/upload-generalized-managed.md). Они также могут использоваться только активными подписчиками Visual Studio и только в целях разработки и тестирования.

В этой [статье](../windows/client-images.md) описываются требования к доступности при запуске клиента Windows в Azure и использовании образов из коллекции Azure.

## <a name="i-am-not-able-to-see-vm-size-family-that-i-want-when-resizing-my-vm"></a>При изменении размера виртуальной машины не отображается необходимое семейство размеров виртуальных машин

Когда виртуальная машина запущена, она развертывается на физический сервер. Физические серверы в регионах Azure группируются в кластеры общего физического оборудования. Изменение размера виртуальной машины, для которого требуется переместить виртуальную машину в другой кластер оборудования, отличается в зависимости от того, какая модель развертывания использовалась для развертывания данной виртуальной машины.

- На виртуальных машинах, развернутых с помощью классической модели, необходимо удалить развертывание облачной службы и развернуть ее повторно, выбрав для виртуальных машин размер из другого семейства размеров.

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]

- На виртуальных машинах, развернутых с помощью модели Resource Manager, необходимо остановить все виртуальные машины в группе доступности, прежде чем изменять размер любой виртуальной машины в этой группе доступности.

## <a name="the-listed-vm-size-is-not-supported-while-deploying-in-availability-set"></a>Указанный размер виртуальной машины не поддерживается при развертывании в группе доступности

Выберите размер, который поддерживается в кластере группы доступности. При создании группы доступности рекомендуется выбрать самый большой размер виртуальной машины, который может понадобиться, и использовать его в качестве первого развертывания в этой группе доступности.

## <a name="can-i-add-an-existing-classic-vm-to-an-availability-set"></a>Можно ли добавить существующую классическую виртуальную машину в группу доступности?

Да. Существующую классическую виртуальную машину можно добавить в новую или существующую группу доступности. Дополнительные сведения см. в разделе [Добавление существующей виртуальной машины к группе доступности](/previous-versions/azure/virtual-machines/windows/classic/configure-availability-classic#addmachine).


## <a name="next-steps"></a>Дальнейшие действия
Если в любой момент при изучении этой статьи вам потребуется дополнительная помощь, вы можете обратиться к экспертам по Azure на [форумах MSDN Azure и Stack Overflow](https://azure.microsoft.com/support/forums/).

Кроме того, можно зарегистрировать обращение в службу поддержки Azure. Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните **Получить поддержку**.
