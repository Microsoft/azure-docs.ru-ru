---
title: Создание настраиваемого канала Conda для управления пакетами
description: Узнайте, как создать пользовательский канал Conda для управления пакетами
services: synapse-analytics
author: midesa
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 02/26/2020
ms.author: midesa
ms.reviewer: jrasnick
ms.subservice: spark
ms.openlocfilehash: 528ba4a1be3650a81772d78a438f03611b9bd761
ms.sourcegitcommit: 4b7a53cca4197db8166874831b9f93f716e38e30
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102107944"
---
# <a name="create-a-custom-conda-channel-for-package-management"></a>Создание пользовательского канала Conda для управления пакетами 
При установке пакетов Python диспетчер пакетов Conda использует каналы для поиска пакетов. Может потребоваться создать пользовательский канал Conda по различным причинам. Например, может оказаться, что:

- Ваша рабочая область — это защищенные утечка данных, и исходящие подключения блокируются.  
- у вас есть пакеты, которые не нужно передавать в общедоступные репозитории.
- Вы хотите настроить альтернативный репозиторий для пользователей в рабочей области.

В этой статье мы предоставим пошаговое руководство, которое поможет вам создать пользовательский канал Conda в учетной записи Azure Data Lake Storage.

## <a name="set-up-your-local-machine"></a>Настройка локального компьютера

1. Установите Conda на локальном компьютере. Вы можете обратиться к [среде выполнения Azure синапсе Spark](./apache-spark-version-support.md) , чтобы узнать версию Conda, используемую в той же среде выполнения.
   
2. Чтобы создать пользовательский канал, установите conda-Build.
```
conda install conda-build
```
3. Упорядочите все пакеты в для платформы, которую вы хотите обслуживать. В этом примере мы установим Архив Anaconda на локальном компьютере.

```
sudo wget https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh 
sudo chmod +x Anaconda3-4.4.0-Linux-x86_64.sh  
sudo bash Anaconda3-4.4.0-Linux-x86_64.sh -b -p /usr/lib/anaconda3 
export PATH="/usr/lib/anaconda3/bin:$PATH" 
sudo chmod 777 -R /usr/lib/anaconda3a.  
```
## <a name="mount-the-storage-account-onto-your-machine"></a>Подключение учетной записи хранения на компьютере
Далее мы подключаем учетную запись Azure Data Lake Storage 2-го поколения на локальном компьютере. Этот процесс также можно выполнить с помощью учетной записи WASB. Однако мы рассмотрим пример для учетной записи ADLSg2 
 
Дополнительные сведения о том, как подключить учетную запись хранения на локальном компьютере, можно получить на [этой странице](https://github.com/Azure/azure-storage-fuse#blobfuse ). 

1. Вы можете установить blobfuse из репозитория программного обеспечения Linux для продуктов Майкрософт.

```
wget https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb 
sudo dpkg -i packages-microsoft-prod.deb 
sudo apt-get update 
sudo apt-get install blobfuse fuse 
export AZURE_STORAGE_ACCOUNT=<<myaccountname>
export AZURE_STORAGE_ACCESS_KEY=<<myaccountkey>>
export AZURE_STORAGE_BLOB_ENDPOINT=*.dfs.core.windows.net 
```

2. Создайте точку подключения ( ```mkdir /path/to/mount``` ) и подключите контейнер больших двоичных объектов с помощью blobfuse. В этом примере мы используем значение **приватечаннел** для переменной **MyContainer** .
   
```
blobfuse /path/to/mount --container-name=mycontainer --tmp-path=/mnt/blobfusetmp --use-adls=true --log-level=LOG_DEBUG 
sudo mkdir -p /mnt/blobfusetmp
sudo chown <myuser> /mnt/blobfusetmp
```
## <a name="create-the-channel"></a>Создание канала
В следующем наборе действий будет создан пользовательский канал Conda. 

1. На локальном компьютере создайте каталог для упорядочения всех пакетов для пользовательского канала.
   
```
mkdir /home/trusted-service-user/privatechannel 
cd ~/privatechannel/ 
mkdir channel1/linux64 
```

2. Упорядочите все ```tar.bz2``` пакеты из https://repo.anaconda.com/pkgs/main/linux-64/ в подкаталог. Также обязательно включите все зависимые пакеты tar. bz2. 

```
cd channel1 
mkdir noarch 
echo '{}' > noarch/repodata.json 
bzip2 -k noarch/repodata.json 

// Create channel 

conda index channel1/noarch 
conda index channel1/linux-64 
conda index channel1 
```

Дополнительные сведения см. в разделе о создании настраиваемых каналов с [помощью Condaного руководством пользователя](https://docs.conda.io/projects/conda/latest/user-guide/tasks/create-custom-channels.html) . 

## <a name="storage-account-permissions"></a>Разрешения учетной записи хранения
Теперь необходимо проверить разрешения для учетной записи хранения. Чтобы задать эти разрешения, перейдите по пути, по которому будет создан настраиваемый канал. Затем создайте маркер SAS для ```privatechannel``` , который имеет разрешения на чтение, список и выполнение. 

Имя канала теперь будет URL-адресом SAS для большого двоичного объекта, созданным на основе этого процесса.  

## <a name="create-a-sample-conda-environment-configuration-file"></a>Создание примера файла конфигурации среды Conda
Наконец, проверьте процесс установки, создав пример ```environment.yml``` файла Conda. При наличии рабочей области с включенной функцией DEP необходимо указать ``nodefaults`` канал в файле среды.

Ниже приведен пример файла конфигурации Conda:
```
name: sample 
channels: 
  - https://<<storage account name>>.blob.core.windows.net/privatechannel/channel?<<SAS Token>
  - nodefaults 
dependencies: 
  - openssl 
  - ncurses 
```
После создания образца файла Conda можно создать виртуальную среду Conda. 

```
conda env create –file sample.yml  
source activate env 
conda list 
```
Теперь, когда вы проверили пользовательский канал, вы можете использовать процесс [управления пулом Python](./apache-spark-manage-python-packages.md) для обновления библиотек в пуле Apache Spark.

## <a name="next-steps"></a>Дальнейшие действия
- Просмотр библиотек по умолчанию: [Поддержка версий Apache Spark](apache-spark-version-support.md)
- Управление пакетами Python: [Управление пакетами Python](./apache-spark-manage-python-packages.md)

