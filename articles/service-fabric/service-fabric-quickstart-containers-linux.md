---
title: Создание приложения-контейнера Linux в Service Fabric в Azure
description: В этом руководстве вы создадите образ Docker с приложением, отправите образ в реестр контейнеров и развернете контейнер в кластер Service Fabric.
ms.topic: quickstart
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: f58b8feda0e88d8a5e7cddaabbc650b0f0ab3973
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98788168"
---
# <a name="quickstart-deploy-linux-containers-to-service-fabric"></a>Краткое руководство. Развертывание контейнеров Linux в Service Fabric

Azure Service Fabric — это платформа распределенных систем для развертывания масштабируемых надежных микрослужб и контейнеров и управления ими.

В этом руководстве объясняется, как развертывать контейнеры Linux в кластер Service Fabric в Azure. Изучив это руководство, вы создадите приложение для голосования с веб-интерфейсом Python и серверной частью Redis, которое работает в кластере Service Fabric. Вы также узнаете, как выполнить отработку отказа приложения и масштабировать приложение в кластере.

![Веб-страница приложения для голосования][quickstartpic]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством сделайте следующее:

1. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

2. Установите [Azure CLI](/cli/azure/install-azure-cli-apt).

3. Установите [пакет SDK и CLI для Service Fabric](service-fabric-get-started-linux.md#installation-methods).

4. Установка [Git](https://git-scm.com/)


## <a name="get-the-application-package"></a>Получение пакета приложения

Чтобы развернуть контейнеры в кластер Service Fabric, вам понадобится набор файлов манифеста (определение приложения), которые описывают отдельные контейнеры и приложение.

В консоли с помощью Git клонируйте копию определения приложения, а затем замените каталоги клонированного экземпляра каталогом `Voting`.

```bash
git clone https://github.com/Azure-Samples/service-fabric-containers.git

cd service-fabric-containers/Linux/container-tutorial/Voting
```

## <a name="create-a-service-fabric-cluster"></a>Создание кластера Service Fabric

Чтобы развернуть приложение в Azure, требуется кластер Service Fabric, в котором будет выполняться приложение. С помощью следующих команд можно создать кластер из пяти узлов в Azure.  Команды также создают самозаверяющий сертификат, добавляют его в хранилище ключей и скачивают сертификат локально. Новый сертификат используется для защиты кластера во время развертывания и для проверки подлинности клиентов.

```azurecli
#!/bin/bash

# Variables
ResourceGroupName='containertestcluster' 
ClusterName='containertestcluster' 
Location='eastus' 
Password='q6D7nN%6ck@6' 
Subject='containertestcluster.eastus.cloudapp.azure.com' 
VaultName='containertestvault' 
VmPassword='Mypa$$word!321'
VmUserName='sfadminuser'

# Login to Azure and set the subscription
az login

az account set --subscription <mySubscriptionID>

# Create resource group
az group create --name $ResourceGroupName --location $Location 

# Create secure five node Linux cluster. Creates a key vault in a resource group
# and creates a certficate in the key vault. The certificate's subject name must match 
# the domain that you use to access the Service Fabric cluster.  The certificate is downloaded locally.
az sf cluster create --resource-group $ResourceGroupName --location $Location --certificate-output-folder . --certificate-password $Password --certificate-subject-name $Subject --cluster-name $ClusterName --cluster-size 5 --os UbuntuServer1604 --vault-name $VaultName --vault-resource-group $ResourceGroupName --vm-password $VmPassword --vm-user-name $VmUserName
```

> [!Note]
> Служба веб-интерфейса ожидает передачи данных через порт 80 для входящего трафика. По умолчанию порт 80 открыт в виртуальных машинах кластера и в подсистеме балансировки нагрузки Azure.
>

## <a name="configure-your-environment"></a>Настройка среды

Service Fabric предоставляет ряд средств, которые можно использовать для управления кластером и его приложениями:

- средство Service Fabric Explorer на основе браузера;
- интерфейс командной строки (CLI) Service Fabric, который работает поверх Azure CLI; 
- команды PowerShell.

При работе с этим кратким руководством используется интерфейс командной строки Service Fabric и Service Fabric Explorer (сетевой инструмент). Чтобы использовать Service Fabric Explorer, вам необходимо импортировать сертификат PFX-файла в браузер. По умолчанию PFX-файл не имеет пароля.

В Ubuntu 16.04 браузером по умолчанию является Mozilla Firefox. Чтобы импортировать сертификат в Firefox, нажмите кнопку меню в правом верхнем углу браузера, а затем щелкните **Параметры**. На странице **Настройки** введите в поле поиска слово "сертификаты". Нажмите кнопку **Просмотр сертификатов**, выберите вкладку **Ваши сертификаты**, щелкните **Импорт** и следуйте инструкциям на экране, чтобы импортировать сертификат.

   ![Установка сертификата в Firefox](./media/service-fabric-quickstart-containers-linux/install-cert-firefox.png)

## <a name="deploy-the-service-fabric-application"></a>Развертывание приложения Service Fabric

1. Подключитесь к кластеру Service Fabric в Azure с помощью CLI. Конечная точка — это конечная точка управления кластера. В предыдущем разделе вы создали PEM-файл. 

    ```bash
    sfctl cluster select --endpoint https://containertestcluster.eastus.cloudapp.azure.com:19080 --pem containertestcluster22019013100.pem --no-verify
    ```

2. С помощью скрипта установки, предназначенного для копирования определения приложения для голосования в кластер, зарегистрируйте тип приложения и создайте экземпляр приложения.  Файл сертификата PEM должен находиться в том же каталоге, что и файл *install.sh*.

    ```bash
    ./install.sh
    ```

3. Откройте веб-браузер и перейдите в конечную точку Service Fabric Explorer для кластера. Конечная точка имеет формат **https://\<my-azure-service-fabric-cluster-url>:19080/Explorer** (например, `https://containertestcluster.eastus.cloudapp.azure.com:19080/Explorer`). </br>

4. Разверните узел **Приложения**, чтобы увидеть запись для типа приложения голосования и созданный вами экземпляр.

    ![Service Fabric Explorer][sfx]

5. Чтобы подключиться к запущенному контейнеру, откройте веб-браузер и перейдите по URL-адресу кластера, например `http://containertestcluster.eastus.cloudapp.azure.com:80`. Вы должны увидеть приложение голосования в браузере.

    ![Веб-страница приложения для голосования][quickstartpic]

> [!NOTE]
> Можно также развернуть приложения Service Fabric с помощью Docker Compose. Например, следующую команду можно использовать для развертывания и установки приложения в кластере с помощью Docker Compose.
>  ```bash
> sfctl compose create --deployment-name TestApp --file-path ../docker-compose.yml
> ```

## <a name="fail-over-a-container-in-a-cluster"></a>Отработка отказа контейнера в кластере

Платформа Service Fabric обеспечивает автоматический перенос экземпляров контейнера на другие узлы в кластере в случае сбоя. Можно также вручную перенести контейнеры с одного узла на другие узлы в кластере. Service Fabric предоставляет несколько способов масштабирования ваших служб. В приведенных далее шагах используется средство Service Fabric Explorer.

Чтобы выполнить отработку отказа внешнего контейнера, сделайте следующее:

1. Откройте Service Fabric Explorer в своем кластере (например, по ссылке `https://containertestcluster.eastus.cloudapp.azure.com:19080/Explorer`).
2. Щелкните узел **fabric:/Voting/azurevotefront** в представлении в виде дерева и разверните узел раздела (он отображается в виде идентификатора GUID). Найдите имя узла в представлении в виде дерева с узлами, на которых сейчас выполняется контейнер. Пример: `_nodetype_1`.
3. Разверните узел **Узлы** в представлении в виде дерева. Щелкните многоточие (…) рядом с узлом, на котором выполняется контейнер.
4. Выберите **Перезапустить**, чтобы перезапустить этот узел, и подтвердите операцию перезапуска. Перезапуск приведет к отработке отказа контейнера на другой узел в кластере.

    ![Представление узла в Service Fabric Explorer][sfxquickstartshownodetype]

## <a name="scale-applications-and-services-in-a-cluster"></a>Масштабирование приложений и служб в кластере

Службы Service Fabric можно легко масштабировать в пределах кластера в соответствии с изменениями нагрузки на службы. Масштабирование службы осуществляется путем изменения числа экземпляров, запущенных в кластере.

Для масштабирования службы веб-интерфейса выполните следующие действия:

1. Откройте Service Fabric Explorer в своем кластере (например, по ссылке `https://containertestcluster.eastus.cloudapp.azure.com:19080`).
2. Щелкните многоточие (три точки) рядом с узлом **fabric:/Voting/azurevotefront** в представлении в виде дерева и выберите **Масштабировать службу**.

    ![Запуск масштабирования службы Service Fabric Explorer][containersquickstartscale]

    Теперь вы можете изменить количество экземпляров службы веб-интерфейса.

3. Измените количество на **2** и нажмите кнопку **Масштабировать службу**.
4. Щелкните узел **fabric:/Voting/azurevotefront** в представлении в виде дерева и разверните узел раздела (он отображается в виде идентификатора GUID).

    ![Масштабирование службы Service Fabric Explorer завершено][containersquickstartscaledone]

    Теперь вы можете увидеть два экземпляра службы. В представлении в виде дерева можно просмотреть, на каких узлах выполняются эти экземпляры.

С помощью этой простой задачи управления вы удвоили количество ресурсов для обработки пользовательской нагрузки для службы веб-интерфейса. Важно понимать, что для надежной работы службы не требуется запускать несколько экземпляров службы. При сбое в работе службы Service Fabric запускает новый экземпляр службы в кластере.

## <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы удалить экземпляр приложения из кластера и отменить регистрацию типа приложения, используйте скрипт удаления (uninstall.sh), указанный в шаблоне. Очистка экземпляра займет некоторое время. Поэтому не следует запускать скрипт установки сразу же после выполнения скрипта удаления. С помощью Service Fabric Explorer вы сможете определить, что экземпляр удален, а регистрация типа приложения отменена.

```bash
./uninstall.sh
```

Чтобы удалить кластер и все ресурсы, который он использует, проще всего удалить группу ресурсов.

Войдите в Azure и выберите идентификатор подписки, в которой вы хотите удалить кластер. Идентификатор подписки можно узнать, войдя на портал Azure. Удалите группу ресурсов и все ресурсы кластера с помощью команды [az group delete](/cli/azure/group).

```azurecli
az login
az account set --subscription <guid>
ResourceGroupName="containertestcluster"
az group delete --name $ResourceGroupName
```

По завершении работы с кластером можно удалить сертификат из хранилища сертификатов. Пример:
- В Windows: Используйте [оснастку MMC для сертификатов](/dotnet/framework/wcf/feature-details/how-to-view-certificates-with-the-mmc-snap-in). При добавлении оснастки не забудьте выбрать **свою учетную запись пользователя**. Перейдите в папку `Certificates - Current User\Personal\Certificates` и удалите сертификат.
- Для компьютеров Mac. Используйте приложение цепочки ключей.
- В Ubuntu: Выполните шаги, которые вы использовали для просмотра сертификатов, и удалите сертификат.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как развернуть приложение-контейнер Linux в кластере Service Fabric в Azure, выполнить отработку отказа для приложения и масштабировать приложение в кластере. Дополнительные сведения о работе с контейнерами Linux в Service Fabric см. в руководстве для приложений-контейнеров Linux.

> [!div class="nextstepaction"]
> [Создание приложения-контейнера Linux](./service-fabric-tutorial-create-container-images.md)

[sfx]: ./media/service-fabric-quickstart-containers-linux/containersquickstartappinstance.png
[quickstartpic]: ./media/service-fabric-quickstart-containers-linux/votingapp.png
[sfxquickstartshownodetype]:  ./media/service-fabric-quickstart-containers-linux/containersquickstartrestart.png
[containersquickstartscale]: ./media/service-fabric-quickstart-containers-linux/containersquickstartscale.png
[containersquickstartscaledone]: ./media/service-fabric-quickstart-containers-linux/containersquickstartscaledone.png
