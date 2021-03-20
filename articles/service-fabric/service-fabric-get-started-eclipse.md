---
title: Подключаемый модуль Service Fabric Azure для Eclipse
description: Узнайте, как приступить к работе с Azure Service Fabric в Java с помощью Eclipse и Service Fabric предоставленного подключаемого модуля.
author: rapatchi
ms.topic: conceptual
ms.date: 04/06/2018
ms.author: rapatchi
ms.custom: devx-track-java
ms.openlocfilehash: b85206f9b1e92607bba7b6f141b700922c129b65
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97656926"
---
# <a name="service-fabric-plug-in-for-eclipse-java-application-development"></a>Подключаемый модуль Service Fabric для разработки приложений Eclipse на Java
Eclipse является одной из наиболее часто используемых интегрированных сред разработки (IDE) для разработчиков Java. В этой статье описывается, как настроить среду разработки Eclipse для работы с Azure Service Fabric. Вы узнаете, как установить подключаемый модуль Service Fabric, создать приложение Service Fabric и развернуть его в локальном или удаленном кластере Service Fabric в Eclipse. 

> [!NOTE]
> Подключаемый модуль Eclipse сейчас не поддерживается в Windows. 

> [!IMPORTANT]
> Убедитесь, что JDK 8 установлен в системе и выбран в Eclipse.

## <a name="install-or-update-the-service-fabric-plug-in-in-eclipse"></a>Установка и обновление подключаемого модуля Service Fabric в Eclipse
Вы можете установить подключаемый модуль Service Fabric в Eclipse. Он позволяет упростить процесс создания и развертывания служб Java.

