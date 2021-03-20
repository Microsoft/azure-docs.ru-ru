---
title: Устранение распространенных неполадок
description: Узнайте, как устранять распространенные проблемы, возникающие при развертывании, запуске или управлении экземплярами контейнеров Azure.
ms.topic: article
ms.date: 06/25/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: d8e7fb85e369f5f278436370944eafeb1fb6a50e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96779521"
---
# <a name="troubleshoot-common-issues-in-azure-container-instances"></a>Устранение распространенных неполадок с помощью службы "Экземпляры контейнеров Azure"

В этой статье показано, как устранять распространенные неполадки при развертывании контейнеров и управлении ими в службе "Экземпляры контейнеров Azure". См. также [часто задаваемые вопросы](container-instances-faq.md).

Если требуется дополнительная поддержка, см. раздел доступная **Справка и варианты поддержки** в [портал Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

## <a name="issues-during-container-group-deployment"></a>Проблемы при развертывании группы контейнеров
### <a name="naming-conventions"></a>Соглашения об именах

При определении спецификации контейнера некоторые параметры требуют соблюдения ограничений на именование. Ниже приведена таблица особых требований для свойств группы контейнеров. Дополнительные сведения см. в разделе [соглашения об именовании][azure-name-restrictions] в центр архитектуры Azure и [правила именования и ограничения для ресурсов Azure][naming-rules].

| Область | Длина | Регистр | Допустимые знаки | Рекомендуемый шаблон | Пример |
| --- | --- | --- | --- | --- | --- |
| Имя контейнера<sup>1</sup> | 1–63 |Строчные буквы | Буквы, цифры, дефис в любом месте, за исключением первого знака |`<name>-<role>-container<number>` |`web-batch-container1` |
| Порты контейнера | От 1 до 65 535 |Целое число |Целое число от 1 до 65 535. |`<port-number>` |`443` |
| Метка DNS-имени | 5–63 |Без учета регистра |Буквы, цифры, дефис в любом месте, за исключением первого знака |`<name>` |`frontend-site1` |
| Переменная среды | 1–63 |Без учета регистра |Буквы, цифры и подчеркивание (_) в любом месте, за исключением первого знака |`<name>` |`MY_VARIABLE` |
| Имя тома | 5–63 |Строчные буквы |Буквенно-цифровые символы и дефисы в любом месте, кроме первого или последнего символа. Не может содержать два дефиса подряд. |`<name>` |`batch-output-volume` |

<sup>1</sup> Ограничение также для имен групп контейнеров, если оно не указано независимо от экземпляров контейнера, например при `az container create` развертывании команд.

### <a name="os-version-of-image-not-supported"></a>Версия ОС образа не поддерживается

Если вы укажете образ, не поддерживаемый службой "Экземпляры контейнеров Azure", будет возвращена ошибка `OsVersionNotSupported`. Ошибка похожа на следующую, где `{0}` — это имя образа, который вы пытались развернуть:

```json
{
  "error": {
    "code": "OsVersionNotSupported",
    "message": "The OS version of image '{0}' is not supported."
  }
}
```

Чаще всего эта ошибка возникает при развертывании образов Windows на основе Semi-Annual канала выпуска 1709 или 1803, которые не поддерживаются. Сведения о поддерживаемых образах Windows в службе "экземпляры контейнеров Azure" см. в разделе [часто задаваемые вопросы](container-instances-faq.md#what-windows-base-os-images-are-supported).

### <a name="unable-to-pull-image"></a>Сбой получения образа

Если службе "Экземпляры контейнеров Azure" не удалось изначально получить образ, в течение некоторого периода она будет повторять эту операцию. Если операция по извлечению образа продолжает завершаться сбоем, ACI в конечном итоге не выполняет развертывание, и вы можете увидеть ошибку `Failed to pull image`.

Чтобы устранить эту проблему, удалите экземпляр контейнера и повторите развертывание. Убедитесь, что образ существует в реестре и что имя образа введено верно.

Если образ невозможно получить, в выходных данных команды [az container show][az-container-show] отображаются следующие события:

```bash
"events": [
  {
    "count": 3,
    "firstTimestamp": "2017-12-21T22:56:19+00:00",
    "lastTimestamp": "2017-12-21T22:57:00+00:00",
    "message": "pulling image \"mcr.microsoft.com/azuredocs/aci-hellowrld\"",
    "name": "Pulling",
    "type": "Normal"
  },
  {
    "count": 3,
    "firstTimestamp": "2017-12-21T22:56:19+00:00",
    "lastTimestamp": "2017-12-21T22:57:00+00:00",
    "message": "Failed to pull image \"mcr.microsoft.com/azuredocs/aci-hellowrld\": rpc error: code 2 desc Error: image t/aci-hellowrld:latest not found",
    "name": "Failed",
    "type": "Warning"
  },
  {
    "count": 3,
    "firstTimestamp": "2017-12-21T22:56:20+00:00",
    "lastTimestamp": "2017-12-21T22:57:16+00:00",
    "message": "Back-off pulling image \"mcr.microsoft.com/azuredocs/aci-hellowrld\"",
    "name": "BackOff",
    "type": "Normal"
  }
],
```
### <a name="resource-not-available-error"></a>Ошибка при недоступном ресурсе

Из-за изменения нагрузки региональных ресурсов в Azure при попытке развернуть экземпляр контейнера может появляться следующая ошибка:

`The requested resource with 'x' CPU and 'y.z' GB memory is not available in the location 'example region' at this moment. Please retry with a different resource request or in another location.`

Эта ошибка означает, что из-за высокой нагрузки в регионе, в котором вы пытаетесь выполнить развертывание, сейчас не удается выделить ресурсы, указанные для контейнера. Чтобы устранить эту проблему, используйте один или несколько способов, указанных ниже.

* Убедитесь, что параметры развертывания контейнера соответствуют параметрам, определенным в статье [Доступность службы "Экземпляры контейнеров Azure" в регионах](container-instances-region-availability.md).
* Укажите для контейнера более низкие значения ЦП и памяти.
* Выполните развертывание в другом регионе Azure.
* Выполните развертывание позднее.

## <a name="issues-during-container-group-runtime"></a>Проблемы во время выполнения группы контейнеров
### <a name="container-continually-exits-and-restarts-no-long-running-process"></a>Контейнер постоянно завершает работу и перезагружается (не очень долгий процесс)

Контейнерные группы по умолчанию выполняют [политику перезапуска](container-instances-restart-policy.md)**Always**, поэтому контейнеры в группе контейнеров всегда перезапускаются после завершения. Может потребоваться изменить ее на **OnFailure** или **Never**, если планируется запуск контейнеров на основе задач. Если вы настроили **OnFailure**, но контейнер по-прежнему перезагружается, возможно, существует проблема в выполняемом в контейнере приложении или скрипте.

При запуске групп контейнеров без длительных процессов можно видеть повторные выходы и перезапуски образов, такие как Ubuntu или Alpine. Подключение через [EXEC](container-instances-exec.md) не будет работать, так как контейнер не имеет процесса, поддерживающего его. Чтобы устранить эту проблему, включите в развертывание группы контейнеров команду запуска, подобную приведенной ниже, для сохранения работоспособности контейнера.

```azurecli-interactive
## Deploying a Linux container
az container create -g MyResourceGroup --name myapp --image ubuntu --command-line "tail -f /dev/null"
```

```azurecli-interactive 
## Deploying a Windows container
az container create -g myResourceGroup --name mywindowsapp --os-type Windows --image mcr.microsoft.com/windows/servercore:ltsc2019
 --command-line "ping -t localhost"
```

API Экземпляров контейнеров и портала Azure включают свойство `restartCount`. Чтобы узнать количество перезагрузок контейнера, выполните в Azure CLI команду [az container show][az-container-show]. Ниже приведен пример выходных данных (сокращены для удобства), в конце которого можно увидеть свойство `restartCount`.

```json
...
 "events": [
   {
     "count": 1,
     "firstTimestamp": "2017-11-13T21:20:06+00:00",
     "lastTimestamp": "2017-11-13T21:20:06+00:00",
     "message": "Pulling: pulling image \"myregistry.azurecr.io/aci-tutorial-app:v1\"",
     "type": "Normal"
   },
   {
     "count": 1,
     "firstTimestamp": "2017-11-13T21:20:14+00:00",
     "lastTimestamp": "2017-11-13T21:20:14+00:00",
     "message": "Pulled: Successfully pulled image \"myregistry.azurecr.io/aci-tutorial-app:v1\"",
     "type": "Normal"
   },
   {
     "count": 1,
     "firstTimestamp": "2017-11-13T21:20:14+00:00",
     "lastTimestamp": "2017-11-13T21:20:14+00:00",
     "message": "Created: Created container with id bf25a6ac73a925687cafcec792c9e3723b0776f683d8d1402b20cc9fb5f66a10",
     "type": "Normal"
   },
   {
     "count": 1,
     "firstTimestamp": "2017-11-13T21:20:14+00:00",
     "lastTimestamp": "2017-11-13T21:20:14+00:00",
     "message": "Started: Started container with id bf25a6ac73a925687cafcec792c9e3723b0776f683d8d1402b20cc9fb5f66a10",
     "type": "Normal"
   }
 ],
 "previousState": null,
 "restartCount": 0
...
}
```

> [!NOTE]
> Большинство образов контейнера для дистрибутивов Linux настраивают оболочку, например bash, в качестве команды по умолчанию. Оболочка сама по себе не является долго работающей службой. Поэтому, если для этих контейнеров настроена политика перезагрузки **Always**, они сразу же завершают работу и начинают циклически перезагружаться.

### <a name="container-takes-a-long-time-to-start"></a>Контейнер долго запускается

Ниже приведены три основных фактора, которые влияют на время запуска контейнера в службе "экземпляры контейнеров Azure".

* [Размер изображения](#image-size)
* [Расположение образа](#image-location)
* [Кэшированные изображения](#cached-images)

Для образов Windows действуют [дополнительные рекомендации](#cached-images).

#### <a name="image-size"></a>Размер образа

Если контейнер запускается долго, но в конечном итоге успешно, обратите внимание на размер образа контейнера. Так как служба "Экземпляры контейнеров Azure" извлекает образ контейнера по требованию, время запуска непосредственно связано с его размером.

Размер образа контейнера можно просмотреть с помощью команды `docker images` в интерфейсе командной строки Docker:

```console
$ docker images
REPOSITORY                                    TAG       IMAGE ID        CREATED          SIZE
mcr.microsoft.com/azuredocs/aci-helloworld    latest    7367f3256b41    15 months ago    67.6MB
```

Чтобы обеспечить маленький размер образа, нужно добавить в него только то, что требуется во время выполнения. Один из способов — это [многоэтапная сборка][docker-multi-stage-builds]. За счет многоэтапной сборки можно очень просто гарантировать, что итоговый образ содержит артефакты, необходимые для приложения, и в нем отсутствует дополнительное содержимое, которое требовалось во время сборки.

#### <a name="image-location"></a>Расположение образа

Чтобы получение образа меньше влияло на время запуска контейнера, вы также можете разместить образ контейнера в [реестре контейнеров Azure](../container-registry/index.yml) того же региона, в котором будут развернуты экземпляры контейнеров. Таким образом сокращается сетевой путь, который должен пройти образ контейнера, а также значительно сокращается время загрузки.

#### <a name="cached-images"></a>Кэшированные изображения

Служба "экземпляры контейнеров Azure" использует механизм кэширования, позволяющий ускорить время запуска контейнера для образов, построенных на [основе общих базовых образов Windows](container-instances-faq.md#what-windows-base-os-images-are-supported), включая `nanoserver:1809` , `servercore:ltsc2019` и `servercore:1809` . Часто используемые образы Linux, такие как `ubuntu:1604` и `alpine:3.6` , также кэшируются. Для образов Windows и Linux не следует использовать `latest` тег. Ознакомьтесь с рекомендациями по [тегу образа](../container-registry/container-registry-image-tag-version.md) в реестре контейнеров. Чтобы получить актуальный список кэшированных изображений и тегов, используйте API [List с кэшированными изображениями][list-cached-images] .

> [!NOTE]
> Возможность использования образов на основе Windows Server 2019 в службе "Экземпляры контейнеров Azure" предоставляется в режиме предварительной версии.

#### <a name="windows-containers-slow-network-readiness"></a>Контейнеры Windows замедляют период готовности сети

В контейнерах Windows при первоначальном создании в течение 30 секунд (в редких случаях и дольше) может отсутствовать входящее или исходящее подключение. Если приложению-контейнеру требуется подключение к Интернету, добавьте логику задержки и повторного выполнения, позволяющую через 30 секунд установить подключение к Интернету. После начальной установки работа с контейнерами в сети будет восстановлена соответствующим образом.

### <a name="cannot-connect-to-underlying-docker-api-or-run-privileged-containers"></a>Не удается подключиться к основному API Docker или запустить привилегированные контейнеры.

Служба "Экземпляры контейнеров Azure" не предоставляет прямой доступ к базовой инфраструктуре, в которой размещены группы контейнеров. Это включает в себя доступ к API Docker, запущенному на узле контейнера, и выполнение привилегированных контейнеров. Если требуется взаимодействие с Docker, обратитесь к [справочной документации по REST](/rest/api/container-instances/), чтобы узнать о поддерживаемых API в службе "Экземпляры контейнеров Azure". Также можно отправить запрос на [форумах обратной связи ACI](https://aka.ms/aci/feedback).

### <a name="container-group-ip-address-may-not-be-accessible-due-to-mismatched-ports"></a>IP-адрес группы контейнеров может быть недоступен из-за несовпадений портов

Служба "экземпляры контейнеров Azure" пока не поддерживает сопоставление портов, как в обычных конфигурациях DOCKER. Если IP-адрес группы контейнеров недоступен, если вы считаете, что он должен быть, убедитесь, что вы настроили образ контейнера для прослушивания тех же портов, которые вы предоставляете в группе контейнеров, с помощью `ports` Свойства.

Если вы хотите убедиться, что служба "экземпляры контейнеров Azure" может прослушивать порт, настроенный в образе контейнера, протестируйте развертывание `aci-helloworld` образа, предоставляющего порт. Также запустите `aci-helloworld` приложение, чтобы оно прослушивает порт. `aci-helloworld` принимает необязательную переменную среды `PORT` для переопределения порта по умолчанию 80, который он прослушивает. Например, чтобы проверить порт 9000, задайте [переменную среды](container-instances-environment-variables.md) при создании группы контейнеров:

1. Настройте группу контейнеров для предоставления порта 9000 и передайте номер порта в качестве значения переменной среды. Этот пример отформатирован для оболочки bash. Если вы предпочитаете другую оболочку, например PowerShell или командную строку, необходимо соответствующим образом настроить назначение переменных.
    ```azurecli
    az container create --resource-group myResourceGroup \
    --name mycontainer --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --ip-address Public --ports 9000 \
    --environment-variables 'PORT'='9000'
    ```
1. Найдите IP-адрес группы контейнеров в выходных данных команды `az container create` . Найдите значение **IP**. 
1. После успешного завершения подготовки контейнера перейдите к IP-адресу и порту приложения контейнера в браузере, например: `192.0.2.0:9000` . 

    Вы должны увидеть "Добро пожаловать в службу" экземпляры контейнеров Azure ". сообщение, отображаемое веб-приложением.
1. Завершив работу с контейнером, удалите его с помощью `az container delete` команды:

    ```azurecli
    az container delete --resource-group myResourceGroup --name mycontainer
    ```

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [получить журналы и события контейнеров](container-instances-get-logs.md) , чтобы упростить отладку контейнеров.

<!-- LINKS - External -->
[azure-name-restrictions]: /azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging#naming-and-tagging-resources
[naming-rules]: ../azure-resource-manager/management/resource-name-rules.md
[windows-sac-overview]: /windows-server/get-started/semi-annual-channel-overview
[docker-multi-stage-builds]: https://docs.docker.com/engine/userguide/eng-image/multistage-build/
[docker-hub-windows-core]: https://hub.docker.com/_/microsoft-windows-servercore
[docker-hub-windows-nano]: https://hub.docker.com/_/microsoft-windows-nanoserver

<!-- LINKS - Internal -->
[az-container-show]: /cli/azure/container#az-container-show
[list-cached-images]: /rest/api/container-instances/location/listcachedimages
