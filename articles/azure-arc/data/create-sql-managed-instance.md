---
title: Создание управляемого экземпляра SQL Azure в службе "Дуга Azure"
description: Создание управляемого экземпляра SQL Azure в службе "Дуга Azure"
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 0d1d6c7349adc65270e13d09d771963cb9cacea3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93280398"
---
# <a name="create-an-azure-sql-managed-instance-on-azure-arc"></a>Создание управляемого экземпляра SQL Azure в службе "Дуга Azure"

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="login-to-the-azure-arc-data-controller"></a>Вход в контроллер данных ARC в Azure

Перед созданием экземпляра Войдите в контроллер данных ARC в Azure, если вы еще не вошли в систему.

```console
azdata login
```

Затем будет предложено ввести имя пользователя, пароль и пространство имен System.  

```console
Username: arcadmin
Password:
Namespace: arc
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="create-an-azure-sql-managed-instance"></a>Создание Управляемого экземпляра SQL Azure

Чтобы просмотреть доступные параметры Create Форскл Управляемый экземпляр, используйте следующую команду:
```console
azdata arc sql mi create --help
```

Чтобы создать Управляемый экземпляр SQL, используйте следующую команду:

```console
azdata arc sql mi create -n <instanceName> --storage-class-data <storage class> --storage-class-logs <storage class>
```

Пример

```console
azdata arc sql mi create -n sqldemo --storage-class-data managed-premium --storage-class-logs managed-premium
```
> [!NOTE]
>  Длина имен должна быть меньше 13 символов и соответствовать [соглашениям об именовании DNS](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names) .
>
>  При указании выделения памяти и распределения Виртуальное ядро используйте эту формулу, чтобы убедиться, что создание прошло успешно. для каждого 1 Виртуальное ядро требуется по крайней мере 4 ГБ ОЗУ на узле Kubernetes, где будет запущен экземпляр Pod управляемого экземпляра SQL.
>
>  Не используйте в имени верхний регистр при создании экземпляра SQL для подготовки в Azure.
>
>  Чтобы получить список доступных классов хранения в кластере Kubernetes, выполните команду `kubectl get storageclass` 


> [!NOTE]
> Если вы хотите автоматизировать создание экземпляров SQL и избежать интерактивного запроса пароля администратора, можно задать `AZDATA_USERNAME` `AZDATA_PASSWORD` для переменных среды и имя пользователя и пароль перед выполнением `azdata arc sql mi create` команды.
> 
>  Если вы создали контроллер данных с помощью AZDATA_USERNAME и AZDATA_PASSWORD в том же сеансе терминала, то для создания управляемого экземпляра SQL также будут использоваться значения AZDATA_USERNAME и AZDATA_PASSWORD.

> [!NOTE]
> Создание Управляемый экземпляр Azure SQL не приведет к регистрации ресурсов в Azure. Действия по регистрации ресурса приведены в следующих статьях: 
> - [Просмотр журналов и метрик с помощью Kibana и Grafana](monitor-grafana-kibana.md)
> - [Отправьте данные о выставлении счетов в Azure и просмотрите их в портал Azure](view-billing-data-in-azure.md) 


## <a name="view-instance-on-azure-arc"></a>Просмотреть экземпляр в службе "Дуга Azure"

Чтобы просмотреть экземпляр, используйте следующую команду:

```console
azdata arc sql mi list
```

Выходные данные должны выглядеть следующим образом:

```console
Name    Replicas    ServerEndpoint    State
------  ----------  ----------------  -------
sqldemo 1/1         10.240.0.4:32023  Ready
```

Если вы используете AKS или `kubeadm` OpenShift и т. д., вы можете скопировать внешний IP-адрес и номер порта отсюда и подключиться к нему с помощью любимого средства для подключения к экземпляру SQL Server или SQL Azure, например Azure Data Studio или SQL Server Management Studio. Однако если вы используете виртуальную машину быстрого запуска, см. Дополнительные инструкции в статье [Подключение к службе Arc Azure с поддержкой SQL управляемый экземпляр](connect-managed-instance.md) .


## <a name="next-steps"></a>Дальнейшие действия
- [Подключение к SQL Управляемый экземпляр с включенной службой Arc](connect-managed-instance.md)
- [Регистрация экземпляра в Azure и отправка метрик и журналов о вашем экземпляре](upload-metrics-and-logs-to-azure-monitor.md)
- [Развертывание управляемого экземпляра SQL Azure с помощью Azure Data Studio](create-sql-managed-instance-azure-data-studio.md)
