---
title: Создание приложения-контейнера Windows в Service Fabric в Azure
description: В этом кратком руководстве вы создадите первое приложение-контейнер Windows в Azure Service Fabric.
ms.topic: quickstart
ms.date: 07/10/2019
ms.author: atsenthi
ms.custom: mvc
ms.openlocfilehash: 934b679813323fe144f935811eb613ab68b561db
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "87089623"
---
# <a name="quickstart-deploy-windows-containers-to-service-fabric"></a>Краткое руководство. Развертывание контейнеров Windows в Service Fabric

Azure Service Fabric — это платформа распределенных систем для развертывания масштабируемых надежных микрослужб и контейнеров и управления ими.

Чтобы запустить существующее приложение в контейнере Windows кластера Service Fabric, не требуется вносить изменения в приложение. В этом кратком руководстве показано, как развернуть готовый образ контейнера Docker в приложении Service Fabric. В итоге у вас будет рабочий Windows Server Core 2016 и контейнер для IIS. В этом кратком руководстве описывается развертывание контейнера Windows. Развертывание контейнера Linux описывается [в этом кратком руководстве](service-fabric-quickstart-containers-linux.md).

![Страница служб IIS по умолчанию][iis-default]

Из этого краткого руководства вы узнаете, как выполнять следующие задачи:

