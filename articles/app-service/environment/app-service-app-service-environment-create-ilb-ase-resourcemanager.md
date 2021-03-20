---
title: Создание ILB ASE v1
description: Создайте среду службы приложений с внутренней подсистемой балансировки нагрузки (ILB ASE). Этот документ предоставляется только для клиентов, использующих устаревшую версию ASE (версию 1).
author: stefsch
ms.assetid: 091decb6-b0de-42a1-9f2f-c18d9b2e67df
ms.topic: article
ms.date: 07/11/2017
ms.author: stefsch
ms.custom: seodec18
ms.openlocfilehash: 2a03b791f37868010e107214ddcb7cf42174e4e1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "85833559"
---
# <a name="how-to-create-an-ilb-ase-using-azure-resource-manager-templates"></a>Создание среды службы приложений с внутренним балансировщиком нагрузки с помощью шаблонов Azure Resource Manager

> [!NOTE] 
> Эта статья посвящена среде службы приложений версии 1. Имеется более новая версия среды службы приложений, которая проще в использовании и которая работает на более мощной инфраструктуре. Чтобы узнать больше о новой версии, начните с [введения в среда службы приложений](intro.md).
>

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Обзор
Вместо общедоступного виртуального IP-адреса для создания среды службы приложений (ASE) можно использовать внутренний адрес виртуальной сети.  Этот внутренний адрес принадлежит компоненту Azure, который называется внутренним балансировщиком нагрузки.  Среду ASE с внутренним балансировщиком нагрузки можно создать на портале Azure.  Ее также можно создать автоматически с помощью шаблонов Azure Resource Manager.  В этой статье описаны действия и синтаксис, необходимые для создания ASE с внутренним балансировщиком нагрузки с помощью шаблонов Azure Resource Manager.

Автоматическое создание среды ASE с внутренним балансировщиком нагрузки предусматривает три шага:

1. Сначала в виртуальной сети создается базовая среда ASE, для чего вместо общедоступного виртуального IP-адреса используется адрес внутреннего балансировщика нагрузки.  Кроме того, на этом шаге ASE с внутренним балансировщиком нагрузки присваивается имя корневого домена.
2. После создания ASE ILB отправляется сертификат TLS/SSL.  
3. Переданный сертификат TLS/SSL явным образом назначается ILB ASE в качестве сертификата TLS/SSL по умолчанию.  Этот сертификат TLS/SSL будет использоваться для TLS-трафика в приложения на ILB ASE, когда приложения обрабатываются с помощью общего корневого домена, назначенного ASE (например, `https://someapp.mycustomrootcomain.com` ).

## <a name="creating-the-base-ilb-ase"></a>Создание базовой среды ASE с внутренним балансировщиком нагрузки
Пример шаблона Azure Resource Manager и соответствующий файл параметров доступны на сайте GitHub [здесь][quickstartilbasecreate].

Большинство параметров в файле *azuredeploy.parameters.json* используются при создании как ASE с внутренним балансировщиком нагрузки, так и ASE с привязкой к общедоступному виртуальному IP-адресу.  Ниже приведены параметры с особыми примечаниями и уникальные параметры, используемые при создании ASE с внутренним балансировщиком нагрузки.

* *интерналлоадбаланЦингмоде*. в большинстве случаев это значение равно 3. Это означает, что трафик HTTP/HTTPS на портах 80/443, а также порты канала элемента управления и данных, прослушиваемые службой FTP на ASE, будут привязаны к внутреннему виртуальному адресу виртуальной сети, выделенному ilB.  Если задать для этого свойства значение 2, к адресу внутреннего балансировщика нагрузки будут привязаны только порты, связанные со службой FTP (канала данных и канала управления), а HTTP- и HTTP-трафик останется привязанным к общедоступному виртуальному IP-адресу.
* *dnsSuffix*. Этот параметр определяет корневой домен по умолчанию, который будет назначен среде службы приложений.  В общедоступной версии службы приложений Azure корневой домен по умолчанию для всех веб-приложений — *azurewebsites.net*.  Однако так как ASE с внутренним балансировщиком нагрузки является внутренней для виртуальной сети пользователя, бессмысленно использовать корневой домен по умолчанию общедоступной службы.  Вместо этого ASE с внутренним балансировщиком нагрузки необходимо назначить корневой домен по умолчанию, который целесообразно использовать в пределах внутренней виртуальной сети компании.  Например, гипотетическая компания Contoso может использовать корневой домен по умолчанию *internal-contoso.com* для приложений, которые должны быть разрешаемыми и доступными только в виртуальной сети компании Contoso. 
* *ipSslAddressCount*. Для этого параметра в файле *azuredeploy.json* автоматически задается значение 0, так как средам службы приложений с внутренним балансировщиком нагрузки назначен только адрес этого балансировщика.  Для ASE с внутренним балансировщиком нагрузки нет явно заданных адресов SSL на основе IP, поэтому для пула этих адресов необходимо задать значение 0. В противном случае произойдет ошибка подготовки. 

