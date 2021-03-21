---
title: Установка драйвера GPU для виртуальных машин Azure серии N в Linux
description: Как установить драйверы NVIDIA GPU для виртуальных машин серии N под управлением Linux в Azure
services: virtual-machines
author: vikancha-MSFT
ms.service: virtual-machines
ms.subervice: vm-sizes-gpu
ms.collection: linux
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 11/11/2019
ms.author: vikancha
ms.openlocfilehash: c4c6bee6d3f9e423d83458ad48d213fe65223514
ms.sourcegitcommit: e6de1702d3958a3bea275645eb46e4f2e0f011af
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102551766"
---
# <a name="install-nvidia-gpu-drivers-on-n-series-vms-running-linux"></a>Установка драйверов GPU NVIDIA на виртуальные машины серии N под управлением Linux

Чтобы воспользоваться всеми преимуществами GPU виртуальных машин Azure серии N, необходимо установить драйверы GPU NVIDIA. [Расширение драйвера GPU NVIDIA](../extensions/hpccompute-gpu-linux.md) устанавливает необходимые драйверы CUDA или GRID NVIDIA на виртуальную машину серии N. Для установки расширения и управления им можно использовать портал Azure или такие инструменты, как Azure CLI и шаблоны Azure Resource Manager. Сведения о поддерживаемых дистрибутивах и этапах развертывания см. в [документации по расширению драйвера GPU NVIDIA](../extensions/hpccompute-gpu-linux.md).

Если вы решили установить драйверы GPU NVIDIA вручную, то в этой статье вы найдете сведения о поддерживаемых дистрибутивах, ссылки на драйверы и инструкции по установке и проверке. Сведения о ручной установке драйверов также доступны для [виртуальных машин Windows](../windows/n-series-driver-setup.md).

Характеристики виртуальных машин серии N, сведения о дисках и объеме памяти см. в статье [Размеры виртуальных машин Linux, оптимизированных для GPU](../sizes-gpu.md?toc=/azure/virtual-machines/linux/toc.json). 

[!INCLUDE [virtual-machines-n-series-linux-support](../../../includes/virtual-machines-n-series-linux-support.md)]

## <a name="install-cuda-drivers-on-n-series-vms"></a>Установка драйверов CUDA на виртуальные машины серии N

Ниже приведены инструкции по установке драйверов CUDA из набора средств NVIDIA CUDA Toolkit на виртуальные машины серии N. 

Разработчики на языках C и C++ могут дополнительно установить полный набор средств для создания приложений, использующих ускорение на GPU. Дополнительные сведения см. в [руководстве по установке CUDA](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html).

Чтобы установить драйверы CUDA, установите SSH-подключение к каждой виртуальной машине. Убедитесь, что в системе есть графический процессор с архитектурой CUDA, выполнив следующую команду:

```bash
lspci | grep -i NVIDIA
```
Вы увидите данные, похожие на следующий пример (для адаптера NVIDIA Tesla K80):

![результаты выполнения команды lspci](./media/n-series-driver-setup/lspci.png)

lspci перечисляет устройства PCIe на виртуальной машине, включая сетевую карту InfiniBand и GPU (если они есть). Если lspci не возвращается, может потребоваться установить LIS на CentOS или RHEL (см. инструкции ниже).
Затем выполните команды установки, относящиеся к используемому дистрибутиву.

### <a name="ubuntu"></a>Ubuntu 

