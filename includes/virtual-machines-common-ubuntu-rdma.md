---
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 10/26/2018
ms.author: cynthn
ms.openlocfilehash: d41b86b902d9a58b144e251e6922fbd95d459031
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96016140"
---
1. Установите dapl, rdmacm, ibverbs и mlx4.

   ```bash
   sudo apt-get update

   sudo apt-get install libdapl2 libmlx4-1

   ```

2. В /etc/waagent.conf включите RDMA, раскомментировав следующие строки конфигурации. Для изменения этого файла требуется доступ с правами root.
  
   ```
   OS.EnableRDMA=y

   OS.UpdateRdmaDriver=y
   ```

3. Добавьте или измените следующие параметры памяти в КБ в файле /etc/security/limits.conf. Для изменения этого файла требуется доступ с правами root. В целях тестирования можно также задать неограниченное значение для параметра memlock. Например: `<User or group name>   hard    memlock   unlimited`.

   ```
   <User or group name> hard    memlock <memory required for your application in KB>

   <User or group name> soft    memlock <memory required for your application in KB>
   ```
  
4. Установите библиотеку Intel MPI [Купите и скачайте](https://software.intel.com/intel-mpi-library/) библиотеку с сайта Intel или скачайте [бесплатную пробную версию](https://registrationcenter.intel.com/en/forms/?productid=1740).

   ```bash
   wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/9278/l_mpi_p_5.1.3.223.tgz
   ```
 
   Поддерживаются только среды выполнения Intel MPI 5.x.
 
   Шаги установки см. в [руководстве по установке библиотеки Intel MPI](https://registrationcenter-download.intel.com/akdlm/irc_nas/1718/INSTALL.html?lang=en&fileExt=.html).

5. Включите ptrace для некорневых процессов без отладчика (требуется для последних версий Intel MPI).
 
   ```bash
   echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope
   ```
