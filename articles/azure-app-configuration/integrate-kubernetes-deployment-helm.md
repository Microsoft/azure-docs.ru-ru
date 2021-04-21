---
title: Интеграция службы "Конфигурация приложений Azure" с развертыванием Kubernetes с помощью Helm
description: Сведения о том, как использовать динамические конфигурации для развертываний Kubernetes с помощью Helm.
services: azure-app-configuration
author: shenmuxiaosen
manager: zhenlan
ms.service: azure-app-configuration
ms.topic: tutorial
ms.date: 04/14/2020
ms.author: shuawan
ms.openlocfilehash: 6276fc2027e92d5b7baaf9237a928e7828a3b021
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107775774"
---
# <a name="integrate-with-kubernetes-deployment-using-helm"></a>Интеграция с развертыванием Kubernetes с помощью Helm

Helm предоставляет способ для определения, установки и обновления приложений, работающих в Kubernetes. Чарт Helm содержит важные сведения для создания экземпляра приложения Kubernetes. Эта конфигурация хранится вне чарта, в файле с именем *values.yaml*. 

В процессе выпуска Helm объединяет чарт с правильной конфигурацией для запуска приложения. Например, определенные в файле *values.yaml* переменные можно использовать как переменные среды в выполняющихся контейнерах. Также Helm поддерживает создание секретов Kubernetes, которые можно подключать как тома данных или предоставлять как переменные среды.

Вы можете переопределить значения, сохраненные в *values.yaml*, предоставив дополнительные файлы конфигурации на основе YAML в командной строке при запуске Helm. Служба "Конфигурация приложений Azure" поддерживает экспорт значений конфигурации в YAML-файлы. Интеграция этой функции экспорта в развертывание позволяет приложениям Kubernetes использовать значения конфигурации, хранящиеся в службе "Конфигурация приложений".

В этом руководстве описано следующее:
> [!div class="checklist"]
> * использование значений из службы "Конфигурация приложений" при развертывании приложения в Kubernetes с помощью Helm;
> * создание секрета Kubernetes на основе ссылки на Key Vault в службе "Конфигурация приложений".

В этом руководстве предполагается, что у вас есть базовое понимание принципов управления Kubernetes с помощью Helm. Дополнительные сведения об установке приложений с помощью Helm в Службе Azure Kubernetes см. [здесь](../aks/kubernetes-helm.md).

