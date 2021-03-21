---
title: Развертывание виртуальных машин Amazon Web Services с помощью модуля runbook службы автоматизации Azure
description: В этой статье рассказывается, как автоматизировать создание виртуальных машин Amazon Web Services.
services: automation
ms.subservice: process-automation
ms.date: 03/16/2018
ms.topic: conceptual
ms.openlocfilehash: 03f1f1659d120a2aa1b827063cb2bc84138f3655
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98896076"
---
# <a name="deploy-an-amazon-web-services-vm-with-a-runbook"></a>Развертывание виртуальных машин Amazon Web Services с помощью модуля runbook

В этой статье вы узнаете, как с помощью службы автоматизации Azure подготовить виртуальную машину в подписке Amazon Web Service (AWS) и присвоить виртуальной машине конкретное имя, которое в AWS называется "тегом" виртуальной машины.

## <a name="prerequisites"></a>Предварительные требования

Для выполнения инструкций из этой статьи вам понадобится учетная запись службы автоматизации Azure и подписка Amazon Web Services (AWS). Дополнительные сведения о настройке учетной записи службы автоматизации Azure и об указании в ней учетных данных подписки AWS см. в статье [Проверка подлинности модулей Runbook с помощью Amazon Web Services](automation-config-aws-account.md). Перед выполнением дальнейших действий необходимо создать эту учетную запись с учетными данными подписки AWS или указать в ней эти учетные данные, так как вы будете ссылаться на нее при выполнении действий в разделах ниже.

## <a name="deploy-amazon-web-services-powershell-module"></a>Развертывание модуля PowerShell для Amazon Web Services

В процессе работы ваш модуль runbook для подготовки виртуальной машины использует модуль PowerShell для AWS. Выполните следующие действия, чтобы добавить в учетную запись службы автоматизации модуль, настроенный с учетными данными подписки AWS.  

