---
title: Использование Istio для обеспечения интеллектуальной маршрутизации
titleSuffix: Azure Kubernetes Service
description: Сведения об использовании Istio для обеспечения интеллектуальной маршрутизации и развертывания ранних выпусков в кластере Службы Azure Kubernetes (AKS)
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
zone_pivot_groups: client-operating-system
ms.openlocfilehash: d66f3099ba225fbdd2bfc3d54db56ffd8ed2c43f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94684038"
---
# <a name="use-intelligent-routing-and-canary-releases-with-istio-in-azure-kubernetes-service-aks"></a>Интеллектуальная маршрутизация и ранние выпуски с использованием Istio в службе Azure Kubernetes (AKS)

Сетка [Istio][istio-github] — это решение с открытым исходным кодом, которое предоставляет набор ключевых функций для микрослужб в кластере Kubernetes, включая управление трафиком, удостоверения служб и защиту служб, а также применение политик и возможность мониторинга. См. дополнительные сведения об [Istio][istio-docs-concepts].

В этой статье показано, как использовать возможности управления трафиком в Istio. Пример приложения AKS для голосования позволит ознакомиться с интеллектуальной маршрутизацией и ранними выпусками.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Развертывание приложения
> * Обновление приложения
> * Ранний выпуск приложения
> * Финализация выпуска

## <a name="before-you-begin"></a>Перед началом

> [!NOTE]
> Этот сценарий проверяется с использованием Istio версии `1.3.2`.

В шагах, описанных в этой статье, предполагается, что вы создали кластер AKS (Kubernetes `1.13` и выше с включенным KUBERNETES RBAC) и установили `kubectl` Подключение к кластеру. Также в кластере потребуется установить Istio.

Если требуется помощь по любому из этих вопросов, см. [краткое руководство по AKS][aks-quickstart] и [руководство по установке Istio в AKS][istio-install].

## <a name="about-this-application-scenario"></a>Сведения об этом сценарии приложения

Пример приложения AKS для голосования предоставляет пользователям два варианта голосования (**кошки** или **собаки**). Предоставляется компонент хранения, который сохраняет число голосов за каждый вариант. Кроме того, есть компонент аналитики, предоставляющий подробные сведения о голосах, поданных за каждый вариант.

В этом сценарии приложения описано, как развернуть версию `1.0` приложения для голосования и версию `1.0` компонента аналитики. Компонент аналитики обеспечивает простой подсчет количества голосов. Приложение для голосования и компонент аналитики взаимодействуют с версией `1.0` компонента хранения, поддержку которого обеспечивает Redis.

Выполняется обновление компонента аналитики до версии `1.1`, которая предоставляет счетчики, а также итоги и проценты.

Часть пользователей тестирует версию `2.0` приложения в виде раннего выпуска. В этой новой версии используется компонент хранения, поддерживаемый базой данных MySQL.

Убедившись в том, что версия `2.0` работает ожидаемым образом у группы пользователей, вы выпускаете версию `2.0` для всех пользователей.

## <a name="deploy-the-application"></a>Развертывание приложения

Начнем с развертывания приложения в кластере службы Azure Kubernetes (AKS). На следующей схеме показано, что выполняется в конце этого раздела — версия `1.0` всех компонентов с обслуживанием входящих запросов через шлюз входящего трафика Istio:

![Схема, в которой показаны версии 1,0 всех компонентов с входящими запросами, обслуживаемыми через входящий шлюз Istio.](media/servicemesh/istio/scenario-routing-components-01.png)

Необходимые для выполнения инструкций этой статьи артефакты доступны в репозитории [Azure-Samples/aks-voting-app][github-azure-sample] на GitHub. Вы можете скачать эти артефакты или клонировать репозиторий следующим образом:

```console
git clone https://github.com/Azure-Samples/aks-voting-app.git
```

Перейдите в следующую папку в скачанном или клонированном репозитории и выполните из этой папки все следующие шаги:

```console
cd aks-voting-app/scenarios/intelligent-routing-with-istio
```

Прежде всего создайте пространство имен в кластере AKS для примера приложения AKS для голосования с именем `voting` следующим образом:

```console
kubectl create namespace voting
```

