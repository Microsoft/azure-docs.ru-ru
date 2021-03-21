---
title: Установка кластера Azure Service Fabric на платформе Linux в Windows
description: В этой статье описывается установка кластеров Service Fabric на платформе Linux на компьютерах разработки Windows. Этот подход удобен для межплатформенной разработки.
ms.topic: conceptual
ms.date: 10/16/2020
ms.openlocfilehash: 7b25a84e76773baea9f17430df1b7ba13aa661aa
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93087083"
---
# <a name="set-up-a-linux-service-fabric-cluster-on-your-windows-developer-machine"></a>Установка кластера Service Fabric на платформе Linux на компьютере разработчики Windows

В этом документе описано, как настроить локальный кластер Linux Service Fabric на компьютере с разработкой Windows. Установка локального кластера Linux удобна для быстрого тестирования приложений, нацеленных на кластеры Linux, но разработанных на компьютере Windows.

## <a name="prerequisites"></a>Предварительные требования
Кластеры Service Fabric под управлением Linux не работают в Windows, но для включения межплатформенного создания прототипов мы предоставили в Linux Service Fabric контейнер DOCKER кластера с одним ящиком, который можно развернуть с помощью Docker для Windows.

Перед началом работы вам потребуются:

* не менее 4 ГБ ОЗУ;
* Последняя версия [DOCKER для Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)
* DOCKER должен работать в режиме контейнеров Linux

>[!TIP]
> Чтобы установить DOCKER на компьютере с Windows, выполните действия, описанные в [документации по DOCKER](https://store.docker.com/editions/community/docker-ce-desktop-windows/plans/docker-ce-desktop-windows-tier?tab=instructions). Затем [проверьте правильность установки](https://docs.docker.com/docker-for-windows/#check-versions-of-docker-engine-compose-and-machine).
>

## <a name="create-a-local-container-and-setup-service-fabric"></a>Создание локального контейнера и настройка Service Fabric
Чтобы настроить локальный контейнер DOCKER и запустить на нем кластер Service Fabric, выполните следующие действия.


1. Обновите конфигурацию управляющей программы Docker на узле следующим образом и перезапустите управляющую программу Docker: 

    ```json
    {
      "ipv6": true,
      "fixed-cidr-v6": "2001:db8:1::/64"
    }
    ```
    Рекомендуемый способ обновления заключается в переходе по адресу: 

    * Значок DOCKER > параметры > подсистеме DOCKER
    * Добавить новые поля, перечисленные выше
    * Применить & перезапустить. Перезапустите управляющую программу DOCKER, чтобы изменения вступили в силу.

2. Запустите кластер с помощью PowerShell.<br/>
    <b>Ubuntu 18,04 LTS:</b>
    ```powershell
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u18
    ```

    <b>Ubuntu 16,04 LTS:</b>
    ```powershell
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u16
    ```

    >[!TIP]
    > По умолчанию вы получите образ с последней версией Service Fabric. Конкретные редакции см. на странице [Service Fabric OneBox](https://hub.docker.com/_/microsoft-service-fabric-onebox) в центре DOCKER.



3. (Необязательно.) создайте расширенный образ Service Fabric.

    В новом каталоге создайте файл `Dockerfile` с именем для создания настраиваемого образа:

    >[!NOTE]
    >Вы можете адаптировать изображение выше с помощью Dockerfile, чтобы добавить в контейнер дополнительные программы или зависимости.
    >Например, добавленный элемент `RUN apt-get install nodejs -y` разрешает поддержку приложений `nodejs` в качестве гостевых исполняемых файлов.
    ```Dockerfile
    FROM mcr.microsoft.com/service-fabric/onebox:u18
    RUN apt-get install nodejs -y
    EXPOSE 19080 19000 80 443
    WORKDIR /home/ClusterDeployer
    CMD ["./ClusterDeployer.sh"]
    ```
    
    >[!TIP]
    > По умолчанию вы получите образ с последней версией Service Fabric. Конкретные редакции см. на странице [центра DOCKER](https://hub.docker.com/r/microsoft/service-fabric-onebox/) .

    Чтобы создать повторно используемое изображение из `Dockerfile` , откройте терминал, а затем запустите его, а `cd` `Dockerfile` затем выполните команду:

    ```powershell 
    docker build -t mysfcluster .
    ```
    
    >[!NOTE]
    >Для выполнения этой операции потребуется некоторое время, но она выполняется только один раз.

    Теперь можно быстро запустить локальную копию Service Fabric при необходимости, запустив:

    ```powershell 
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mysfcluster
    ```

    >[!TIP]
    >Укажите имя экземпляра контейнера, которое будет более удобным для чтения. 
    >
    >Если приложение ожидает передачи данных через определенные порты, их необходимо указать с помощью дополнительных тегов `-p`. Например, если приложение ожидает передачи данных через порт 8080, добавьте следующий тег `-p`:
    >
    >`docker run -itd -p 19000:19000 -p 19080:19080 -p 8080:8080 --name sfonebox mcr.microsoft.com/service-fabric/onebox:u18`
    >


4. Для запуска кластера потребуется некоторое время. Чтобы просмотреть состояние работоспособности кластера, можно перейти к панели мониторинга кластера `http://localhost:19080` или просмотреть журналы с помощью следующей команды:

    ```powershell 
    docker logs sftestcluster
    ```

5. После успешного развертывания кластера, как показано на шаге 4, можно перейти ``http://localhost:19080`` с компьютера под Windows, чтобы найти панель мониторинга Service Fabric Explorer. На этом этапе можно подключиться к этому кластеру с помощью средств на компьютере разработчика Windows и развернуть приложения, предназначенные для кластеров Linux Service Fabric. 

    > [!NOTE]
    > Подключаемый модуль Eclipse сейчас не поддерживается в Windows. 

6. По завершении закройте и очистите контейнер с помощью следующей команды:

    ```powershell 
    docker rm -f sftestcluster
    ```

### <a name="known-limitations"></a>Известные ограничения 
 
 Ниже перечислены известные ограничения для локального кластера, запущенного в контейнере для Mac: 
 
 * Служба DNS не запускается и в настоящее время не поддерживается в контейнере. [#132 проблемы](https://github.com/Microsoft/service-fabric/issues/132)
 * Для выполнения приложений на основе контейнеров необходимо запустить на узле Linux. Вложенные приложения контейнера в настоящее время не поддерживаются.

## <a name="next-steps"></a>Дальнейшие действия
* [Создание первого приложения Azure Service Fabric](service-fabric-create-your-first-linux-application-with-java.md)
* Приступите к работе с [Eclipse](./service-fabric-get-started-eclipse.md).
* Просмотрите другие [примеры Java](https://github.com/Azure-Samples/service-fabric-java-getting-started).
* Дополнительные сведения о [вариантах поддержки Service Fabric](service-fabric-support.md)


<!-- Image references -->

[publishdialog]: ./media/service-fabric-manage-multiple-environment-app-configuration/publish-dialog-choose-app-config.png
[app-parameters-solution-explorer]:./media/service-fabric-manage-multiple-environment-app-configuration/app-parameters-in-solution-explorer.png
