---
title: Оптимизации пропускной способности сети для виртуальной машины | Документация Майкрософт
description: Оптимизируйте пропускную способность сети для Microsoft Azure виртуальных машин Windows и Linux, включая такие основные дистрибутивы, как Ubuntu, CentOS и Red Hat.
services: virtual-network
documentationcenter: na
author: steveesp
manager: Gerald DeGrace
editor: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/06/2020
ms.author: steveesp
ms.openlocfilehash: bb9235f4d1190bf7f71ddc007f09c9666c353234
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98216807"
---
# <a name="optimize-network-throughput-for-azure-virtual-machines"></a>Оптимизации пропускной способности сети для виртуальной машины Azure

Виртуальные машины Azure имеют сетевые параметры по умолчанию, с помощью которых можно дополнительно оптимизировать пропускную способность сети. Из этой статьи вы узнаете, как оптимизировать пропускную способность сети для виртуальных машин Microsoft Azure, которые работают под управлением Windows и Linux, включая такие основные дистрибутивы, как Ubuntu, CentOS и Red Hat.

## <a name="windows-vm"></a>Виртуальная машина Windows

Если виртуальная машина Windows поддерживает [ускорение работы в сети](create-vm-accelerated-networking-powershell.md), включив эту функцию, можно оптимизировать конфигурацию для пропускной способности. Остальные виртуальные машины Windows, использующие масштабирование размера приема (RSS), могут иметь большую максимальную пропускную способность, чем виртуальная машина без RSS. По умолчанию функция RSS может быть отключена на виртуальной машине Windows. Чтобы определить, включена ли функция RSS, и при необходимости включить ее, выполните следующие действия:

1. Определите, включена ли функция RSS для сетевого адаптера при помощи команды PowerShell `Get-NetAdapterRss`. В следующем примере показаны результаты выполнения команды `Get-NetAdapterRss`, когда функция RSS не включена.

    ```powershell
    Name                    : Ethernet
    InterfaceDescription    : Microsoft Hyper-V Network Adapter
    Enabled                 : False
    ```
2. Чтобы включить RSS, введите следующую команду:

    ```powershell
    Get-NetAdapter | % {Enable-NetAdapterRss -Name $_.Name}
    ```
    Предыдущая команда не имеет выходные данные. Команда изменяет параметры сетевого адаптера, в результате чего около одной минуты отсутствует подключение. В это время появится диалоговое окно повторного подключения. Обычно после третьей попытки подключение восстанавливается.
3. Убедитесь, что функция RSS включена на виртуальной машине. Для этого еще раз выполните команду `Get-NetAdapterRss`. При успешном выполнении возвращается следующий результат:

    ```powershell
    Name                    : Ethernet
    InterfaceDescription    : Microsoft Hyper-V Network Adapter
    Enabled                  : True
    ```

## <a name="linux-vm"></a>Виртуальная машина Linux

Функция RSS по умолчанию всегда включена на виртуальной машине Azure по управлением Linux. Ядра Linux, выпущенные после октября 2017 года, включают новые параметры оптимизации сети, которые обеспечивают более высокую пропускную способность сети для виртуальной машины Linux.

### <a name="ubuntu-for-new-deployments"></a>Ubuntu для новых развертываний

Ядро Azure Ubuntu является наиболее оптимизированным для производительности сети в Azure. Чтобы получить последние оптимизации, сначала установите последнюю поддерживаемую версию 18,04-LTS, как показано ниже.

```json
"Publisher": "Canonical",
"Offer": "UbuntuServer",
"Sku": "18.04-LTS",
"Version": "latest"
```

Затем выполните следующие команды, чтобы получить последние обновления. Эти шаги также применимы к виртуальным машинам, которые сейчас выполняются на базе ядра Ubuntu Azure.

```bash
#run as root or preface with sudo
apt-get -y update
apt-get -y upgrade
apt-get -y dist-upgrade
```

Следующий набор необязательных команд может быть полезным для существующих развертываний Ubuntu на базе ядра Azure, но для которых не удалось выполнить дальнейшие обновления без ошибок.

```bash
#optional steps may be helpful in existing deployments with the Azure kernel
#run as root or preface with sudo
apt-get -f install
apt-get --fix-missing install
apt-get clean
apt-get -y update
apt-get -y upgrade
apt-get -y dist-upgrade
```

#### <a name="ubuntu-azure-kernel-upgrade-for-existing-vms"></a>Обновление ядра Ubuntu Azure для существующих виртуальных машин

Обновив ядро Azure Linux, можно существенно повысить пропускную способность. Чтобы определить необходимость обновления, проверьте вашу версию ядра. Оно должно быть таким же или более поздним, чем в примере.

```bash
#Azure kernel name ends with "-azure"
uname -r

#sample output on Azure kernel:
#4.13.0-1007-azure
```

Если на виртуальной машине нет ядра Azure, номер версии обычно начинается с цифр 4.4. Если на виртуальной машине нет ядра Azure, выполните следующие команды как корневые:

```bash
#run as root or preface with sudo
apt-get update
apt-get upgrade -y
apt-get dist-upgrade -y
apt-get install "linux-azure"
reboot
```

### <a name="centos"></a>CentOS

Чтобы получить последние оптимизации, лучше всего создать виртуальную машину, используя последнюю поддерживаемую версию, указав следующие параметры:

```json
"Publisher": "OpenLogic",
"Offer": "CentOS",
"Sku": "7.7",
"Version": "latest"
```

Вы можете воспользоваться преимуществами использования новых и существующих виртуальных машин, установив последнюю версию служб интеграции Linux (LIS). Оптимизация пропускной способности реализована в LIS начиная с 4.2.2-2. Более поздние версии содержат дополнительные усовершенствования. Введите следующие команды для установки новейшей версии LIS:

```bash
sudo yum update
sudo reboot
sudo yum install microsoft-hyper-v
```

### <a name="red-hat"></a>Red Hat

Чтобы получить оптимизации, лучше всего создать виртуальную машину, используя последнюю поддерживаемую версию, указав следующие параметры:

```json
"Publisher": "RedHat"
"Offer": "RHEL"
"Sku": "7-RAW"
"Version": "latest"
```

Вы можете воспользоваться преимуществами использования новых и существующих виртуальных машин, установив последнюю версию служб интеграции Linux (LIS). Возможность оптимизации пропускной способности предусмотрена в службах LIS начиная с версии 4.2. Выполните следующие команды, чтобы загрузить и установить LIS:

```bash
wget https://aka.ms/lis
tar xvf lis
cd LISISO
sudo ./install.sh #or upgrade.sh if prior LIS was previously installed
```

Дополнительные сведения о версии 4.2 служб интеграции Linux (LIS) для Hyper-V см. на [странице скачивания](https://www.microsoft.com/download/details.aspx?id=55106).

## <a name="next-steps"></a>Дальнейшие действия
* Развертывание виртуальных машин близко друг к другу для низкой задержки с [группой размещения с близкой](../virtual-machines/co-location.md) назначением
* Просмотрите оптимизированные результаты для своего сценария, используя сведения в статье [Проверка пропускной способности (NTTTCP)](virtual-network-bandwidth-testing.md).
* Узнайте о том [, как пропускная способность выделяется виртуальным машинам](virtual-machine-network-throughput.md) .
* Дополнительные сведения см. в статье [Виртуальная сеть Azure: часто задаваемые вопросы](virtual-networks-faq.md).