---
title: Настройка Jenkins для приложения Java в Service Fabric в Azure
description: Из этого руководства вы узнаете, как настроить непрерывную интеграцию с помощью Jenkins для развертывания приложения Java Service Fabric.
ms.topic: tutorial
ms.date: 08/27/2018
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 4e29104f0586a8e18afb7a640903c57ce988b692
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91531978"
---
# <a name="tutorial-configure-a-jenkins-environment-to-enable-cicd-for-a-java-application-on-service-fabric"></a>Руководство по Настройка среды Jenkins для включения непрерывной интеграции и непрерывного развертывания для приложения Java в Service Fabric

Это руководство представляет собой пятую часть цикла. В нем показано, как развернуть обновления в приложении с помощью Jenkins. В этом руководстве приложение Voting развертывается в кластере с помощью подключаемого модуля Jenkins для Service Fabric в сочетании с репозиторием GitHub, в котором размещено это приложение.

В пятой части цикла вы узнаете, как выполнять такие задачи:
> [!div class="checklist"]
> * Развертывание контейнера Jenkins для Service Fabric на компьютере.
> * Настройка среды Jenkins для развертывания в Service Fabric.
> * Обновление приложения.

Из этого цикла руководств вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * [Развертывание приложения Java служб Service Fabric Reliable Services в Azure.](service-fabric-tutorial-create-java-app.md)
> * [Развертывание и отладка приложения в локальном кластере.](service-fabric-tutorial-debug-log-local-cluster.md)
> * [Развертывание приложения в кластере Azure.](service-fabric-tutorial-java-deploy-azure.md)
> * [Настройка мониторинга и диагностики приложения](service-fabric-tutorial-java-elk.md)
> * Настройка непрерывной интеграции и непрерывного развертывания.

## <a name="prerequisites"></a>Предварительные требования

