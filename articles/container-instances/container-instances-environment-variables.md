---
title: Установка переменных среды в экземпляре контейнера
description: Узнайте, как настроить переменные среды в контейнерах, выполняемых в службе "Экземпляры контейнеров Azure"
ms.topic: article
ms.date: 04/17/2019
ms.openlocfilehash: 92ae59f69b7cb43fee1d3ce8190a85fc20a11f60
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86169771"
---
# <a name="set-environment-variables-in-container-instances"></a>Установка переменных среды в экземплярах контейнеров

Используя переменные среды в экземплярах контейнеров, вы можете динамически изменять конфигурацию приложения или скрипта, запущенных в контейнере. Эта функция работает так же, как и аргумент `--env` в команде `docker run`. 

Чтобы установить переменные среды в контейнере, укажите их при создании экземпляра контейнера. В этой статье приведены примеры настройки переменных среды при запуске контейнера с [Azure CLI](#azure-cli-example), [Azure PowerShell](#azure-powershell-example)и [портал Azure](#azure-portal-example). 

Например, при запуске образа контейнера Microsoft [ACI-WordCount][aci-wordcount] можно изменить его поведение, указав следующие переменные среды:

*NumWords* задает число слов для вывода в STDOUT.

*MinLength* обозначает минимальную длину учитываемых слов в символах. Если увеличить это значение, скрипт будет пропускать самые распространенные слова, например of и the.

Если нужно передать секреты в качестве переменных среды, служба "Экземпляры контейнеров Azure" поддерживает [безопасные значения](#secure-values) для контейнеров Windows и Linux.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="azure-cli-example"></a>Пример для Azure CLI

Чтобы просмотреть выходные данные по умолчанию для контейнера [ACI-WordCount][aci-wordcount] , сначала запустите его с помощью команды [AZ Container Create][az-container-create] (не указаны переменные среды):

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer1 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure
```

Чтобы изменить выходные данные, запустите второй контейнер с `--environment-variables` добавленным аргументом, указав значения для переменных *Нумвордс* и *minLength* . (В этом примере предполагается, что вы используете интерфейс командной строки оболочки Bash или Azure Cloud Shell. Если вы используете командную строку Windows, заключите переменные в двойные кавычки, например: `--environment-variables "NumWords"="5" "MinLength"="8"`.)

```azurecli-interactive
az container create \
    --resource-group myResourceGroup \
    --name mycontainer2 \
    --image mcr.microsoft.com/azuredocs/aci-wordcount:latest \
    --restart-policy OnFailure \
    --environment-variables 'NumWords'='5' 'MinLength'='8'
```

Как только состояние обоих контейнеров станет *Terminated* (Завершен) (используйте [az container show][az-container-show] для проверки состояния), откройте их журналы с помощью [az container logs][az-container-logs], чтобы просмотреть выходные данные.

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer1
az container logs --resource-group myResourceGroup --name mycontainer2
```

Выходные данные контейнеров показывают, как вы изменили поведение скрипта второго контейнера, установив переменные среды.

**mycontainer1**
```output
[('the', 990),
 ('and', 702),
 ('of', 628),
 ('to', 610),
 ('I', 544),
 ('you', 495),
 ('a', 453),
 ('my', 441),
 ('in', 399),
 ('HAMLET', 386)]
```

**mycontainer2**
```output
[('CLAUDIUS', 120),
 ('POLONIUS', 113),
 ('GERTRUDE', 82),
 ('ROSENCRANTZ', 69),
 ('GUILDENSTERN', 54)]
```

## <a name="azure-powershell-example"></a>Пример для Azure PowerShell

Установка переменных среды в PowerShell аналогична действиям в CLI, за исключением использования аргумента командной строки `-EnvironmentVariable`.

Сначала запустите контейнер [ACI-WordCount][aci-wordcount] в конфигурации по умолчанию с помощью этой команды [New-азконтаинерграуп][new-Azcontainergroup] :

```azurepowershell-interactive
New-AzContainerGroup `
    -ResourceGroupName myResourceGroup `
    -Name mycontainer1 `
    -Image mcr.microsoft.com/azuredocs/aci-wordcount:latest
```

Теперь выполните следующую команду [New-азконтаинерграуп][new-Azcontainergroup] . Она задает переменные среды *NumWords* и *MinLength* после заполнения переменной массива `envVars`:

```azurepowershell-interactive
$envVars = @{'NumWords'='5';'MinLength'='8'}
New-AzContainerGroup `
    -ResourceGroupName myResourceGroup `
    -Name mycontainer2 `
    -Image mcr.microsoft.com/azuredocs/aci-wordcount:latest `
    -RestartPolicy OnFailure `
    -EnvironmentVariable $envVars
```

После того как оба контейнера *прерываются* (используйте команду [Get-азконтаинеринстанцелог][azure-instance-log] для проверки состояния), вытяните свои журналы с помощью команды [Get-азконтаинеринстанцелог][azure-instance-log] .

```azurepowershell-interactive
Get-AzContainerInstanceLog -ResourceGroupName myResourceGroup -ContainerGroupName mycontainer1
Get-AzContainerInstanceLog -ResourceGroupName myResourceGroup -ContainerGroupName mycontainer2
```

Выходные данные каждого контейнера показывают, как вы изменили скрипт, выполняемый контейнером, установив переменные среды.

```console
PS Azure:\> Get-AzContainerInstanceLog -ResourceGroupName myResourceGroup -ContainerGroupName mycontainer1
[('the', 990),
 ('and', 702),
 ('of', 628),
 ('to', 610),
 ('I', 544),
 ('you', 495),
 ('a', 453),
 ('my', 441),
 ('in', 399),
 ('HAMLET', 386)]

Azure:\
PS Azure:\> Get-AzContainerInstanceLog -ResourceGroupName myResourceGroup -ContainerGroupName mycontainer2
[('CLAUDIUS', 120),
 ('POLONIUS', 113),
 ('GERTRUDE', 82),
 ('ROSENCRANTZ', 69),
 ('GUILDENSTERN', 54)]

Azure:\
```

## <a name="azure-portal-example"></a>Пример на портале Azure

Чтобы задать переменные среды при запуске контейнера в портал Azure, укажите их на странице **Дополнительно** при создании контейнера.

1. На странице **Дополнительно** задайте для параметра **Политика перезапуска** значение *при сбое* .
2. В разделе **переменные среды** введите `NumWords` со значением `5` для первой переменной и введите `MinLength` со значением `8` для второй переменной. 
1. Выберите **проверить и создать** , чтобы проверить, а затем развернуть контейнер.

![Страница портала, отображающая переменную среды, кнопку "Включить" и текстовые поля][portal-env-vars-01]

Чтобы просмотреть журналы контейнера, в разделе **Параметры** выберите **контейнеры**, а затем **журналы**. Подобно тому, как это было показано в предыдущих разделах для CLI и PowerShell, вы можете увидеть, как поведение скрипта было изменено после указания переменных среды. Отображаются только пять слов, каждое из которых имеет длину не менее восьми символов.

![Выходные данные журналов контейнера на портале][portal-env-vars-02]

## <a name="secure-values"></a>Безопасные значения

Объекты с безопасными значениями предназначены для хранения конфиденциальных данных, например паролей или ключей приложения. Использовать безопасные значения для переменных среды — более безопасный и гибкий способ по сравнению с добавлением значений в образ контейнера. Другой вариант — использовать тома secret, описанные в статье [Подключение тома secret в службе "Экземпляры контейнеров Azure"](container-instances-volume-secret.md).

Переменные среды с безопасными значениями не видны в свойствах контейнера — доступ к их значениям можно получить только из контейнера. Например, свойства контейнера, просматриваемые на портале Azure или с помощью Azure CLI, отображают только имя безопасной переменной, а не ее значение.

Установите безопасную переменную среды, указав для типа переменной свойство `secureValue` вместо обычного `value`. В следующем файле YAML представлены две переменные двух разных типов.

### <a name="yaml-deployment"></a>Развертывание файла YAML

Создайте файл `secure-env.yaml` со следующим фрагментом кода.

```yaml
apiVersion: 2019-12-01
location: eastus
name: securetest
properties:
  containers:
  - name: mycontainer
    properties:
      environmentVariables:
        - name: 'NOTSECRET'
          value: 'my-exposed-value'
        - name: 'SECRET'
          secureValue: 'my-secret-value'
      image: nginx
      ports: []
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Выполните следующую команду, чтобы развернуть группу контейнеров с помощью YAML (при необходимости измените имя группы ресурсов):

```azurecli-interactive
az container create --resource-group myResourceGroup --file secure-env.yaml
```

### <a name="verify-environment-variables"></a>Проверка переменных среды

Запустите команду [az container show][az-container-show], чтобы запросить переменные среды контейнера.

```azurecli-interactive
az container show --resource-group myResourceGroup --name securetest --query 'containers[].environmentVariables'
```

Ответ JSON отображает ключ и значение небезопасной переменной среды и только имя безопасной переменной среды:

```json
[
  [
    {
      "name": "NOTSECRET",
      "secureValue": null,
      "value": "my-exposed-value"
    },
    {
      "name": "SECRET",
      "secureValue": null,
      "value": null
    }
  ]
]
```

С помощью команды [az container exec][az-container-exec], которая позволяет выполнить команду в запущенном контейнере, вы можете проверить, была ли установлена безопасная переменная среды. Выполните следующую команду, чтобы запустить интерактивный сеанс Bash в контейнере:

```azurecli-interactive
az container exec --resource-group myResourceGroup --name securetest --exec-command "/bin/bash"
```

После открытия интерактивной оболочки в контейнере вы можете получить доступ к значению переменной `SECRET`:

```console
root@caas-ef3ee231482549629ac8a40c0d3807fd-3881559887-5374l:/# echo $SECRET
my-secret-value
```

## <a name="next-steps"></a>Дальнейшие действия

В скриптах на основе задач, таких как пакетная обработка большого набора данных с несколькими контейнерами, могут использоваться пользовательские переменные среды во время выполнения. Дополнительные сведения о выполнении контейнеров на основе задач см. [в статье выполнение контейнерных задач с помощью политик перезапуска](container-instances-restart-policy.md).

<!-- IMAGES -->
[portal-env-vars-01]: ./media/container-instances-environment-variables/portal-env-vars-01.png
[portal-env-vars-02]: ./media/container-instances-environment-variables/portal-env-vars-02.png

<!-- LINKS - External -->
[aci-wordcount]: https://hub.docker.com/_/microsoft-azuredocs-aci-wordcount

<!-- LINKS Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-exec]: /cli/azure/container#az-container-exec
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[azure-cli-install]: /cli/azure/
[azure-instance-log]: /powershell/module/az.containerinstance/get-azcontainerinstancelog
[azure-powershell-install]: /powershell/azure/install-Az-ps
[new-Azcontainergroup]: /powershell/module/az.containerinstance/new-azcontainergroup
[portal]: https://portal.azure.com
