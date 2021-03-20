---
title: Создание тома NFS Ubuntu Linux Server
titleSuffix: Azure Kubernetes Service
description: Узнайте, как вручную создать том сервера Ubuntu Linux NFS для использования с модулями Pod в службе Azure Kubernetes Service (AKS).
services: container-service
author: ozboms
ms.topic: article
ms.date: 4/25/2019
ms.author: obboms
ms.openlocfilehash: 4e817d572a98ffb8135adf58d13f50ccacbc8746
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86252000"
---
# <a name="manually-create-and-use-an-nfs-network-file-system-linux-server-volume-with-azure-kubernetes-service-aks"></a>Создание и использование тома NFS сервера Linux (сетевой файловой системы) вручную с помощью службы Kubernetes Azure (AKS)
Обмен данными между контейнерами часто является необходимым компонентом служб и приложений, основанных на контейнерах. Обычно у вас есть различные модули, которым требуется доступ к одной и той же информации на внешнем постоянном томе.    
Хотя служба файлов Azure является возможностью, создание NFS-сервера на виртуальной машине Azure является другой формой постоянного общего хранилища. 

В этой статье показано, как создать NFS Server на виртуальной машине Ubuntu. Кроме того, предоставьте контейнерам AKS доступ к этой общей файловой системе.

## <a name="before-you-begin"></a>Перед началом
В этой статье предполагается, что у вас уже есть кластер AKS. Если вам нужен кластер AKS, ознакомьтесь с кратким руководством по AKS, [используя Azure CLI][aks-quickstart-cli] или [с помощью портал Azure][aks-quickstart-portal].

Кластер AKS должен быть активным в тех же или одноранговых виртуальных сетях, что и сервер NFS. Кластер должен быть создан в существующей виртуальной сети, которая может быть той же виртуальной машиной, что и виртуальная машина.

Действия по настройке с помощью существующей виртуальной сети описаны в документации по [созданию кластера AKS в существующей виртуальной][aks-virtual-network] сети и [подключению виртуальных сетей с помощью пиринга виртуальной][peer-virtual-networks] сети.

Также предполагается, что вы создали Ubuntu Linux виртуальную машину (например, 18,04 LTS). Параметры и размер могут быть доступны вам и могут быть развернуты с помощью Azure. Краткое руководство по Linux см. в статье [Управление виртуальными машинами Linux][linux-create].

При первом развертывании кластера AKS Azure автоматически заполняет поле виртуальной сети при развертывании компьютера Ubuntu, делая его активным в той же виртуальной сети. Но если вы хотите работать с одноранговыми сетями, обратитесь к документации, приведенной выше.

## <a name="deploying-the-nfs-server-onto-a-virtual-machine"></a>Развертывание NFS-сервера на виртуальной машине
Ниже приведен сценарий для настройки NFS-сервера в виртуальной машине Ubuntu.
```bash
#!/bin/bash

# This script should be executed on Linux Ubuntu Virtual Machine

EXPORT_DIRECTORY=${1:-/export/data}
DATA_DIRECTORY=${2:-/data}
AKS_SUBNET=${3:-*}

echo "Updating packages"
apt-get -y update

echo "Installing NFS kernel server"

apt-get -y install nfs-kernel-server

echo "Making data directory ${DATA_DIRECTORY}"
mkdir -p ${DATA_DIRECTORY}

echo "Making new directory to be exported and linked to data directory: ${EXPORT_DIRECTORY}"
mkdir -p ${EXPORT_DIRECTORY}

echo "Mount binding ${DATA_DIRECTORY} to ${EXPORT_DIRECTORY}"
mount --bind ${DATA_DIRECTORY} ${EXPORT_DIRECTORY}

echo "Giving 777 permissions to ${EXPORT_DIRECTORY} directory"
chmod 777 ${EXPORT_DIRECTORY}

parentdir="$(dirname "$EXPORT_DIRECTORY")"
echo "Giving 777 permissions to parent: ${parentdir} directory"
chmod 777 $parentdir

echo "Appending bound directories into fstab"
echo "${DATA_DIRECTORY}    ${EXPORT_DIRECTORY}   none    bind  0  0" >> /etc/fstab

echo "Appending localhost and Kubernetes subnet address ${AKS_SUBNET} to exports configuration file"
echo "/export        ${AKS_SUBNET}(rw,async,insecure,fsid=0,crossmnt,no_subtree_check)" >> /etc/exports
echo "/export        localhost(rw,async,insecure,fsid=0,crossmnt,no_subtree_check)" >> /etc/exports

nohup service nfs-kernel-server restart
```
Сервер будет перезапущен (из-за скрипта), и сервер NFS можно подключить к AKS.

