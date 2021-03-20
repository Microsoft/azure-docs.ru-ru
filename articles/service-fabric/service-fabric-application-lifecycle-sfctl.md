---
title: Управление приложениями Service Fabric Azure с помощью sfctl
description: Узнайте, как развертывать приложения в кластере Azure Service Fabric и удалять их из него с помощью интерфейса командной строки Azure Service Fabric.
author: Christina-Kang
ms.topic: conceptual
ms.date: 07/31/2018
ms.author: bikang
ms.openlocfilehash: 7d361d44c349bc7a6e3c041f78d00ad66182fa15
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "84711040"
---
# <a name="manage-an-azure-service-fabric-application-by-using-azure-service-fabric-cli-sfctl"></a>Управление приложениями Azure Service Fabric с помощью интерфейса командной строки Azure Service Fabric (sfctl)

Узнайте, как создавать и удалять приложения, выполняющиеся в кластере Azure Service Fabric.

## <a name="prerequisites"></a>Предварительные требования

* Установите интерфейс командной строки Service Fabric и выберите кластер Service Fabric. Дополнительные сведения см. в статье [Azure Service Fabric command line](service-fabric-cli.md) (Интерфейс командной строки Azure Service Fabric).

* Также требуется готовый к развертыванию пакет приложения Service Fabric. Дополнительные сведения о том, как создавать и упаковывать приложения, представлены в [документации по модели приложения Service Fabric](service-fabric-application-model.md).

## <a name="overview"></a>Обзор

Чтобы развернуть новое приложение, выполните указанные ниже действия.

1. Отправьте пакет приложения в хранилище образов Service Fabric.
2. Подготовьте тип приложения.
3. Удалите содержимое из хранилища образов.
4. Укажите приложение и создайте его.
5. Укажите службы и создайте их.

Чтобы удалить существующее приложение, выполните следующие действия.

1. Удалите приложение.
2. Отмените подготовку связанного типа приложения.

## <a name="deploy-a-new-application"></a>Развертывание нового приложения

Чтобы развернуть новое приложение, выполните указанные ниже действия.

### <a name="upload-a-new-application-package-to-the-image-store"></a>Передача нового пакета приложения в хранилище образов

Прежде чем создавать приложение, необходимо отправить пакет приложения в хранилище образов Service Fabric.

Предположим, что пакет приложения существует в каталоге `app_package_dir`. Чтобы отправить каталог, выполните следующие команды:

```shell
sfctl application upload --path ~/app_package_dir
```

Для больших пакетов приложений вы можете указать параметр `--show-progress`, чтобы отобразить ход передачи.

### <a name="provision-the-application-type"></a>Подготовка типа приложения

По завершении отправки необходимо подготовить приложение. Чтобы подготовить приложение, выполните следующую команду:

```shell
sfctl application provision --application-type-build-path app_package_dir
```

Параметр `application-type-build-path` должен совпадать с именем каталога, содержащего пакет приложения, который был отправлен ранее.

### <a name="delete-the-application-package"></a>Удаление пакета приложения

Рекомендуется удалить пакет приложения после успешной регистрации приложения.  Удаление пакетов приложений из хранилища образов освобождает системные ресурсы.  Если хранить неиспользуемые пакеты, они занимают место на диске и создают проблемы производительности приложения. 

Чтобы удалить пакет приложения из хранилища образов, выполните следующую команду:

```shell
sfctl store delete --content-path app_package_dir
```

Параметр `content-path` должен совпадать с именем каталога, загруженного при создании приложения.

### <a name="create-an-application-from-an-application-type"></a>Создание приложения из типа приложения

После подготовки приложения можно присвоить ему имя и создать приложение, выполнив следующую команду:

```shell
sfctl application create --app-name fabric:/TestApp --app-type TestAppType --app-version 1.0
```

`app-name` — это имя, которое получит экземпляр приложения. Другие параметры можно найти с помощью манифеста приложения, который был подготовлен ранее.

Имя приложения должно начинаться с префикса `fabric:/`.

### <a name="create-services-for-the-new-application"></a>Создание служб для нового приложения

После создания приложения из него можно создавать службы. В этом примере мы создадим из приложения службу без отслеживания состояния. Службы, которые можно создать из приложения, определены в манифесте службы внутри подготовленного ранее пакета приложения.

```shell
sfctl service create --app-id TestApp --name fabric:/TestApp/TestSvc --service-type TestServiceType \
--stateless --instance-count 1 --singleton-scheme
```

## <a name="verify-application-deployment-and-health"></a>Проверка развертывания и работоспособности приложения

Чтобы убедиться, что все находится в работоспособном состоянии, выполните приведенные ниже команды.

```shell
sfctl application list
sfctl service list --application-id TestApp
```

Чтобы проверить, что служба работоспособна, выполните аналогичные команды, чтобы получить сведения о работоспособности службы и приложения.

```shell
sfctl application health --application-id TestApp
sfctl service health --service-id TestApp/TestSvc
```

Параметр `HealthState` работоспособных служб и приложений должен иметь значение `Ok`.

## <a name="remove-an-existing-application"></a>Удаление имеющегося приложения

Чтобы удалить приложение, выполните указанные ниже действия.

### <a name="delete-the-application"></a>Удаление приложения

Удалите приложение, выполнив следующую команду:

```shell
sfctl application delete --application-id TestEdApp
```

### <a name="unprovision-the-application-type"></a>Отмена подготовки типа приложения

После удаления приложения можно также отменить подготовку типа приложения (если он больше не требуется). Для этого выполните следующую команду:

```shell
sfctl application unprovision --application-type-name TestAppType --application-type-version 1.0
```

Имя и версия типа должны соответствовать имени и версии в манифесте приложения, который был подготовлен ранее.

## <a name="upgrade-application"></a>Обновление приложения

После создания приложения можно повторить тот же набор шагов, чтобы подготовить вторую версию приложения. Затем с помощью функции обновления приложения Service Fabric можно перейти к использованию второй версии приложения. Дополнительные сведения см. в документации по [обновлению приложения Service Fabric](service-fabric-application-upgrade.md).

Чтобы выполнить обновление, необходимо сначала подготовить следующую версию приложения, используя ту же команду, что и раньше.

```shell
sfctl application upload --path ~/app_package_dir_2
sfctl application provision --application-type-build-path app_package_dir_2
sfctl store delete --content-path app_package_dir_2
```

Затем рекомендуется выполнить отслеживаемое автоматическое обновление. Запустите обновление, выполнив следующую команду.

```shell
sfctl application upgrade --app-id TestApp --app-version 2.0.0 --parameters "{\"test\":\"value\"}" --mode Monitored
```

Обновления переопределяют существующие параметры вне зависимости от способа их задания. При необходимости параметры приложения можно передать как аргументы команды обновления. Параметры приложения должны быть закодированы в виде объекта JSON.

Чтобы получить какие-либо указанные ранее параметры, можно использовать команду `sfctl application info`.

Если выполняется обновление приложения, его состояние можно получить с помощью команды `sfctl application upgrade-status`.

Наконец, если идет обновление и его необходимо отменить, можно использовать команду `sfctl application upgrade-rollback`, чтобы выполнить откат обновления.

## <a name="next-steps"></a>Дальнейшие действия

* [Azure Service Fabric command line](service-fabric-cli.md) (Командная строка Azure Service Fabric)
* [Подготовка среды разработки в Linux](service-fabric-get-started-linux.md)
* [Обновление приложения Service Fabric](service-fabric-application-upgrade.md)
