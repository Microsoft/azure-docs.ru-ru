---
title: Добавление или обновление секрета по запросу Red Hat в кластере Azure Red Hat OpenShift 4
description: Добавление или обновление секрета для запроса Red Hat на существующих кластерах АТО 4. x
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 05/21/2020
keywords: секрет по запросу, АТО, openshift, Red Hat
ms.openlocfilehash: 58c0eb2be3423783a69d005277ffe75aaf59415f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100633739"
---
# <a name="add-or-update-your-red-hat-pull-secret-on-an-azure-red-hat-openshift-4-cluster"></a>Добавление или обновление секрета по запросу Red Hat в кластере Azure Red Hat OpenShift 4

В этом руководстве описывается добавление или обновление секрета по запросу Red Hat для существующего кластера Azure Red Hat OpenShift (АТО) 4. x.

При создании кластера в первый раз можно добавить свой секрет при создании кластера. Дополнительные сведения о создании кластера АТО с секретом по запросу Red Hat см. в статье [Создание кластера Azure Red Hat OpenShift 4](tutorial-create-cluster.md#get-a-red-hat-pull-secret-optional).

## <a name="before-you-begin"></a>Перед началом

В этом учебнике предполагается наличие существующего кластера Azure Red Hat OpenShift 4. Убедитесь, что у вас есть доступ администратора к кластеру.

## <a name="prepare-your-pull-secret"></a>Подготовка секрета на вытягивание
При создании кластера АТО без добавления секрета по запросу Red Hat секрет по запросу по-прежнему создается в кластере автоматически. Однако этот секрет для извлечения не полностью заполнен.

В этом разделе описано, как обновить этот опрашивающий секрет на дополнительные значения из секрета по запросу Red Hat.

1. Извлеките секрет с именем `pull-secret` в `openshift-config` пространстве имен и сохраните его в отдельный файл, выполнив следующую команду: 

    ```console
    oc get secrets pull-secret -n openshift-config -o template='{{index .data ".dockerconfigjson"}}' | base64 -d > pull-secret.json
    ```

    Выходные данные должны быть похожи на следующие. (Обратите внимание, что фактическое значение секрета было удалено.)

    ```json
    {
        "auths": {
            "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
        }
    }
    ```

2. Перейдите на [портал диспетчера кластеров Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) и выберите **скачать секрет по запросу**. Секретный код для Red Hat будет выглядеть следующим образом. (Обратите внимание, что фактические значения секрета были удалены.)

    ```json
    {
        "auths": {
            "cloud.openshift.com": {
                "auth": "<my-crc-secret>",
                "email": "klamenzo@redhat.com"
            },
            "quay.io": {
                "auth": "<my-quayio-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.connect.redhat.com": {
                "auth": "<my-registry.connect.redhat.com-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.redhat.io": {
                "auth": "<my-registry.redhat.io-secret>",
                "email": "klamenzo@redhat.com"
            }
        }
    }
    ```

3. Измените файл с секретом на извлечение, полученный из кластера, добавив записи в записях, найденных в секрете на извлечение Red Hat. 

    > [!IMPORTANT]
    > Включение `cloud.openshift.com` записи из секрета по запросу Red Hat приведет к тому, что кластер начнет отправлять данные телеметрии в Red Hat. Включайте этот раздел только в том случае, если вы хотите отправить данные телеметрии. В противном случае не закрывайте следующий раздел.    
    > ```json
    > {
    >         "cloud.openshift.com": {
    >             "auth": "<my-crc-secret>",
    >             "email": "klamenzo@redhat.com"
    >         }
    > ```

    > [!CAUTION]
    > Не удаляйте и не изменяйте пользовательскую `arosvc.azurecr.io` запись из секрета на вытягивание. Этот раздел необходим для правильной работы кластера.

    ```json
    "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
    ```

    Окончательный файл должен выглядеть следующим образом. (Обратите внимание, что фактические значения секрета были удалены.)

    ```json
    {
        "auths": {
            "cloud.openshift.com": {
                "auth": "<my-crc-secret>",
                "email": "klamenzo@redhat.com"
            },
            "quay.io": {
                "auth": "<my-quayio-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.connect.redhat.com": {
                "auth": "<my-registry.connect.redhat.com-secret>",
                "email": "klamenzo@redhat.com"
            },
            "registry.redhat.io": {
                "auth": "<my-registry.redhat.io-secret>",
                "email": "klamenzo@redhat.com"
            },
            "arosvc.azurecr.io": {
                "auth": "<my-aroscv.azurecr.io-secret>"
            }
        }
    }
    ```

4. Убедитесь, что файл является допустимым JSON. Существует множество способов проверки JSON. В следующем примере используется JQ:

    ```json
    cat pull-secret.json | jq
    ```

    > [!NOTE]
    > Если в файле содержится ошибка, она отображается как `parse error` .

## <a name="add-your-pull-secret-to-your-cluster"></a>Добавление секрета для опрашивающего запроса в кластер

Выполните следующую команду, чтобы обновить секрет по запросу.

> [!NOTE]
> Выполнение этой команды приведет к тому, что узлы кластера перезапускают их по мере обновления. 

```console
oc set data secret/pull-secret -n openshift-config --from-file=.dockerconfigjson=./pull-secret.json
```

После настройки секрета вы можете включить операторы сертификации Red Hat.

### <a name="modify-the-configuration-files"></a>Изменение файлов конфигурации

Измените следующие объекты, чтобы включить операторы Red Hat.

Сначала измените файл конфигурации оператора Samples. Затем можно выполнить следующую команду, чтобы изменить файл конфигурации:

```
oc edit configs.samples.operator.openshift.io/cluster -o yaml
```

Измените `spec.architectures.managementState` значения и `status.architecture.managementState` с `Removed` на `Managed` . 

В следующем фрагменте YAML показаны только соответствующие разделы измененного файла YAML:

```yaml
apiVersion: samples.operator.openshift.io/v1
kind: Config
metadata:
  
  ...
  
spec:
  architectures:
  - x86_64
  managementState: Managed
status:
  architectures:

  ...

  managementState: Managed
  version: 4.3.27
```

Во-вторых, выполните следующую команду, чтобы изменить файл конфигурации концентратора оператора:  

```console
oc edit operatorhub cluster -o yaml
```

Измените `Spec.Sources.Disabled` значения и `Status.Sources.Disabled` с `true` на на `false` для всех источников, которые требуется включить.

В следующем фрагменте YAML показаны только соответствующие разделы измененного файла YAML:

```yaml
Name:         cluster

...
                 dd3310b9-e520-4a85-98e5-8b4779ee0f61
Spec:
  Sources:
    Disabled:  false
    Name:      certified-operators
    Disabled:  false
    Name:      redhat-operators
Status:
  Sources:
    Disabled:  false
    Name:      certified-operators
    Status:    Success
    Disabled:  false
    Name:      community-operators
    Status:    Success
    Disabled:  false
    Name:      redhat-operators
    Status:    Success
Events:        <none>
```

Сохраните файл, чтобы применить изменения.

## <a name="validate-that-your-secret-is-working"></a>Проверка работоспособности секрета

После добавления секрета для получения и изменения правильных файлов конфигурации для обновления кластера может потребоваться несколько минут. Чтобы проверить, обновлен ли кластер, выполните следующую команду, чтобы отобразить доступные источники операторы Certified и Red Hat:

```console
$ oc get catalogsource -A
NAMESPACE               NAME                  DISPLAY               TYPE   PUBLISHER   AGE
openshift-marketplace   certified-operators   Certified Operators   grpc   Red Hat     10s
openshift-marketplace   community-operators   Community Operators   grpc   Red Hat     18h
openshift-marketplace   redhat-operators      Red Hat Operators     grpc   Red Hat     11s
```

Если вы не видите операторов Certified и Red Hat, подождите несколько минут и повторите попытку.

Чтобы убедиться, что секрет по запросу обновлен и работает правильно, откройте Операторхуб и проверьте любой оператор проверки Red Hat. Например, проверьте, доступен ли оператор хранилища контейнера OpenShift, и проверьте, есть ли у вас разрешения на установку.

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о важных секретах для Red Hat см. в разделе [использование секретных ключей образа](https://docs.openshift.com/container-platform/4.6/openshift_images/managing_images/using-image-pull-secrets.html).

Дополнительные сведения о Red Hat OpenShift 4 см. в [документации по платформе контейнеров Red Hat OpenShift](https://docs.openshift.com/container-platform/4.6/welcome/index.html).
