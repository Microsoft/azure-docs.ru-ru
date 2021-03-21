---
title: Использование контейнеров речевых служб с Kubernetes и Helm
titleSuffix: Azure Cognitive Services
description: Используя Kubernetes и Helm для определения образов контейнеров преобразования речи в текст и преобразования текста в речь, мы создадим пакет Kubernetes. Этот пакет будет развернут в кластере Kubernetes в локальной среде.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/30/2020
ms.author: aahi
ms.openlocfilehash: 78ac9ae4aa8611f50caa94c84d3e6c95e58fc91c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102200739"
---
# <a name="use-speech-service-containers-with-kubernetes-and-helm"></a>Использование контейнеров речевых служб с Kubernetes и Helm

Одним из вариантов управления речевыми контейнерами в локальной среде является использование Kubernetes и Helm. Используя Kubernetes и Helm для определения образов контейнеров преобразования речи в текст и преобразования текста в речь, мы создадим пакет Kubernetes. Этот пакет будет развернут в кластере Kubernetes в локальной среде. Наконец, мы продемонстрируем, как тестировать развернутые службы и различные параметры конфигурации. Дополнительные сведения о запуске контейнеров DOCKER без оркестрации Kubernetes см. в [статье Установка и запуск контейнеров речевых служб](speech-container-howto.md).

## <a name="prerequisites"></a>Предварительные условия

Перед использованием контейнеров речи в локальной среде выполните следующие предварительные требования:

| Обязательно | Назначение |
|----------|---------|
| Учетная запись Azure | Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись][free-azure-account], прежде чем начинать работу. |
| Доступ к реестру контейнеров | Чтобы Kubernetes мог извлечь образы DOCKER в кластер, ему потребуется доступ к реестру контейнеров. |
| Kubernetes CLI | Интерфейс [командной строки Kubernetes][kubernetes-cli] требуется для управления общими учетными данными из реестра контейнеров. Kubernetes также требуется перед Helm, который является диспетчером пакетов Kubernetes. |
| Интерфейс командной строки Helm | Установите интерфейс [командной строки Helm][helm-install], который используется для установки диаграммы Helm (определение пакета контейнера). |
|Речевой ресурс |Для использования контейнеров необходимо следующее:<br><br>_Речевой_ ресурс Azure для получения связанного ключа выставления счета и URI конечной точки выставления счетов. Оба значения доступны на страницах "Обзор **речи** " и "ключи" портал Azure и необходимы для запуска контейнера.<br><br>**{API_KEY}**: ключ ресурса<br><br>**{ENDPOINT_URI}**: пример URI конечной точки: `https://westus.api.cognitive.microsoft.com/sts/v1.0`|

## <a name="the-recommended-host-computer-configuration"></a>Рекомендуемая конфигурация главного компьютера

См. сведения о [главном компьютере контейнера службы речевой][speech-container-host-computer] обработки в качестве справочной информации. Эта *Helm диаграмма* автоматически вычисляет требования к ЦП и памяти в зависимости от того, сколько декодирований (параллельных запросов) указывает пользователь. Кроме того, он будет изменяться в зависимости от того, настроены ли в качестве оптимизации для ввода звука и текста `enabled` . По умолчанию Helm диаграмма имеет значение, два параллельных запроса и отключена оптимизация.

| Служба | ЦП или контейнер | Память/контейнер |
|--|--|--|
| **Преобразование речи в текст** | для одного декодера требуется не менее 1 150 миллиардах. Если `optimizedForAudioFile` включен, то требуется 1 950 миллиардах. (по умолчанию: два декодера) | Требуется: 2 ГБ<br>Ограничено: 4 ГБ |
| **Преобразование текста в речь** | для одного одновременного запроса требуется минимум 500 миллиардах. Если `optimizeForTurboMode` включен, то требуется 1 000 миллиардах. (по умолчанию: два одновременных запроса) | Требуется: 1 ГБ<br> Ограничено: 2 ГБ |

## <a name="connect-to-the-kubernetes-cluster"></a>Подключение к кластеру Kubernetes

Предполагается, что на главном компьютере есть доступный кластер Kubernetes. Ознакомьтесь с руководством по [развертыванию кластера Kubernetes](../../aks/tutorial-kubernetes-deploy-cluster.md) для концептуального понимания того, как развернуть кластер Kubernetes на главном компьютере.

## <a name="configure-helm-chart-values-for-deployment"></a>Настройка значений диаграммы Helm для развертывания

Посетите [центр Microsoft Helm][ms-helm-hub] для всех общедоступных диаграмм Helm, предлагаемых корпорацией Майкрософт. В центре Microsoft Helm вы найдете **локальную диаграмму Cognitive Services речь**. **Локальная Cognitive Services речь** — это диаграмма, которую мы будем устанавливать, но сначала необходимо создать `config-values.yaml` файл с явной конфигурацией. Начнем с добавления репозитория Майкрософт в наш экземпляр Helm.