После того, как *azuredeploy.parameters.jsв* файле заполнится для ASEа ILB, ilB ASE можно будет создать с помощью следующего фрагмента кода PowerShell.  Измените пути к файлам в соответствии с расположением файлов шаблонов Azure Resource Manager на компьютере.  Кроме того, введите собственные значения для имени развертывания Azure Resource Manager и группы ресурсов.

```azurepowershell-interactive
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

После отправки шаблона Azure Resource Manager для создания ASE с внутренним балансировщиком нагрузки потребуется несколько часов.  Созданная ASE с внутренним балансировщиком нагрузки появится на портале в списке сред службы приложений для подписки, использованной для развертывания.

## <a name="uploading-and-configuring-the-default-tlsssl-certificate"></a>Отправка и Настройка сертификата TLS/SSL по умолчанию
После создания ASE ILB сертификат TLS/SSL должен быть связан с ASE как сертификат TLS/SSL по умолчанию, используемый для установки подключений TLS/SSL к приложениям.  Продолжая работу с гипотетическим примером Contoso Corporation, если DNS-суффикс ASE по умолчанию — *internal-contoso.com*, то для подключения к *`https://some-random-app.internal-contoso.com`* требуется сертификат TLS/SSL, допустимый для **. internal-contoso.com*. 

Существует множество способов получить допустимый сертификат TLS/SSL, включая внутренние центры сертификации, Купить сертификат у внешнего поставщика и использовать самозаверяющий сертификат.  Независимо от источника сертификата TLS/SSL, необходимо правильно настроить следующие атрибуты сертификата.

* *Subject* — для этого атрибута необходимо задать значение \**.имя_корневого_домена.com*.
* *Subject Alternative Name* — этот атрибут должен содержать два значения: \**.имя_корневого_домена.com* и \**.scm.имя_корневого_домена.com*.  Причина второй записи заключается в том, что TLS-подключения к сайту SCM или KUDU, связанным с каждым приложением, будут выполняться с использованием адреса формы *Your-App-Name.SCM.Your-root-Domain-here.com*.

Используя действительный сертификат TLS/SSL, необходимо выполнить два дополнительных подготовительных действия.  Сертификат TLS/SSL необходимо преобразовать или сохранить в виде PFX-файла.  PFX-файл должен содержать все промежуточные и корневые сертификаты, а также быть защищен паролем.

Затем файл resultd. PFX необходимо преобразовать в строку Base64, так как сертификат TLS/SSL будет отправлен с помощью шаблона Azure Resource Manager.  Эти шаблоны представлены в виде текстовых файлов. Такое преобразование требуется, чтобы добавить PFX-файл в качестве параметра шаблона.

В следующем фрагменте кода PowerShell приведен пример создания самозаверяющего сертификата, экспорта сертификата в виде PFX-файла, преобразования PFX-файла в строку в кодировке Base64 и последующего сохранения строки в кодировке Base64 в отдельный файл.  Код PowerShell для кодирования Base64 был адаптирован из [блога сценариев PowerShell][examplebase64encoding].

```azurepowershell-interactive
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.pfx"
Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password     

$fileContentBytes = get-content -encoding byte $fileName
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
$fileContentEncoded | set-content ($fileName + ".b64")
```

После того как сертификат TLS/SSL успешно создан и преобразован в строку в кодировке Base64, можно использовать пример шаблона Azure Resource Manager на сайте GitHub для [настройки сертификата TLS/SSL по умолчанию][configuringDefaultSSLCertificate] .