Задайте для пространства имен метку `istio-injection=enabled`. Эта метка указывает Istio автоматически внедрить прокси-серверы istio-proxy в качестве службы-расширения во все группы pod в этом пространстве имен.

```console
kubectl label namespace voting istio-injection=enabled
```

Теперь давайте создадим компоненты для приложения AKS Voting. Эти компоненты следует создать в пространстве имен `voting`, созданном на предыдущем шаге.

```console
kubectl apply -f kubernetes/step-1-create-voting-app.yaml --namespace voting
```

В следующем примере выходных данных показаны создаваемые ресурсы:

```output
deployment.apps/voting-storage-1-0 created
service/voting-storage created
deployment.apps/voting-analytics-1-0 created
service/voting-analytics created
deployment.apps/voting-app-1-0 created
service/voting-app created
```

> [!NOTE]
> Istio устанавливает определенные требования к группам pod и службам. Дополнительные сведения см. в [документации по требованиям к группам pod и службам в Istio][istio-requirements-pods-and-services].

Чтобы просмотреть созданные группы pod, выполните команду [kubectl get pods][kubectl-get] следующим образом:

```console
kubectl get pods -n voting --show-labels
```

В следующем примере выходных данных показано, что существуют три экземпляра pod `voting-app` и по одному экземпляру pod `voting-analytics` и `voting-storage`. Каждая группа pod содержит два контейнера. Один из этих контейнеров представляет собой компонент, а другой — `istio-proxy`:

```output
NAME                                    READY     STATUS    RESTARTS   AGE   LABELS
voting-analytics-1-0-57c7fccb44-ng7dl   2/2       Running   0          39s   app=voting-analytics,pod-template-hash=57c7fccb44,version=1.0
voting-app-1-0-956756fd-d5w7z           2/2       Running   0          39s   app=voting-app,pod-template-hash=956756fd,version=1.0
voting-app-1-0-956756fd-f6h69           2/2       Running   0          39s   app=voting-app,pod-template-hash=956756fd,version=1.0
voting-app-1-0-956756fd-wsxvt           2/2       Running   0          39s   app=voting-app,pod-template-hash=956756fd,version=1.0
voting-storage-1-0-5d8fcc89c4-2jhms     2/2       Running   0          39s   app=voting-storage,pod-template-hash=5d8fcc89c4,version=1.0
```