```console
helm repo add microsoft https://microsoft.github.io/charts/repo
```

Далее предстоит настроить наши значения диаграммы Helm. Скопируйте и вставьте следующий YAML в файл с именем `config-values.yaml` . Дополнительные сведения о настройке **Cognitive Servicesной диаграммы для перевода речи в локальной среде** см. в разделе [Настройка диаграмм Helm](#customize-helm-charts). Замените `# {ENDPOINT_URI}` комментарии и `# {API_KEY}` собственными значениями.

```yaml
# These settings are deployment specific and users can provide customizations
# speech-to-text configurations
speechToText:
  enabled: true
  numberOfConcurrentRequest: 3
  optimizeForAudioFile: true
  image:
    registry: mcr.microsoft.com
    repository: azure-cognitive-services/speechservices/speech-to-text
    tag: latest
    pullSecrets:
      - mcr # Or an existing secret
    args:
      eula: accept
      billing: # {ENDPOINT_URI}
      apikey: # {API_KEY}

# text-to-speech configurations
textToSpeech:
  enabled: true
  numberOfConcurrentRequest: 3
  optimizeForTurboMode: true
  image:
    registry: mcr.microsoft.com
    repository: azure-cognitive-services/speechservices/speech-to-text
    tag: latest
    pullSecrets:
      - mcr # Or an existing secret
    args:
      eula: accept
      billing: # {ENDPOINT_URI}
      apikey: # {API_KEY}
```

> [!IMPORTANT]
> Если `billing` значения и `apikey` не указаны, срок действия служб истечет через 15 минут. Аналогичным образом проверка завершится ошибкой, так как службы будут недоступны.

### <a name="the-kubernetes-package-helm-chart"></a>Пакет Kubernetes (диаграмма Helm)

*Диаграмма Helm* содержит конфигурацию образов DOCKER, которые необходимо извлечь из `mcr.microsoft.com` реестра контейнеров.

> [Helm диаграмма][helm-charts] — это коллекция файлов, описывающих связанный набор ресурсов Kubernetes. Отдельную диаграмму можно использовать для развертывания чего-либо простого, такого как memcached Pod или что-то сложного, подобно полному стеку веб-приложений с серверами HTTP, базами данных, кэшами и т. д.

Предоставленные *Helm диаграммы* извлекают образы DOCKER службы Speech, а также текстовые и речевые службы из `mcr.microsoft.com` реестра контейнеров.

## <a name="install-the-helm-chart-on-the-kubernetes-cluster"></a>Установка диаграммы Helm в кластере Kubernetes

Чтобы установить *диаграмму Helm* , необходимо выполнить [`helm install`][helm-install-cmd] команду, заменив параметр на `<config-values.yaml>` соответствующий путь и аргумент имени файла. `microsoft/cognitive-services-speech-onpremise`Диаграмма Helm, указанная ниже, доступна в [центре Helm Майкрософт][ms-helm-hub-speech-chart].

```console
helm install onprem-speech microsoft/cognitive-services-speech-onpremise \
    --version 0.1.1 \
    --values <config-values.yaml> 
```

Ниже приведен пример выходных данных, которые можно ожидать при успешном выполнении установки.

```console
NAME:   onprem-speech
LAST DEPLOYED: Tue Jul  2 12:51:42 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                             READY  STATUS             RESTARTS  AGE
speech-to-text-7664f5f465-87w2d  0/1    Pending            0         0s
speech-to-text-7664f5f465-klbr8  0/1    ContainerCreating  0         0s
text-to-speech-56f8fb685b-4jtzh  0/1    ContainerCreating  0         0s
text-to-speech-56f8fb685b-frwxf  0/1    Pending            0         0s

==> v1/Service
NAME            TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)       AGE
speech-to-text  LoadBalancer  10.0.252.106  <pending>    80:31811/TCP  1s
text-to-speech  LoadBalancer  10.0.125.187  <pending>    80:31247/TCP  0s

==> v1beta1/PodDisruptionBudget
NAME                                MIN AVAILABLE  MAX UNAVAILABLE  ALLOWED DISRUPTIONS  AGE
speech-to-text-poddisruptionbudget  N/A            20%              0                    1s
text-to-speech-poddisruptionbudget  N/A            20%              0                    1s

==> v1beta2/Deployment
NAME            READY  UP-TO-DATE  AVAILABLE  AGE
speech-to-text  0/2    2           0          0s
text-to-speech  0/2    2           0          0s

==> v2beta2/HorizontalPodAutoscaler
NAME                       REFERENCE                  TARGETS        MINPODS  MAXPODS  REPLICAS  AGE
speech-to-text-autoscaler  Deployment/speech-to-text  <unknown>/50%  2        10       0         0s
text-to-speech-autoscaler  Deployment/text-to-speech  <unknown>/50%  2        10       0         0s


NOTES:
cognitive-services-speech-onpremise has been installed!
Release is named onprem-speech
```

Для завершения развертывания Kubernetes может потребоваться несколько минут. Чтобы убедиться в правильности развертывания и доступности модулей Pod и служб, выполните следующую команду:

```console
kubectl get all
```

Должны отобразиться примерно такие результаты:

```console
NAME                                  READY     STATUS    RESTARTS   AGE
pod/speech-to-text-7664f5f465-87w2d   1/1       Running   0          34m
pod/speech-to-text-7664f5f465-klbr8   1/1       Running   0          34m
pod/text-to-speech-56f8fb685b-4jtzh   1/1       Running   0          34m
pod/text-to-speech-56f8fb685b-frwxf   1/1       Running   0          34m

NAME                     TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
service/kubernetes       ClusterIP      10.0.0.1       <none>           443/TCP        3h
service/speech-to-text   LoadBalancer   10.0.252.106   52.162.123.151   80:31811/TCP   34m
service/text-to-speech   LoadBalancer   10.0.125.187   65.52.233.162    80:31247/TCP   34m

NAME                             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/speech-to-text   2         2         2            2           34m
deployment.apps/text-to-speech   2         2         2            2           34m

NAME                                        DESIRED   CURRENT   READY     AGE
replicaset.apps/speech-to-text-7664f5f465   2         2         2         34m
replicaset.apps/text-to-speech-56f8fb685b   2         2         2         34m

NAME                                                            REFERENCE                   TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
horizontalpodautoscaler.autoscaling/speech-to-text-autoscaler   Deployment/speech-to-text   1%/50%    2         10        2          34m
horizontalpodautoscaler.autoscaling/text-to-speech-autoscaler   Deployment/text-to-speech   0%/50%    2         10        2          34m
```

### <a name="verify-helm-deployment-with-helm-tests"></a>Проверка развертывания Helm с помощью тестов Helm

Установленные диаграммы Helm определяют *тесты Helm*, которые служат для удобства проверки. Эти тесты проверяют готовность службы. Чтобы проверить службы преобразования **речи в текст** и преобразования **текста в речь** , мы выполним команду [Helm Test][helm-test] .

```console
helm test onprem-speech
```

> [!IMPORTANT]
> Эти тесты завершатся ошибкой, если состояние POD не равно `Running` или если развертывание не указано в `AVAILABLE` столбце. Подождите, так как выполнение этого может занять больше десяти минут.

Эти тесты выводят различные результаты состояния:

```console
RUNNING: speech-to-text-readiness-test
PASSED: speech-to-text-readiness-test
RUNNING: text-to-speech-readiness-test
PASSED: text-to-speech-readiness-test
```

В качестве альтернативы выполнению *тестов Helm* можно получить *внешние IP-* адреса и соответствующие порты из `kubectl get all` команды. Используя IP-адрес и порт, откройте веб-браузер и перейдите к, `http://<external-ip>:<port>:/swagger/index.html` чтобы просмотреть страницы SWAGGER API.

## <a name="customize-helm-charts"></a>Настройка диаграмм Helm

Helm диаграммы являются иерархическими. Иерархия позволяет выполнять наследование диаграммы, оно также относится к концепции особенностей, где параметры, которые более специфичны для переопределения унаследованных правил.

[!INCLUDE [Speech umbrella-helm-chart-config](includes/speech-umbrella-helm-chart-config.md)]

[!INCLUDE [Speech-to-Text Helm Chart Config](includes/speech-to-text-chart-config.md)]

[!INCLUDE [Text-to-Speech Helm Chart Config](includes/text-to-speech-chart-config.md)]

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об установке приложений с помощью Helm в службе Kubernetes Azure (AKS) см. [здесь][installing-helm-apps-in-aks].

> [!div class="nextstepaction"]
> [Контейнеры Cognitive Services][cog-svcs-containers]

<!-- LINKS - external -->
[free-azure-account]: https://azure.microsoft.com/free
[git-download]: https://git-scm.com/downloads
[azure-cli]: /cli/azure/install-azure-cli
[docker-engine]: https://www.docker.com/products/docker-engine
[kubernetes-cli]: https://kubernetes.io/docs/tasks/tools/install-kubectl
[helm-install]: https://helm.sh/docs/intro/install/
[helm-install-cmd]: https://helm.sh/docs/intro/using_helm/#helm-install-installing-a-package
[tiller-install]: https://helm.sh/docs/install/#installing-tiller
[helm-charts]: https://helm.sh/docs/topics/charts/
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[helm-test]: https://v2.helm.sh/docs/helm/#helm-test
[ms-helm-hub]: https://hub.helm.sh/charts/microsoft
[ms-helm-hub-speech-chart]: https://hub.helm.sh/charts/microsoft/cognitive-services-speech-onpremise

<!-- LINKS - internal -->
[speech-container-host-computer]: speech-container-howto.md#the-host-computer
[installing-helm-apps-in-aks]: ../../aks/kubernetes-helm.md
[cog-svcs-containers]: ../cognitive-services-container-support.md