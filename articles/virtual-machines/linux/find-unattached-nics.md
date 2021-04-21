---
title: Поиск и удаление неподключенных сетевых адаптеров Azure
description: Как найти и удалить не подключенные к виртуальным машинам сетевые адаптеры Azure с помощью Azure CLI.
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 04/10/2018
ms.author: cynthn
ms.openlocfilehash: 5b28226cfe6bc51a2619c4dd63e666ddd51dad2a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96016205"
---
# <a name="how-to-find-and-delete-unattached-network-interface-cards-nics-for-azure-vms"></a>Как найти и удалить неподключенные сетевые карты (NIC) на виртуальных машинах Azure
Когда вы удаляете виртуальную машину в Azure, ее сетевые карты не удаляются по умолчанию. При создании и удалении нескольких виртуальных машин неиспользуемые сетевые карты продолжают занимать выделенные им внутренние IP-адреса. Когда вы будете создавать другие сетевые карты виртуальной машины, они, возможно, не смогут получить IP-адрес в адресном пространстве подсети. В этой статье показано, как найти и удалить неподключенные сетевые карты.

## <a name="find-and-delete-unattached-nics"></a>Поиск и удаление неподключенных сетевых адаптеров

В свойстве *virtualMachine* сетевой карты хранится идентификатор и группа ресурсов виртуальной машины, к которой подключена данная сетевая карта. Приведенный ниже скрипт просматривает все сетевые карты в подписке и проверяет, имеет ли свойство *virtualMachine* значение null. Если это свойство имеет значение null, сетевая карта не подключена к виртуальной машине.

Чтобы просмотреть все неподключенные сетевые карты, мы настоятельно рекомендуем сначала выполнить скрипт, задав для переменной *deleteUnattachedNics* значение *0*. Чтобы удалить все неподключенные сетевые карты после просмотра списка всех найденных, запустите скрипт со значением *deleteUnattachedNics* равным *1*.

```azurecli
# Set deleteUnattachedNics=1 if you want to delete unattached NICs
# Set deleteUnattachedNics=0 if you want to see the Id(s) of the unattached NICs
deleteUnattachedNics=0

unattachedNicsIds=$(az network nic list --query '[?virtualMachine==`null`].[id]' -o tsv)
for id in ${unattachedNicsIds[@]}
do
   if (( $deleteUnattachedNics == 1 ))
   then

       echo "Deleting unattached NIC with Id: "$id
       az network nic delete --ids $id
       echo "Deleted unattached NIC with Id: "$id
   else
       echo $id
   fi
done
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о создании виртуальных сетей и управлении ими в Azure см. в статье [Управление виртуальными сетями Azure и виртуальными машинами Linux с помощью Azure CLI](tutorial-virtual-network.md).