1. Откройте браузер и перейдите к [коллекции PowerShell](https://www.powershellgallery.com/packages/AWSPowerShell/), а затем нажмите кнопку **Deploy to Azure Automation** (Развернуть в службе автоматизации Azure).<br><br> ![Импорт модуля AWS PS](./media/automation-scenario-aws-deployment/powershell-gallery-download-awsmodule.png)
2. Вы перейдете на страницу входа в Azure. После проверки подлинности откроется следующая страница портала Azure.<br><br> ![Страница импорта модуля](./media/automation-scenario-aws-deployment/deploy-aws-powershell-module-parameters.png)
3. Выберите учетную запись службы автоматизации и нажмите кнопку **OK** для запуска развертывания.

   > [!NOTE]
   > Когда служба автоматизации Azure импортирует модуль PowerShell, она извлекает командлеты. Действия не отображаются, пока служба автоматизации не завершит импорт модуля и извлечение командлетов полностью. Это может занять несколько минут.  
   > <br>

1. На портале Azure выберите свою учетную запись службы автоматизации.
2. Щелкните плитку **Ресурсы** и на странице "Ресурсы" выберите **Модули**.
3. На странице "Модули" вы увидите в списке модуль **AWSPowerShell**.

## <a name="create-aws-deploy-vm-runbook"></a>Создание модуля Runbook для развертывания виртуальной машины в AWS

После развертывания модуля AWS PowerShell можно создать модуль Runbook для автоматизации подготовки виртуальной машины в AWS с помощью сценария PowerShell. Приведенные ниже шаги показывают, как использовать собственный сценарий PowerShell в службе автоматизации Azure.  

> [!NOTE]
> Дополнительные параметры и сведения, касающиеся этого сценария, можно найти в [коллекции PowerShell](https://www.powershellgallery.com/packages/New-AwsVM/).
> 

1. Загрузите сценарий PowerShell New-AwsVM из коллекции PowerShell, открыв сеанс PowerShell и введя следующую команду:<br>
   ```powershell
   Save-Script -Name New-AwsVM -Path <path>
   ```
   <br>
2. На портале Azure откройте учетную запись автоматизации и выберите **Модули runbook** в разделе **Автоматизация процессов**.  
3. На странице "Модули runbook" выберите **Добавить runbook**.
4. На странице "Добавить runbook" выберите **Быстрое создание** для создания нового модуля runbook.
5. В области "Свойства runbook" введите имя модуля runbook.
6. В раскрывающемся списке **Тип runbook** выберите **PowerShell**, а затем щелкните **Создать**.<br><br> ![Панель создания модуля Runbook](./media/automation-scenario-aws-deployment/runbook-quickcreate-properties.png)
7. При открытии страницы "Изменение сценария PowerShell для модуля Runbook" скопируйте и вставьте сценарий PowerShell в область создания модуля Runbook.<br><br> ![Сценарий PowerShell для модуля Runbook](./media/automation-scenario-aws-deployment/runbook-powershell-script.png)<br>
   
    При работе с примером сценария PowerShell имейте в виду следующее:

    * Модуль Runbook содержит несколько параметров по умолчанию. Проверьте все значения по умолчанию и при необходимости обновите их.
    * Если вы сохранили учетные данные AWS как ресурс учетных данных с именем, отличным от `AWScred`, потребуется соответствующим образом обновить сценарий в строке 57.  
    * При работе с командами AWS в PowerShell, особенно в этом примере модуля Runbook, необходимо указывать регион AWS. В противном случае командлеты завершатся с ошибкой. Дополнительные сведения на сайте AWS см. в разделе [Specify AWS Region](https://docs.aws.amazon.com/powershell/latest/userguide/pstools-installing-specifying-region.html) (Указание региона AWS) документации "AWS Tools for Windows PowerShell" (Инструменты AWS для Windows PowerShell).  

8. Чтобы получить список имен образов из подписки AWS, запустите PowerShell ISE и импортируйте модуль AWS PowerShell. Аутентификация в AWS путем замены `Get-AutomationPSCredential` в среде ISE на `AWScred = Get-Credential`. Этот оператор выдаст запрос на ввод учетных данных. Идентификатор ключа доступа можно указать в качестве имени пользователя, а секретный ключ доступа в качестве пароля. 

      ```powershell
        #Sample to get the AWS VM available images
        #Please provide the path where you have downloaded the AWS PowerShell module
        Import-Module AWSPowerShell
        $AwsRegion = "us-west-2"
        $AwsCred = Get-Credential
        $AwsAccessKeyId = $AwsCred.UserName
        $AwsSecretKey = $AwsCred.GetNetworkCredential().Password
   
        # Set up the environment to access AWS
        Set-AwsCredentials -AccessKey $AwsAccessKeyId -SecretKey $AwsSecretKey -StoreAs AWSProfile
        Set-DefaultAWSRegion -Region $AwsRegion
   
        Get-EC2ImageByName -ProfileName AWSProfile
      ```
        
    Возвращается следующий результат:<br><br>
   ![Получение образов AWS](./media/automation-scenario-aws-deployment/powershell-ise-output.png)<br>  
9. Скопируйте и вставьте имя одного из образов в переменную службы автоматизации, которая в модуле runbook указана как `$InstanceType`. Так как в этом примере используется подписка AWS уровня "Бесплатный", в примере модуля runbook нужно использовать **t2.micro**.  
10. Сохраните модуль Runbook, нажмите кнопку **Опубликовать**, чтобы опубликовать модуль Runbook, а затем кнопку **Да** в появившемся запросе.

### <a name="test-the-aws-vm-runbook"></a>Тестирование модуля runbook виртуальной машины AWS

1. Убедитесь, что модуль runbook создает ресурс с именем `AWScred` для проверки подлинности в AWS или обновите сценарий, чтобы он ссылался на имя ресурса учетных данных.    
2. Проверьте новый модуль runbook и убедитесь, что все необходимые значения параметров были обновлены.
Убедитесь, что модуль PowerShell для AWS импортирован в службу автоматизации Azure.  
3. В службе автоматизации Azure для параметров **Подробные записи в журнале** и **Записи ведения журнала** в операции **Ведение журналов и трассировка** модуля runbook задайте значение **Включено**.<br><br> ![Ведение журнала и трассировка для модуля runbook](./media/automation-scenario-aws-deployment/runbook-settings-logging-and-tracing.png).  
4. Чтобы запустить модуль Runbook, щелкните **Запустить**, а затем, когда откроется страница "Запуск Runbook", нажмите кнопку **OK**.
5. На странице "Запуск модуля Runbook" укажите имя виртуальной машины. Примите значения по умолчанию для остальных параметров, которые были указаны в сценарии. Нажмите кнопку **ОК**, чтобы запустить задание Runbook.<br><br> ![Запуск модуля Runbook: New-AwsVM](./media/automation-scenario-aws-deployment/runbook-start-job-parameters.png)
6. Откроется область заданий с созданным заданием runbook. Закройте эту область.
7. Ход выполнения задания и потоки вывода можно просмотреть, выбрав плитку **Все журналы** на странице задания runbook.<br><br> ![Раздел "Поток": выходные данные](./media/automation-scenario-aws-deployment/runbook-job-streams-output.png)
8. Чтобы убедиться, что виртуальная машина подготавливается, войдите в консоль управления AWS, если вы еще не вошли в систему.<br><br> ![Консоль AWS: развернутая виртуальная машина](./media/automation-scenario-aws-deployment/aws-instances-status.png)

## <a name="next-steps"></a>Дальнейшие действия
 
* Чтобы узнать, какие модули runbook поддерживаются, см. статью [Типы runbook службы автоматизации Azure](automation-runbook-types.md).
* Сведения о работе с модулями runbook см. в статье [Управление модулями runbook в службе автоматизации Azure](manage-runbooks.md).
* Дополнительные сведения о PowerShell см. в [документации PowerShell](/powershell/scripting/overview).
* Сведения о поддержке сценариев см. в [Встроенная поддержка скриптов PowerShell в службе автоматизации Azure](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/).
* Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
