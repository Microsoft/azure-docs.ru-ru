---
title: Подключение к SQL Управляемый экземпляр с включенной службой Arc
description: Подключение к SQL Управляемый экземпляр с включенной службой Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: abd27e15ccf5b421e69e78b2b726d192ffdecacb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92372367"
---
# <a name="connect-to-azure-arc-enabled-sql-managed-instance"></a>Подключение к SQL Управляемый экземпляр с включенной службой Arc

В этой статье объясняется, как подключиться к службе SQL Управляемый экземпляр с поддержкой ARC. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="view-azure-arc-enabled-sql-managed-instances"></a>Просмотр управляемых экземпляров SQL, включенных в дугу Azure

Чтобы просмотреть SQL Управляемый экземпляр с поддержкой дуги Azure, а внешние конечные точки используют следующую команду:

```console
azdata arc sql mi list
```

Выходные данные должны выглядеть следующим образом:

```console
Name    Replicas    ExternalEndpoint    State
------  ----------  ----------------  -------
sqldemo 1/1         10.240.0.4:32023  Ready
```

Если вы используете AKS или кубеадм или OpenShift и т. д., вы можете скопировать внешний IP-адрес и номер порта отсюда, а также подключиться к нему с помощью любимого средства для подключения к экземпляру SQL Server или SQL Azure, например Azure Data Studio или SQL Server Management Studio.  Однако если вы используете виртуальную машину быстрого запуска, ознакомьтесь со статьей ниже, чтобы получить дополнительные сведения о подключении к этой виртуальной машине извне Azure. 

> [!NOTE]
> Корпоративные политики могут блокировать доступ к IP-адресу и порту, особенно если он создан в общедоступном облаке.

## <a name="connect"></a>Подключение 

Подключение с помощью Azure Data Studio, SQL Server Management Studio или SQLCMD

Откройте Azure Data Studio и подключитесь к экземпляру с помощью IP-адреса внешней конечной точки и номера порта, приведенного выше. Если вы используете виртуальную машину Azure, вам потребуется _общедоступный_ IP-адрес, который можно идентифицировать с помощью [специального примечания о развертывании виртуальных машин Azure](#special-note-about-azure-virtual-machine-deployments).

Пример:

- Сервер: 52.229.9.30, 30913
- Имя пользователя: SA
- Пароль: указанный пароль SQL во время подготовки.

> [!NOTE]
> Вы можете использовать Azure Data Studio [просмотра панелей мониторинга управляемого экземпляра SQL](azure-data-studio-dashboards.md#view-the-sql-managed-instance-dashboards).

Для подключения с помощью SQLCMD или Linux или Windows можно использовать команду, подобную следующей. Введите пароль SQL при появлении запроса:

```bash
sqlcmd -S 52.229.9.30,30913 -U sa
```

## <a name="special-note-about-azure-virtual-machine-deployments"></a>Специальное примечание о развертывании виртуальных машин Azure

Если вы используете виртуальную машину Azure, IP-адрес конечной точки не будет показывать общедоступный IP-адрес. Чтобы узнать внешний IP-адрес, используйте следующую команду:

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

Затем можно объединить общедоступный IP-адрес с портом, чтобы установить соединение.

Также может потребоваться предоставить порт экземпляра SQL через шлюз безопасности сети (NSG). Чтобы разрешить трафик через (NSG), необходимо добавить правило, которое можно выполнить с помощью следующей команды.

Чтобы задать правило, необходимо знать имя NSG, которое можно узнать с помощью следующей команды:

```azurecli
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

После получения имени NSG можно добавить правило брандмауэра, используя следующую команду. Приведенные здесь примеры значений создают правило NSG для порта 30913 и допускают подключение с **любого** исходного IP-адреса.  Это не рекомендуется по соображениям безопасности.  Вы можете более эффективно выполнять блокирование, указав значение -source-address-prefixes, характерное для IP-адреса клиента, или диапазон IP-адресов, который охватывает IP-адреса вашей команды или организации.

Замените значение `--destination-port-ranges` параметра ниже номером порта, полученным из `azdata sql instance list` команды F выше.

```azurecli
az network nsg rule create -n db_port --destination-port-ranges 30913 --source-address-prefixes '*' --nsg-name azurearcvmNSG --priority 500 -g azurearcvm-rg --access Allow --description 'Allow port through for db access' --destination-address-prefixes '*' --direction Inbound --protocol Tcp --source-port-ranges '*'
```

## <a name="next-steps"></a>Дальнейшие действия

- [Просмотр панелей мониторинга управляемого экземпляра SQL](azure-data-studio-dashboards.md#view-the-sql-managed-instance-dashboards)
- [Просмотр Управляемый экземпляр SQL в портал Azure](view-arc-data-services-inventory-in-azure-portal.md)
