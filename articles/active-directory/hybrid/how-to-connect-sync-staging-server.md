---
title: 'Синхронизация Azure AD Connect: рабочие задачи и рекомендации | Документация Майкрософт'
description: В этой статье описываются рабочие задачи служб синхронизации Azure AD Connect и подготовка к работе с этим компонентом.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: ''
ms.assetid: b29c1790-37a3-470f-ab69-3cee824d220d
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/27/2018
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 48584fa4042cf53fa1084e519dca0e64f530ca59
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90090131"
---
# <a name="azure-ad-connect-staging-server-and-disaster-recovery"></a>Azure AD Connect: промежуточный сервер и аварийное восстановление
Когда сервер работает в промежуточном режиме, вы можете вносить изменения в конфигурацию и просматривать их, прежде чем активировать сервер. Этот режим также позволяет запустить полный импорт и полную синхронизацию, чтобы проверить, все ли изменения имеют нужный вид, прежде чем вносить их в рабочую среду.

## <a name="staging-mode"></a>Промежуточный режим
Промежуточный режим может использоваться в нескольких ситуациях, например:

* обеспечение высокой доступности; Каждый набор масштабирования помещает свои виртуальные машины в группу доступности с 5 доменами сбоя (FD) и 5 доменами обновления (UD) для обеспечения доступности (дополнительные сведения о доменах сбоя и обновления см.
* тестирование и развертывание новых изменений в конфигурации;
* введение в эксплуатацию нового сервера и списание старого.

Во время установки для сервера можно выбрать **промежуточный режим**. В таком случае сервер будет выполнять импорт и синхронизацию, но не будет выполнять операции экспорта. Сервер в промежуточном режиме не выполняет синхронизацию и обратную запись паролей, даже если эти функции активированы при установке. При отключении промежуточного режима сервер начнет выполнять экспорт, а также синхронизацию и обратную запись паролей.

> [!NOTE]
> Предположим, что у вас установлена служба Azure AD Connect с включенной функцией синхронизации хэшей паролей. При включении промежуточного режима сервер останавливает синхронизацию изменений паролей из локального каталога AD. При отключении промежуточного режима сервер возобновляет синхронизацию изменений паролей с того места, на котором она была остановлена. Если сервер останется в промежуточном режиме в течение продолжительного периода времени, то ему может потребоваться некоторое время, чтобы синхронизировать все изменения паролей, произошедшие за этот период.
>
>

Вы по-прежнему можете выполнить экспорт принудительно с помощью Synchronization Service Manager.

Сервер в промежуточном режиме по-другому принимает изменения от Active Directory и Azure AD и может быстро перехватить обязанности другого сервера в случае сбоя. При внесении изменений в конфигурацию сервера-источника обязательно внесите те же изменения в конфигурацию серверов в промежуточном режиме.

Тем, кто знаком с более ранними технологиями синхронизации, следует помнить об отличиях промежуточного режима, поскольку сервер имеет собственную базу данных SQL. Благодаря такой архитектуре сервер в промежуточном режиме может располагаться в другом центре обработки данных.

### <a name="verify-the-configuration-of-a-server"></a>Проверка конфигурации сервера
Чтобы использовать этот метод, выполните следующие действия.

1. [Подготовка](#prepare)
2. [Конфигурация](#configuration)
3. [Импорт и синхронизация](#import-and-synchronize)
4. [Проверка](#verify)
5. [Переключение активного сервера](#switch-active-server)

#### <a name="prepare"></a>Подготовка
1. Установите Azure AD Connect, выберите **промежуточный режим** и снимите флажок **Запустить синхронизацию** на последней странице мастера установки. В этом режиме можно вручную запустить модуль синхронизации.
   ![На снимке экрана показана страница все готово для настройки в диалоговом окне Azure AD Connect.](./media/how-to-connect-sync-staging-server/readytoconfigure.png)
2. Выйдите из системы и снова войдите в нее, а затем в меню "Пуск" выберите пункт **Служба синхронизации**.

#### <a name="configuration"></a>Конфигурация
Если вы внесли какие-либо изменения на сервере-источнике и хотите сравнить конфигурацию с промежуточным сервером, то воспользуйтесь [средством документирования конфигураций Azure AD Connect](https://github.com/Microsoft/AADConnectConfigDocumenter).

#### <a name="import-and-synchronize"></a>Импорт и синхронизация
1. Выберите пункт **Соединители**, а затем выберите первый соединитель с типом **Доменные службы Active Directory**. Щелкните **Запуск**, выберите **Полный импорт** и нажмите кнопку **ОК**. Проделайте это для всех соединителей данного типа.
2. Выберите соединитель типа **Azure Active Directory (Майкрософт)**. Щелкните **Запуск**, выберите **Полный импорт** и нажмите кнопку **ОК**.
3. Убедитесь, что все еще выбрана вкладка "Соединители". Для каждого соединителя типа **Доменные службы Active Directory** щелкните **Запуск**, выберите **Синхронизация изменений** и нажмите кнопку **ОК**.
4. Выберите соединитель типа **Azure Active Directory (Майкрософт)**. Щелкните **Запуск**, выберите **Синхронизация изменений**) и нажмите кнопку **ОК**.

Теперь изменения экспорта размещены в Azure AD и локальном каталоге AD (если используется гибридное развертывание Exchange). С помощью описанных далее действий вы сможете проверить предстоящие изменения, прежде чем фактически запустить экспорт в каталоги.

#### <a name="verify"></a>Проверка
1. Откройте командную строку и перейдите в каталог `%ProgramFiles%\Microsoft Azure AD Sync\bin`.
2. Выполните команду `csexport "Name of Connector" %temp%\export.xml /f:x`. Имя соединителя можно найти в службе синхронизации. Он имеет имя, похожее на "contoso.com — Azure AD" для Azure AD.
3. Выполните команду `CSExportAnalyzer %temp%\export.xml > %temp%\export.csv`. Теперь в папке %temp% есть файл export.csv, который можно просмотреть в Microsoft Excel. Этот файл содержит все изменения, которые будут экспортированы.
4. Внесите необходимые изменения в данные и конфигурацию и выполните описанные выше действия (импорт, синхронизация и проверка) повторно, чтобы привести изменения, которые предстоит экспортировать, в нужный вид.

**Изучите файл export.csv**. Большая часть файла не нуждается в объяснении. Вот некоторые сокращения, которые помогут понять содержимое.
* OMODT — тип изменения объекта (от англ. Object Modification Type). Указывает, к какому типу относится операция на уровне объекта: Add (добавление), Update (обновление) или Delete (удаление).
* AMODT — тип изменения атрибута (от англ. Attribute Modification Type). Указывает, к какому типу относится операция на уровне атрибута: Add (добавление), Update (обновление) или Delete (удаление).

**Получите общие идентификаторы**. Файл export.csv содержит все изменения, которые необходимо экспортировать. Каждая строка соответствует изменению для объекта в пространстве соединителя, и этот объект определяется атрибутом различающегося имени. Атрибут различающегося имени — это уникальный идентификатор, назначаемый объекту в пространстве соединителя. Если в файле export.csv имеется большое количество строк и изменений для анализа, может быть сложно понять, какие объекты, для которых указаны изменения, основываются только на атрибуте различающегося имени. Чтобы упростить процесс анализа изменений, используйте сценарий PowerShell csanalyzer.ps1. Этот сценарий получает общие идентификаторы объектов (например, displayName, userPrincipalName). Использование сценария
1. Скопируйте сценарий PowerShell из раздела [CSAnalyzer](#appendix-csanalyzer) в файл `csanalyzer.ps1`.
2. Откройте окно PowerShell и перейдите в папку, в которой вы создали сценарий PowerShell.
3. Выполните команду `.\csanalyzer.ps1 -xmltoimport %temp%\export.xml`.
4. Теперь у вас есть файл **processedusers1.csv**, который можно просмотреть в Microsoft Excel. Обратите внимание на то, что файл обеспечивает сопоставление атрибута различающегося имени с общими идентификаторами (например, displayName и userPrincipalName). В настоящее время он не включает в себя фактические изменения атрибутов, которые должны быть экспортированы.

#### <a name="switch-active-server"></a>Переключение активного сервера
1. Отключите текущий сервер (DirSync, FIM или Azure AD Sync), чтобы он не выполнял экспорт в Azure AD, или переведите его в промежуточный режим (Azure AD Connect).
2. Запустите мастер установки на сервере в **промежуточном режиме** и отключите **промежуточный режим**.
   ![ReadyToConfigure](./media/how-to-connect-sync-staging-server/additionaltasks.png)

## <a name="disaster-recovery"></a>Аварийное восстановление
Реализация предусматривает возможность планирования действий в случае потери подключения к серверу синхронизации. Вы можете воспользоваться одной из нескольких моделей. Выбор зависит от ответов на следующие вопросы:

* Насколько приемлема для вас ситуация, когда вы не можете вносить изменения в объекты в Azure AD во время простоя?
* Если вы используете синхронизацию паролей, согласятся ли пользователи использовать старые пароли в Azure AD в случае их изменения в локальной среде?
* Зависите ли вы от выполняемых в реальном времени операций, таких как обратная запись паролей?

В зависимости от политики вашей организации и ответов на эти вопросы можно реализовать одну из следующих стратегий:

* Восстановление при необходимости.
* Наличие запасного резервного сервера ( **промежуточный режим**).
* Использование виртуальных машин.

Если встроенная база данных SQL Express не используется, просмотрите раздел [Высокий уровень доступности сервера SQL](#sql-high-availability) .

### <a name="rebuild-when-needed"></a>Восстановление при необходимости
Одна из эффективных стратегий заключается в возможности восстановления сервера при необходимости. Обычно установка модуля синхронизации и выполнение начального импорта и синхронизации занимают несколько часов. Если нет резервного сервера, модуль синхронизации можно временно разместить на контроллере домена.

Сервер модуля синхронизации не сохраняет состояние объектов, поэтому базу данных можно восстановить из данных в Active Directory и Azure AD. Для объединения объектов из локальной среды и облака используется атрибут **sourceAnchor** . Если восстанавливается сервер с имеющимися объектами в локальной среде и в облаке, то при повторной установке модуль синхронизации повторно сопоставляет эти объекты. Изменения в конфигурации сервера, в частности изменения в правилах фильтрации и синхронизации, необходимо задокументировать и сохранить. Эти пользовательские конфигурации необходимо применить повторно перед началом синхронизации.

### <a name="have-a-spare-standby-server---staging-mode"></a>Наличие запасного резервного сервера (промежуточный режим)
В более сложной среде рекомендуется использовать один или несколько резервных серверов. Во время установки один из серверов можно перевести в **промежуточный режим**.

Дополнительные сведения см. в разделе [промежуточный режим](#staging-mode).

### <a name="use-virtual-machines"></a>Использование виртуальных машин
Также поддерживается распространенный метод, предусматривающий работу модуля синхронизации на виртуальной машине. Если на узле возникнут проблемы, образ сервера с модулем синхронизации можно будет перенести на другой сервер.

### <a name="sql-high-availability"></a>Высокий уровень доступности сервера SQL
Если вы не используете выпуск SQL Server Express, входящий в Azure AD Connect, вам следует также рассмотреть обеспечение высокого уровня доступности для SQL Server. Поддерживаются такие решения высокого уровня доступности, как кластеры SQL и группы доступности AlwaysOn. Такие решения, как зеркальное отображение, не поддерживаются.

Поддержка групп доступности AlwaysOn SQL добавлена в Azure AD Connect в версии 1.1.524.0. Перед установкой Azure AD Connect включите функцию SQL AOA. При установке Azure AD Connect определяет, включена ли в указанном экземпляре SQL функция SQL AOA. Если функция SQL AOA включена, то Azure AD Connect определяет, какая в SQL AOA настроена репликация (синхронная или асинхронная). При настройке прослушивателя группы доступности рекомендуется задать свойству RegisterAllProvidersIP значение 0. Это требуется то той причине, что для подключения к SQL служба Azure AD Connect сейчас использует SQL Native Client, а SQL Native Client не поддерживает свойство MultiSubNetFailover.

## <a name="appendix-csanalyzer"></a>Приложение CSAnalyzer
Сведения об использовании этого сценария см. в разделе [Проверка](#verify).

```
Param(
    [Parameter(Mandatory=$true, HelpMessage="Must be a file generated using csexport 'Name of Connector' export.xml /f:x)")]
    [string]$xmltoimport="%temp%\exportedStage1a.xml",
    [Parameter(Mandatory=$false, HelpMessage="Maximum number of users per output file")][int]$batchsize=1000,
    [Parameter(Mandatory=$false, HelpMessage="Show console output")][bool]$showOutput=$false
)

#LINQ isn't loaded automatically, so force it
[Reflection.Assembly]::Load("System.Xml.Linq, Version=3.5.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089") | Out-Null

[int]$count=1
[int]$outputfilecount=1
[array]$objOutputUsers=@()

#XML must be generated using "csexport "Name of Connector" export.xml /f:x"
write-host "Importing XML" -ForegroundColor Yellow

#XmlReader.Create won't properly resolve the file location,
#so expand and then resolve it
$resolvedXMLtoimport=Resolve-Path -Path ([Environment]::ExpandEnvironmentVariables($xmltoimport))

#use an XmlReader to deal with even large files
$result=$reader = [System.Xml.XmlReader]::Create($resolvedXMLtoimport) 
$result=$reader.ReadToDescendant('cs-object')
do 
{
    #create the object placeholder
    #adding them up here means we can enforce consistency
    $objOutputUser=New-Object psobject
    Add-Member -InputObject $objOutputUser -MemberType NoteProperty -Name ID -Value ""
    Add-Member -InputObject $objOutputUser -MemberType NoteProperty -Name Type -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name DN -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name operation -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name UPN -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name displayName -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name sourceAnchor -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name alias -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name primarySMTP -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name onPremisesSamAccountName -Value ""
    Add-Member -inputobject $objOutputUser -MemberType NoteProperty -Name mail -Value ""

    $user = [System.Xml.Linq.XElement]::ReadFrom($reader)
    if ($showOutput) {Write-Host Found an exported object... -ForegroundColor Green}

    #object id
    $outID=$user.Attribute('id').Value
    if ($showOutput) {Write-Host ID: $outID}
    $objOutputUser.ID=$outID

    #object type
    $outType=$user.Attribute('object-type').Value
    if ($showOutput) {Write-Host Type: $outType}
    $objOutputUser.Type=$outType

    #dn
    $outDN= $user.Element('unapplied-export').Element('delta').Attribute('dn').Value
    if ($showOutput) {Write-Host DN: $outDN}
    $objOutputUser.DN=$outDN

    #operation
    $outOperation= $user.Element('unapplied-export').Element('delta').Attribute('operation').Value
    if ($showOutput) {Write-Host Operation: $outOperation}
    $objOutputUser.operation=$outOperation

    #now that we have the basics, go get the details

    foreach ($attr in $user.Element('unapplied-export-hologram').Element('entry').Elements("attr"))
    {
        $attrvalue=$attr.Attribute('name').Value
        $internalvalue= $attr.Element('value').Value

        switch ($attrvalue)
        {
            "userPrincipalName"
            {
                if ($showOutput) {Write-Host UPN: $internalvalue}
                $objOutputUser.UPN=$internalvalue
            }
            "displayName"
            {
                if ($showOutput) {Write-Host displayName: $internalvalue}
                $objOutputUser.displayName=$internalvalue
            }
            "sourceAnchor"
            {
                if ($showOutput) {Write-Host sourceAnchor: $internalvalue}
                $objOutputUser.sourceAnchor=$internalvalue
            }
            "alias"
            {
                if ($showOutput) {Write-Host alias: $internalvalue}
                $objOutputUser.alias=$internalvalue
            }
            "proxyAddresses"
            {
                if ($showOutput) {Write-Host primarySMTP: ($internalvalue -replace "SMTP:","")}
                $objOutputUser.primarySMTP=$internalvalue -replace "SMTP:",""
            }
        }
    }

    $objOutputUsers += $objOutputUser

    Write-Progress -activity "Processing ${xmltoimport} in batches of ${batchsize}" -status "Batch ${outputfilecount}: " -percentComplete (($objOutputUsers.Count / $batchsize) * 100)

    #every so often, dump the processed users in case we blow up somewhere
    if ($count % $batchsize -eq 0)
    {
        Write-Host Hit the maximum users processed without completion... -ForegroundColor Yellow

        #export the collection of users as a CSV
        Write-Host Writing processedusers${outputfilecount}.csv -ForegroundColor Yellow
        $objOutputUsers | Export-Csv -path processedusers${outputfilecount}.csv -NoTypeInformation

        #increment the output file counter
        $outputfilecount+=1

        #reset the collection and the user counter
        $objOutputUsers = $null
        $count=0
    }

    $count+=1

    #need to bail out of the loop if no more users to process
    if ($reader.NodeType -eq [System.Xml.XmlNodeType]::EndElement)
    {
        break
    }

} while ($reader.Read)

#need to write out any users that didn't get picked up in a batch of 1000
#export the collection of users as CSV
Write-Host Writing processedusers${outputfilecount}.csv -ForegroundColor Yellow
$objOutputUsers | Export-Csv -path processedusers${outputfilecount}.csv -NoTypeInformation
```

## <a name="next-steps"></a>Дальнейшие действия
**Обзорные статьи**  

* [Синхронизация Azure AD Connect: общие сведений о синхронизации и ее настройка](how-to-connect-sync-whatis.md)  
* [Интеграция локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md)  