## <a name="prerequisites"></a>Предварительные требования

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
- Установленный [интерфейс командной строки Azure](/cli/azure/install-azure-cli) версии 2.4.0 или более поздней.
- Установленный [Helm](https://helm.sh/docs/intro/install/) версии 2.14.0 или более поздней.
- Кластер Kubernetes.

## <a name="create-an-app-configuration-store"></a>Создание хранилища Конфигурации приложений

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

7. Выберите **Обозреватель конфигураций** > **Создать**, чтобы добавить указанные ниже пары "ключ — значение":

    | Клавиши | Значение |
    |---|---|
    | settings.color | White |
    | settings.message | Данные из конфигурации приложения Azure |

    Поля **Метка** и **Тип контента** пока заполнять не нужно.

## <a name="add-a-key-vault-reference-to-app-configuration"></a>Добавление ссылки на Key Vault в службу "Конфигурация приложений Azure"
1. Войдите на [портал Azure](https://portal.azure.com) и добавьте в [Key Vault](../key-vault/secrets/quick-create-portal.md#add-a-secret-to-key-vault) секрет с именем **Password** и значением **myPassword**. 
2. Выберите экземпляр хранилища службы "Конфигурация приложений", который вы создали в предыдущем разделе.

3. Выберите **Обозреватель конфигураций**.

4. Выберите **+ Создать** > **Ссылка на хранилище ключей** и укажите следующие значения:
    - **Ключ**: Выберите **secrets.password**.
    - **Метка** — оставьте это значение пустым.
    - **Подписка**, **Группа ресурсов** и **Хранилище ключей**: введите значения, соответствующие значениям хранилища ключей, созданного на предыдущем шаге.
    - **Секрет**: выберите секрет с именем **Password**, созданный в предыдущем разделе.

## <a name="create-helm-chart"></a>Создание чарта Helm ##
Сначала создайте пример чарта Helm с помощью такой команды:
```console
helm create mychart
```

Helm создает новый каталог с именем mychart и показанной ниже структурой. 

> [!TIP]
> Дополнительные сведения см. в [руководстве по чартам](https://helm.sh/docs/chart_template_guide/getting_started/).

```
mychart
|-- Chart.yaml
|-- charts
|-- templates
|   |-- NOTES.txt
|   |-- _helpers.tpl
|   |-- deployment.yaml
|   |-- ingress.yaml
|   `-- service.yaml
`-- values.yaml
```

Затем обновите раздел **spec:template:spec:containers** в файле *deployment.yaml*. Следующий фрагмент кода добавляет для контейнера две переменные среды. Значения вы будете задавать динамически во время развертывания.

```yaml
env:
- name: Color
    value: {{ .Values.settings.color }}
- name: Message
    value: {{ .Values.settings.message }}
``` 

Полный файл *deployment.yaml* после обновления должен выглядеть следующим образом.

```yaml
apiVersion: apps/v1beta2
kind: Deployment
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "mychart.name" . }}
    helm.sh/chart: {{ include "mychart.chart" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
    app.kubernetes.io/managed-by: {{ .Release.Service }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "mychart.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "mychart.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          env:
            - name: Color
              value: {{ .Values.settings.color }}
            - name: Message
              value: {{ .Values.settings.message }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
{{ toYaml .Values.resources | indent 12 }}
    {{- with .Values.nodeSelector }}
      nodeSelector:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.affinity }}
      affinity:
{{ toYaml . | indent 8 }}
    {{- end }}
    {{- with .Values.tolerations }}
      tolerations:
{{ toYaml . | indent 8 }}
    {{- end }}
```

Чтобы сохранить конфиденциальные данные в виде секретов Kubernetes, добавьте файл *secrets.yaml* в папку templates.

> [!TIP]
> См. дополнительные сведения об использовании [секретов Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: {{ .Values.secrets.password | b64enc }}
```

Наконец, обновите файл *values.yaml*, добавив следующее содержимое со значениями по умолчанию для параметров конфигурации и секретов, указанных в файлах *deployment.yaml* и *secrets.yaml*. Их фактические значения будут перезаписаны конфигурацией, извлеченной из службы "Конфигурация приложений".

```yaml
# settings will be overwritten by App Configuration
settings:
    color: red
    message: myMessage
```

## <a name="pass-configuration-from-app-configuration-in-helm-install"></a>Передача конфигурации из службы "Конфигурация приложений" при установке Helm ##
Сначала в службе "Конфигурация приложений" скачайте конфигурацию в файл *myConfig.yaml*. Чтобы скачать только те ключи, которые начинаются со строки **settings.** , примените фильтр ключей. Если для вашего случая фильтр ключей не может исключить все упомянутые в Key Vault ключи, попробуйте использовать для их исключения аргумент **--skip-keyvault**. 

> [!TIP]
> Дополнительные сведения см. в статье о [команде export](/cli/azure/appconfig/kv#az_appconfig_kv_export). 

```azurecli-interactive
az appconfig kv export -n myAppConfiguration -d file --path myConfig.yaml --key "settings.*"  --separator "." --format yaml
```

Затем скачайте секреты в файл с именем *mySecrets.yaml*. Аргумент командной строки **--resolve-keyvault** разрешает ссылки на Key Vault, получая фактические значения из Key Vault. Для выполнения этой команды вам придется указать учетные данные с правами на доступ к соответствующему Key Vault.

> [!WARNING]
> Так как этот файл содержит конфиденциальные сведения, храните его с осторожностью и очистите, когда потребность в нем отпадет.

```azurecli-interactive
az appconfig kv export -n myAppConfiguration -d file --path mySecrets.yaml --key "secrets.*" --separator "." --resolve-keyvault --format yaml
```

Используйте аргумент **-f** для обновления Helm, чтобы передать два созданных файла конфигурации. Они переопределят значения конфигурации, определенные в файле *values.yaml*, экспортированными из службы "Конфигурация приложений" значениями.

```console
helm upgrade --install -f myConfig.yaml -f mySecrets.yaml "example" ./mychart 
```

Вы также можете передать литеральные значения ключей с помощью аргумента **--set** для обновления Helm. Использование аргумента **--set** позволяет избежать сохранения конфиденциальных данных на диск. 

```powershell
$secrets = az appconfig kv list -n myAppConfiguration --key "secrets.*" --resolve-keyvault --query "[*].{name:key, value:value}" | ConvertFrom-Json

foreach ($secret in $secrets) {
  $keyvalues += $secret.name + "=" + $secret.value + ","
}

if ($keyvalues){
  $keyvalues = $keyvalues.TrimEnd(',')
  helm upgrade --install --set $keyvalues "example" ./mychart 
}
else{
  helm upgrade --install "example" ./mychart 
}

```

Убедитесь, что настройки и секреты были успешно заданы, открыв [панель мониторинга Kubernetes](../aks/kubernetes-dashboard.md). Вы увидите, что значения **color** и **message** из службы "Конфигурация приложений" успешно перенесены в переменные среды контейнера.

![Краткое руководство. Запуск приложения, размещенного локально](./media/kubernetes-dashboard-env-variables.png)

Один из секретов с именем **password**, который хранит в службе "Конфигурация приложений" ссылку на Key Vault, также был добавлен в секреты Kubernetes. 

![Снимок экрана, на котором выделено поле для ввода пароля в разделе "Данные".](./media/kubernetes-dashboard-secrets.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как экспортировать данные из службы "Конфигурация приложений Azure", чтобы использовать их в Helm при развертывании Kubernetes. Чтобы узнать больше об использовании службы "Конфигурация приложений", перейдите к примерам скриптов Azure CLI.

> [!div class="nextstepaction"]
> [Azure CLI](/cli/azure/appconfig)
