---
title: Использование политик безопасности Pod в службе Kubernetes Azure (AKS)
description: Узнайте, как управлять допуском Pod с помощью Подсекуритиполици в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 02/12/2021
ms.openlocfilehash: cb317e5e0d1f558121e675f569bad37811768ca6
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102180315"
---
# <a name="preview---secure-your-cluster-using-pod-security-policies-in-azure-kubernetes-service-aks"></a>Предварительная версия — защита кластера с помощью политик безопасности Pod в службе Kubernetes Azure (AKS)

> [!WARNING]
> **Функция, описанная в этом документе, политика безопасности Pod (Предварительная версия), задается для устаревания и больше не будет доступна после 30 июня 2021** в пользу [политики Azure для AKS](use-azure-policy.md). Дата устаревания была продлена с предыдущей даты 15 октября 2020 г.
>
> После этой даты политику безопасности pod (предварительная версия) нужно будет отключить на всех затронутых существующих кластерах, чтобы сохранить возможность обновления кластеров в будущем и обеспечить поддержку Azure.
>
> Мы настоятельно рекомендуем начать тестирование сценариев с помощью политики Azure для AKS, которая предоставляет встроенные политики для защиты модулей Pod и встроенных инициатив, которые сопоставляются с политиками безопасности модуля. Чтобы выполнить миграцию из политики безопасности Pod, необходимо выполнить следующие действия в кластере.
> 
> 1. [Отключение политики безопасности Pod](#clean-up-resources) в кластере
> 1. Включение [надстройки политики Azure][kubernetes-policy-reference]
> 1. Включение требуемых политик Azure из [доступных встроенных политик][policy-samples]
> 1. Проверка [изменений поведения между политикой безопасности Pod и политикой Azure](#behavior-changes-between-pod-security-policy-and-azure-policy)

Чтобы повысить безопасность кластера AKS, можно ограничить набор модулей, которые можно запланировать. Модули, которые запрашивают неразрешенные ресурсы, не могут выполняться в кластере AKS. Этот доступ определяется с помощью политик безопасности Pod. В этой статье показано, как использовать политики безопасности Pod для ограничения развертывания модулей AKS.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас есть кластер AKS. Если вам нужен кластер AKS, обратитесь к краткому руководству по работе с AKS [с помощью Azure CLI][aks-quickstart-cli] или [портала Azure][aks-quickstart-portal].

Необходимо установить и настроить Azure CLI версии 2.0.61 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

### <a name="install-aks-preview-cli-extension"></a>Установка расширения интерфейса командной строки предварительной версии AKS

Для использования политик безопасности Pod требуется расширение CLI *AKS-Preview* версии 0.4.1 или более поздней. Установите расширение Azure CLI *aks-preview* с помощью команды [az extension add][az-extension-add], а затем проверьте наличие доступных обновлений с помощью команды [az extension update][az-extension-update].

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="register-pod-security-policy-feature-provider"></a>Регистрация поставщика функций политики безопасности Pod

**Этот документ и функция задаются для устаревания 15 октября, 2020.**

Чтобы создать или обновить кластер AKS для использования политик безопасности Pod, сначала включите в подписке флаг функции. Чтобы зарегистрировать флаг функции *подсекуритиполиципревиев* , используйте команду [AZ Feature Register][az-feature-register] , как показано в следующем примере:

```azurecli-interactive
az feature register --name PodSecurityPolicyPreview --namespace Microsoft.ContainerService
```

Через несколько минут отобразится состояние *Registered* (Зарегистрировано). Состояние регистрации можно проверить с помощью команды [az feature list][az-feature-list].

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/PodSecurityPolicyPreview')].{Name:name,State:properties.state}"
```

Когда все будет готово, обновите регистрацию поставщика ресурсов *Microsoft. ContainerService* с помощью команды [AZ Provider Register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="overview-of-pod-security-policies"></a>Общие сведения о политиках безопасности Pod

В кластере Kubernetes контроллер допуска используется для перехвата запросов к серверу API при создании ресурса. Контроллер допуска может затем *проверить* запрос ресурса на соответствие набору *правил или изменить* его на изменение параметров развертывания.

*Подсекуритиполици* — это контроллер допуска, который проверяет спецификацию Pod на соответствие определенным требованиям. Эти требования могут ограничивать использование привилегированных контейнеров, доступ к определенным типам хранилища, а также пользователя или группу, от имени которых может работать контейнер. При попытке развернуть ресурс, в котором спецификации Pod не соответствуют требованиям, изложенным в политике безопасности Pod, запрос отклоняется. Эта возможность контролировать, какие модули управления доступом к данным могут быть запланированы в кластере AKS, предотвращая некоторые возможные уязвимости или эскалацию привилегий.

При включении политики безопасности Pod в кластере AKS применяются некоторые политики по умолчанию. Эти политики по умолчанию предоставляют готовый интерфейс для определения модулей, которые можно запланировать. Однако пользователи кластера могут столкнуться с проблемами при развертывании модулей Pod, пока не будут определены собственные политики. Рекомендуемый подход:

* Создание кластера AKS
* Определение собственных политик безопасности Pod
* Включение функции политики безопасности Pod

Чтобы продемонстрировать, как политики по умолчанию ограничивают развертывания Pod, в этой статье мы сначала включили функции политик безопасности Pod, а затем создадим настраиваемую политику.

### <a name="behavior-changes-between-pod-security-policy-and-azure-policy"></a>Изменения в работе политики безопасности Pod и политики Azure

Ниже приведена сводка изменений в поведении политики безопасности Pod и политики Azure.

|Сценарий| Политика безопасности Pod | Политика Azure |
|---|---|---|
|Установка|Включить компонент политики безопасности Pod |Включить надстройку политики Azure
|Развертывание политик| Развертывание ресурса политики безопасности Pod| Назначьте политики Azure области подписки или группы ресурсов. Для приложений ресурсов Kubernetes требуется надстройка политики Azure.
| Политики по умолчанию | Если политика безопасности Pod включена в AKS, применяются привилегированные и неограниченные политики по умолчанию. | Политики по умолчанию не применяются при включении надстройки политики Azure. Необходимо явно включить политики в политике Azure.
| Кто может создавать и назначать политики | Администратор кластера создает ресурс политики безопасности Pod | Пользователи должны иметь минимальную роль "владелец" или "участник политики ресурсов" в группе ресурсов кластера AKS. С помощью API пользователи могут назначать политики в области ресурсов кластера AKS. У пользователя должно быть как минимум разрешение "владелец" или "участник политики ресурсов" в ресурсе кластера AKS. — В портал Azure политики могут быть назначены на уровне группы управления, подписки или группы ресурсов.
| Политики авторизации| Пользователям и учетным записям служб требуются явные разрешения на использование политик безопасности Pod. | Для авторизации политик дополнительное назначение не требуется. После назначения политик в Azure все пользователи кластера могут использовать эти политики.
| Применимость политик | Пользователь с правами администратора обходит принудительное применение политик безопасности Pod. | Все пользователи (администратор & без прав администратора) видят одни и те же политики. Специальные регистры на основе пользователей не предусмотрены. Приложение политики можно исключить на уровне пространства имен.
| Область политики | Политики безопасности Pod не являются пространствами имен | Шаблоны ограничений, используемые политикой Azure, не являются пространствами имен.
| Действие "запретить/аудит/изменения" | Политики безопасности Pod поддерживают только запрещенные действия. Изменения можно выполнять со значениями по умолчанию для запросов на создание. Проверку можно выполнить во время запросов на обновление.| Политика Azure поддерживает как аудит, так & запрещать действия. Изменения пока не поддерживаются, но были запланированы.
| Соответствие политике безопасности Pod | Не существует сведений о соответствии модулей Pod, существовавших до включения политики безопасности модуля. Несоответствующие Pod, созданные после включения политик безопасности Pod, отклоняются. | Несоответствующие модули, существовавшие до применения политик Azure, будут отображаться в нарушениях политики. Несоответствующие модули Pod, созданные после включения политик Azure, отклоняются, если для политик задано действие Deny.
| Просмотр политик в кластере | `kubectl get psp` | `kubectl get constrainttemplate` — Возвращаются все политики.
| Политика безопасности Pod "Стандартный — привилегированный" | Ресурс политики безопасности Pod, созданный по умолчанию, создается при включении этой функции. | Привилегированный режим не подразумевает ограничений, поэтому он эквивалентен отсутствию назначения политики Azure.
| [Политика безопасности Pod "Стандартный" — "базовый"/"по умолчанию"](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline-default) | Пользователь устанавливает базовый ресурс политики безопасности Pod. | Политика Azure предоставляет [встроенную базовую инициативу](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) , которая сопоставляется с базовой политикой безопасности Pod.
| [Политика безопасности Pod — ограничена по стандарту](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted) | Пользователь устанавливает ресурс с ограничением политики безопасности Pod. | Политика Azure предоставляет [встроенную ограниченную инициативу](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00) , которая сопоставляется с ограниченной политикой безопасности Pod.

## <a name="enable-pod-security-policy-on-an-aks-cluster"></a>Включение политики безопасности Pod в кластере AKS

Включить или отключить политику безопасности Pod можно с помощью команды [AZ AKS Update][az-aks-update] . В следующем примере включается политика безопасности Pod для имени кластера *myAKSCluster* в группе ресурсов с именем *myResourceGroup*.

> [!NOTE]
> Для реального использования не включайте политику безопасности Pod, пока не будут определены собственные пользовательские политики. В этой статье вы включите политику безопасности Pod в качестве первого шага, чтобы увидеть, как политики по умолчанию ограничивают развертывания Pod.

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --enable-pod-security-policy
```

## <a name="default-aks-policies"></a>Политики AKS по умолчанию

При включении политики безопасности Pod AKS создает одну политику по умолчанию с именем *privileged*. Не изменяйте и не удаляйте политику по умолчанию. Вместо этого создайте собственные политики, определяющие параметры, которые требуется контролировать. Давайте сначала посмотрим, что эти политики по умолчанию влияют на развертывание Pod.

Чтобы просмотреть доступные политики, используйте команду [kubectl Get PSP][kubectl-get] , как показано в следующем примере.

```console
$ kubectl get psp

NAME         PRIV    CAPS   SELINUX    RUNASUSER          FSGROUP     SUPGROUP    READONLYROOTFS   VOLUMES
privileged   true    *      RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
```

Политика безопасности " *привилегированный* модуль" применяется к любому пользователю, прошедшему проверку подлинности в кластере AKS. Это назначение управляется Клустерролес и Клустерролебиндингс. Используйте команду [kubectl Get ролебиндингс][kubectl-get] и выполните поиск по *умолчанию: privileged:* Binding в пространстве имен *KUBE-System* :

```console
kubectl get rolebindings default:privileged -n kube-system -o yaml
```

Как показано в следующем сжатом выводе, параметр *PSP: privileged* клустерроле назначается любой *системе: прошедшим проверку подлинности* пользователям. Эта возможность обеспечивает базовый уровень привилегий без определения собственных политик.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  [...]
  name: default:privileged
  [...]
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp:privileged
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

Важно понимать, как эти политики по умолчанию взаимодействуют с запросами пользователей на планирование модулей Pod, прежде чем приступать к созданию собственных политик безопасности. В следующих разделах мы планируем несколько модулей, чтобы увидеть эти политики по умолчанию в действии.

## <a name="create-a-test-user-in-an-aks-cluster"></a>Создание тестового пользователя в кластере AKS

По умолчанию при использовании команды [AZ AKS Get-Credential][az-aks-get-credentials] учетные данные *администратора* кластера AKS добавляются в `kubectl` конфигурацию. Пользователь с правами администратора обходит принудительное применение политик безопасности Pod. При использовании интеграции Azure Active Directory для кластеров AKS можно выполнить вход с использованием учетных данных пользователя без прав администратора, чтобы увидеть применение политик в действии. В этой статье мы создадим тестовую учетную запись пользователя в кластере AKS, который можно использовать.

Создайте пример пространства имен с именем *PSP-AKS* для тестовых ресурсов с помощью команды [kubectl Create Namespace][kubectl-create] . Затем создайте учетную запись службы с именем *"неадминистративный пользователь* " с помощью команды [kubectl Create учетная запись службы][kubectl-create] :

```console
kubectl create namespace psp-aks
kubectl create serviceaccount --namespace psp-aks nonadmin-user
```

Затем создайте Ролебиндинг для пользователя, который не является *администратором* , чтобы выполнять базовые действия в пространстве имен с помощью команды [kubectl Create ролебиндинг][kubectl-create] :

```console
kubectl create rolebinding \
    --namespace psp-aks \
    psp-aks-editor \
    --clusterrole=edit \
    --serviceaccount=psp-aks:nonadmin-user
```

### <a name="create-alias-commands-for-admin-and-non-admin-user"></a>Создание псевдонимов для администратора и пользователя без прав администратора

Чтобы выделить разницу между обычным пользователем администратора при использовании `kubectl` и пользователь, не являющийся администратором, созданный на предыдущих шагах, создайте два псевдонима командной строки:

* Псевдоним **kubectl-Admin** предназначен для обычного пользователя администратора и ограничивается пространством имен *PSP-AKS* .
* Псевдоним **kubectl-нонадминусер** предназначен для пользователя, который не является *администратором* , созданным на предыдущем шаге, и ограничивается пространством имен *PSP-AKS* .

Создайте эти два псевдонима, как показано в следующих командах:

```console
alias kubectl-admin='kubectl --namespace psp-aks'
alias kubectl-nonadminuser='kubectl --as=system:serviceaccount:psp-aks:nonadmin-user --namespace psp-aks'
```

## <a name="test-the-creation-of-a-privileged-pod"></a>Тестирование создания привилегированного Pod

Давайте сначала проверим, что происходит при планировании Pod с помощью контекста безопасности `privileged: true` . Этот контекст безопасности повышает привилегии Pod. В предыдущем разделе, в котором были показаны политики безопасности AKS Pod по умолчанию, политика *прав доступа* должна отклонить этот запрос.

Создайте файл с именем `nginx-privileged.yaml` и вставьте следующий манифест YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-privileged
spec:
  containers:
    - name: nginx-privileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
      securityContext:
        privileged: true
```

Создайте модуль Pod с помощью команды [kubectl Apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl-nonadminuser apply -f nginx-privileged.yaml
```

Не удалось запланировать модуль Pod, как показано в следующем примере выходных данных:

```console
$ kubectl-nonadminuser apply -f nginx-privileged.yaml

Error from server (Forbidden): error when creating "nginx-privileged.yaml": pods "nginx-privileged" is forbidden: unable to validate against any pod security policy: []
```

Модуль не достигает этапа планирования, поэтому нет ресурсов для удаления перед тем, как вы перейдете.

## <a name="test-creation-of-an-unprivileged-pod"></a>Тестирование создания непривилегированного Pod

В предыдущем примере спецификация Pod запросила привилегированное укрупнение. Этот запрос отклонен политикой безопасности модуля *прав доступа* по умолчанию, поэтому не удалось запланировать модуль Pod. Давайте попробуем запустить этот же модуль NGINX без запроса на эскалацию привилегий.

Создайте файл с именем `nginx-unprivileged.yaml` и вставьте следующий манифест YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
```

Создайте модуль Pod с помощью команды [kubectl Apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl-nonadminuser apply -f nginx-unprivileged.yaml
```

Не удалось запланировать модуль Pod, как показано в следующем примере выходных данных:

```console
$ kubectl-nonadminuser apply -f nginx-unprivileged.yaml

Error from server (Forbidden): error when creating "nginx-unprivileged.yaml": pods "nginx-unprivileged" is forbidden: unable to validate against any pod security policy: []
```

Модуль не достигает этапа планирования, поэтому нет ресурсов для удаления перед тем, как вы перейдете.

## <a name="test-creation-of-a-pod-with-a-specific-user-context"></a>Тестирование создания Pod с конкретным контекстом пользователя

В предыдущем примере образ контейнера автоматически пытался использовать root для привязки NGINX к порту 80. Этот запрос был отклонен политикой безопасности модуля *прав доступа* по умолчанию, поэтому не удается запустить Pod. Давайте попробуем запустить тот же модуль NGINX с конкретным контекстом пользователя, например `runAsUser: 2000` .

Создайте файл с именем `nginx-unprivileged-nonroot.yaml` и вставьте следующий манифест YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged-nonroot
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
      securityContext:
        runAsUser: 2000
```

Создайте модуль Pod с помощью команды [kubectl Apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl-nonadminuser apply -f nginx-unprivileged-nonroot.yaml
```

Не удалось запланировать модуль Pod, как показано в следующем примере выходных данных:

```console
$ kubectl-nonadminuser apply -f nginx-unprivileged-nonroot.yaml

Error from server (Forbidden): error when creating "nginx-unprivileged-nonroot.yaml": pods "nginx-unprivileged-nonroot" is forbidden: unable to validate against any pod security policy: []
```

Модуль не достигает этапа планирования, поэтому нет ресурсов для удаления перед тем, как вы перейдете.

## <a name="create-a-custom-pod-security-policy"></a>Создание пользовательской политики безопасности Pod

Теперь, когда вы узнали о поведении политик безопасности Pod по умолчанию, давайте разберем, что пользователь, не являющийся *администратором* , может успешно запланировать множество модулей.

Давайте создадим политику для отклонения модулей Pod, запрашивающих привилегированный доступ. Другие параметры, такие как *выполняющиеся или* разрешенные *тома*, явно не ограничиваются. Этот тип политики запрещает запрос привилегированного доступа, но в противном случае позволяет кластеру запускать запрошенные модули Pod.

Создайте файл с именем `psp-deny-privileged.yaml` и вставьте следующий манифест YAML:

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp-deny-privileged
spec:
  privileged: false
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'
```

Создайте политику с помощью команды [kubectl Apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl apply -f psp-deny-privileged.yaml
```

Чтобы просмотреть доступные политики, используйте команду [kubectl Get PSP][kubectl-get] , как показано в следующем примере. Сравните политику *PSP-Deny-Privileges* с политикой *привилегий* по умолчанию, которая была применена в предыдущих примерах для создания Pod. Только использование эскалации *Priv* запрещено политикой. Для политики *PSP-Deny-Privileges* не существует ограничений на пользователя или группу.

```console
$ kubectl get psp

NAME                  PRIV    CAPS   SELINUX    RUNASUSER          FSGROUP     SUPGROUP    READONLYROOTFS   VOLUMES
privileged            true    *      RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *
psp-deny-privileged   false          RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *          
```

## <a name="allow-user-account-to-use-the-custom-pod-security-policy"></a>Разрешить учетной записи пользователя использовать пользовательскую политику безопасности Pod

На предыдущем шаге вы создали политику безопасности Pod для отклонения модулей, запрашивающих привилегированный доступ. Чтобы разрешить использование политики, создайте *роль* или *клустерроле*. Затем вы связываете одну из этих ролей с помощью *ролебиндинг* или *клустерролебиндинг*.

В этом примере создайте Клустерроле, который позволит *использовать* политику *PSP-Deny-privileged* , созданную на предыдущем шаге. Создайте файл с именем `psp-deny-privileged-clusterrole.yaml` и вставьте следующий манифест YAML:

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-deny-privileged-clusterrole
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - psp-deny-privileged
  verbs:
  - use
```

Создайте Клустерроле с помощью команды [kubectl Apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl apply -f psp-deny-privileged-clusterrole.yaml
```

Теперь создайте Клустерролебиндинг для использования Клустерроле, созданного на предыдущем шаге. Создайте файл с именем `psp-deny-privileged-clusterrolebinding.yaml` и вставьте следующий манифест YAML:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: psp-deny-privileged-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp-deny-privileged-clusterrole
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:serviceaccounts
```

Создайте Клустерролебиндинг с помощью команды [kubectl Apply][kubectl-apply] и укажите имя манифеста YAML:

```console
kubectl apply -f psp-deny-privileged-clusterrolebinding.yaml
```

> [!NOTE]
> На первом шаге этой статьи функция политики безопасности Pod была включена в кластере AKS. Рекомендуется включить функцию политики безопасности Pod только после определения собственных политик. На этом этапе можно включить функцию политики безопасности Pod. Была определена одна или несколько пользовательских политик, а учетные записи пользователей были связаны с этими политиками. Теперь можно безопасно включить функцию политики безопасности Pod и избежать проблем, вызванных политиками по умолчанию.

## <a name="test-the-creation-of-an-unprivileged-pod-again"></a>Повторное тестирование создания непривилегированного модуля Pod

Применяя пользовательскую политику безопасности Pod и привязку учетной записи пользователя для использования политики, давайте попробуем снова создать непривилегированный модуль. Используйте тот же `nginx-privileged.yaml` манифест для создания Pod с помощью команды [kubectl Apply][kubectl-apply] :

```console
kubectl-nonadminuser apply -f nginx-unprivileged.yaml
```

Модуль POD успешно запланирован. При проверке состояния Pod с помощью команды [kubectl Get][kubectl-get] Pod *выполняется*:

```
$ kubectl-nonadminuser get pods

NAME                 READY   STATUS    RESTARTS   AGE
nginx-unprivileged   1/1     Running   0          7m14s
```

В этом примере показано, как создать пользовательские политики безопасности Pod для определения доступа к кластеру AKS для разных пользователей или групп. Политики AKS по умолчанию предоставляют ограниченный контроль над тем, какие модули Pod могут выполняться, поэтому создайте собственные пользовательские политики, чтобы правильно определить необходимые ограничения.

Удалите непривилегированный модуль NGINX с помощью команды [kubectl Delete][kubectl-delete] и укажите имя манифеста YAML:

```console
kubectl-nonadminuser delete -f nginx-unprivileged.yaml
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы отключить политику безопасности Pod, выполните команду [AZ AKS Update][az-aks-update] еще раз. В следующем примере выполняется отключение политики безопасности Pod для имени кластера *myAKSCluster* в группе ресурсов с именем *myResourceGroup*:

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --disable-pod-security-policy
```

Затем удалите Клустерроле и Клустерролебиндинг:

```console
kubectl delete -f psp-deny-privileged-clusterrolebinding.yaml
kubectl delete -f psp-deny-privileged-clusterrole.yaml
```

Удалите политику безопасности с помощью команды [kubectl Delete][kubectl-delete] и укажите имя манифеста YAML:

```console
kubectl delete -f psp-deny-privileged.yaml
```

Наконец, удалите пространство имен *PSP-AKS* :

```console
kubectl delete namespace psp-aks
```

## <a name="next-steps"></a>Дальнейшие действия

В этой статье показано, как создать политику безопасности Pod, чтобы предотвратить использование привилегированного доступа. Существует множество функций, которые может применять политика, например тип тома или пользователя запуска от имени. Дополнительные сведения о доступных параметрах см. в [справочнике по политикам безопасности Kubernetes Pod][kubernetes-policy-reference].

Дополнительные сведения об ограничении сетевого трафика Pod см. [в статье Защита трафика между модулями данных с помощью сетевых политик в AKS][network-policies].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[kubernetes-policy-reference]: https://kubernetes.io/docs/concepts/policy/pod-security-policy/#policy-reference

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[network-policies]: use-network-policies.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-update]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-update
[az-extension-add]: /cli/azure/extension#az-extension-add
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[policy-samples]: ./policy-reference.md#microsoftcontainerservice
