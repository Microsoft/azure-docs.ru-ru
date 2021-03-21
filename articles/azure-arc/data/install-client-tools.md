---
title: Установка клиентских средств
description: Установите аздата, kubectl, Azure CLI, PSQL, Azure Data Studio (предварительные оценки) и расширение ARC для Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 6f42f712ecca77c00020304b63f5a1b0dbd77ad0
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102172326"
---
# <a name="install-client-tools-for-deploying-and-managing-azure-arc-enabled-data-services"></a>Установка клиентских средств для развертывания служб данных с поддержкой Azure Arc и управления ими

> [!IMPORTANT]
> При обновлении до нового ежемесячного выпуска необходимо также выполнить обновление до последней версии Azure Data Studio, [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] средства, а также [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] расширений и расширения Azure Arc для Azure Data Studio.

В этом документе описано, как установить [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] , Azure Data Studio, Azure CLI (AZ) и средство CLI Kubernetes (kubectl) на клиентском компьютере.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="tools-for-creating-and-managing-azure-arc-enabled-data-services"></a>Средства для создания и управления службами данных с поддержкой Arc Azure 

В следующей таблице перечислены стандартные средства, необходимые для создания и управления службами данных, включенными в службу "Дуга Azure", и способы их установки.

| Инструмент | Обязательно | Описание | Установка |
|---|---|---|---|
| [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] | Да | Программа командной строки для установки кластера больших данных и управления им. [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] также включает служебную программу командной строки для подключения и запроса экземпляров Azure SQL и SQL Server, а также серверов Postgres с помощью команд `azdata sql query` (выполнение одного запроса из командной строки), `azdata sql shell` (интерактивной оболочки) `azdata postgres query` и `azdata postgres shell` . | [Установка](/sql/azdata/install/deploy-install-azdata?toc=/azure/azure-arc/data/toc.json&bc=/azure/azure-arc/data/breadcrumb/toc.json) |
| Azure Data Studio | Да | Полнофункциональное средство для подключения к различным базам данных, включая Azure SQL, SQL Server, Постргрескл и MySQL, а также запросы к ним. Расширения для Azure Data Studio предоставляют возможности администрирования для служб данных, поддерживающих службу "Дуга Azure". | [Установка](/sql/azure-data-studio/download-azure-data-studio) |
| [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] расширение для Azure Data Studio | Да | Расширение для Azure Data Studio, которое будет установлено, [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] Если у вас его еще нет.| Установка из коллекции расширений в Azure Data Studio.|
| Расширение дуги Azure для Azure Data Studio | Да | Расширение для Azure Data Studio, которое предоставляет возможности управления для служб данных, поддерживающих службу "Дуга Azure". Существует зависимость от [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] расширения для Azure Data Studio. | Установка из коллекции расширений в Azure Data Studio.|
| Расширение PostgreSQL в Azure Data Studio | Нет | Расширение PostgreSQL для Azure Data Studio, предоставляющее возможности управления для PostgreSQL. | <!--{need link} [Install](../azure-data-studio/data-virtualization-extension.md) --> Установка из коллекции расширений в Azure Data Studio.|
| Azure CLI (AZ)<sup>1</sup> | Да | Современный интерфейс командной строки для управления службами Azure. Используется с развертываниями AKS и для передачи данных инвентаризации и выставления счетов в Azure с включенной службой ARC. ([Дополнительные сведения](/cli/azure/)). | [Установка](/cli/azure/install-azure-cli) |
| Kubernetes CLI (kubectl)<sup>2</sup> | Да | Программа командной строки для управления кластером Kubernetes ([Дополнительные сведения](https://kubernetes.io/docs/tasks/tools/install-kubectl/)). | [Windows](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-with-powershell-from-psgallery) \| [Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-using-native-package-management) |
| изогнутое <sup>3</sup> | Требуется для некоторых примеров сценариев. | Программа командной строки для передачи данных по URL-адресам. | [Windows](https://curl.haxx.se/windows/) \| Linux: установите пакет cURL |
| даваемым | Требуется для развертываний Red Hat OpenShift и Azure Redhat OpenShift. |`oc` — это интерфейс командной строки (CLI) в Open Shift. | [Установка CLI](https://docs.openshift.com/container-platform/4.4/cli_reference/openshift_cli/getting-started-cli.html#installing-the-cli)



<sup>1</sup> необходимо использовать Azure CLI версии 2.0.4 или более поздней. При необходимости выполните команду `az --version`, чтобы определить версию.

<sup>2</sup> необходимо использовать `kubectl` версию 1,13 или более позднюю. Кроме того, версия `kubectl` должна отстоять от младшей версии кластера Kubernetes не более чем на единицу. Если вы хотите установить определенную версию в клиенте `kubectl`, см. статью [Установка двоичных файлов `kubectl` при помощи curl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-binary-using-curl) (в Windows 10 для запуска curl используйте cmd.exe, а не Windows PowerShell).

<sup>3</sup> если вы используете PowerShell, то фигурный псевдоним для командлета Invoke-WebRequest.

## <a name="next-steps"></a>Дальнейшие действия

[Создание контроллера данных ARC в Azure](create-data-controller.md)