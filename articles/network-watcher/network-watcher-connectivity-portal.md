---
title: Устранение неполадок соединений — портал Azure
titleSuffix: Azure Network Watcher
description: Узнайте, как использовать функцию устранения неполадок подключений с помощью службы "Наблюдатель за сетями Azure" с помощью портала Azure.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/04/2021
ms.author: damendo
ms.openlocfilehash: f33c5f0fdf69737df0d8bd83499ded1e0e0f4f88
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97898119"
---
# <a name="troubleshoot-connections-with-azure-network-watcher-using-the-azure-portal"></a>Устранение неполадок подключений с помощью службы "Наблюдатель за сетями Azure" с помощью портала Azure

> [!div class="op_single_selector"]
> - [Портал](network-watcher-connectivity-portal.md)
> - [PowerShell](network-watcher-connectivity-powershell.md)
> - [Azure CLI](network-watcher-connectivity-cli.md)
> - [Azure REST API](network-watcher-connectivity-rest.md)

Узнайте, как проверить возможность прямого подключения TCP между виртуальной машиной и определенной конечной точкой с помощью функции устранения неполадок подключения.

## <a name="before-you-begin"></a>Перед началом

В данной статье предполагается, что у вас есть следующие ресурсы:

* Экземпляр службы "Наблюдатель за сетями" в регионе, в котором нужно устранить проблему подключения.
* Виртуальные машины, на которых требуется устранить неполадки подключения.

> [!IMPORTANT]
> Чтобы устранить неполадки подключения на виртуальной машине, установите на ней расширение `AzureNetworkWatcherExtension`. Информацию об установке расширения для виртуальной машины Windows см. в статье [Расширение виртуальной машины агента Наблюдателя за сетями для Windows](../virtual-machines/extensions/network-watcher-windows.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json), а для виртуальной машины Linux — в статье [Расширение виртуальной машины агента Наблюдателя за сетями для Linux](../virtual-machines/extensions/network-watcher-linux.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json). Эта расширение не нужно устанавливать на целевой конечной точке.

## <a name="check-connectivity-to-a-virtual-machine"></a>Проверка возможности подключения к виртуальной машине

В этом примере проверяется возможность подключения к целевой виртуальной машине через порт 80.

Перейдите к службе "Наблюдатель за сетями" и щелкните **Устранение неполадок с подключением**. Выберите виртуальную машину, чтобы проверить ее возможность подключения. В разделе **Место назначения** щелкните **Выберите виртуальную машину** и выберите соответствующие виртуальную машины и порт для тестирования.

После щелчка **Проверить** будет проверено подключение между виртуальными машинами через указанный порт. В примере целевая виртуальная машина недоступна и показан список прыжков.

![Результаты проверки возможности подключения виртуальной машины][1]

## <a name="check-remote-endpoint-connectivity"></a>Проверка возможности подключения к удаленной конечной точке

Чтобы проверить возможность подключения и задержки для удаленной конечной точки, установите переключатель **Указать вручную** в разделе **Место назначения**, введите URL-адрес и порт, после чего щелкните **Проверить**.  Это используется для таких удаленных конечных точек, такие как конечные точки веб-сайтов и хранилищ.

![Результаты проверки возможности подключения веб-сайта][2]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об автоматизации записи пакетов с помощью оповещений на виртуальной машине см. в статье, посвященной [созданию записи пакетов, активируемой с использованием оповещений](network-watcher-alert-triggered-packet-capture.md).

Сведения о состоянии (разрешен или запрещен) входящего и исходящего трафика виртуальной машины см. в статье, посвященной [проверке потока IP-адресов](diagnose-vm-network-traffic-filtering-problem.md).

[1]: ./media/network-watcher-connectivity-portal/figure1.png
[2]: ./media/network-watcher-connectivity-portal/figure2.png