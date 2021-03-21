---
title: Создание управляемого экземпляра SQL Azure с помощью Azure Data Studio
description: Создание управляемого экземпляра SQL Azure с помощью Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 651811384a8e831c41c48ca8c4849e5ee3852054
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93280469"
---
# <a name="create-sql-managed-instance---azure-arc-using-azure-data-studio"></a>Создание управляемого экземпляра SQL в службе "Дуга Azure" с помощью Azure Data Studio

В этом документе описано, как установить управляемый экземпляр SQL Azure в Azure ARC с помощью Azure Data Studio

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="log-in-to-the-azure-arc-data-controller"></a>Вход в систему на контроллере данных ARC в Azure

Перед созданием экземпляра Войдите в контроллер данных ARC в Azure, если вы еще не вошли в систему.

```console
azdata login
```

Затем будет предложено ввести пространство имен, в котором создается контроллер данных, имя пользователя и пароль для входа на контроллер.  

> Если необходимо проверить пространство имен, можно запустить, ```kubectl get pods -A``` чтобы получить список всех пространств имен в кластере.

```console
Username: arcadmin
Password:
Namespace: arc
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="create-azure-sql-managed-instance-on-azure-arc"></a>Создание управляемого экземпляра SQL Azure в службе "Дуга Azure"

- Запустить Azure Data Studio
- На вкладке подключения щелкните три точки в верхнем левом углу и выберите "создать развертывание".
- В параметрах развертывания выберите **управляемый экземпляр SQL Azure — Arc Azure** . 
  > [!NOTE]
  > Возможно, вам будет предложено установить этот параметр, [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] если он в настоящее время не установлен.
- Примите условия соглашения о конфиденциальности и лицензии и нажмите кнопку **выбрать** внизу.



- В колонке развертывание управляемого экземпляра SQL Azure в службе "Дуга Azure" введите следующие сведения.
  - Введите имя для экземпляра SQL Server
  - Введите и подтвердите пароль для экземпляра SQL Server
  - Выберите класс хранения данных, соответствующий
  - Выберите класс хранения в качестве подходящего для журналов.

- Нажмите кнопку " **развернуть** "

- Это должно инициировать создание управляемого экземпляра SQL Azure с помощью дуги Azure на контроллере данных.

- Через несколько минут создание будет успешно завершено

## <a name="connect-to-azure-sql-managed-instance---azure-arc-from-azure-data-studio"></a>Подключение к управляемому экземпляру SQL Azure из Azure Data Studio

- Войдите на контроллер данных ARC в Azure, указав пространство имен, имя пользователя и пароль для контроллера данных: 
```console
azdata login
```

- Просмотрите все подготовленные управляемые экземпляры SQL Azure с помощью следующих команд:

```console
azdata arc sql mi list
```

Выходные данные должны выглядеть так, скопируйте Серверендпоинт (включая номер порта) отсюда.

```console

Name          Replicas    ServerEndpoint     State
------------  ----------  -----------------  -------
sqlinstance1  1/1         25.51.65.109:1433  Ready
```

- В Azure Data Studio на вкладке " **подключения** " щелкните **новое подключение** в представлении **серверы** .
- В колонке **Подключение** вставьте серверендпоинт в текстовое поле Server (сервер).
- Выбор **имени входа SQL** в качестве типа проверки подлинности
- Введите *SA* в качестве имени пользователя
- Введите пароль для `sa` учетной записи
- При необходимости введите имя конкретной базы данных для подключения
- При необходимости выберите или добавьте новую группу серверов.
- Нажмите кнопку " **Подключиться** ", чтобы подключиться к управляемому экземпляру SQL Azure с помощью дуги Azure




## <a name="next-steps"></a>Дальнейшие действия

Теперь попробуйте [отслеживать ваш экземпляр SQL](monitor-grafana-kibana.md)