* Необходимо установить Git на локальный компьютер, используя [страницу загрузки Git](https://git-scm.com/downloads). Дополнительные сведения о Git см. в [документации по Git](https://git-scm.com/docs).
* Необходимо иметь опыт работы с [Jenkins](https://jenkins.io/).
* Необходимо создать учетную запись [GitHub](https://github.com/) и уметь использовать GitHub.
* Необходимо установить [Docker](https://www.docker.com/community-edition) на компьютер.

## <a name="pull-and-deploy-service-fabric-jenkins-container-image"></a>Получение и развертывание образа контейнера Jenkins для Service Fabric

Jenkins можно настроить внутри или за пределами кластера Service Fabric. Ниже показано, как с помощью предоставленного образа Docker настроить его за пределами кластера. Тем не менее также можно использовать предварительно настроенную среду сборки Jenkins. Текущий образ контейнера поставляется вместе с установленным подключаемым модулем Service Fabric, готовым к работе с Service Fabric.

1. Получите образ контейнера Jenkins для Service Fabric: ``docker pull rapatchi/jenkins:v10``. Этот образ предварительно установлен в подключаемом модуле Jenkins Service Fabric.

1. Запустите образ контейнера из расположения, в котором хранятся сертификаты Azure, на подключенном локальном компьютере.

    ```bash
    docker run -itd -p 8080:8080 -v /service-fabric-java-quickstart/AzureCluster rapatchi/jenkins:v10
    ```

1. Получите идентификатор экземпляра образа контейнера. Вы можете вывести список всех контейнеров Docker с помощью команды ``docker ps –a``.

1. Получите пароль своего экземпляра Jenkins, выполнив следующую команду:

    ```sh
    docker exec [first-four-digits-of-container-ID] cat /var/jenkins_home/secrets/initialAdminPassword
    ```

    Если идентификатор контейнера имеет значение 2d24a73b5964, используйте 2d24.
    * Этот пароль необходим для входа на панель мониторинга Jenkins с портала ``http://<HOST-IP>:8080``.
    * После первого входа можно создать учетную запись пользователя или пользоваться учетной записью администратора.

1. Настройте GitHub для работы с Jenkins, выполнив шаги, описанные в статье [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) (Создание ключа SSH и его добавление в агент SSH). Так как команды запускаются из контейнера Docker, необходимо следовать инструкциям для среды Linux.
   * Для создания SSH-ключа используйте инструкции, предоставленные GitHub. Затем добавьте SSH-ключ в учетную запись GitHub, в которой находится репозиторий.
   * В оболочке Jenkins или Docker (а не на узле) выполните команды, перечисленные в приведенной выше статье.
   * Чтобы войти в оболочку Jenkins с вашего узла, выполните следующие команды:

     ```sh
     docker exec -t -i [first-four-digits-of-container-ID] /bin/bash
     ```

     Убедитесь, что кластер или компьютер, где размещен образ контейнера Jenkins, имеет общедоступный IP-адрес, Наличие общедоступного IP-адреса позволяет экземпляру Jenkins получать уведомления от GitHub.

## <a name="create-and-configure-a-jenkins-job"></a>Создание и настройка задания Jenkins

1. Сначала необходимо создать репозиторий, который можно использовать для размещения проекта Voting на GitHub (если у вас его еще нет). До завершения работы с руководством репозиторий будет называться **dev_test**.

1. Создайте **элемент** на панели мониторинга Jenkins по адресу ``http://<HOST-IP>:8080``.

1. Введите имя элемента, например **MyJob**. Выберите **проект в свободной форме** и нажмите кнопку **ОК**.

1. Перейдите на страницу задания и щелкните **Configure** (Настройка).

   а. В разделе общих сведений установите флажок напротив **проекта GitHub** и укажите URL-адрес этого проекта. Это URL-адрес, где размещено приложение Service Fabric на Java, в которое необходимо интегрировать процесс непрерывной интеграции и доставки Jenkins (например, ``https://github.com/testaccount/dev_test``).

   b. В разделе **Source Code Management** (Управление исходным кодом) выберите **Git**. Укажите URL-адрес репозитория, где размещено приложение Service Fabric на Java, в которое необходимо интегрировать поток средств непрерывной интеграции и непрерывного развертывания Jenkins (например, *`https://github.com/testaccount/dev_test.git`* ). Также здесь можно указать, какую ветвь необходимо создать (например, **/master**).

1. Настройте свой *Github* (где размещен репозиторий), чтобы он мог обмениваться данными с Jenkins. Выполните указанные ниже действия.

   а. Перейдите на страницу репозитория GitHub. Последовательно выберите пункты **Settings** (Параметры) > **Integrations and Services** (Интеграции и службы).

   b. Выберите **Add Service** (Добавить службу), введите **Jenkins** и выберите **Jenkins-Github plugin** (Модуль Jenkins-Github).

   c. Введите URL-адрес webhook для Jenkins (по умолчанию это должен быть ``http://<PublicIPorFQDN>:8081/github-webhook/``). **Добавьте или обновите службу**.

   d. В ваш экземпляр Jenkins будет отправлено событие проверки. Рядом с веб-перехватчиком в GitHub должен появиться зеленый флажок. Это означает, что проект создается.

   ![Настройка Jenkins для Service Fabric](./media/service-fabric-tutorial-java-jenkins/jenkinsconfiguration.png)

1. В разделе **Build Triggers** (Создание триггеров) выберите требуемый параметр сборки. В этом примере мы планируем инициировать создание сборки каждый раз, когда будет происходить отправка в репозиторий, поэтому выберите параметр **GitHub hook trigger for GITScm polling** (Обработчик триггера Github для опроса GITScm).

1. В раскрывающемся списке **Add build step** (Добавить шаг сборки) раздела **Build** (Создание) выберите параметр **Invoke Gradle Script** (Вызов скрипта Gradle). В появившемся мини-приложении откройте расширенное меню и укажите путь к **корневому скрипту сборки** вашего приложения. Этот скрипт выбирает build.gradle по указанному пути и действует соответствующим образом.

    ![Действие при сборке Jenkins для Service Fabric](./media/service-fabric-tutorial-java-jenkins/jenkinsbuildscreenshot.png)

1. В раскрывающемся списке **Post-Build Actions** (Действия после сборки) выберите **Deploy Service Fabric Project** (Развернуть проект Service Fabric). Здесь необходимо предоставить сведения о кластере, где будет развернуто приложение Service Fabric, скомпилированное с помощью Jenkins, Путь к сертификату ведет в расположение, в котором был подключен том (/tmp/myCerts).

    Кроме того, можно предоставить дополнительные сведения, необходимые для развертывания приложения. Ниже приведен снимок экрана с примером дополнительных сведений о приложении.

    ![Диалоговое окно Post-Build Actions (Действия после сборки) с дополнительными параметрами Deploy Service Fabric Project (Развернуть проект Service Fabric)](./media/service-fabric-tutorial-java-jenkins/sfjenkins.png)

    > [!NOTE]
    > Если образ контейнера Jenkins развертывается с помощью Service Fabric, можно использовать тот же кластер, на котором размещается приложение-контейнер Jenkins.
    >

1. Выберите команду **Сохранить**.

## <a name="update-your-existing-application"></a>Обновление имеющегося приложения

1. Обновите заголовок HTML в файле *VotingApplication/VotingWebPkg/Code/wwwroot/index.html*, использовав **Service Fabric Voting Sample V2**.

    ```html
    <div ng-app="VotingApp" ng-controller="VotingAppController" ng-init="refresh()">
        <div class="container-fluid">
            <div class="row">
                <div class="col-xs-8 col-xs-offset-2 text-center">
                    <h2>Service Fabric Voting Sample V2</h2>
                </div>
            </div>
        </div>
    </div>
    ```

1. Обновите версию **ApplicationTypeVersion** и **ServiceManifestVersion** до версии **2.0.0** в файле *Voting/VotingApplication/ApplicationManifest.xml*.

    ```xml
    <?xml version="1.0" encoding="utf-8" standalone="no"?>
    <ApplicationManifest xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingApplicationType" ApplicationTypeVersion="2.0.0">
      <Description>Voting Application</Description>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="2.0.0"/>
      </ServiceManifestImport>
      <ServiceManifestImport>
            <ServiceManifestRef ServiceManifestName="VotingDataServicePkg" ServiceManifestVersion="1.0.0"/>
        </ServiceManifestImport>
        <DefaultServices>
          <Service Name="VotingWeb">
             <StatelessService InstanceCount="1" ServiceTypeName="VotingWebType">
                <SingletonPartition/>
             </StatelessService>
          </Service>
       <Service Name="VotingDataService">
                <StatefulService MinReplicaSetSize="3" ServiceTypeName="VotingDataServiceType" TargetReplicaSetSize="3">
                    <UniformInt64Partition HighKey="9223372036854775807" LowKey="-9223372036854775808" PartitionCount="1"/>
                </StatefulService>
            </Service>
        </DefaultServices>
    </ApplicationManifest>
    ```

1. Обновите поле **Версия** в **ServiceManifest** и поле **Версия** в теге **CodePackage** в файле *Voting/VotingApplication/VotingWebPkg/ServiceManifest.xml* до версии **2.0.0**.

    ```xml
    <CodePackage Name="Code" Version="2.0.0">
    <EntryPoint>
        <ExeHost>
        <Program>entryPoint.sh</Program>
        </ExeHost>
    </EntryPoint>
    </CodePackage>
    ```

1. Чтобы инициализировать задание Jenkins, выполняющее обновление приложения, отправьте новые изменения в репозиторий GitHub.

1. В Service Fabric Explorer нажмите раскрывающийся список **Приложения**. Чтобы увидеть состояние обновления, нажмите вкладку **Upgrades in Progress** (Выполняемые обновления).

    ![Выполняемые обновления](./media/service-fabric-tutorial-create-java-app/upgradejava.png)

1. Теперь при доступе к **http://\<Host-IP>:8080** приложение Voting со всеми функциями запущено и работает.

    ![Локальное приложение для голосования](./media/service-fabric-tutorial-java-jenkins/votingv2.png)

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали, как выполнять следующие задачи:

> [!div class="checklist"]
> * Развертывание контейнера Jenkins для Service Fabric на компьютере.
> * Настройка среды Jenkins для развертывания в Service Fabric.
> * Обновление приложения.

* Просмотрите другие [примеры Java](https://github.com/Azure-Samples/service-fabric-java-getting-started).