Чтобы просмотреть сведения о группе pod, воспользуйтесь командой [kubectl describe pod][kubectl-describe] с селекторами меток для выбора группы pod `voting-analytics`. Мы отфильтруем выходные данные, чтобы отобразить сведения о двух контейнерах, присутствующих в группе pod:

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Bash - routing scenario - show autoinjected proxy](includes/servicemesh/istio/scenario-routing-show-proxy-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [Bash - routing scenario - show autoinjected proxy](includes/servicemesh/istio/scenario-routing-show-proxy-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [PowerShell - routing scenario - show autoinjected proxy](includes/servicemesh/istio/scenario-routing-show-proxy-powershell.md)]

::: zone-end

Подключиться к приложению для голосования не удастся до тех пор, пока не будут созданы [шлюз][istio-reference-gateway] Istio и [виртуальная служба][istio-reference-virtualservice]. Эти ресурсы Istio направляют трафик из шлюза Istio для входящего трафика по умолчанию в наше приложение.

> [!NOTE]
> **Шлюз** — это компонент на границе сетки служб, который получает входящий или исходящий трафик HTTP и TCP.
> 
> **Виртуальная служба** определяет набор правил маршрутизации для одной или нескольких служб назначения.

С помощью команды `kubectl apply` выполните развертывание YAML шлюза и виртуальной службы. Не забудьте указать пространство имен, в котором развертываются эти ресурсы.

```console
kubectl apply -f istio/step-1-create-voting-app-gateway.yaml --namespace voting
```

В следующем примере выходных данных показаны создаваемые шлюз и виртуальная служба:

```output
virtualservice.networking.istio.io/voting-app created
gateway.networking.istio.io/voting-app-gateway created
```

Получите IP-адрес шлюза входящего трафика Istio с помощью следующей команды:

```output
kubectl get service istio-ingressgateway --namespace istio-system -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
```

В следующем примере выходных данных показан IP-адрес шлюза входящего трафика:

```output
20.188.211.19
```

Откройте браузер и вставьте в адресную строку IP-адрес. Отобразится пример приложения AKS для голосования.

![Приложение AKS для голосования, работающее в кластере AKS с поддержкой Istio.](media/servicemesh/istio/scenario-routing-deploy-app-01.png)

Сведения в нижней части экрана показывают, что приложение использует версию `1.0` компонента `voting-app` и версию `1.0` компонента `voting-storage` (Redis).

## <a name="update-the-application"></a>Обновление приложения

Давайте развернем новую версию компонента аналитики. Эта новая версия `1.1` отображает итоги и проценты в дополнение к подсчету голосов за каждую категорию.

На следующей схеме показано, что будет выполняться в конце этого раздела — трафик из компонента `voting-app` направляется только в версию `1.1` нашего компонента `voting-analytics`. Хотя версия `1.0` нашего компонента `voting-analytics` продолжает выполняться и на нее ссылается служба `voting-analytics`, прокси-серверы Istio запрещают трафик к ней и от нее.

![На схеме, отображающей только версию 1,1 компонента голосования-Analytics, трафик направляется из компонента голосования-App.](media/servicemesh/istio/scenario-routing-components-02.png)

Давайте развернем версию `1.1` компонента `voting-analytics`. Создайте этот компонент в пространстве имен `voting`:

```console
kubectl apply -f kubernetes/step-2-update-voting-analytics-to-1.1.yaml --namespace voting
```

В следующем примере выходных данных показаны создаваемые ресурсы:

```output
deployment.apps/voting-analytics-1-1 created
```

Откройте снова пример приложения AKS для голосования в браузере, используя IP-адрес шлюза входящего трафика Istio, полученный на предыдущем шаге.

Браузер переключается между двумя представлениями, показанными ниже. Так как [служба Kubernetes][kubernetes-service] используется для компонента `voting-analytics` только с селектором одиночной метки (`app: voting-analytics`), Kubernetes применяет поведение по умолчанию — циклическое переключение между группами pod, соответствующими этому селектору. В нашем примере это обе версии `1.0` и `1.1` групп pod `voting-analytics`.

![Версия 1.0 компонента аналитики, выполняющаяся в приложении AKS Voting.](media/servicemesh/istio/scenario-routing-deploy-app-01.png)

![Версия 1.1 компонента аналитики, выполняющаяся в приложении AKS Voting.](media/servicemesh/istio/scenario-routing-update-app-01.png)

Вы можете визуализировать переключение между двумя версиями компонента `voting-analytics` следующим образом. Не забудьте указать IP-адрес вашего шлюза входящего трафика Istio.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Bash - routing scenario - loop through results](includes/servicemesh/istio/scenario-routing-loop-results-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [Bash - routing scenario - loop through results](includes/servicemesh/istio/scenario-routing-loop-results-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [PowerShell - routing scenario - loop through results](includes/servicemesh/istio/scenario-routing-loop-results-powershell.md)]

::: zone-end

В следующем примере выходных данных показана значимая часть возвращаемого веб-сайта при переключении между версиями.

```output
  <div id="results"> Cats: 2 | Dogs: 4 </div>
  <div id="results"> Cats: 2 | Dogs: 4 </div>
  <div id="results"> Cats: 2/6 (33%) | Dogs: 4/6 (67%) </div>
  <div id="results"> Cats: 2 | Dogs: 4 </div>
  <div id="results"> Cats: 2/6 (33%) | Dogs: 4/6 (67%) </div>
```

### <a name="lock-down-traffic-to-version-11-of-the-application"></a>Ограничение трафика версией 1.1 приложения

Теперь давайте заблокируем трафик, чтобы он поступал только к версии `1.1` компонента `voting-analytics` и версии `1.0` компонента `voting-storage`. Затем следует определить правила маршрутизации для всех остальных компонентов.

> * **Виртуальная служба** определяет набор правил маршрутизации для одной или нескольких служб назначения.
> * **Правило назначения** определяет политики трафика и политики, связанные с определенной версией.
> * **Политика** определяет, какие методы проверки подлинности допустимы для рабочих нагрузок.

С помощью команды `kubectl apply` следует заменить определение виртуальной службы для `voting-app` и добавить [правила назначения][istio-reference-destinationrule] и [виртуальные службы][istio-reference-virtualservice] для остальных компонентов. Следует добавить [политику][istio-reference-policy] к пространству имен `voting`, чтобы обеспечить защиту всех передаваемых между службами данных с помощью взаимной поддержки TLS и сертификатов клиентов.

* В политике для `peers.mtls.mode` устанавливается значение `STRICT`, принудительно применяющее взаимную поддержку TLS для обмена данными между службами в пределах пространства имен `voting`.
* Мы также зададим для `trafficPolicy.tls.mode` значение `ISTIO_MUTUAL` во всех правилах назначения. Istio предоставляет службы с надежными удостоверениями и обеспечивает защиту обмена данными между службами с помощью взаимной поддержки TLS и сертификатов клиентов, которыми Istio управляет прозрачным образом.

```console
kubectl apply -f istio/step-2-update-and-add-routing-for-all-components.yaml --namespace voting
```

В следующем примере выходных данных показано, что новая политика, правила назначения и виртуальные службы обновляются или создаются:

```output
virtualservice.networking.istio.io/voting-app configured
policy.authentication.istio.io/default created
destinationrule.networking.istio.io/voting-app created
destinationrule.networking.istio.io/voting-analytics created
virtualservice.networking.istio.io/voting-analytics created
destinationrule.networking.istio.io/voting-storage created
virtualservice.networking.istio.io/voting-storage created
```

Если снова открыть в браузере приложение AKS для голосования, компонентом `voting-app` будет использоваться только новая версия `1.1` компонента `voting-analytics`.

![Версия 1.1 компонента аналитики, выполняющаяся в приложении AKS Voting.](media/servicemesh/istio/scenario-routing-update-app-01.png)

Вы можете отобразить перенаправление только к версии `1.1` компонента `voting-analytics` следующим образом. Не забудьте указать IP-адрес вашего шлюза входящего трафика Istio:

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Bash - routing scenario - loop through results](includes/servicemesh/istio/scenario-routing-loop-results-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [Bash - routing scenario - loop through results](includes/servicemesh/istio/scenario-routing-loop-results-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [PowerShell - routing scenario - loop through results](includes/servicemesh/istio/scenario-routing-loop-results-powershell.md)]

::: zone-end

В следующем примере выходных данных показана требуемая часть возвращаемого веб-сайта:

```output
  <div id="results"> Cats: 2/6 (33%) | Dogs: 4/6 (67%) </div>
  <div id="results"> Cats: 2/6 (33%) | Dogs: 4/6 (67%) </div>
  <div id="results"> Cats: 2/6 (33%) | Dogs: 4/6 (67%) </div>
  <div id="results"> Cats: 2/6 (33%) | Dogs: 4/6 (67%) </div>
  <div id="results"> Cats: 2/6 (33%) | Dogs: 4/6 (67%) </div>
```

Теперь убедимся, что Istio использует взаимную поддержку TLS для защиты обмена данными между каждой из наших служб. Для этого мы будем использовать команду [authn tls-check][istioctl-authn-tls-check] в двоичном файле клиента `istioctl`, которая принимает следующий вид.

```console
istioctl authn tls-check <pod-name[.namespace]> [<service>]
```

Этот набор команд предоставляет сведения о доступе к указанным службам из всех групп pod, которые находятся в пространстве имен и соответствуют набору меток:

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Bash - routing scenario - verify mtls](includes/servicemesh/istio/scenario-routing-verify-mtls-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [Bash - routing scenario - verify mtls](includes/servicemesh/istio/scenario-routing-verify-mtls-bash.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [PowerShell - routing scenario - verify mtls](includes/servicemesh/istio/scenario-routing-verify-mtls-powershell.md)]

::: zone-end

В следующем примере выходных данных показано, что взаимная поддержка TLS принудительно применяется для каждого из приведенных выше запросов. В выходных данных также отображаются политика и правила назначения, которые обеспечивают взаимную поддержку TLS:

```output
# mTLS configuration between istio ingress pods and the voting-app service
HOST:PORT                                    STATUS     SERVER     CLIENT     AUTHN POLICY       DESTINATION RULE
voting-app.voting.svc.cluster.local:8080     OK         mTLS       mTLS       default/voting     voting-app/voting

# mTLS configuration between each of the voting-app pods and the voting-analytics service
HOST:PORT                                          STATUS     SERVER     CLIENT     AUTHN POLICY       DESTINATION RULE
voting-analytics.voting.svc.cluster.local:8080     OK         mTLS       mTLS       default/voting     voting-analytics/voting
HOST:PORT                                          STATUS     SERVER     CLIENT     AUTHN POLICY       DESTINATION RULE
voting-analytics.voting.svc.cluster.local:8080     OK         mTLS       mTLS       default/voting     voting-analytics/voting
HOST:PORT                                          STATUS     SERVER     CLIENT     AUTHN POLICY       DESTINATION RULE
voting-analytics.voting.svc.cluster.local:8080     OK         mTLS       mTLS       default/voting     voting-analytics/voting

# mTLS configuration between each of the voting-app pods and the voting-storage service
HOST:PORT                                        STATUS     SERVER     CLIENT     AUTHN POLICY       DESTINATION RULE
voting-storage.voting.svc.cluster.local:6379     OK         mTLS       mTLS       default/voting     voting-storage/voting
HOST:PORT                                        STATUS     SERVER     CLIENT     AUTHN POLICY       DESTINATION RULE
voting-storage.voting.svc.cluster.local:6379     OK         mTLS       mTLS       default/voting     voting-storage/voting
HOST:PORT                                        STATUS     SERVER     CLIENT     AUTHN POLICY       DESTINATION RULE
voting-storage.voting.svc.cluster.local:6379     OK         mTLS       mTLS       default/voting     voting-storage/voting

# mTLS configuration between each of the voting-analytics version 1.1 pods and the voting-storage service
HOST:PORT                                        STATUS     SERVER     CLIENT     AUTHN POLICY       DESTINATION RULE
voting-storage.voting.svc.cluster.local:6379     OK         mTLS       mTLS       default/voting     voting-storage/voting
```

## <a name="roll-out-a-canary-release-of-the-application"></a>Ранний выпуск приложения

Теперь развернем новую версию `2.0` компонентов `voting-app`, `voting-analytics` и `voting-storage`. В новом компоненте `voting-storage` используется MySQL вместо Redis, а компоненты `voting-app` и `voting-analytics` обновляются, что позволяет им использовать новый компонент `voting-storage`.

Теперь компонент `voting-app` поддерживает использование флага компонента. Флаг компонента позволяет протестировать возможность Istio осуществить ранний выпуск для группы пользователей.

На следующей схеме показано, что будет выполняться в конце этого раздела.

* Версия `1.0` компонента `voting-app`, версия `1.1` компонента `voting-analytics` и версия `1.0` компонента `voting-storage` могут взаимодействовать между собой.
* Версия `2.0` компонента `voting-app`, версия `2.0` компонента `voting-analytics` и версия `2.0` компонента `voting-storage` могут взаимодействовать между собой.
* Версия `2.0` компонента `voting-app` доступна только пользователям, для которых установлен определенный флаг компонента. Управление этим изменением осуществляется с помощью флага компонента через файл cookie.

![Схема, на которой показано, что вы используете в конце этого раздела.](media/servicemesh/istio/scenario-routing-components-03.png)

Прежде всего обновите правила назначения и виртуальные службы Istio для работы с новыми компонентами. Это гарантирует, что трафик не будет несоответствующим образом направляться к новым компонентам, и пользователи не получат доступ непредвиденным образом:

```console
kubectl apply -f istio/step-3-add-routing-for-2.0-components.yaml --namespace voting
```

В следующем примере выходных данных показано, что правила назначения и виртуальные службы обновляются:

```output
destinationrule.networking.istio.io/voting-app configured
virtualservice.networking.istio.io/voting-app configured
destinationrule.networking.istio.io/voting-analytics configured
virtualservice.networking.istio.io/voting-analytics configured
destinationrule.networking.istio.io/voting-storage configured
virtualservice.networking.istio.io/voting-storage configured
```

Далее добавим объекты Kubernetes для компонентов новой версии `2.0`. Следует также обновить службу `voting-storage`, чтобы включить в нее порт `3306` для MySQL:

```console
kubectl apply -f kubernetes/step-3-update-voting-app-with-new-storage.yaml --namespace voting
```

В следующем примере выходных данных показано, что объекты Kubernetes успешно обновлены или созданы:

```output
service/voting-storage configured
secret/voting-storage-secret created
deployment.apps/voting-storage-2-0 created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/voting-analytics-2-0 created
deployment.apps/voting-app-2-0 created
```

Подождите, пока не будут выполняться все группы pod версии `2.0`. Используйте команду [kubectl get pods][kubectl-get] с переключателем контрольного значения `-w`, чтобы отслеживать изменения во всех группах pod в пространстве имен `voting`:

```console
kubectl get pods --namespace voting -w
```

Теперь у вас должна быть возможность переключаться между версией `1.0` и версией `2.0` (ранний выпуск) приложения для голосования. Переключатель флага компонента в нижней части экрана устанавливает файл cookie. Этот файл cookie используется виртуальной службой `voting-app` для направления пользователей к новой версии `2.0`.

![Версия 1.0 приложения AKS Voting — флаг компонента НЕ задан.](media/servicemesh/istio/scenario-routing-canary-release-01.png)

![Версия 2.0 приложения AKS Voting — флаг компонента ЗАДАН.](media/servicemesh/istio/scenario-routing-canary-release-02.png)

Количество голосов различается между версиями приложения. Это различие подчеркивает, что вы используете два разных внутренних средства для хранения данных.

## <a name="finalize-the-rollout"></a>Финализация выпуска

После успешного тестирования раннего выпуска обновите виртуальную службу `voting-app`, чтобы направить весь трафик к версии `2.0` компонента `voting-app`. После этого всем пользователям будет отображаться версия `2.0` приложения независимо от того, задан ли флаг компонента:

![Схема, показывающая, что пользователи видят версию 2,0 приложения, независимо от того, установлен ли флаг компонента.](media/servicemesh/istio/scenario-routing-components-04.png)

Обновите все правила назначения, чтобы удалить версии компонентов, которые больше не нужны в активном состоянии. Затем обновите все виртуальные службы, убрав ссылки на эти версии.

Так как трафик больше не направляется ни на одну из старых версий компонентов, все эти развернутые компоненты теперь можно безопасно удалить.

![Компоненты и маршрутизация приложения AKS для голосования.](media/servicemesh/istio/scenario-routing-components-05.png)

Вы успешно выпустили новую версию приложения для голосования AKS Voting.

## <a name="clean-up"></a>Очистка 

Приложение AKS для голосования, которое использовалось в этом сценарии, можно удалить из кластера AKS, удалив пространство имен `voting`, как показано ниже.

```console
kubectl delete namespace voting
```

В следующем примере выходных данных показано, что все компоненты приложения AKS для голосования были удалены из кластера AKS.

```output
namespace "voting" deleted
```

## <a name="next-steps"></a>Дальнейшие действия

Вы можете исследовать дополнительные сценарии, используя [пример приложения Istio Bookinfo][istio-bookinfo-example].

<!-- LINKS - external -->
[github-azure-sample]: https://github.com/Azure-Samples/aks-voting-app
[istio-github]: https://github.com/istio/istio

[istio]: https://istio.io
[istio-docs-concepts]: https://istio.io/docs/concepts/what-is-istio/
[istio-requirements-pods-and-services]: https://istio.io/docs/setup/kubernetes/prepare/requirements/
[istio-reference-gateway]: https://istio.io/docs/reference/config/networking/v1alpha3/gateway/
[istio-reference-policy]: https://istio.io/docs/reference/config/istio.mesh.v1alpha1/#AuthenticationPolicy
[istio-reference-virtualservice]: https://istio.io/docs/reference/config/networking/v1alpha3/virtual-service/
[istio-reference-destinationrule]: https://istio.io/docs/reference/config/networking/v1alpha3/destination-rule/
[istio-bookinfo-example]: https://istio.io/docs/examples/bookinfo/
[istioctl-authn-tls-check]: https://istio.io/docs/reference/commands/istioctl/#istioctl-authn-tls-check

[kubernetes-service]: https://kubernetes.io/docs/concepts/services-networking/service/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe

<!-- LINKS - internal -->
[aks-quickstart]: ./kubernetes-walkthrough.md
[istio-install]: ./servicemesh-istio-install.md