>[!IMPORTANT]  
>Не забудьте заменить **AKS_SUBNET** на правильный из вашего кластера, иначе "*" откроет сервер NFS для всех портов и подключений.

После создания виртуальной машины Скопируйте приведенный выше сценарий в файл. Затем вы можете переместить его с локального компьютера или с того места, где находится сценарий, на виртуальную машину, используя: 
```console
scp /path/to/script_file username@vm-ip-address:/home/{username}
```
После того как сценарий находится на виртуальной машине, вы можете использовать SSH-подключение к виртуальной машине и выполнить его с помощью команды:
```console
sudo ./nfs-server-setup.sh
```
Если выполнение завершается сбоем из-за ошибки отказа в разрешении, задайте разрешение на выполнение с помощью команды:
```console
chmod +x ~/nfs-server-setup.sh
```

## <a name="connecting-aks-cluster-to-nfs-server"></a>Подключение кластера AKS к серверу NFS
Можно подключить сервер NFS к кластеру, выполнив подготовку постоянного тома и постоянного тома, который указывает, как получить доступ к тому.

Подключение двух служб в одной или в одноранговой виртуальной сети является обязательным. Инструкции по настройке кластера в одной виртуальной сети: [Создание кластера AKS в существующей виртуальной сети][aks-virtual-network]

После того как они находятся в одной виртуальной сети (или в одноранговой), необходимо подготавливать постоянный том и постоянное требование к тому в кластере AKS. Затем контейнеры могут подключать диск NFS к их локальному каталогу.

Ниже приведен пример определения Kubernetes для постоянного тома (это определение предполагает, что ваш кластер и виртуальная машина находятся в одной виртуальной сети):

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <NFS_NAME>
  labels:
    type: nfs
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  nfs:
    server: <NFS_INTERNAL_IP>
    path: <NFS_EXPORT_FILE_PATH>
```
Замените **NFS_INTERNAL_IP**, **NFS_NAME** и **NFS_EXPORT_FILE_PATH** сведениями о сервере NFS.

Вам также потребуется постоянный файл утверждения тома. Ниже приведен пример того, что следует включить.

>[!IMPORTANT]  
>**"сторажекласснаме"** должен оставаться пустой строкой, иначе утверждение не будет работать.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <NFS_NAME>
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: ""
  resources:
    requests:
      storage: 1Gi
  selector: 
    matchLabels:
      type: nfs
```

## <a name="troubleshooting"></a>Устранение неполадок
Если не удается подключиться к серверу из кластера, то проблема может быть в экспортированном или родительском каталоге и не имеет достаточных разрешений для доступа к серверу.

Убедитесь, что каталог экспорта и его родительский каталог имеют разрешения 777.

Разрешения можно проверить, выполнив приведенную ниже команду, и каталоги должны иметь разрешения *"дрвксрвксрвкс"* :
```console
ls -l
```

## <a name="more-information"></a>Дополнительные сведения
Чтобы получить полное пошаговое руководство или помочь вам в отладке программы установки сервера NFS, ознакомьтесь с подробным руководством:
  - [Учебник по NFS][nfs-tutorial]

## <a name="next-steps"></a>Дальнейшие действия

Соответствующие рекомендации см. в разделе [Рекомендации по хранению и резервному копированию в AKS][operator-best-practices-storage].

<!-- LINKS - external -->
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/volumes/
[linux-create]: ../virtual-machines/linux/tutorial-manage-vm.md
[nfs-tutorial]: https://help.ubuntu.com/community/SettingUpNFSHowTo#Pre-Installation_Setup
[aks-virtual-network]: ./configure-kubenet.md#create-an-aks-cluster-in-the-virtual-network
[peer-virtual-networks]: ../virtual-network/tutorial-connect-virtual-networks-portal.md

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[operator-best-practices-storage]: operator-best-practices-storage.md
