---
title: Распространенные вопросы о виртуальной машине в Azure Marketplace
description: Распространенные вопросы, возникающие при создании виртуальной машины в Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: guide
author: iqshahmicrosoft
ms.author: iqshah
ms.date: 10/15/2020
ms.openlocfilehash: d045af3b170d585b4bf1f8c57b7ba924c6b30695
ms.sourcegitcommit: 8d1b97c3777684bd98f2cfbc9d440b1299a02e8f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102489792"
---
# <a name="common-questions-about-vm-in-azure-marketplace"></a>Распространенные вопросы о виртуальной машине в Azure Marketplace

В статье часто задаваемые вопросы рассматриваются распространенные проблемы, которые могут возникнуть при создании предложения виртуальной машины в Azure Marketplace.

## <a name="how-do-i-configure-a-virtual-private-network-vpn-to-work-with-my-vms"></a>Как настроить виртуальную частную сеть (VPN) для работы с моими виртуальными машинами?

Если вы используете модель развертывания Azure Resource Manager, у вас есть три варианта:

- [Создание VPN-шлюза на основе маршрутов с помощью портала Azure](../vpn-gateway/tutorial-create-gateway-portal.md)
- [Создание VPN-шлюза на основе маршрутов с помощью Azure PowerShell](../vpn-gateway/create-routebased-vpn-gateway-powershell.md)
- [Создание VPN-шлюза на основе маршрута с помощью CLI](../vpn-gateway/create-routebased-vpn-gateway-cli.md)

## <a name="what-are-microsoft-support-policies-for-running-microsoft-server-software-on-azure-based-vms"></a>Каковы политики поддержки корпорации Майкрософт для запуска серверного программного обеспечения Майкрософт на виртуальных машинах Azure?

Подробнее см. в статье [Поддержка серверного ПО Майкрософт для виртуальных машин Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines).

## <a name="in-a-vm-how-do-i-manage-the-custom-script-extension-in-the-startup-task"></a>Как выполняется управление расширением настраиваемых сценариев при запуске виртуальной машины?

Сведения об использовании расширения настраиваемых сценариев с помощью модуля Azure PowerShell, шаблонов Azure Resource Manager, а также сведения о действиях по устранению неполадок в системах Windows см. в статье [Расширение настраиваемого сценария в ОС Windows](../virtual-machines/extensions/custom-script-windows.md).

## <a name="are-32-bit-applications-or-services-supported-in-azure-marketplace"></a>Поддерживаются ли 32-разрядные приложения или службы в Azure Marketplace?

Нет. Поддерживаемые операционные системы и стандартные службы для виртуальных машин Azure являются 64-разрядными. Несмотря на то что большинство 64-разрядных операционных систем поддерживают 32-разрядные версии приложений для обеспечения обратной совместимости, использование 32-разрядных приложений в рамках решения виртуальной машины не поддерживается и не рекомендуется к использованию. Повторно создайте приложение как 64-разрядный проект.

Дополнительные сведения вы найдете в следующих статьях:

- [Статья о выполнении 32-разрядных приложений](/windows/desktop/WinProg64/running-32-bit-applications)
- [Статья о поддержке 32-разрядных операционных системах на виртуальных машинах Azure](https://support.microsoft.com/help/4021388/support-for-32-bit-operating-systems-in-azure-virtual-machines)
- [Поддержка серверного ПО Майкрософт для виртуальных машин Microsoft Azure](https://support.microsoft.com/help/2721672/microsoft-server-software-support-for-microsoft-azure-virtual-machines)

## <a name="error-vhd-is-already-registered-with-image-repository-as-the-resource"></a>Ошибка: Виртуальный жесткий диск уже зарегистрирован в репозитории образов как ресурс

Каждый раз при попытке создать образ виртуальных жестких дисков в Azure PowerShell отображается следующее сообщение об ошибке: "VHD уже зарегистрирован в репозитории образов как ресурс". Я не создавал образы раньше, и в Azure отсутствует образ с таким именем. Как решить эту проблему?

Эта проблема обычно возникает, если вы создали виртуальную машину на основе виртуального жесткого диска, который имеет блокировку. Убедитесь, что виртуальная машина не выделяется из такого виртуального жесткого диска, а затем повторите операцию. Если проблема не устранена, отправьте запрос в службу поддержки. См. статью [Поддержка для Центра партнеров](support.md).

## <a name="how-do-i-test-a-hidden-preview-image"></a>Разделы справки протестировать скрытое изображение для предварительного просмотра?

Вы можете развернуть скрытые изображения предварительного просмотра с помощью шаблонов быстрого запуска.
Чтобы развернуть образ предварительной версии Linux, 
1. Перейти к этому [шаблону быстрого запуска](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vm-simple-linux), выберите "развернуть в Azure". Вы должны перейти к портал Azure
2. В портал Azure выберите "изменить шаблон".
3. В шаблоне JSON найдите imageReference и обновите PublisherId, OfferId, skuId и версию образа. Чтобы проверить изображение предварительного просмотра, добавьте "-PREVIEW" в OfferId.
 ![image](https://user-images.githubusercontent.com/79274470/110191995-71c7d500-7de0-11eb-9f3c-6a42f55d8f03.png)
4. Щелкните Сохранить
5. Заполните остальные детали. Проверка и создание



## <a name="next-steps"></a>Дальнейшие действия

- [Устранение неполадок с сертификацией виртуальной машины](azure-vm-create-certification-faq.md)