Ниже перечислены параметры, содержащиеся в файле *azuredeploy.parameters.json* .

* *appServiceEnvironmentName* — имя настраиваемой среды службы приложений с внутренним балансировщиком нагрузки.
* *existingAseLocation* — текстовая строка, содержащая регион Azure, где развернута среда службы приложений с внутренним балансировщиком нагрузки.  Например, центрально-южная часть США.
* *pfxBlobString* — представление PFX-файла в виде строки в кодировке Base64.  Нужно скопировать строку, содержащуюся в файле exportedcert.pfx.b64, и вставить ее в качестве значения атрибута *pfxBlobString* в фрагменте кода выше.
* *пароль*. пароль, используемый для защиты PFX-файла.
* *certificateThumbprint* — отпечаток сертификата.  Если вы получаете это значение из PowerShell (например, *$Certificate. Отпечаток* из предыдущего фрагмента кода) можно использовать значение "как есть".  Однако если вы скопируете это значение в диалоговом окне сертификатов Windows, в нем нужно удалить лишние пробелы.  *CertificateThumbprint* должен выглядеть примерно так: AF3143EB61D43F6727842115BB7F17BBCECAECAE
* *certificateName* — понятный идентификатор строки для сертификата. Пользователь может выбрать его по своему усмотрению.  Имя используется как часть уникального идентификатора Azure Resource Manager для сущности *Microsoft. Web/Certificates* , представляющей сертификат TLS/SSL.  Имя **должно** заканчиваться следующим суффиксом:  \_ yourASENameHere_InternalLoadBalancingASE.  Этот суффикс сообщает порталу о том, что для обеспечения безопасности ASE с поддержкой ILB используется сертификат.

Ниже приведен сокращенный пример файла *azuredeploy.parameters.json* .

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "appServiceEnvironmentName": {
            "value": "yourASENameHere"
        },
        "existingAseLocation": {
            "value": "East US 2"
        },
        "pfxBlobString": {
            "value": "MIIKcAIBAz...snip...snip...pkCAgfQ"
        },
        "password": {
            "value": "PASSWORDGOESHERE"
        },
        "certificateThumbprint": {
            "value": "AF3143EB61D43F6727842115BB7F17BBCECAECAE"
        },
        "certificateName": {
            "value": "DefaultCertificateFor_yourASENameHere_InternalLoadBalancingASE"
        }
    }
}
```

После заполнения *azuredeploy.parameters.js* в файле можно настроить сертификат TLS/SSL по умолчанию с помощью следующего фрагмента кода PowerShell.  Измените пути к файлам в соответствии с расположением файлов шаблонов Azure Resource Manager на компьютере.  Кроме того, введите собственные значения для имени развертывания Azure Resource Manager и группы ресурсов.

```azurepowershell-interactive
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

Когда шаблон Azure Resource Manager будет отправлен, применение изменений для одного внешнего интерфейса ASE займет примерно сорок минут.  Например, для среды ASE с размером по умолчанию, содержащей два внешних интерфейса, реализация шаблона займет примерно один час и двадцать минут.  Во время выполнения шаблона нельзя масштабировать среду ASE.  

После завершения шаблона доступ к приложениям на ILB ASE может осуществляться по протоколу HTTPS, а подключения будут защищаться с помощью SSL-сертификата по умолчанию.  Сертификат TLS/SSL по умолчанию будет использоваться при адресации приложений на ILB ASE с помощью сочетания имени приложения и имени узла по умолчанию.  Например *`https://mycustomapp.internal-contoso.com`* , будет использоваться сертификат TLS/SSL по умолчанию для **. internal-contoso.com*.

Однако, как и приложения, работающие в общедоступной многоклиентской службе, разработчики могут также настраивать пользовательские имена узлов для отдельных приложений, а затем настраивать уникальные привязки сертификатов SNI TLS/SSL для отдельных приложений.  

## <a name="getting-started"></a>Начало работы
Сведения о том, как начать работу со средами службы приложений, см. в статье [Введение в среду службы приложений](app-service-app-service-environment-intro.md).

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[quickstartilbasecreate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-create/
[examplebase64encoding]: https://powershellscripts.blogspot.com/2007/02/base64-encode-file.html 
[configuringDefaultSSLCertificate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl/ 

