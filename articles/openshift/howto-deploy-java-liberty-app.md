---
title: Развертывание приложения Java с помощью Open Liberty/WebSphere Liberty в кластере Azure Red Hat OpenShift 4
description: Разверните приложение Java с помощью Open Liberty/WebSphere Liberty в кластере Azure Red Hat OpenShift 4.
author: jiangma
ms.author: jiangma
ms.service: azure-redhat-openshift
ms.topic: conceptual
ms.date: 10/30/2020
keywords: Java, жакартаи, Java, Микропрофиль, Open-Liberty, WebSphere-Liberty, АТО, openshift, Red Hat
ms.openlocfilehash: 08fd3ab112498a983b438d5ba1f1f100816cbf5d
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102213000"
---
# <a name="deploy-a-java-application-with-open-libertywebsphere-liberty-on-an-azure-red-hat-openshift-4-cluster"></a>Развертывание приложения Java с помощью Open Liberty/WebSphere Liberty в кластере Azure Red Hat OpenShift 4

В этом руководстве показано, как запустить приложение Java, Java EE, [Джакарта ee](https://jakarta.ee/)или [Микропрофиль](https://microprofile.io/) в открытой среде выполнения Liberty/WebSphere Liberty, а затем развернуть контейнерное приложение в кластере Azure Red Hat OpenShift (АТО) 4 с помощью оператора Open Liberty. В этой статье описывается подготовка приложения Liberty, создание образа DOCKER приложения и запуск контейнерного приложения в кластере АТО 4.  Дополнительные сведения об Open Liberty см. [на странице Открытие проекта Liberty](https://openliberty.io/). Дополнительные сведения о IBM WebSphere Liberty см. на [странице продукта WebSphere Liberty](https://www.ibm.com/cloud/websphere-liberty).

[!INCLUDE [aro-support](includes/aro-support.md)]

## <a name="prerequisites"></a>Предварительные требования

Чтобы успешно пройти это руководство, выполните следующие предварительные требования.

> [!NOTE]
> Для создания и запуска кластера OpenShift в Azure Red Hat OpenShift требуется не менее 40 ядер. Стандартная квота ресурсов Azure для новой подписки Azure не соответствует этому требованию. Чтобы запросить увеличение ограничений ресурсов, см. статью об [ увеличении ограничений для различных серий виртуальных машин](../azure-portal/supportability/per-vm-quota-requests.md). Обратите внимание, что бесплатная пробная подписка не подходит для увеличения квоты, [обновления до подписки с оплатой по мере](../cost-management-billing/manage/upgrade-azure-subscription.md) использования перед запросом на увеличение квоты.

1. Подготовка локального компьютера с установленной операционной системой, похожей на UNIX (например, Ubuntu, macOS).
1. Установите реализацию Java SE (например, [Адоптопенждк OpenJDK 8 LTS/OpenJ9](https://adoptopenjdk.net/?variant=openjdk8&jvmVariant=openj9)).
1. Установите [Maven](https://maven.apache.org/download.cgi) 3.5.0 или более поздней версии.
1. Установите [DOCKER](https://docs.docker.com/get-docker/) для вашей ОС.
1. Установите [Azure CLI](/cli/azure/install-azure-cli) 2.0.75 или более поздней версии.
1. Проверьте и установите флажок, [`envsubst`](https://command-not-found.com/envsubst) если он не был предварительно установлен в вашей операционной системе.
1. Клонировать код для этого примера в локальную систему. Пример находится на [GitHub](https://github.com/Azure-Samples/open-liberty-on-aro).
1. Следуйте инструкциям в разделе [Создание кластера Azure Red Hat OpenShift 4](./tutorial-create-cluster.md).

   Хотя шаг «получение секрета по запросу Red Hat» помечен как необязательный, **он необходим для работы с этой статьей**.  Секрет опрашивающего запроса позволяет кластеру Azure Red Hat OpenShift найти оператор Open Liberty.

   Если вы планируете запускать приложения, интенсивно использующие память, в кластере, укажите соответствующий размер виртуальной машины для рабочих узлов с помощью `--worker-vm-size` параметра. Например, `Standard_E4s_v3` — это минимальный размер виртуальной машины для установки оператора Elasticsearch в кластере. Дополнительные сведения см. в разделе:

   * [Azure CLI для создания кластера](/cli/azure/aro#az-aro-create)
   * [Поддерживаемые размеры виртуальных машин для оптимизированной для памяти](./support-policies-v4.md#memory-optimized)
   * [Необходимые условия для установки оператора Elasticsearch](https://docs.openshift.com/container-platform/4.3/logging/cluster-logging-deploying.html#cluster-logging-deploy-eo-cli_cluster-logging-deploying)

1. Подключитесь к кластеру, выполнив действия, описанные в подразделе [Подключение к кластеру Azure Red Hat OpenShift 4](./tutorial-connect-cluster.md).
   * Обязательно выполните действия, описанные в разделе "Установка OpenShift CLI", так как мы будем использовать эту `oc` команду Далее в этой статье.
   * Запишите URL-адрес консоли кластера, который выглядит как `https://console-openshift-console.apps.<random>.<region>.aroapp.io/` .
   * Запишите `kubeadmin` учетные данные.

1. Убедитесь, что вы можете войти в OpenShift CLI с маркером для пользователя `kubeadmin` .

### <a name="enable-the-built-in-container-registry-for-openshift"></a>Включение встроенного реестра контейнеров для OpenShift

В этом руководстве описывается создание образа DOCKER, который должен быть помещен в реестр контейнеров, доступный для OpenShift. Самый простой вариант — использовать встроенный реестр, предоставляемый OpenShift. Чтобы включить встроенный реестр контейнеров, выполните действия, описанные в статье [Настройка встроенного реестра контейнеров для Azure Red Hat OpenShift 4](built-in-container-registry.md). В этой статье используются три элемента из этих шагов.

* Имя и пароль пользователя Azure AD для входа в веб-консоль OpenShift.
* Выходные данные `oc whoami` после выполнения действий для входа в OPENSHIFT CLI.  Это значение называется **AAD-User** для обсуждения.
* URL-адрес реестра контейнеров.

Запишите эти элементы вниз по завершении действий, чтобы включить встроенный реестр контейнеров.

### <a name="create-an-openshift-namespace-for-the-java-app"></a>Создание пространства имен OpenShift для приложения Java

1. Войдите в веб-консоль OpenShift из браузера, используя `kubeadmin` учетные данные.
2. Перейдите к разделу **Администрирование**  >  **пространства имен**  >  **создать пространство имен**.
3. Заполните поля `open-liberty-demo` **имя** и выберите **создать**, как показано далее.

   ![создать пространство имен](./media/howto-deploy-java-liberty-app/create-namespace.png)

### <a name="create-an-administrator-for-the-demo-project"></a>Создание администратора для демонстрационного проекта

Помимо управления образами, **AAD-User** также предоставляет административные разрешения для управления ресурсами в демонстрационном проекте кластера АТО 4.  Войдите в OpenShift CLI и предоставьте **AAD-User** необходимые привилегии, выполнив следующие действия.

1. Войдите в веб-консоль OpenShift из браузера, используя `kubeadmin` учетные данные.
1. В правой верхней части веб-консоли разверните контекстное меню пользователя, выполнившего вход, и выберите **команду Копировать имя входа**.
1. При необходимости войдите в новое окно вкладки с тем же пользователем.
1. Выберите **Показать маркер**.
1. Скопируйте указанное ниже значение **Login с этим маркером** в буфер обмена и запустите его в оболочке, как показано ниже.
1. Выполните следующие команды, чтобы предоставить `admin` роли **AAD-User** в пространстве имен `open-liberty-demo` .

   ```bash
   # Switch to project "open-liberty-demo"
   oc project open-liberty-demo
   Now using project "open-liberty-demo" on server "https://api.x8xl3f4y.eastus.aroapp.io:6443".
   # Note: replace "<aad-user>" with the one noted by executing the steps in
   # Configure built-in container registry for Azure Red Hat OpenShift 4
   oc adm policy add-role-to-user admin <aad-user>
   clusterrole.rbac.authorization.k8s.io/admin added: "kaaIjx75vFWovvKF7c02M0ya5qzwcSJ074RZBfXUc34"
   ```

### <a name="install-the-open-liberty-openshift-operator"></a>Установка оператора Open Liberty OpenShift

После создания кластера и подключения к нему установите оператор Open Liberty.  Главная начальная страница для оператора Open Liberty — на [GitHub](https://github.com/OpenLiberty/open-liberty-operator).

1. Войдите в веб-консоль OpenShift из браузера, используя `kubeadmin` учетные данные.
2. Перейдите к **операторам**  >  **операторхуб** и найдите **оператор Open Liberty**.
3. В результатах поиска выберите **Открыть оператор Liberty** .
4. Выберите пункт **Установить**.
5. В меню **создать подписку на оператор** просмотрите **все пространства имен в кластере (по умолчанию)** для **режима установки**, **бета-версия** **канала обновления** и **Автоматическое** **утверждение для стратегии утверждения**:

   ![создать подписку оператора для открытого оператора Liberty](./media/howto-deploy-java-liberty-app/install-operator.png)
6. Выберите **подписываться** и подождите одну минуту, пока не отобразится оператор Open Liberty.
7. Обратите внимание на оператор Open Liberty с состоянием "успех".  Если это не так, определите и устраните проблему, прежде чем продолжить.
   :::image type="content" source="media/howto-deploy-java-liberty-app/open-liberty-operator-installed.png" alt-text="Установлены операторы, отображающие Open Liberty.":::

## <a name="prepare-the-liberty-application"></a>Подготовка приложения Liberty

Мы будем использовать приложение Java EE 8 в качестве примера в этом разделе. Open Liberty — это сервер, совместимый с [Java EE 8](https://javaee.github.io/javaee-spec/javadocs/) , который позволяет легко запускать приложение.  Open Liberty также является [Джакарта, совместимым с полным профилем ee](https://jakarta.ee/specifications/platform/8/apidocs/).

### <a name="run-the-application-on-open-liberty"></a>Запуск приложения при открытии Liberty

Чтобы запустить приложение в Open Liberty, необходимо создать файл конфигурации Open Liberty Server, чтобы [подключаемый модуль Liberty Maven](https://github.com/OpenLiberty/ci.maven#liberty-maven-plugin) мог упаковать приложение для развертывания. Подключаемый модуль Liberty Maven не требуется для развертывания приложения в OpenShift.  Однако мы будем использовать его в этом примере в режиме разработчика Open Liberty (dev).  Режим разработчика позволяет легко запускать приложение локально. На локальном компьютере выполните следующие действия.

1. Копировать `2-simple/src/main/liberty/config/server.xml` в `1-start/src/main/liberty/config` , перезаписывая существующий файл нулевой длины. Это `server.xml` настраивает Открытый сервер Liberty с компонентами Java EE.
1. Копировать `2-simple/pom.xml` в `1-start/pom.xml` .  Этот шаг добавляет `liberty-maven-plugin` к POM.
1. Измените каталог на `1-start` локальный клон.
1. Запустите `mvn clean package` в консоли, чтобы создать WAR-пакет `javaee-cafe.war` в каталоге `./target` .
1. Запустите `mvn liberty:dev` , чтобы запустить Open Liberty в режиме разработки.
1. Дождитесь запуска сервера. Выходные данные консоли должны заканчиваться следующим сообщением:

   ```Text
   [INFO] CWWKM2015I: Match number: 1 is [6/10/20 10:26:09:517 CST] 00000022 com.ibm.ws.kernel.feature.internal.FeatureManager            A CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 6.447 seconds..
   [INFO] Press the Enter key to run tests on demand. To stop the server and quit dev mode, use Ctrl-C or type 'q' and press the Enter key.
   [INFO] Source compilation was successful.
   ```

1. Откройте `http://localhost:9080/` в браузере, чтобы перейти на домашнюю страницу приложения. Приложение будет выглядеть примерно так, как показано на следующем рисунке:

   ![Веб-интерфейс Java кафе](./media/howto-deploy-java-liberty-app/javaee-cafe-web-ui.png)
1. Нажмите **Control-C** , чтобы прерывать работу приложения и открыть Liberty Server.

Каталог `2-simple` локального клона показывает проект Maven с уже примененными ранее изменениями.

## <a name="prepare-the-application-image"></a>Подготовка образа приложения

Чтобы развернуть и запустить приложение Liberty в кластере АТО 4, контейнеризовать приложение как образ DOCKER с помощью [открытых Liberty контейнеров](https://github.com/OpenLiberty/ci.docker) или [образов контейнеров WebSphere Liberty](https://github.com/WASdev/ci.docker).

### <a name="build-application-image"></a>Сборка образа приложения

Выполните следующие действия, чтобы создать образ приложения.

1. Измените каталог на `2-simple` локальный клон.
2. Выполните команду `mvn clean package` , чтобы упаковать приложение.
3. Выполните одну из следующих команд, чтобы создать образ приложения.
   * Сборка с открытым базовым образом Liberty:

     ```bash
     # Build and tag application image. This will cause Docker to pull the necessary Open Liberty base images.
     docker build -t javaee-cafe-simple:1.0.0 --pull .
     ```

   * Сборка с помощью базового образа WebSphere Liberty:

     ```bash
     # Build and tag application image. This will cause Docker to pull the necessary WebSphere Liberty base images.
     docker build -t javaee-cafe-simple:1.0.0 --pull --file=Dockerfile-wlp .
     ```

### <a name="run-the-application-locally-with-docker"></a>Запуск приложения локально с помощью DOCKER

Перед развертыванием контейнерного приложения в удаленном кластере запустите с помощью локального DOCKER, чтобы проверить, работает ли он:

1. Запустите `docker run -it --rm -p 9080:9080 javaee-cafe-simple:1.0.0` в консоли.
2. Дождитесь запуска сервера Liberty и разверните приложение для успешного развертывания.
3. Откройте `http://localhost:9080/` в браузере, чтобы перейти на домашнюю страницу приложения.
4. Нажмите **Control-C** , чтобы прерывать работу приложения и сервера Liberty.

### <a name="push-the-image-to-the-container-image-registry"></a>Отправка образа в реестр образов контейнеров

Если вы удовлетворены состоянием приложения, отправьте его во встроенный реестр образов контейнера, следуя приведенным ниже инструкциям.

#### <a name="log-in-to-the-openshift-cli-as-the-azure-ad-user"></a>Войдите в OpenShift CLI в качестве пользователя Azure AD.

1. Войдите в веб-консоль OpenShift из браузера, используя учетные данные пользователя Azure AD.

   1. Используйте InPrivate, режиме инкогнито или другую эквивалентную функцию окна браузера для входа в консоль.
   1. Выбор **OpenID Connect**

   > [!NOTE]
   > Запишите имя пользователя и пароль, которые вы используете для входа. Это имя пользователя и пароль будут работать как администратор для других действий в этой и других статьях.
1. Выполните вход с помощью интерфейса командной строки OpenShift, выполнив следующие действия.  Для обсуждения этот процесс называется `oc login` .
   1. В правой верхней части веб-консоли разверните контекстное меню пользователя, выполнившего вход, и выберите **команду Копировать имя входа**.
   1. При необходимости войдите в новое окно вкладки с тем же пользователем.
   1. Выберите **Показать маркер**.
   1. Скопируйте указанное ниже значение **Login с этим маркером** в буфер обмена и запустите его в оболочке, как показано ниже.

       ```bash
       oc login --token=XOdASlzeT7BHT0JZW6Fd4dl5EwHpeBlN27TAdWHseob --server=https://api.aqlm62xm.rnfghf.aroapp.io:6443
       Logged into "https://api.aqlm62xm.rnfghf.aroapp.io:6443" as "kube:admin" using the token provided.

       You have access to 57 projects, the list has been suppressed. You can list all projects with 'oc projects'

       Using project "default".
       ```

#### <a name="push-the-container-image-to-the-container-registry-for-openshift"></a>Отправка образа контейнера в реестр контейнеров для OpenShift

Выполните следующие команды, чтобы отправить образ в реестр контейнеров для OpenShift.

```bash
# Note: replace "<Container_Registry_URL>" with the fully qualified name of the registry
Container_Registry_URL=<Container_Registry_URL>

# Create a new tag with registry info that refers to source image
docker tag javaee-cafe-simple:1.0.0 ${Container_Registry_URL}/open-liberty-demo/javaee-cafe-simple:1.0.0

# Sign in to the built-in container image registry
docker login -u $(oc whoami) -p $(oc whoami -t) ${Container_Registry_URL}
```

Успешный вывод будет выглядеть следующим образом.

```bash
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
```

Отправьте образ во встроенный реестр образов контейнера с помощью следующей команды.

```bash

docker push ${Container_Registry_URL}/open-liberty-demo/javaee-cafe-simple:1.0.0
```

## <a name="deploy-application-on-the-aro-4-cluster"></a>Развертывание приложения в кластере АТО 4

Теперь вы можете развернуть пример приложения Liberty в кластере Azure Red Hat OpenShift 4, созданном ранее при работе с предварительными требованиями.

### <a name="deploy-the-application-from-the-web-console"></a>Развертывание приложения из веб-консоли

Поскольку для управления приложениями Liberty используется оператор Open Liberty, необходимо создать экземпляр его *пользовательского определения ресурса* типа "опенлибертяппликатион". Затем оператор берет на себя все аспекты управления ресурсами OpenShift, необходимыми для развертывания.

1. Войдите в веб-консоль OpenShift из браузера, используя учетные данные пользователя Azure AD.
1. Разверните узел **Главная** и выберите **проекты**  >  **Open-Liberty-Demo**.
1. Перейдите к **операторам**,  >  **установленным** операторами.
1. В середине страницы выберите **Open Liberty operator (открыть оператор**).
1. В середине страницы выберите **Open Liberty Application (открыть приложение приложения**).  Навигация по элементам в пользовательском интерфейсе отражает реальную иерархию вложений используемых технологий.
   <!-- Diagram source https://github.com/Azure-Samples/open-liberty-on-aro/blob/master/diagrams/aro-java-containment.vsdx -->
   ![Включение АТО Java](./media/howto-deploy-java-liberty-app/aro-java-containment.png)
1. Выберите **создать опенлибертяппликатион**
1. Замените созданный YAML своим, который находится в папке `<path-to-repo>/2-simple/openlibertyapplication.yaml` .
1. Нажмите кнопку **создания**. Вы вернетесь к списку Опенлибертяппликатионс.
1. Выберите **Java-кафе-Simple**.
1. В середине страницы выберите **ресурсы**.
1. В таблице выберите ссылку для **Java-кафе-Simple** с **видом** **маршрута**.
1. На открывшейся странице выберите ссылку в **расположении** ниже.

Вы увидите домашнюю страницу приложения, открытую в браузере.

### <a name="delete-the-application-from-the-web-console"></a>Удаление приложения из веб-консоли

Завершив работу с приложением, выполните следующие действия, чтобы удалить приложение из окна Shift.

1. В области навигации слева разверните элемент **Операторы**.
1. Выберите **установленные операторы**.
1. Выберите **Open Liberty operator (открыть оператор**).
1. В середине страницы выберите **Open Liberty Application (открыть приложение**).
1. Щелкните вертикальное многоточие (три вертикальные точки) и выберите **удалить приложение опенлиберти**.

### <a name="deploy-the-application-from-cli"></a>Развертывание приложения из интерфейса командной строки

Вместо использования графического интерфейса веб-консоли можно развернуть приложение из интерфейса командной строки. Если вы еще не сделали этого, скачайте и установите `oc` программу командной строки, следуя документации по Red Hat [Начало работы с CLI](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html).

1. Войдите в веб-консоль OpenShift из браузера, используя учетные данные пользователя Azure AD.
2. Войдите в OpenShift CLI с маркером для пользователя Azure AD.
3. Измените каталог на `2-simple` локальный клон и выполните следующие команды, чтобы развернуть приложение Liberty в кластере АТО 4.  Выходные данные команды также отображаются как встроенные.

   ```bash
   # Switch to namespace "open-liberty-demo" where resources of demo app will belong to
   oc project open-liberty-demo

   Now using (or already on) project "open-liberty-demo" on server "https://api.aqlm62xm.rnfghf.aroapp.io:6443".

   # Create OpenLibertyApplication "javaee-cafe-simple"
   oc create -f openlibertyapplication.yaml

   openlibertyapplication.openliberty.io/javaee-cafe-simple created

   # Check if OpenLibertyApplication instance is created
   oc get openlibertyapplication javaee-cafe-simple

   NAME                 IMAGE                      EXPOSED   RECONCILED   AGE
   javaee-cafe-simple   javaee-cafe-simple:1.0.0   true      True         36s

   # Check if deployment created by Operator is ready
   oc get deployment javaee-cafe-simple

   NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
   javaee-cafe-simple   1/1     1            0           102s
   ```

4. `1/1`Перед продолжением убедитесь, что в столбце находится элемент `READY` .  Если нет, Исследуйте и устраните проблему, прежде чем продолжить.
5. Найдите узел маршрута к приложению с помощью `oc get route` команды, как показано здесь.

   ```bash
   # Get host of the route
   HOST=$(oc get route javaee-cafe-simple --template='{{ .spec.host }}')
   echo "Route Host: $HOST"

   Route Host: javaee-cafe-simple-open-liberty-demo.apps.aqlm62xm.rnfghf.aroapp.io
   ```

   Когда приложение Liberty заработает, откройте выходные данные **узла Route** в браузере, чтобы перейти на домашнюю страницу приложения.

### <a name="delete-the-application-from-cli"></a>Удаление приложения из интерфейса командной строки

Удалите приложение из интерфейса командной строки, выполнив следующую команду.

```bash
oc delete -f openlibertyapplication.yaml
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Удалите кластер АТО, выполнив действия, описанные в [руководстве по удалению кластера Azure Red Hat OpenShift 4.](./tutorial-delete-cluster.md)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководство вы узнали, как выполнять следующие задачи:
> [!div class="checklist"]
>
> * Подготовка приложения Liberty
> * Создание образа приложения
> * Запуск контейнерного приложения в кластере АТО 4 с помощью графического интерфейса пользователя и интерфейса командной строки

Дополнительные сведения можно узнать из ссылок, используемых в этом руководство:

* [Открыть Liberty](https://openliberty.io/)
* [Azure Red Hat OpenShift](https://azure.microsoft.com/services/openshift/)
* [Open Liberty, оператор](https://github.com/OpenLiberty/open-liberty-operator)
* [Открыть конфигурацию сервера Liberty](https://openliberty.io/docs/ref/config/)
* [Подключаемый модуль Liberty Maven](https://github.com/OpenLiberty/ci.maven#liberty-maven-plugin)
* [Открытие образов контейнеров Liberty](https://github.com/OpenLiberty/ci.docker)
* [Образы контейнеров WebSphere Liberty](https://github.com/WASdev/ci.docker)