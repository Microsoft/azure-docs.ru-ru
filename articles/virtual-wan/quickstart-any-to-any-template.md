---
title: Краткое руководство. Создание конфигурации подключения "любой к любому" с помощью шаблона ARM
titleSuffix: Azure Virtual WAN
description: В этом кратком руководстве показано, как создать конфигурацию подключения "любой к любому" с помощью шаблона Resource Manager (шаблона ARM).
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: quickstart
ms.date: 02/02/2021
ms.author: cherylmc
ms.custom: subject-armqs
ms.openlocfilehash: d31f490baec49e8e0b6fcf89caa8c19202fdf763
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102431386"
---
# <a name="quickstart-create-an-any-to-any-configuration-using-an-arm-template"></a>Краткое руководство. Создание конфигурации подключения "любой к любому" с помощью шаблона ARM

В этом кратком руководстве показано, как с помощью шаблона Resource Manager (шаблона ARM) создать сценарий подключения "любой к любому", когда каждый периферийный сервер может обращаться к другому такому серверу.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Если среда соответствует предварительным требованиям и вы знакомы с использованием шаблонов ARM, нажмите кнопку **Развертывание в Azure**. Шаблон откроется на портале Azure.

[![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f201-virtual-wan-with-all-gateways%2fazuredeploy.json)

## <a name="prerequisites"></a>Предварительные требования

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
* Для этой конфигурации требуются данные сертификата открытого ключа. Пример таких данных приведен в статье. Но этот пример предоставляется только для того, чтобы обеспечить соответствие требованиям для шаблона при создании шлюза P2S. Завершив выполнение шаблона и развертывание ресурсов, вам необходимо указать в этом поле данные вашего сертификата, чтобы эта конфигурация работала. См. сведения о [пользовательских сертификатах VPN](certificates-point-to-site.md#cer).

## <a name="review-the-template"></a><a name="review"></a>Изучение шаблона

Шаблон, используемый в этом кратком руководстве, взят из [шаблонов быстрого запуска Azure](https://azure.microsoft.com/resources/templates/201-virtual-wan-with-all-gateways). Шаблон для этой статьи слишком длинный, чтобы его можно было здесь отобразить. Вы можете просмотреть этот шаблон в файле [azuredeploy.json](https://github.com/Azure/azure-quickstart-templates/blob/master/201-virtual-wan-with-all-gateways/azuredeploy.json).

В этом кратком руководстве показано, как создать развертывание Виртуальной глобальной сети Azure с несколькими концентраторами, включая все необходимые шлюзы и подключения к виртуальной сети. Список входных параметров намеренно сведен к минимально возможному. Схему IP-адресации можно изменить, изменив переменные в этом шаблоне. Этот сценарий подробнее описан в статье о [сценарии подключения "любой к любому"](scenario-any-to-any.md).

:::image type="content" source="./media/routing-scenarios/any-any/figure-1.png" alt-text="Архитектура развертывания":::

Этот шаблон создает полнофункциональную среду Виртуальной глобальной сети Azure со следующими ресурсами:

* два разных концентратора в разных регионах;
* четыре виртуальные сети Azure;
* два подключения к виртуальной сети для каждого концентратора глобальной виртуальной сети;
* один VPN-шлюз типа "точка — сеть" (P2S) в каждом концентраторе;
* один VPN-шлюз типа "сеть — сеть" (S2S) в каждом концентраторе;
* один шлюз ExpressRoute в каждом концентраторе.

В шаблоне определено несколько ресурсов Azure:

* [**Microsoft.Network/virtualwans**](/azure/templates/microsoft.network/virtualwans)
* [**Microsoft.Network/virtualhubs**](/azure/templates/microsoft.network/virtualhubs)
* [**Microsoft.Network/virtualnetworks**](/azure/templates/microsoft.network/virtualnetworks)
* [**Microsoft.Network/hubvirtualnetworkconnections**](/azure/templates/microsoft.network/virtualhubs/hubvirtualnetworkconnections)
* [**Microsoft.Network/p2svpngateways**](/azure/templates/microsoft.network/p2svpngateways)
* [**Microsoft.Network/vpngateways**](/azure/templates/microsoft.network/vpngateways) 
* [**Microsoft.Network/expressroutegateways**](/azure/templates/microsoft.network/expressroutegateways)
* [**Microsoft.Network/vpnserverconfigurations**](/azure/templates/microsoft.network/vpnserverconfigurations)

>[!NOTE]
> Этот шаблон ARM не создает на стороне клиента ресурсы, требуемые для гибридного подключения. После развертывания шаблона вам придется дополнительно создать и настроить VPN-клиенты P2S, ветви VPN (локальные сайты) и подключить каналы ExpressRoute.
>

Дополнительные примеры шаблонов быстрого запуска для Azure вы найдете [здесь](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a><a name="deploy"></a>Развертывание шаблона

Чтобы правильно развернуть этот шаблон, необходимо использовать кнопку "Развернуть в Azure" и портал Azure, но не другие методы. Причины такого ограничения:

* Чтобы создать конфигурацию P2S, необходимо передать данные корневого сертификата. При использовании PowerShell или CLI поле данных не принимает данные в формате сертификата.
* Этот шаблон неправильно работает с Cloud Shell из-за передачи данных сертификата.
* Кроме того, на портале вы можете легко изменять шаблон и любые параметры, чтобы они соответствовали используемым диапазонам IP-адресов и другим значениям.

1. Нажмите кнопку **Развернуть в Azure**.

   [![Развертывание в Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f201-virtual-wan-with-all-gateways%2fazuredeploy.json)
1. Чтобы просмотреть шаблон, щелкните **Изменить шаблон**. На этой странице вы можете изменить некоторые значения, например адресное пространство или имена определенных ресурсов. Кнопки **Сохранить** и **Отменить** позволяют выбрать, что нужно сделать с внесенными изменениями.
1. На странице шаблона введите нужные значения. Для этого шаблона требуются данные открытого сертификата P2S. Если вы используете эту статью, выполняя упражнение, можно применить для обоих концентраторов следующие данные из CER-файла в примере. Когда завершится выполнение шаблона и развертывание решения, для работы в конфигурации P2S эту информацию необходимо заменить [данными сертификата](certificates-point-to-site.md#cer) открытого ключа, созданного для вашего развертывания.

   ```certificate-data
   MIIC5zCCAc+gAwIBAgIQGxd3Av1q6LJDZ71e3TzqcTANBgkqhkiG9w0BAQsFADAW
   MRQwEgYDVQQDDAtQMlNSb290Q2VydDAeFw0yMDExMDkyMjMxNTVaFw0yMTExMDky
   MjUxNTVaMBYxFDASBgNVBAMMC1AyU1Jvb3RDZXJ0MIIBIjANBgkqhkiG9w0BAQEF
   AAOCAQ8AMIIBCgKCAQEA33fFra/E0YmGuXLKmYcdvjsYpKwQmw8DjjDkbwhE9jcc
   Dp50e7F1P6Rxo1T6Hm3dIhEji+0QkP4Ie0XPpw0eW77+RWUiG9XJxGqtJ3Q4tyRy
   vBfsHORcqMlpV3VZOXIxrk+L/1sSm2xAc2QGuOqKaDNNoKmjrSGNVAeQHigxbTQg
   zCcyeuhFxHxAaxpW0bslK2hEZ9PhuAe22c2SHht6fOIDeXkadzqTFeV8wEZdltLr
   6Per0krxf7N2hFo5Cfz0KgWlvgdKLL7dUc9cjHo6b6BL2pNbLh8YofwHQOQbwt6H
   miAkEnx1EJ5N8AWuruUTByR2jcWyCnEAUSH41+nk4QIDAQABozEwLzAOBgNVHQ8B
   Af8EBAMCAgQwHQYDVR0OBBYEFJMgnJSYHH5AJ+9XB11usKRwjbjNMA0GCSqGSIb3
   DQEBCwUAA4IBAQBOy8Z5FBd/nvgDcjvAwNCw9h5RHzgtgQqDP0qUjEqeQv3ALeC+
   k/F2Tz0OWiPEzX5N+MMrf/jiYsL2exXuaPWCF5U9fu8bvs89GabHma8MGU3Qua2x
   Imvt0whWExQMjoyU8SNUi2S13fnRie9ZlSwNh8B/OIUUEtVhQsd4OfuZZFVH4xGp
   ibJMSMe5JBbZJC2tCdSdTLYfYJqrLkVuTjynXOjmz2JXfwnDNqEMdIMMjXzlNavR
   J8SNtAoptMOK5vAvlySg4LYtFyXkl0W0vLKIbbHf+2UszuSCijTUa3o/Y1FoYSfi
   eJH431YTnVLuwdd6fXkXFBrXDhjNsU866+hE
   ```

1. Завершив ввод значений, щелкните **Просмотр и создание**.
1. На странице **Просмотр и создание** дождитесь завершения проверки и щелкните **Создать**.
1. Завершение развертывания займет примерно 75 минут. Ход его выполнения можно отслеживать на **странице обзора** шаблона.  Вы можете спокойно закрыть портал, это не повлияет на ход развертывания.

   :::image type="content" source="./media/quickstart-any-to-any-template/template.png" alt-text="Пример успешно завершенного развертывания":::

## <a name="validate-the-deployment"></a><a name="validate"></a>Проверка развертывания

1. Войдите на [портал Azure](https://portal.azure.com).
1. В области слева выберите **Группы ресурсов**.
1. Выберите группу ресурсов, созданную при работе с предыдущим разделом. На **странице обзор** а вы увидите примерно следующее: :::image type="content" source="./media/quickstart-any-to-any-template/resources.png" alt-text="Пример ресурсов" lightbox="./media/quickstart-any-to-any-template/resources.png":::

1. Щелкните виртуальную глобальную сеть, чтобы открыть список концентраторов. На странице виртуальной глобальной сети можно щелкнуть каждый концентратор, чтобы просмотреть сведения о его подключениях и других параметрах.
   :::image type="content" source="./media/quickstart-any-to-any-template/hub.png" alt-text="Примеры концентраторов" lightbox="./media/quickstart-any-to-any-template/hub.png":::

## <a name="complete-the-hybrid-configuration"></a><a name="complete"></a>Завершение гибридной конфигурации

Этот шаблон настраивает не все параметры, требуемые для гибридной сети. В зависимости от конкретных требований вам необходимо дополнительно подготовить следующие конфигурации и настройки.

* [Настройка ветвей VPN — локальные сайты](virtual-wan-site-to-site-portal.md#site)
* [Завершение настройки VPN для P2S](virtual-wan-point-to-site-portal.md)
* [Подключение каналов ExpressRoute](virtual-wan-expressroute-portal.md)

## <a name="clean-up-resources"></a>Очистка ресурсов

Если созданные ресурсы вам больше не нужны, удалите их. Некоторые ресурсы Виртуальной глобальной сети необходимо удалять в определенном порядке с учетом зависимостей. Удаление может занять около 30 минут.

[!INCLUDE [Delete resources](../../includes/virtual-wan-resource-cleanup.md)]

## <a name="next-steps"></a>Следующие шаги

> [!div class="nextstepaction"]
> [Завершение настройки VPN для P2S](virtual-wan-point-to-site-portal.md)

> [!div class="nextstepaction"]
> [Настройка ветвей VPN — локальные сайты](virtual-wan-site-to-site-portal.md#site)

> [!div class="nextstepaction"]
> [Подключение каналов ExpressRoute](virtual-wan-expressroute-portal.md)