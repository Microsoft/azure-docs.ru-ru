---
title: Быстрое развертывание существующего приложения в кластере
description: Используйте кластер Azure Service Fabric, чтобы разместить имеющееся приложение Node.js с помощью Visual Studio.
ms.topic: conceptual
ms.date: 12/06/2017
ms.custom: devx-track-js
ms.openlocfilehash: dfd1fe9db54925bc17a53a7686fa34d5ea53cd5a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96013604"
---
# <a name="host-a-nodejs-application-on-azure-service-fabric"></a>Размещение приложения Node.js в Azure Service Fabric

Это краткое руководство поможет вам развернуть имеющееся приложение (Node.js в этом примере) в кластере Service Fabric, выполняющемся в Azure.

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы [настройте среду разработки](service-fabric-get-started.md). В состав которого входит установка пакета SDK для Service Fabric и Visual Studio 2019 или 2015.

У вас также должно быть имеющееся приложение Node.js для развертывания. В этом кратком руководстве используется простой веб-сайт Node.js, который можно загрузить [здесь][download-sample]. Извлеките этот файл в папку `<path-to-project>\ApplicationPackageRoot\<package-name>\Code\` после создания проекта на следующем шаге.

Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure][create-account].

## <a name="create-the-service"></a>Создание службы

Запустите Visual Studio от имени **администратора**.

Создайте проект, нажав клавиши `CTRL`+`SHIFT`+`N`.

В диалоговом окне **Создание проекта** выберите **Облако > Приложение Service Fabric**.

Присвойте приложению имя **мигуестапп** и нажмите кнопку **создать**.

>[!IMPORTANT]
>Node.js может легко преодолеть лимит в 260 знаков для путей Windows. Используйте короткий путь для самого проекта, например **c:\code\svc1**. При необходимости можно выполнить **[эти инструкции](https://stackoverflow.com/a/41687101/1664231)** , чтобы включить длинные пути к файлам в Windows 10.
   
![Диалоговое окно "Новый проект" в Visual Studio][new-project]

Вы можете создать любой тип службы Service Fabric из следующего диалогового окна. В рамках этого краткого руководства выберите **Гостевой исполняемый файл**.

Назовите службу **MyGuestService** и задайте параметрам справа следующие значения:

| Параметр                   | Значение |
| ------------------------- | ------ |
| Папка пакета кода       | _&lt;папка с приложением Node.js&gt;_ |
| Поведение пакета кода     | Скопируйте содержимое папки в проект |
| Программа                   | node.exe |
| Аргументы                 | server.js |
| Рабочая папка            | CodePackage |

Нажмите кнопку **ОК**.

![Диалоговое окно "Новая служба" в Visual Studio][new-service]

Visual Studio создаст проект приложения и проект службы субъекта, а затем отобразит их в обозревателе решений.

Проект приложения (**мигуестапп**) не содержит никакого кода напрямую. Проект ссылается на набор проектов служб. Кроме того, он содержит три других типа содержимого:

* **Профили публикации**  
Средства настройки для различных сред.

* **Создание и запуск скриптов PowerShell из консоли Configuration Manager**  
Сценарий PowerShell для развертывания и обновления приложения.

* **Определение приложения**  
Содержит манифест приложения в разделе *ApplicationPackageRoot*. Связанные файлы параметров приложения расположены в разделе *ApplicationParameters*. Они определяют характеристики этого приложения, и с помощью них можно настроить приложение для конкретной среды.
    
Обзор содержимого проекта службы вы найдете в статье [Начало работы со службами Reliable Services в Service Fabric](service-fabric-reliable-services-quick-start.md).

## <a name="set-up-networking"></a>Настройка сети

Пример приложения Node.js, которое мы развертываем, использует порт **80**, и нам нужно сообщить Service Fabric о необходимости использования этого порта.

Откройте файл **ServiceManifest.xml** в проекте. В нижней части манифеста есть `<Resources> \ <Endpoints>` запись с уже определенным элементом. Измените эту запись, чтобы добавить `Port`, `Protocol` и `Type`. 

```xml
  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="MyGuestAppServiceTypeEndpoint" Port="80" Protocol="http" Type="Input" />
    </Endpoints>
  </Resources>
```

## <a name="deploy-to-azure"></a>Развернуть в Azure

Если нажать клавишу **F5** и запустить проект, он будет развернут в локальном кластере. Тем не менее давайте лучше развернем его в Azure.

Щелкните проект правой кнопкой мыши и выберите **Опубликовать**, чтобы открыть диалоговое окно для публикации в Azure.

![Публикация в диалоговом окне Azure для службы Service Fabric][publish]

Выберите целевой профиль **PublishProfiles\Cloud.xml**.

Выберите учетную запись Azure для развертывания, если вы этого еще не сделали. Если у вас ее нет, [зарегистрируйте ее][create-account].

В разделе **Конечная точка подключения:** выберите кластер Service Fabric для развертывания. Если у вас ее нет, выберите **&lt; создать новый кластер... &gt;** открывает окно веб-браузера для портал Azure. Дополнительные сведения см. в разделе [Создание кластера на портале Azure](service-fabric-cluster-creation-via-portal.md#create-cluster-in-the-azure-portal). 

При создании кластера Service Fabric обязательно установите для параметра **настраиваемых конечных точек** значение **80**.

![Конфигурация типа узла Service Fabric с настраиваемой конечной точкой][custom-endpoint]

Создание нового кластера Service Fabric займет некоторое время. После создания вернитесь в диалоговое окно Публикация и выберите **&lt; обновить &gt;**. Новый кластер появится в окне раскрывающегося списка. Выберите его.

Щелкните **Опубликовать** и дождитесь завершения развертывания.

Это может занять несколько минут. После завершения развертывания необходимо подождать некоторое время, чтобы приложение стало полностью доступным.

## <a name="test-the-website"></a>Тестирование веб-сайта

После публикации службы проверьте ее в веб-браузере. 

Сначала откройте портал Azure и найдите службу Service Fabric.

Просмотрите колонку обзора адреса службы. Используйте доменное имя из свойства _конечной точки подключения клиента_. Например, `http://mysvcfab1.westus2.cloudapp.azure.com`.

![Колонка обзора Service Fabric на портале Azure][overview]

Перейдите по этому адресу, где вы увидите `HELLO WORLD` ответ.

## <a name="delete-the-cluster"></a>Удаление кластера

Не забудьте удалить все ресурсы, созданные для этого краткого руководства, так как плата за эти ресурсы будет заряжена.

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь с дополнительными сведениями о [гостевых исполняемых файлах](service-fabric-guest-executables-introduction.md).

<!-- Image References -->

[new-project]: ./media/quickstart-guest-app/new-project.png
[new-service]: ./media/quickstart-guest-app/template.png
[solution-exp]: ./media/quickstart-guest-app/solution-explorer.png
[publish]: ./media/quickstart-guest-app/publish.png
[overview]: ./media/quickstart-guest-app/overview.png
[custom-endpoint]: ./media/quickstart-guest-app/custom-endpoint.png

[download-sample]: https://github.com/MicrosoftDocs/azure-cloud-services-files/raw/temp/service-fabric-node-website.zip
[create-account]: https://azure.microsoft.com/free/?WT.mc_id=A261C142F