* упаковка контейнера образов Docker;
* настройка обмена данными;
* создание и упаковка приложений Service Fabric;
* развертывание приложения-контейнера в Azure.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure. Вы можете создать [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Компьютер для разработки, на котором установлено ПО, перечисленное ниже.
  * Visual Studio 2019 или Windows 2019.
  * [Пакет SDK и средства для Service Fabric](service-fabric-get-started.md).

## <a name="package-a-docker-image-container-with-visual-studio"></a>Упаковка контейнера образов Docker с помощью Visual Studio

Пакет SDK и средства для Service Fabric предоставляют шаблон службы для развертывания контейнера в кластере Service Fabric.

Запустите Visual Studio от имени администратора.  Выберите **File** > **New** > **Project** ( Файл > Создать > Проект).

Выберите **Приложение Service Fabric**, назовите его MyFirstContainer и щелкните **Создать**.

Выберите **Контейнер** в шаблонах **размещенных контейнеров и приложений**.

В поле **Имя образа** введите mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2016. Это [базовый образ Windows Server Core Server и IIS](https://hub.docker.com/_/microsoft-windows-servercore-iis).

Настройте сопоставление порта контейнера с портом узла, чтобы входящие запросы к службе через порт 80 сопоставлялись с портом 80 в контейнере.  Укажите **порт контейнера** (80) и **узла** (80).  

Присвойте службе имя MyContainerService и нажмите кнопку **ОК**.

![Диалоговое окно создания службы][new-service]

## <a name="specify-the-os-build-for-your-container-image"></a>Укажите сборку операционной системы для образа контейнера

Контейнеры, созданные с помощью одной версии Windows Server могут не работать на узле под управлением другой версии Windows Server. Например, созданные с помощью Windows Server версии 1709 контейнеры не работают на узлах под управлением Windows Server 2016. Дополнительные сведения см. в разделе [ОС контейнера Windows Server и совместимость ОС узлов](service-fabric-get-started-containers.md#windows-server-container-os-and-host-os-compatibility). 

В версии 6.1 среды выполнения Service Fabric и более поздних можно указать несколько образов операционной системы на контейнер и отметить каждый из них, указав версию сборки операционной системы, для которой он будет развертываться. Это помогает обеспечить запуск приложения на узлах под управлением других версий ОС Windows. Дополнительные сведения см. в разделе [Указание сборок ОС для образов контейнеров](service-fabric-get-started-containers.md#specify-os-build-specific-container-images). 

Корпорация Майкрософт публикует различные образы для версий IIS, встроенных в разных версиях Windows Server. Чтобы убедиться в том, что Service Fabric развертывает контейнер, совместимый с версией Windows Server на узлах кластера, где он развертывает приложение, добавьте следующие строки в файл *ApplicationManifest.xml*. Версия сборки для Windows Server 2016 — 14393, а версия сборки для Windows Server версии 1709 — 16299.

```xml
    <ContainerHostPolicies CodePackageRef="Code"> 
      <ImageOverrides> 
        ...
          <Image Name="mcr.microsoft.com/windows/servercore/iis:windowsservercore-1803" /> 
          <Image Name= "mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2016" Os="14393" /> 
          <Image Name="mcr.microsoft.com/windows/servercore/iis:windowsservercore-1709" Os="16299" /> 
      </ImageOverrides> 
    </ContainerHostPolicies> 
```

Манифест служб продолжает указывать только один образ для наносервера, `mcr.microsoft.com/windows/servercore/iis:windowsservercore-ltsc2016`.

Кроме того, в файле *ApplicationManifest.xml* измените значение параметра **PasswordEncrypted** на **false**. Учетная запись и пароль не указаны для образа общедоступного контейнера, находящегося в центре Docker, поэтому мы отключили возможность шифрования, так как шифрование пустого пароля приведет к возникновению ошибки сборки.

```xml
<RepositoryCredentials AccountName="" Password="" PasswordEncrypted="false" />
```

## <a name="create-a-cluster"></a>Создание кластера

В следующем примере сценария создается кластер Service Fabric из пяти узлов, защищенный с помощью сертификата X.509. Команда создает самозаверяющий сертификат и отправляет его в новое хранилище ключей. Сертификат также копируется в локальный каталог. Дополнительные сведения о создании кластера с помощью этого сценария см. в статье [Создание кластера Service Fabric](scripts/service-fabric-powershell-create-secure-cluster-cert.md).

При необходимости установите Azure PowerShell с помощью инструкций, приведенных в [руководстве по Azure PowerShell](/powershell/azure/).

Прежде чем запустить следующий сценарий, в PowerShell выполните командлет `Connect-AzAccount`, чтобы создать подключение к Azure.

Скопируйте приведенный ниже сценарий в буфер обмена и откройте **интегрированную среду сценариев Windows PowerShell**.  Вставьте содержимое в пустое окно Untitled1.ps1. Затем укажите значения для содержащихся в сценарии переменных: `subscriptionId`, `certpwd`, `certfolder`, `adminuser`, `adminpwd` и т. д.  Указанный для `certfolder` каталог должен существовать до запуска сценария.

[!code-powershell[main](../../powershell_scripts/service-fabric/create-secure-cluster/create-secure-cluster.ps1 "Create a Service Fabric cluster")]

Предоставив значения для переменных, нажмите клавишу **F5**, чтобы запустить сценарий.

После выполнения сценария и создания кластера в выходных данных найдите значение `ClusterEndpoint`. Пример:

```powershell
...
ClusterEndpoint : https://southcentralus.servicefabric.azure.com/runtime/clusters/b76e757d-0b97-4037-a184-9046a7c818c0
```

### <a name="install-the-certificate-for-the-cluster"></a>Установка сертификата для кластера

Теперь мы установим PFX-файл в хранилище сертификатов *CurrentUser\My*. PFX-файл будет расположен в каталоге, указанном с помощью переменной среды `certfolder` в приведенном выше сценарии PowerShell.

Перейдите в этот каталог, а затем выполните следующую команду PowerShell, подставив имя PFX-файла, который находится в каталоге `certfolder`, и пароль, указанный в переменной `certpwd`. В этом примере текущий каталог получает значение каталога, указанного в переменной `certfolder` в сценарии PowerShell. Из этого каталога выполняется команда `Import-PfxCertificate`:

```powershell
PS C:\mycertificates> Import-PfxCertificate -FilePath .\mysfclustergroup20190130193456.pfx -CertStoreLocation Cert:\CurrentUser\My -Password (ConvertTo-SecureString Password#1234 -AsPlainText -Force)
```

Эта команда возвращает отпечаток:

```powershell
  ...
  PSParentPath: Microsoft.PowerShell.Security\Certificate::CurrentUser\My

Thumbprint                                Subject
----------                                -------
0AC30A2FA770BEF566226CFCF75A6515D73FC686  CN=mysfcluster.SouthCentralUS.cloudapp.azure.com
```

Запомните значение отпечатка для выполнения следующего шага.

## <a name="deploy-the-application-to-azure-using-visual-studio"></a>Развертывание приложения в Azure с помощью Visual Studio

Теперь, когда приложение готово, можно развернуть его в кластер напрямую из Visual Studio.

Щелкните правой кнопкой мыши **MyFirstContainer** в обозревателе решений и выберите команду **Опубликовать**. Появится диалоговое окно "Опубликовать".

Скопируйте содержимое после **CN=** из результатов выполнения приведенной выше команды `Import-PfxCertificate` в окне PowerShell и добавьте к нему порт `19000`. Например, `mysfcluster.SouthCentralUS.cloudapp.azure.com:19000`. Скопируйте его в поле **Конечная точка подключения**. Запомните это значение, так как оно понадобится при выполнении предстоящего шага.

Щелкните **Расширенные параметры подключения** и проверьте сведения о параметрах подключения.  Значения *FindValue* и *ServerCertThumbprint* должны соответствовать отпечатку сертификата, установленного при выполнении командлета `Import-PfxCertificate` на предыдущем шаге.

![Диалоговое окно "Публикация"](./media/service-fabric-quickstart-containers/publish-app.png)

Нажмите кнопку **Опубликовать**.

Имя каждого приложения в кластере должно быть уникальным. В случае конфликта имен переименуйте проект Visual Studio и повторите развертывание.

Откройте браузер и перейдите по адресу, указанному в поле **Конечная точка подключения** на предыдущем шаге. При необходимости можно добавить в начало URL-адреса идентификатор схемы `http://` и порт `:80`. Например http:\//mysfcluster.SouthCentralUS.cloudapp.azure.com:80.

 Откроется веб-страница IIS по умолчанию. ![Веб-страница IIS по умолчанию][iis-default]

## <a name="clean-up"></a>Очистка

Пока кластер работает, с вас будет взиматься плата. Рекомендуем [удалить кластер](./service-fabric-tutorial-delete-cluster.md).

## <a name="next-steps"></a>Дальнейшие действия

В этом кратком руководстве рассматривались следующие темы:

* упаковка контейнера образов Docker;
* настройка обмена данными;
* создание и упаковка приложений Service Fabric;
* развертывание приложения-контейнера в Azure.

Дополнительные сведения о работе с контейнерами Windows в Service Fabric см. в руководстве для приложений-контейнеров Windows.

> [!div class="nextstepaction"]
> [Создание приложения-контейнера Windows](./service-fabric-host-app-in-a-container.md)

[iis-default]: ./media/service-fabric-quickstart-containers/iis-default.png
[publish-dialog]: ./media/service-fabric-quickstart-containers/publish-dialog.png
[new-service]: ./media/service-fabric-quickstart-containers/NewService.png