1. Скачайте драйверы CUDA на веб-сайте NVIDIA и установите их. 
    > [!NOTE]
   >  В примере ниже показан путь к пакету CUDA для Ubuntu 16,04. Замените путь, относящийся к версии, которую вы планируете использовать. 
   >  
   >  Посетите [Центр загрузки NVIDIA] ( https://developer.download.nvidia.com/compute/cuda/repos/) полный путь, относящийся к каждой версии. 
   > 
   ```bash
   CUDA_REPO_PKG=cuda-repo-ubuntu1604_10.0.130-1_amd64.deb
   wget -O /tmp/${CUDA_REPO_PKG} https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/${CUDA_REPO_PKG} 

   sudo dpkg -i /tmp/${CUDA_REPO_PKG}
   sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/7fa2af80.pub 
   rm -f /tmp/${CUDA_REPO_PKG}

   sudo apt-get update
   sudo apt-get install cuda-drivers
   ```

   Установка может занять несколько минут.
 

2. Чтобы дополнительно установить полный набор средств CUDA, введите следующую команду.

   ```bash
   sudo apt-get install cuda
   ```

3. Перезапустите виртуальную машину и приступите к проверке установки.

#### <a name="cuda-driver-updates"></a>Обновления драйверов CUDA

Рекомендуется периодически обновлять драйверы CUDA после развертывания.

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
sudo apt-get install cuda-drivers

sudo reboot
```

### <a name="centos-or-red-hat-enterprise-linux"></a>CentOS или Red Hat Enterprise Linux

1. Обновите ядро (рекомендуется). Если вы решите не делать это, убедитесь, что версии `kernel-devel` и `dkms` соответствуют ядру.

   ```
   sudo yum install kernel kernel-tools kernel-headers kernel-devel
   sudo reboot
   ```

2. Установите последнюю версию [Linux Integration Services для Hyper-V и Azure](https://www.microsoft.com/download/details.aspx?id=55106). Проверьте, требуется ли LIS, проверив результаты lspci. Если все устройства GPU перечислены должным образом (и описаны выше), установка LIS не требуется.

   Обратите внимание, что LIS применим к Red Hat Enterprise Linux, CentOS и Oracle Linuxам ядра 5.2-5.11, 6.0-6.10 и 7.0-7.7, совместимых с Red Hat. Ознакомьтесь с [документацией по Linux Integration Services] ( https://www.microsoft.com/en-us/download/details.aspx?id=55106) для получения дополнительных сведений. 
   Пропустите этот шаг, если вы планируете использовать CentOS/RHEL 7,8 (или более поздние версии), так как LIS больше не требуется для этих версий.

      ```bash
      wget https://aka.ms/lis
      tar xvzf lis
      cd LISISO

      sudo ./install.sh
      sudo reboot
      ```

3. Повторно подключитесь к виртуальной машине и продолжите установку с помощью следующих команд:

   ```bash
   sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   sudo yum install dkms
   
   CUDA_REPO_PKG=cuda-repo-rhel7-10.0.130-1.x86_64.rpm
   wget https://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/${CUDA_REPO_PKG} -O /tmp/${CUDA_REPO_PKG}

   sudo rpm -ivh /tmp/${CUDA_REPO_PKG}
   rm -f /tmp/${CUDA_REPO_PKG}

   sudo yum install cuda-drivers
   ```

   Установка может занять несколько минут. 

4. Чтобы дополнительно установить полный набор средств CUDA, введите следующую команду.

   ```bash
   sudo yum install cuda
   ```
   > [!NOTE]
   >  Если отображается сообщение об ошибке, связанное с отсутствующими пакетами, например вулкан-FileSystem, то, возможно, потребуется изменить/ЕТК/Юм.Репос.д/РХ-Клауд, найти Optional-RPMS и задать для параметра Enabled значение 1.
   >  

5. Перезапустите виртуальную машину и приступите к проверке установки.

### <a name="verify-driver-installation"></a>Проверка установки драйверов

Чтобы запросить состояние устройства GPU, установите SSH-подключение к виртуальной машине и запустите служебную программу командной строки [nvidia smi](https://developer.nvidia.com/nvidia-system-management-interface), установленную вместе с драйвером. 

Если драйвер установлен, то отобразятся выходные данные, аналогичные приведенным ниже. Обратите внимание, что **GPU-Util** отобразит 0 %, если в данный момент графический процессор не выполняет рабочую нагрузку на виртуальной машине. Версия драйвера и сведения о GPU могут отличаться от показанных на изображении.

![Состояние устройства NVIDIA](./media/n-series-driver-setup/smi.png)

## <a name="rdma-network-connectivity"></a>Сетевое подключение RDMA

Сетевое подключение RDMA можно включить на виртуальных машинах серии N с поддержкой RDMA, таких как NC24r, развернутых в той же группе доступности или в пределах одной группы размещения в масштабируемом наборе виртуальных машин. Сеть RDMA поддерживает трафик MPI (Message Passing Interface) для приложений, использующих Intel MPI 5.x или более поздней версии. Дополнительные требования приведены ниже.

### <a name="distributions"></a>Дистрибутивы

Разверните виртуальные машины серии N с поддержкой RDMA из одного из следующих образов Azure Marketplace, поддерживающих подключение RDMA на виртуальных машинах серии N:
  
* **Ubuntu 16.04 LTS**. Настройте драйверы RDMA на виртуальной машине и выполните регистрацию на сайте Intel, чтобы скачать Intel MPI.

  [!INCLUDE [virtual-machines-common-ubuntu-rdma](../../../includes/virtual-machines-common-ubuntu-rdma.md)]

* **HPC версии 7.4 на основе CentOS.** Драйверы RDMA и Intel MPI 5.1 будут установлены на виртуальной машине.

* **HPC на основе CentOS** — CENTOS-HPC 7,6 и более поздних версий (для SKU, где InfiniBand поддерживается поверх SR-IOV). На этих изображениях предварительно установлены библиотеки Mellanox ОФЕД и MPI.

> [!NOTE]
> Карты CX3-Pro поддерживаются только с помощью LTS версий Mellanox ОФЕД. Используйте LTS Mellanox ОФЕД Version (4.9-0.1.7.0) на виртуальных машинах серии N с картами ConnectX3-Pro. Дополнительные сведения см. в статье [драйверы Linux](https://www.mellanox.com/products/infiniband-drivers/linux/mlnx_ofed).
>
> Кроме того, некоторые из последних образов HPC для Azure Marketplace имеют Mellanox ОФЕД 5,1 и более поздних версий, которые не поддерживают карты ConnectX3-Pro. Проверьте версию ОФЕД для Mellanox в образе HPC, прежде чем использовать ее на виртуальных машинах с картами ConnectX3-Pro.
>
> Ниже приведены последние образы CentOS-HPC, поддерживающие карты ConnectX3-Pro:
>
> - OpenLogic: CentOS-HPC: 7.6:7.6.2020062900
> - OpenLogic: CentOS-HPC: 7_6gen2:7.6.2020062901
> - OpenLogic: CentOS-HPC: 7.7:7.7.2020062600
> - OpenLogic: CentOS-HPC: 7_7-Gen2:7.7.2020062601
> - OpenLogic: CentOS-HPC: 8_1:8.1.2020062400
> - OpenLogic: CentOS-HPC: 8_1-Gen2:8.1.2020062401
>

## <a name="install-grid-drivers-on-nv-or-nvv3-series-vms"></a>Установка драйверов GRID на виртуальные машины серии NV или NVv3

Чтобы установить драйверы GRID NVIDIA на виртуальных машинах серии NV или NVv3, подключитесь по протоколу SSH к каждой виртуальной машине и выполните действия, необходимые для вашего дистрибутива Linux. 

### <a name="ubuntu"></a>Ubuntu 

1. Выполните команду `lspci`. Убедитесь, что карта или карты NVIDIA M60 отображаются как устройства PCI.

2. Установите обновления.

   ```bash
   sudo apt-get update
   sudo apt-get upgrade -y
   sudo apt-get dist-upgrade -y
   sudo apt-get install build-essential ubuntu-desktop -y
   sudo apt-get install linux-azure -y
   ```
3. Отключите драйвер ядра Nouveau, который несовместим с драйвером NVIDIA. (На виртуальных машинах NV или NVv2 используйте только драйвер NVIDIA.) Чтобы сделать это, создайте в `/etc/modprobe.d` файл с именем `nouveau.conf` и следующим содержимым:

   ```
   blacklist nouveau
   blacklist lbm-nouveau
   ```


4. Перезагрузите виртуальную машину и подключитесь повторно. Выйдите из X-сервера:

   ```bash
   sudo systemctl stop lightdm.service
   ```

5. Загрузите и установите драйвер GRID.

   ```bash
   wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=874272  
   chmod +x NVIDIA-Linux-x86_64-grid.run
   sudo ./NVIDIA-Linux-x86_64-grid.run
   ``` 

6. Когда появится запрос на запуск служебной программы nvidia-xconfig для обновления файла конфигурации X, выберите **Да**.

7. После завершения установки скопируйте /etc/nvidia/gridd.conf.template в новый файл gridd.conf в расположении /etc/nvidia/.

   ```bash
   sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
   ```

8. Добавьте следующую строку в файл `/etc/nvidia/gridd.conf`:
 
   ```
   IgnoreSP=FALSE
   EnableUI=FALSE
   ```
   
9. Удалите следующее содержимое из `/etc/nvidia/gridd.conf`, если оно присутствует:
 
   ```
   FeatureType=0
   ```
10. Перезапустите виртуальную машину и приступите к проверке установки.


### <a name="centos-or-red-hat-enterprise-linux"></a>CentOS или Red Hat Enterprise Linux 

1. Обновите ядро и DKMS (рекомендуется). Если вы решите не делать это, убедитесь, что версии `kernel-devel` и `dkms` соответствуют ядру.
 
   ```bash  
   sudo yum update
   sudo yum install kernel-devel
   sudo rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
   sudo yum install dkms
   sudo yum install hyperv-daemons
   ```

2. Отключите драйвер ядра Nouveau, который несовместим с драйвером NVIDIA. (Используйте драйвер NVIDIA только на виртуальных машинах NV или NV3.) Для этого создайте файл с `/etc/modprobe.d` именем `nouveau.conf` со следующим содержимым:

   ```
   blacklist nouveau
   blacklist lbm-nouveau
   ```

3. Перезагрузите виртуальную машину, повторно подключитесь и установите последнюю версию [служб интеграции Linux для Hyper-V и Azure](https://www.microsoft.com/download/details.aspx?id=55106). Проверьте, требуется ли LIS, проверив результаты lspci. Если все устройства GPU перечислены должным образом (и описаны выше), установка LIS не требуется. 

   Пропустите этот шаг, если вы планируете использовать CentOS/RHEL 7,8 (или более поздние версии), так как LIS больше не требуется для этих версий.

      ```bash
      wget https://aka.ms/lis
      tar xvzf lis
      cd LISISO

      sudo ./install.sh
      sudo reboot

      ```
 
4. Повторно подключитесь к виртуальной машине и выполните команду `lspci`. Убедитесь, что карта или карты NVIDIA M60 отображаются как устройства PCI.
 
5. Загрузите и установите драйвер GRID.

   ```bash
   wget -O NVIDIA-Linux-x86_64-grid.run https://go.microsoft.com/fwlink/?linkid=874272  
   chmod +x NVIDIA-Linux-x86_64-grid.run

   sudo ./NVIDIA-Linux-x86_64-grid.run
   ``` 
6. Когда появится запрос на запуск служебной программы nvidia-xconfig для обновления файла конфигурации X, выберите **Да**.

7. После завершения установки скопируйте /etc/nvidia/gridd.conf.template в новый файл gridd.conf в расположении /etc/nvidia/.
  
   ```bash
   sudo cp /etc/nvidia/gridd.conf.template /etc/nvidia/gridd.conf
   ```
  
8. Добавьте следующую строку в файл `/etc/nvidia/gridd.conf`:
 
   ```
   IgnoreSP=FALSE
   EnableUI=FALSE 
   ```
9. Удалите следующее содержимое из `/etc/nvidia/gridd.conf`, если оно присутствует:
 
   ```
   FeatureType=0
   ```
10. Перезапустите виртуальную машину и приступите к проверке установки.


### <a name="verify-driver-installation"></a>Проверка установки драйверов


Чтобы запросить состояние устройства GPU, установите SSH-подключение к виртуальной машине и запустите служебную программу командной строки [nvidia smi](https://developer.nvidia.com/nvidia-system-management-interface), установленную вместе с драйвером. 

Если драйвер установлен, то отобразятся выходные данные, аналогичные приведенным ниже. Обратите внимание, что **GPU-Util** отобразит 0 %, если в данный момент графический процессор не выполняет рабочую нагрузку на виртуальной машине. Версия драйвера и сведения о GPU могут отличаться от показанных на изображении.

![Снимок экрана, на котором показаны выходные данные при запросе состояния устройства GPU.](./media/n-series-driver-setup/smi-nv.png)
 

### <a name="x11-server"></a>Сервер X11
Если вам требуется сервер X11 для удаленных подключений к виртуальной машине NV или NVv2, рекомендуем использовать [x11vnc](http://www.karlrunge.com/x11vnc/), так как он поддерживает аппаратное ускорение графики. BusID устройства M60 необходимо вручную добавить в конфигурационный файл X11 (обычно `etc/X11/xorg.conf`). Добавьте раздел `"Device"`, аналогичный приведенному ниже.
 
```
Section "Device"
    Identifier     "Device0"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    BoardName      "Tesla M60"
    BusID          "PCI:0@your-BusID:0:0"
EndSection
```
 
Чтобы использовать это устройство, обновите раздел `"Screen"`.
 
Чтобы найти десятичное число BusID, выполните эту команду:

```bash
nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print $4}'
```
 
BusID может измениться, если виртуальная машина будет перераспределена или перезагружена. Для обновления BusID в конфигурации X11 при перезагрузке виртуальной машины можно создать и использовать сценарий. Например, создайте сценарий `busidupdate.sh` (или выберите другое имя) примерно со следующим содержимым:

```bash 
#!/bin/bash
XCONFIG="/etc/X11/xorg.conf"
OLDBUSID=`awk '/BusID/{gsub(/"/, "", $2); print $2}' ${XCONFIG}`
NEWBUSID=`nvidia-xconfig --query-gpu-info | awk '/PCI BusID/{print $4}'`