> [!IMPORTANT]
> Для подключаемого модуля Service Fabric требуется Eclipse Neon или более поздняя версия. Чтобы проверить версию Eclipse, выполните инструкции под этим примечанием. Если у вас установлена более ранняя версия Eclipse, скачайте более новые версии с [сайта Eclipse](https://www.eclipse.org). Не рекомендуется устанавливать Eclipse поверх существующей установки (перезаписывать). Вы можете удалить ее перед запуском установщика или установить новую версию в другом каталоге. 
> 
> В Ubuntu мы рекомендуем установить Eclipse непосредственно с сайта, а не с помощью установщика пакета (`apt` или `apt-get`). Так вы получите последнюю версию Eclipse. 

Установите Eclipse Neon или более поздней версии с [сайта Eclipse](https://www.eclipse.org).  Также установите Buildship 2.2.1 или более поздней версии (подключаемый модуль Service Fabric не совместим с более ранними версиями Buildship):
-   Чтобы проверить версии установленных компонентов, в Eclipse выберите **Справка**  >    >  **сведения об установке** Eclipse.
-   Сведения об обновлении Buildship см. на странице [Eclipse Buildship: Eclipse Plug-ins for Gradle][buildship-update] (Eclipse Buildship. Подключаемые модули Eclipse для Gradle).
-   Чтобы проверить и установить обновления для Eclipse, перейдите к разделу **Справка**  >  **Проверка обновлений**.

Установка подключаемого модуля Service Fabric в Eclipse перейдите к разделу **Справка**  >  **Установка нового программного обеспечения**.
1. В поле **работать с** введите https: \/ /DL.Microsoft.com/Eclipse.
2. Нажмите кнопку **Добавить**.

   ![Подключаемый модуль Service Fabric для Eclipse][sf-eclipse-plugin-install]
3. Выберите подключаемый модуль Service Fabric и нажмите кнопку **Next** (Далее).
4. Выполните установку и примите условия лицензионного соглашения на использование программного обеспечения корпорации Майкрософт.

Если подключаемый модуль Service Fabric уже установлен, необходимо использовать последнюю версию. 
1. Чтобы проверить наличие доступных обновлений, перейдите в раздел **Справка**  >  **по**  >  **установке** Eclipse. 
2. В списке установленных подключаемых модулей выберите Service Fabric и щелкните **Update** (Обновить). После этого будут установлены доступные обновления.
3. После обновления подключаемого модуля Service Fabric также обновите проект Gradle.  Щелкните правой кнопкой мыши **build.gradle** и выберите **Обновить**.

> [!NOTE]
> Если установка или обновление подключаемого модуля Service Fabric выполняется медленно, это может быть связано с настройкой Eclipse. Eclipse собирает метаданные обо всех изменениях на сайтах обновления, зарегистрированных с помощью экземпляра Eclipse. Чтобы ускорить процесс проверки и установки обновления подключаемого модуля Service Fabric, щелкните **Available Software Sites** (Доступные сайты программного обеспечения). Снимите флажки для всех сайтов, кроме тех, которые указывают на расположение подключаемого модуля Service Fabric (HTTPS: \/ /DL.Microsoft.com/Eclipse/Azure/servicefabric).

> [!NOTE]
>Если Eclipse не работает должным образом на компьютере Mac или требуется запуск от имени суперпользователя, перейдите в папку **ECLIPSE_INSTALLATION_PATH** и откройте подпапку **Eclipse.app/Contents/MacOS**. Запустите Eclipse, выполнив команду `./eclipse`.

## <a name="create-a-service-fabric-application-in-eclipse"></a>Создание приложения Service Fabric в Eclipse

1.  В Eclipse выберите **файл**  >  **создать**  >  **другой**. Выберите **Service Fabric Project** (Проект Service Fabric) и нажмите кнопку **Next** (Далее).

    ![Создание проекта Service Fabric, страница 1][create-application/p1]

2.  Введите имя проекта и нажмите кнопку **Next** (Далее).

    ![Создание проекта Service Fabric, страница 2][create-application/p2]

3.  В списке шаблонов выберите **Service Template** (Шаблон службы). Выберите тип шаблона службы (субъект, без отслеживания состояния, контейнер или гостевой двоичный файл) и нажмите кнопку **Next** (Далее).

    ![Создание проекта Service Fabric, страница 3][create-application/p3]

4.  Введите имя службы и сведения о ней, а затем нажмите кнопку **Finish** (Готово).

    ![Создание проекта Service Fabric, страница 4][create-application/p4]

5. Если вы создаете первый проект Service Fabric, в диалоговом окне **Open Associated Perspective** (Открыть связанную перспективу) нажмите кнопку **Yes** (Да).

    ![Создание проекта Service Fabric, страница 5][create-application/p5]

6.  Новый проект выглядит следующим образом:

    ![Создание проекта Service Fabric, страница 6][create-application/p6]

## <a name="build-a-service-fabric-application-in-eclipse"></a>Создание приложения Service Fabric в Eclipse

1.  Щелкните правой кнопкой мыши новое приложение Service Fabric, а затем выберите **Service Fabric**.

    ![Контекстное меню Service Fabric][publish/RightClick]

2. В контекстном меню выберите один из следующих вариантов:
    -   Чтобы выполнить сборку приложения без очистки, щелкните **Build Application** (Создать приложение).
    -   Для выполнения чистой сборки приложения щелкните **Rebuild Application** (Повторно создать приложение).
    -   Чтобы выполнить очистку приложения от артефактов сборки, щелкните **Clean Application** (Очистить приложение).

## <a name="deploy-a-service-fabric-application-to-the-local-cluster-with-eclipse"></a>Развертывание приложения Service Fabric в локальный кластер с помощью Eclipse

После создания приложения Service Fabric выполните следующие действия, чтобы развернуть его в локальный кластер.

1. Если вы еще не запустили локальный кластер, следуйте инструкциям, описанным в разделе о [настройке локального кластера](./service-fabric-get-started-linux.md#set-up-a-local-cluster), чтобы запустить его и проверить, что он работает.
2. Щелкните правой кнопкой мыши приложение Service Fabric, а затем выберите **Service Fabric**.

    ![Контекстное меню Service Fabric][publish/RightClick]

3.  В контекстном меню выберите **Deploy Application** (Развернуть приложение).
4.  Ход выполнения операции развертывания можно отслеживать в окне консоли.
5.  Чтобы проверить, что приложение работает, откройте Service Fabric Explorer в локальном кластере в окне браузера `http://localhost:19080/Explorer`. Разверните узел **приложений** и проверьте, работает ли приложение. 

Сведения об отладке приложения в Eclipse с помощью локального кластера см. в статье [Отладка приложения Java Service Fabric с помощью Eclipse](./service-fabric-debugging-your-application-java.md).

Вы также можете развернуть приложение в локальный кластер с помощью команды **публикации приложения**:

1. Щелкните правой кнопкой мыши приложение Service Fabric, а затем выберите **Service Fabric**.
2. В контекстном меню щелкните **Publish Application...** (Опубликовать приложение...).
3. В окне **публикации приложения** выберите **PublishProfiles/Local.json** как целевой профиль, а затем нажмите кнопку **Publish** (Опубликовать).

    ![Диалоговое окно публикации локально](./media/service-fabric-get-started-eclipse/localjson.png)

    По умолчанию профиль публикации Local.json настроен для публикации в локальный кластер. Дополнительные сведения о параметрах подключения и конечной точки, присутствующих в профилях публикации, см. в следующем разделе.

## <a name="publish-your-service-fabric-application-to-azure-with-eclipse"></a>Публикация приложения Service Fabric в Azure с помощью Eclipse

Чтобы опубликовать приложение в облако, сделайте следующее:

1. Чтобы опубликовать свое приложение в защищенный кластер в облаке, вам понадобится сертификат X.509 для обмена данными с кластером. В средах тестирования и разработки используемый сертификат часто является сертификатом кластера. В рабочих средах должен использоваться сертификат клиента, отличный от сертификата кластера. Вам нужен сертификат и закрытый ключ. Файл сертификата (и ключа) должен быть в формате PEM. Вы можете создать PEM-файл, содержащий сертификат и закрытый ключ, из PFX-файла с помощью следующей команды openssl:

    ```bash
    openssl pkcs12 -in your-cert-file.pfx -out your-cert-file.pem -nodes -passin pass:your-pfx-password
    ```

   Если PFX-файл не защищен паролем, используйте `--passin pass:` для последнего параметра.

2. Откройте файл **Cloud.json** в каталоге **PublishProfiles**. Необходимо правильно настроить конечную точку кластера и учетные данные безопасности.

   - В поле `ConnectionIPOrURL` содержится IP-адрес или URL-адрес кластера. Обратите внимание, что значение не содержит схему URL-адреса (`https://`).
   - По умолчанию для поля `ConnectionPort` должно использоваться значение `19080`, если только вы не изменили порт для кластера.
   - Поле `ClientKey` должно указывать на файл в формате PEM — с расширениями .pem или .key на локальном компьютере, который содержит закрытый ключ для сертификата клиента или кластера.
   - Поле `ClientCert` должно указывать на файл в формате PEM — с расширениями .pem или .crt на локальном компьютере, который содержит данные сертификата клиента или кластера. 

     ```bash
     {
         "ClusterConnectionParameters":
         {
            "ConnectionIPOrURL": "lnxxug0tlqm5.westus.cloudapp.azure.com",
            "ConnectionPort": "19080",
            "ClientKey": "[path_to_your_pem_file_on_local_machine]",
            "ClientCert": "[path_to_your_pem_file_on_local_machine]"
         }
     }
     ```

2. Щелкните правой кнопкой мыши приложение Service Fabric, а затем выберите **Service Fabric**.
3. В контекстном меню щелкните **Publish Application...** (Опубликовать приложение...).
3. В окне **публикации приложения** выберите **PublishProfiles/Cloud.json** как целевой профиль, а затем нажмите кнопку **Publish** (Опубликовать).

    ![Диалоговое окно публикации в облаке](./media/service-fabric-get-started-eclipse/cloudjson.png)

4. Ход выполнения операции публикации можно отслеживать в окне консоли.
5. Чтобы проверить, что приложение работает, откройте Service Fabric Explorer в кластере Azure в окне браузера. В примере выше это будет выглядеть примерно так: `https://lnxxug0tlqm5.westus.cloudapp.azure.com:19080/Explorer`. Разверните узел **приложений** и проверьте, работает ли приложение. 

В защищенных кластерах Linux, если ваше приложение содержит службы Reliable Services, вам также потребуется настроить сертификат, который ваши службы могут использовать для вызова API-интерфейсов среды выполнения Service Fabric. Дополнительные сведения см. в разделе [Настройка приложения Reliable Services для запуска на кластерах Linux](./service-fabric-configure-certificates-linux.md#configure-a-reliable-services-app-to-run-on-linux-clusters).

Краткое руководство по развертыванию приложения Service Fabric Reliable Services, написанного на Java, в защищенном кластере Linux см. в разделе [Краткое руководство. Развертывание приложения java Reliable Services](./service-fabric-quickstart-java-reliable-services.md).

## <a name="deploy-a-service-fabric-application-by-using-eclipse-run-configurations"></a>Развертывание приложения Service Fabric с помощью конфигураций запуска Eclipse

Альтернативным способом развертывания приложения Service Fabric является использование конфигураций запуска Eclipse.

1. В Eclipse перейдите к разделу **Запуск**  >  **конфигураций** запуска.
2. Выберите конфигурацию запуска **ServiceFabricDeployer** в разделе **Grade Project** (Проект Gradle).
3. В правой области на вкладке **Arguments** (Аргументы) проверьте, чтобы параметры **ip**, **port**, **clientCert** и **clientKey** были соответствующим образом настроены для развертывания. По умолчанию параметры заданы для развертывания в локальный кластер, как показано на следующем снимке экрана. Для публикации своего приложения в Azure вы можете изменить параметры, чтобы они содержали сведения о конечной точке и учетные данные безопасности для кластера Azure. Дополнительные сведения см. в разделе выше, посвященном [публикации приложения Service Fabric в Azure с помощью Eclipse](#publish-your-service-fabric-application-to-azure-with-eclipse).

    ![Локальное диалоговое окно конфигураций запуска](./media/service-fabric-get-started-eclipse/run-config-local.png)

5. **Рабочая папка** должна указывать на приложение, которое необходимо развернуть. Чтобы изменить приложение, нажмите кнопку **Workspace** (Рабочая область) и выберите нужное приложение.
6. Щелкните **Apply** (Применить), а затем — **Run** (Запуск).

Приложение будет собрано и развернуто через несколько секунд. Состояние развертывания можно отслеживать в Service Fabric Explorer.  

## <a name="add-a-service-fabric-service-to-your-service-fabric-application"></a>Добавление службы Service Fabric в приложение Service Fabric

Чтобы добавить службу Service Fabric в имеющееся приложение Service Fabric, сделайте следующее:

1.  Щелкните правой кнопкой мыши проект, в который нужно добавить службу, и выберите **Service Fabric**.

    ![Добавление службы Service Fabric, страница 1][add-service/p1]

2.  Щелкните **Add Service Fabric Service** (Добавить службу Service Fabric) и выполните действия для добавления службы в проект.
3.  Выберите шаблон службы и нажмите кнопку **Next** (Далее).

    ![Добавление службы Service Fabric, страница 2][add-service/p2]

4.  Введите имя службы (и другие сведения при необходимости), а затем нажмите кнопку **Add Service** (Добавить службу).  

    ![Добавление службы Service Fabric, страница 3][add-service/p3]

5.  После добавления службы общая структура проекта будет аналогична приведенной ниже.

    ![Добавление службы Service Fabric, страница 4][add-service/p4]

## <a name="edit-manifest-versions-of-your-service-fabric-java-application"></a>Изменение версий манифеста приложения Service Fabric на Java

Чтобы изменить версии манифеста, щелкните проект правой кнопкой мыши, щелкните пункт **Service Fabric**, а затем в раскрывающемся меню выберите **Изменить версии манифеста...** В мастере можно обновить версии манифеста приложения, служб и версии пакетов **кода**, **конфигурации** и **данных**.

Если вы установите флажок **Автоматическое обновление версий приложения и службы**, а затем обновите версию, то версии манифестов обновятся автоматически. Например, если вы сначала установите флажок, а затем обновите версию **кода** с 0.0.0 до 0.0.1, а затем щелкните **Готово**, то версия манифеста служб и манифеста приложения будет автоматически обновлена до 0.0.1.

## <a name="upgrade-your-service-fabric-java-application"></a>Обновление приложения Service Fabric на Java

Чтобы рассмотреть сценарий обновления, предположим, вы создали проект **App1** с помощью подключаемого модуля Service Fabric в Eclipse. Он был развернут с помощью подключаемого модуля для создания приложения с именем **fabric:/App1Application**. Тип приложения — **App1ApplicationType**, а версия приложения — 1.0. Теперь необходимо обновить приложение, не прерывая доступ к нему.

Сначала внесите изменения в приложение, а затем выполните повторную сборку службы в измененном виде. Замените файл манифеста измененной службы (ServiceManifest.xml) обновленной версией для этой службы (а также все необходимые файлы из каталогов Code, Config и Data). Замените также манифест приложения (ApplicationManifest.xml) обновленной версией для приложения и измененной службы.  

Чтобы обновить приложение с помощью Eclipse, можно создать дубликат профиля конфигурации запуска. Затем вы сможете использовать его для обновления приложения.

1.  Перейдите к разделу **Запуск**  >  **конфигураций запуска**. Щелкните небольшую стрелку слева от **Gradle Project** (Проект Gradle) в области слева.
2.  Щелкните правой кнопкой мыши **ServiceFabricDeployer** и выберите **Duplicate** (Дублировать). Введите новое имя для этой конфигурации, например **ServiceFabricUpgrader**.
3.  На панели справа на вкладке **Arguments** (Аргументы) измените **-Pconfig='deploy'** на **-Pconfig='upgrade'**, а затем нажмите кнопку **Apply** (Применить).

Это действие позволяет создать и сохранить профиль конфигурации запуска, который можно использовать для обновления приложения в любое время. При этом вы также получаете последнюю обновленную версию типа приложения из файла манифеста приложения.

Обновление приложения занимает несколько минут. Его можно отслеживать в Service Fabric Explorer.

## <a name="migrating-old-service-fabric-java-applications-to-be-used-with-maven"></a>Перенос устаревших приложений Java из Service Fabric для использования с Maven
Мы недавно переместили библиотеки Java для Service Fabric из пакета SDK для Java Service Fabric в репозиторий Maven. В новых приложениях, созданных с помощью Eclipse, создаются последние обновленные проекты (что позволяет работать с Maven). Но вы можете обновить существующие приложения Service Fabric без учета состояния или Java Reliable Actor, в которых ранее применялся пакет SDK для Service Fabric Java, чтобы использовать зависимости Java для Service Fabric из Maven. Выполните [инструкции](service-fabric-migrate-old-javaapp-to-use-maven.md), чтобы устаревшие приложения могли взаимодействовать с Maven.

## <a name="next-steps"></a>Дальнейшие действия

- Краткие инструкции по созданию приложения надежной службы Java и его развертыванию локально и в Azure см. в разделе [Краткое руководство. Развертывание приложения java Reliable Services](./service-fabric-quickstart-java-reliable-services.md).
- Сведения об отладке приложения Java в локальном кластере см. в статье [Отладка приложения Java Service Fabric с помощью Eclipse](./service-fabric-debugging-your-application-java.md).
- Сведения о мониторинге и диагностике приложений Service Fabric см. в статье [Мониторинг и диагностика состояния служб в локальной среде разработки](./service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally-linux.md).

<!-- Images -->

[sf-eclipse-plugin-install]: ./media/service-fabric-get-started-eclipse/service-fabric-eclipse-plugin.png

[create-application/p1]:./media/service-fabric-get-started-eclipse/create-application/p1.png
[create-application/p2]:./media/service-fabric-get-started-eclipse/create-application/p2.png
[create-application/p3]:./media/service-fabric-get-started-eclipse/create-application/p3.png
[create-application/p4]:./media/service-fabric-get-started-eclipse/create-application/p4.png
[create-application/p5]:./media/service-fabric-get-started-eclipse/create-application/p5.png
[create-application/p6]:./media/service-fabric-get-started-eclipse/create-application/p6.png

[publish/Publish]: ./media/service-fabric-get-started-eclipse/publish/Publish.png
[publish/RightClick]: ./media/service-fabric-get-started-eclipse/publish/RightClick.png

[add-service/p1]: ./media/service-fabric-get-started-eclipse/add-service/p1.png
[add-service/p2]: ./media/service-fabric-get-started-eclipse/add-service/p2.png
[add-service/p3]: ./media/service-fabric-get-started-eclipse/add-service/p3.png
[add-service/p4]: ./media/service-fabric-get-started-eclipse/add-service/p4.png

<!-- Links -->
[buildship-update]: https://projects.eclipse.org/projects/tools.buildship
