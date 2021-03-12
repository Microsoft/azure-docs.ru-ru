---
title: Создание контроллера данных с помощью средств Kubernetes
description: Создание контроллера данных с помощью средств Kubernetes
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 03/02/2021
ms.topic: how-to
ms.openlocfilehash: 500587dc6564aa55eb430365eb67bb958bbd2482
ms.sourcegitcommit: 956dec4650e551bdede45d96507c95ecd7a01ec9
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102519985"
---
# <a name="create-azure-arc-data-controller-using-kubernetes-tools"></a>Создание контроллера данных ARC в Azure с помощью средств Kubernetes

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Предварительные требования

Ознакомьтесь с разделом [Создание контроллера данных ARC в Azure](create-data-controller.md) для получения общих сведений.

Чтобы создать контроллер данных ARC в Azure с помощью средств Kubernetes, необходимо установить средства Kubernetes.  Примеры в этой статье будут использовать `kubectl` , но аналогичные подходы можно использовать с другими средствами Kubernetes, такими как панель мониторинга Kubernetes, `oc` или `helm` Если вы знакомы с этими инструментами и Kubernetes YAML/JSON.

[Установка средства kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

> [!NOTE]
> Для выполнения некоторых действий по созданию контроллера данных ARC в Azure, указанных ниже, требуются разрешения администратора кластера Kubernetes.  Если вы не являетесь администратором кластера Kubernetes, вам потребуется Администратор кластера Kubernetes выполнить эти действия от вашего имени.

### <a name="cleanup-from-past-installations"></a>Очистка от прошлых установок

Если вы установили в прошлом контроллер данных ARC для Azure, в том же кластере и удалили контроллер данных ARC в Azure с помощью `azdata arc dc delete` команды, возможно, существуют некоторые объекты уровня кластера, которые по-прежнему потребуется удалить. Выполните следующие команды, чтобы удалить объекты уровня кластера для контроллера данных ARC в Azure:

```console
# Cleanup azure arc data service artifacts
kubectl delete crd datacontrollers.arcdata.microsoft.com 
kubectl delete crd sqlmanagedinstances.sql.arcdata.microsoft.com 
kubectl delete crd postgresql-11s.arcdata.microsoft.com 
kubectl delete crd postgresql-12s.arcdata.microsoft.com
```

## <a name="overview"></a>Обзор

Создание контроллера данных ARC в Azure имеет следующие общие шаги:
1. Создайте пользовательские определения ресурсов для контроллера данных Arc, управляемого экземпляра SQL Azure и PostgreSQLного масштабирования. **[Требуются разрешения администратора кластера Kubernetes]**
2. Создайте пространство имен, в котором будет создан контроллер данных. **[Требуются разрешения администратора кластера Kubernetes]**
3. Создайте службу начального загрузчика, включая набор реплик, учетную запись службы, роль и привязку роли.
4. Создайте секрет для имени пользователя и пароля администратора контроллера данных.
5. Создайте контроллер данных.
   
## <a name="create-the-custom-resource-definitions"></a>Создание пользовательских определений ресурсов

Выполните следующую команду, чтобы создать пользовательские определения ресурсов.  **[Требуются разрешения администратора кластера Kubernetes]**

```console
kubectl create -f https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/custom-resource-definitions.yaml
```

## <a name="create-a-namespace-in-which-the-data-controller-will-be-created"></a>Создание пространства имен, в котором будет создан контроллер данных

Выполните команду, аналогичную приведенной ниже, чтобы создать новое выделенное пространство имен, в котором будет создан контроллер данных.  В этом примере и оставшейся части примеров в этой статье будет использоваться имя пространства имен `arc` . Если вы решили использовать другое имя, используйте то же самое имя в пределах.

```console
kubectl create namespace arc
```

Если другие пользователи будут использовать это пространство имен, не являющиеся администраторами кластера, рекомендуется создать роль администратора пространства имен и предоставить эту роль этим пользователям с помощью привязки ролей.  Администратор пространства имен должен иметь полный доступ к пространству имен.  Дополнительные сведения будут предоставлены позже, как предоставить пользователям более детализированный доступ на основе ролей.

## <a name="create-the-bootstrapper-service"></a>Создание службы начального загрузчика

Служба начального загрузчика обрабатывает входящие запросы на создание, изменение и удаление настраиваемых ресурсов, таких как контроллер данных, управляемый экземпляр SQL или PostgreSQLная группа серверов.

Выполните следующую команду, чтобы создать службу начального загрузчика, учетную запись службы для службы начального загрузчика, а также привязку роли и роли для учетной записи службы начального загрузчика.

```console
kubectl create --namespace arc -f https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/bootstrapper.yaml
```

Убедитесь, что модуль загрузчика выполняется с помощью следующей команды.  Может потребоваться выполнить его несколько раз, пока состояние не изменится на `Running` .

```console
kubectl get pod --namespace arc
```

Файл шаблона начального загрузчика. YAML по умолчанию извлекает образ контейнера загрузчика из реестра контейнеров Майкрософт (мкр).  Если у вашей среды нет доступа непосредственно к реестру контейнеров Microsoft, можно выполнить следующие действия.
- Выполните действия, чтобы [извлечь образы контейнеров из реестра контейнеров Майкрософт и отправить их в частный реестр контейнеров](offline-deployment.md).
- [Создайте секрет для извлечения образа](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-lin) для частного реестра контейнеров.
- Добавление секрета извлечения образа в контейнер начального загрузчика. См. пример ниже.
- Измените расположение образа для образа начального загрузчика. См. пример ниже.

В приведенном ниже примере предполагается, что вы создали имя секрета для извлечения образа, `regcred` как указано в документации по Kubernetes.

```yaml
#just showing only the relevant part of the bootstrapper.yaml template file here
containers:
      - env:
        - name: ACCEPT_EULA
          value: "Y"
        #image: mcr.microsoft.com/arcdata/arc-bootstrapper:public-preview-dec-2020  <-- template value to change
        image: <your registry DNS name or IP address>/<your repo>/arc-bootstrapper:<your tag>
        imagePullPolicy: IfNotPresent
        name: bootstrapper
        resources: {}
        securityContext:
          runAsUser: 21006
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      imagePullSecrets:
      - name: regcred
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      serviceAccount: sa-mssql-controller
      serviceAccountName: sa-mssql-controller
      terminationGracePeriodSeconds: 30

```

## <a name="create-a-secret-for-the-data-controller-administrator"></a>Создание секрета для администратора контроллера данных

Имя пользователя и пароль администратора контроллера данных используются для проверки подлинности API контроллера данных для выполнения административных функций.  Выберите безопасный пароль и предоставьте к нему доступ только тем пользователям, которым требуются права администратора кластера.

Секрет Kubernetes хранится в виде строки в кодировке Base64 — одна для имени пользователя, а другая — для пароля.

Вы можете использовать онлайн-инструмент для кодирования требуемого имени пользователя и пароля в Base64 или использовать встроенные средства CLI в зависимости от платформы.

PowerShell

```console
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('<your string to encode here>'))

#Example
#[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('example'))

```

Linux/macOS

```console
echo -n '<your string to encode here>' | base64

#Example
# echo -n 'example' | base64
```

После кодирования имени пользователя и пароля можно создать файл на основе [файла шаблона](https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/controller-login-secret.yaml) и заменить значения username и Password собственными.

Затем выполните следующую команду, чтобы создать секрет.

```console
kubectl create --namespace arc -f <path to your data controller secret file>

#Example
kubectl create --namespace arc -f C:\arc-data-services\controller-login-secret.yaml
```

## <a name="create-the-data-controller"></a>Создание контроллера данных

Теперь все готово для создания самого контроллера данных.

Сначала создайте копию [файла шаблона](https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/data-controller.yaml) на локальном компьютере, чтобы можно было изменить некоторые параметры.

При необходимости измените следующее:

**Обязательно**
- **Расположение**: измените это расположение Azure, где будут храниться _метаданные_ о контроллере данных.  Список доступных расположений Azure можно просмотреть в статье [Обзор создания контроллера данных](create-data-controller.md) .
- **resourceGroup**— группа ресурсов Azure, в которой вы хотите создать ресурс контроллера данных azure в Azure Resource Manager.  Обычно эта группа ресурсов уже существует, но она не требуется до момента передачи данных в Azure.
- **Подписка**. идентификатор GUID подписки Azure для подписки, в которой вы хотите создать ресурсы Azure.

**РЕКОМЕНДУЕТСЯ ДЛЯ ПРОСМОТРА И, ВОЗМОЖНО, ИЗМЕНЕНИЯ ЗНАЧЕНИЙ ПО УМОЛЧАНИЮ**
- **хранилище.. className**: класс хранения, используемый для файлов данных и журналов контроллера данных.  Если вы не знаете доступных классов хранения в кластере Kubernetes, можно выполнить следующую команду: `kubectl get storageclass` .  По умолчанию `default` предполагается, что существует класс хранения с именем, а `default` не существует класс хранения, который _является_ значением по умолчанию.  Примечание. в качестве требуемого класса хранения можно задать два параметра className — один для данных и один для журналов.
- **serviceType**. Измените тип службы на, `NodePort` если не используется балансировщик нагрузки.  Примечание. необходимо изменить два параметра serviceType.
- На платформе контейнеров Azure Red Hat OpenShift или Red Hat OpenShift необходимо применить ограничение контекста безопасности перед созданием контроллера данных. Следуйте инструкциям в разделе [применение ограничения контекста безопасности для служб данных, включенных в службу "Дуга Azure" на OpenShift](how-to-apply-security-context-constraint.md).
- **Безопасность** Для платформы контейнеров Azure Red Hat OpenShift или Red Hat OpenShift замените `security:` параметры следующими значениями в файле YAML контроллера данных. 

```yml
  security:
    allowDumps: true
    allowNodeMetricsCollection: false
    allowPodMetricsCollection: false
    allowRunAsRoot: false
```

**ИСПОЛЬЗУЕМЫХ**
- **имя**. имя по умолчанию контроллера данных — `arc` , но при необходимости его можно изменить.
- **DisplayName**: задает то же значение, что и атрибут Name в верхней части файла.
- **Реестр**. Реестр контейнеров Майкрософт используется по умолчанию.  Если вы извлекаете образы из реестра контейнеров Майкрософт и отправляете [их в частный реестр контейнеров](offline-deployment.md), введите здесь IP-адрес или DNS-имя реестра.
- **доккеррегистри**: секрет для извлечения образа, используемый для извлечения образов из закрытого реестра контейнеров, если это необходимо.
- **репозиторий**. репозиторий по умолчанию в реестре контейнеров Майкрософт — `arcdata` .  Если вы используете частный реестр контейнеров, введите путь к папке или репозиторию, содержащему образы контейнеров служб данных с поддержкой дуги Azure.
- **имажетаг**: в шаблоне используется новый тег последней версии, но его можно изменить, если вы хотите использовать более старую версию.

В следующем примере показан завершенный файл YAML контроллера данных. Обновите пример для своей среды в соответствии с вашими требованиями и приведенными выше сведениями.

```yaml
apiVersion: arcdata.microsoft.com/v1alpha1
kind: datacontroller
metadata:
  generation: 1
  name: arc
spec:
  credentials:
    controllerAdmin: controller-login-secret
    #dockerRegistry: arc-private-registry - optional if you are using a private container registry that requires authentication using an image pull secret
    serviceAccount: sa-mssql-controller
  docker:
    imagePullPolicy: Always
    imageTag: public-preview-dec-2020 
    registry: mcr.microsoft.com
    repository: arcdata
  security:
    allowDumps: true
    allowNodeMetricsCollection: true
    allowPodMetricsCollection: true
    allowRunAsRoot: false
  services:
  - name: controller
    port: 30080
    serviceType: LoadBalancer
  - name: serviceProxy
    port: 30777
    serviceType: LoadBalancer
  settings:
    ElasticSearch:
      vm.max_map_count: "-1"
    azure:
      connectionMode: Indirect
      location: eastus
      resourceGroup: myresourcegroup
      subscription: c82c901a-129a-435d-86e4-cc6b294590ae
    controller:
      displayName: arc
      enableBilling: "True"
      logs.rotation.days: "7"
      logs.rotation.size: "5000"
  storage:
    data:
      accessMode: ReadWriteOnce
      className: default
      size: 15Gi
    logs:
      accessMode: ReadWriteOnce
      className: default
      size: 10Gi
```

Сохраните измененный файл на локальном компьютере и выполните следующую команду, чтобы создать контроллер данных:

```console
kubectl create --namespace arc -f <path to your data controller file>

#Example
kubectl create --namespace arc -f C:\arc-data-services\data-controller.yaml
```

## <a name="monitoring-the-creation-status"></a>Наблюдение за состоянием создания

Создание контроллера займет несколько минут. Вы можете отслеживать ход выполнения в другом окне терминала с помощью следующих команд:

> [!NOTE]
>  В примерах команд ниже предполагается, что вы создали контроллер данных и пространство имен Kubernetes с именем `arc` .  Если вы использовали другое имя пространства имен или контроллера данных, вы можете заменить его на `arc` свое имя.

```console
kubectl get datacontroller/arc --namespace arc
```

```console
kubectl get pods --namespace arc
```

Вы также можете проверить состояние создания любого конкретного модуля, выполнив команду, как показано ниже.  Это особенно полезно для устранения любых проблем.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/control-2g7bl --namespace arc
```

Расширение Arc Azure для Azure Data Studio предоставляет записную книжку, в которой вы узнаете, как настроить Kubernetes Azure Arc и настроить ее для мониторинга репозитория Git, содержащего пример файла YAML SQL Управляемый экземпляр. Когда все подключено, в кластер Kubernetes будет развернута новая Управляемый экземпляр SQL.

См. статью **развертывание управляемый экземпляр SQL с помощью Kubernetes и** записной книжки Flux в расширении azure для Azure Data Studio.

## <a name="troubleshooting-creation-problems"></a>Устранение неполадок при создании

Если у вас возникли роняли с созданием, ознакомьтесь с [руководством по устранению неполадок](troubleshoot-guide.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Создание управляемого экземпляра SQL с помощью собственных средств Kubernetes](./create-sql-managed-instance-using-kubernetes-native-tools.md)
- [Создание группы PostgreSQL Scale Server с помощью Kubernetes-Native Tools](./create-postgresql-hyperscale-server-group-kubernetes-native-tools.md)
