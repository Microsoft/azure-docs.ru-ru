---
title: Создание управляемого экземпляра SQL с помощью средств Kubernetes
description: Создание управляемого экземпляра SQL с помощью средств Kubernetes
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 02/11/2021
ms.topic: how-to
ms.openlocfilehash: d23df80a3f80ed96779297bac12ef0ed8d2927d5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101687929"
---
# <a name="create-azure-sql-managed-instance-using-kubernetes-tools"></a>Создание управляемого экземпляра SQL Azure с помощью средств Kubernetes

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="prerequisites"></a>Предварительные условия

Вы уже создали [контроллер данных Arc Azure](./create-data-controller.md).

Чтобы создать управляемый экземпляр SQL с помощью средств Kubernetes, необходимо установить средства Kubernetes.  Примеры в этой статье будут использовать `kubectl` , но аналогичные подходы можно использовать с другими средствами Kubernetes, такими как панель мониторинга Kubernetes, `oc` или `helm` Если вы знакомы с этими инструментами и Kubernetes YAML/JSON.

[Установка средства kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## <a name="overview"></a>Обзор

Чтобы создать управляемый экземпляр SQL, необходимо создать секрет Kubernetes для безопасного хранения имени входа и пароля системного администратора, а также настраиваемого ресурса управляемого экземпляра SQL на основе определения настраиваемого ресурса склманажединстанце.

## <a name="create-a-yaml-file"></a>Создание файла YAML

Файл [шаблона YAML](https://raw.githubusercontent.com/microsoft/azure_arc/main/arc_data_services/deploy/yaml/sqlmi.yaml) можно использовать в качестве отправной точки для создания собственного пользовательского файла управляемого экземпляра SQL YAML.  Скачайте этот файл на локальный компьютер и откройте его в текстовом редакторе.  Полезно использовать текстовый редактор, например [VS Code](https://code.visualstudio.com/download) , поддерживающий выделение синтаксиса и linting для файлов YAML.

Это пример файла YAML:

```yaml
apiVersion: v1
data:
  password: <your base64 encoded password>
  username: <your base64 encoded user name. 'sa' is not allowed>
kind: Secret
metadata:
  name: sql1-login-secret
type: Opaque
---
apiVersion: sql.arcdata.microsoft.com/v1alpha1
kind: sqlmanagedinstance
metadata:
  name: sql1
spec:
  limits:
    memory: 4Gi
    vcores: "4"
  requests:
    memory: 2Gi
    vcores: "1"
  service:
    type: LoadBalancer
  storage:
    data:
      className: default
      size: 5Gi
    logs:
      className: default
      size: 1Gi
```

### <a name="customizing-the-login-and-password"></a>Настройка имени входа и пароля

Секрет Kubernetes хранится в виде строки в кодировке Base64 — одна для имени пользователя, а другая — для пароля.  Необходимо, чтобы система Base64 заменяла имя входа и пароль системного администратора и поместились в заполнитель в расположение по адресу `data.password` и `data.username` .  Не включайте `<` символы и, `>` указанные в шаблоне.

> [!NOTE]
> Для обеспечения оптимальной безопасности использование значения SA для имени входа не допускается.
> Следуйте [политике сложности паролей](/sql/relational-databases/security/password-policy#password-complexity).

Вы можете использовать онлайн-инструмент для кодирования требуемого имени пользователя и пароля в Base64 или использовать встроенные средства CLI в зависимости от платформы.

PowerShell

```console
[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('<your string to encode here>'))

#Example
#[Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes('example'))

```

Linux/macOS

```console
echo -n '<your string to encode here>' | base64

#Example
# echo -n 'example' | base64
```

### <a name="customizing-the-name"></a>Настройка имени

Шаблон имеет значение "SQL1" для атрибута Name.  Его можно изменить, но должны быть символами, которые следуют стандартам DNS-имен.  Необходимо также изменить имя секрета на Match.  Например, если изменить имя управляемого экземпляра SQL на "sql2", необходимо изменить имя секрета с "SQL1-login-Secret" на "sql2-login-Secret".

### <a name="customizing-the-resource-requirements"></a>Настройка требований к ресурсам

При необходимости можно изменить требования к ресурсам для ОЗУ и ядер и запросов.  

> [!NOTE]
> Дополнительные сведения см. в статье [Управление ресурсами Kubernetes](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes).

Требования к ограничениям ресурсов и запросам:
- В целях выставления счетов **требуется** значение ограничения ядер.
- Остальные запросы и ограничения ресурсов являются необязательными.
- Ограничение количества ядер и запрос должны быть положительным целым числом, если оно указано.
- Минимальное количество ядер, необходимое для запроса Core, —, если оно указано.
- Формат значения памяти соответствует нотации Kubernetes.  
- Для запроса памяти требуется минимум 2Gi, если он указан.
- В качестве общей рекомендации необходимо иметь 4 ГБ ОЗУ для каждого 1 ядра в рабочих вариантах использования.

### <a name="customizing-service-type"></a>Настройка типа службы

При необходимости тип службы можно изменить на Нодепорт.  Будет назначен случайный номер порта.

### <a name="customizing-storage"></a>Настройка хранилища

Классы хранения можно настроить для хранения в соответствии с вашей средой.  Если вы не уверены, какие классы хранилища доступны, можно выполнить команду, `kubectl get storageclass` чтобы просмотреть их.  Шаблон имеет значение по умолчанию "default".  Это означает, что существует класс хранения _с именем_ Default, не имеющий класса _хранения, используемого_ по умолчанию.  При необходимости можно также изменить размер хранилища.  Дополнительные сведения о [конфигурации хранилища](./storage-configuration.md)см. в этой статье.

## <a name="creating-the-sql-managed-instance"></a>Создание управляемого экземпляра SQL

Теперь, когда вы настроили файл YAML SQL с управляемым экземпляром, вы можете создать управляемый экземпляр SQL Server, выполнив следующую команду:

```console
kubectl create -n <your target namespace> -f <path to your yaml file>

#Example
#kubectl create -n arc -f C:\arc-data-services\sqlmi.yaml
```


## <a name="monitoring-the-creation-status"></a>Наблюдение за состоянием создания

Создание управляемого экземпляра SQL займет несколько минут. Вы можете отслеживать ход выполнения в другом окне терминала с помощью следующих команд:

> [!NOTE]
>  В примерах команд ниже предполагается, что вы создали управляемый экземпляр SQL с именем "SQL1" и пространством имен Kubernetes с именем "Arc".  Если вы использовали другое пространство имен или управляемое имя экземпляра SQL, вы можете заменить "Arc" и "склми" своими именами.

```console
kubectl get sqlmi/sql1 --namespace arc
```

```console
kubectl get pods --namespace arc
```

Вы также можете проверить состояние создания любого конкретного модуля, выполнив команду, как показано ниже.  Это особенно полезно для устранения любых проблем.

```console
kubectl describe po/<pod name> --namespace arc

#Example:
#kubectl describe po/sql1-0 --namespace arc
```

## <a name="troubleshooting-creation-problems"></a>Устранение неполадок при создании

Если у вас возникли роняли с созданием, ознакомьтесь с [руководством по устранению неполадок](troubleshoot-guide.md).

## <a name="next-steps"></a>Дальнейшие действия

[Подключение к SQL Управляемый экземпляр с включенной службой Arc](connect-managed-instance.md)
