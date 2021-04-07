---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: c0a0b44b3de088539a070f1182248c4e6db7303f
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97510283"
---
## <a name="create-a-self-hosted-integration-runtime"></a>Создание локальной среды выполнения интеграции

В этом разделе вы создадите локальную среду выполнения интеграции и свяжете ее с локальным компьютером, на котором находится база данных SQL Server. Локальная среда выполнения интеграции — это компонент, который копирует данные с SQL Server на компьютере в Базу данных SQL Azure. 

1. Создайте переменную для имени среды выполнения интеграции. Используйте уникальное имя и запишите его. Оно будет использоваться далее в этом руководстве. 

    ```powershell
   $integrationRuntimeName = "ADFTutorialIR"
    ```
2. Создайте локальную среду выполнения интеграции. 

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -Name $integrationRuntimeName -Type SelfHosted -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName
   ```

   Пример выходных данных:

   ```console
    Name              : <Integration Runtime name>
    Type              : SelfHosted
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Description       : 
    Id                : /subscriptions/<subscription ID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.DataFactory/factories/<DataFactoryName>/integrationruntimes/ADFTutorialIR
    ```
  
3. Чтобы получить состояние созданной среды выполнения интеграции, выполните следующую команду. Убедитесь, что свойству **State** присвоено значение **NeedRegistration**. 

   ```powershell
   Get-AzDataFactoryV2IntegrationRuntime -name $integrationRuntimeName -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Status
   ```

   Пример выходных данных:

   ```console  
   State                     : NeedRegistration
   Version                   : 
   CreateTime                : 9/24/2019 6:00:00 AM
   AutoUpdate                : On
   ScheduledUpdateDate       : 
   UpdateDelayOffset         : 
   LocalTimeZoneOffset       : 
   InternalChannelEncryption : 
   Capabilities              : {}
   ServiceUrls               : {eu.frontend.clouddatahub.net}
   Nodes                     : {}
   Links                     : {}
   Name                      : ADFTutorialIR
   Type                      : SelfHosted
   ResourceGroupName         : <ResourceGroup name>
   DataFactoryName           : <DataFactory name>
   Description               : 
   Id                        : /subscriptions/<subscription ID>/resourceGroups/<ResourceGroup name>/providers/Microsoft.DataFactory/factories/<DataFactory name>/integrationruntimes/<Integration Runtime name>
   ```

4. Чтобы получить ключи проверки подлинности для регистрации локальной среды выполнения интеграции в службе фабрики данных Azure в облаке, выполните следующую команду: 

   ```powershell
   Get-AzDataFactoryV2IntegrationRuntimeKey -Name $integrationRuntimeName -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName | ConvertTo-Json
   ```

   Пример выходных данных:

   ```json
   {
    "AuthKey1": "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=",
    "AuthKey2":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy="
   }
   ```    

5. Скопируйте один из ключей (без двойных кавычек), используемый для регистрации локальной среды выполнения интеграции, которую вы установите на компьютер на следующих шагах.  

## <a name="install-the-integration-runtime-tool"></a>Установка средства среды выполнения интеграции

1. Если на вашем компьютере уже установлена среда выполнения интеграции, удалите ее в разделе **Установка и удаление программ**. 

2. [Скачайте](https://www.microsoft.com/download/details.aspx?id=39717) локальную среду выполнения интеграции на локальный компьютер под управлением Windows. Запустите установку.

3. На странице **приветствия мастера установки Microsoft Integration Runtime** нажмите кнопку **Далее**.

4. На странице **Лицензионное соглашение** примите условия использования и лицензионное соглашение и нажмите кнопку **Далее**.

5. На странице **Конечная папка** нажмите кнопку **Далее**.

6. На странице **Ready to install Microsoft Integration Runtime** (Все готово для установки Microsoft Integration Runtime) нажмите кнопку **Установить**.

7. На странице **Completed the Microsoft Integration Runtime Setup** (Мастер установки Microsoft Integration Runtime завершил работу) нажмите кнопку **Готово**.

8. На странице **Регистрация Integration Runtime (Self-hosted)** вставьте ключ, созданный в предыдущем разделе, и выберите **Зарегистрировать**. 

    ![Регистрация среды выполнения интеграции](media/data-factory-create-install-integration-runtime/register-integration-runtime.png)

9. На странице **Новый узел среды выполнения интеграции (с локальным размещением)** нажмите кнопку **Finish** (Завершить) 

10. Когда локальная среда выполнения интеграции будет успешно зарегистрирована, вы увидите следующее сообщение:

    ![Успешно зарегистрирована](media/data-factory-create-install-integration-runtime/registered-successfully.png)

14. На странице **Регистрация Integration Runtime (Self-hosted)** выберите **Запустить Configuration Manager**.

15. Когда узел будет подключен к облачной службе, отобразится следующая страница:

    ![Страница Node is connected (Узел подключен)](media/data-factory-create-install-integration-runtime/node-is-connected.png)

16. Теперь проверьте возможность подключения к базе данных SQL Server.

    ![Вкладка «Диагностика»](media/data-factory-create-install-integration-runtime/config-manager-diagnostics-tab.png)   

    а. На странице **Configuration Manager** перейдите на вкладку **Diagnostics** (Диагностика).

    b. В качестве типа источника данных выберите **SqlServer**.

    c. Введите имя сервера.

    d. Введите имя базы данных.

    д) Выберите режим проверки подлинности.

    е) Введите имя пользователя.

    ж. Введите пароль, связанный с именем пользователя.

    h. Нажмите кнопку **проверки**, чтобы убедиться, что эта среда выполнения интеграции может подключаться к серверу SQL Server. Если подключение установлено успешно, появится зеленый флажок. Если подключение не установлено, появится сообщение об ошибке. Исправьте ошибки и проверьте, может ли среда выполнения интеграции подключаться к SQL Server.    

    > [!NOTE]
    > Запишите значения типа проверки подлинности, сервера, базы данных, пользователя и пароль. Они будут использоваться далее в этом руководстве.