if [[ "${OLDBUSID}" == "${NEWBUSID}" ]] ; then
        echo "NVIDIA BUSID not changed - nothing to do"
else
        echo "NVIDIA BUSID changed from \"${OLDBUSID}\" to \"${NEWBUSID}\": Updating ${XCONFIG}" 
        sed -e 's|BusID.*|BusID          '\"${NEWBUSID}\"'|' -i ${XCONFIG}
fi
```

Затем создайте запись для сценария обновления в `/etc/rc.d/rc3.d`, чтобы этот сценарий вызывался в качестве корневого сценария во время загрузки.

## <a name="troubleshooting"></a>Устранение неполадок

* Когда необходимо запрашивать карты, для быстрого получения выходных данных команды можно задать режим сохранения с помощью команды `nvidia-smi`. Чтобы задать режим сохранения, выполните `nvidia-smi -pm 1`. Обратите внимание, что в случае перезапуска виртуальной машины настройка режима не сохранится. Всегда можно написать сценарий настройки режима для выполнения при запуске.
* Если вы обновили драйверы CUDA NVIDIA до последней версии и обнаруживаете, что подключение RDMA больше не работает, [переустановите драйверы RDMA](#rdma-network-connectivity), чтобы восстановить подключение. 
* Если при установке LIS определенная версия ОС CentOS/RHEL (или ядро) не поддерживается для LIS, выдается ошибка "неподдерживаемая версия ядра". Сообщите об этой ошибке вместе с версиями ОС и ядра.

## <a name="next-steps"></a>Дальнейшие действия

* Чтобы записать образ виртуальной машины Linux с установленными драйверами NVIDIA, см. статью [Как подготовить к работе и записать образ виртуальной машины Linux](capture-image.md).
