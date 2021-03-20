---
title: Проверка подлинности Azure Stream Analytics для Azure Data Lake Storage 1-го поколения
description: В этой статье объясняется, как использовать управляемые удостоверения для аутентификации заданий Azure Stream Analytics для вывода данных в Azure Data Lake Storage 1-го поколения.
author: enkrumah
ms.author: ebnkruma
ms.service: stream-analytics
ms.topic: how-to
ms.date: 03/16/2021
ms.custom: seodec18
ms.openlocfilehash: d5b406d260b1c6518d9227c51d84ab7eb3754329
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104598156"
---
# <a name="authenticate-stream-analytics-to-azure-data-lake-storage-gen1-using-managed-identities"></a>Проверка подлинности Stream Analytics для Azure Data Lake Storage 1-го поколения с помощью управляемых удостоверений

Azure Stream Analytics поддерживает аутентификацию с помощью управляемого удостоверения для вывода данных в Azure Data Lake Storage (ADLS) 1-го поколения. Удостоверение — это зарегистрированное в Azure Active Directory управляемое приложение, представляющее данное задание Stream Analytics и используемое для аутентификации в целевом ресурсе. Управляемые удостоверения устраняют ограничения пользовательских методов аутентификации, такие как необходимость повторной аутентификации из-за изменения пароля или после истечения срока действия пользовательских токенов (каждые 90 дней). Кроме того, управляемые удостоверения позволяют автоматизировать развертывания заданий Stream Analytics, которые выводят данные в Azure Data Lake Storage 1-го поколения.

В этой статье показано три способа включения управляемого удостоверения для задания Azure Stream Analytics, которое выводит данные в Azure Data Lake Storage 1-го поколения: с помощью портала Azure (развертывания шаблона Azure Resource Manager) и средств Azure Stream Analytics для Visual Studio.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="azure-portal"></a>Портал Azure

1. Начните с создания задания Stream Analytics или открытия имеющегося задания на портале Azure. В строке меню, расположенной в левой части экрана, выберите **управляемое удостоверение** , расположенное в разделе **Настройка**.

   ![Настройка управляемого удостоверения Stream Analytics](./media/stream-analytics-managed-identities-adls/stream-analytics-managed-identity-preview.png)

2. Установите флажок **использовать управляемое системой удостоверение** из окна, отображаемого справа. Щелкните **сохранить** в субъекте-службе, чтобы получить удостоверение задания Stream Analytics в Azure Active Directory. Жизненным циклом нового удостоверения будет управлять Azure. При удалении задания Stream Analytics Azure автоматически удаляет связанное удостоверение (то есть субъект-службу).

   Если конфигурация сохраняется, идентификатор объекта (OID) субъекта-службы отображается в качестве идентификатора субъекта, как показано ниже:

   ![Идентификатор субъекта-службы Stream Analytics](./media/stream-analytics-managed-identities-adls/stream-analytics-principal-id.png)
 
   Субъект-служба имеет то же имя, что и задание Stream Analytics. Например, если имя задания — **MyASAJob**, имя созданного субъекта-службы будет также **MyASAJob**.

3. В окне Выходные свойства приемника выходных данных ADLS 1-го поколения щелкните раскрывающийся список режим проверки подлинности и выберите пункт **управляемое удостоверение**.

4. Укажите остальные свойства. Дополнительные сведения о создании выходных данных для ADLS см. в статье [Потоковая передача данных из большого двоичного объекта службы хранилища Azure в Data Lake Storage 1-го поколения с помощью Azure Stream Analytics](../data-lake-store/data-lake-store-stream-analytics.md). Закончив, нажмите кнопку **Сохранить**.

   ![Настройка Azure Data Lake Storage](./media/stream-analytics-managed-identities-adls/stream-analytics-configure-adls.png)
 
5. Перейдите к странице обзора ADLS 1-го поколения и щелкните **Обозреватель данных**.

   ![Обзор настройки Data Lake Storage](./media/stream-analytics-managed-identities-adls/stream-analytics-adls-overview.png)

6. На панели обозревателя данных выберите **Доступ** и щелкните **Добавить** в области доступа.

   ![Настройка доступа к Data Lake Storage](./media/stream-analytics-managed-identities-adls/stream-analytics-adls-access.png)

7. В текстовом поле в области **Select user or group** (Выбор пользователя или группы) укажите имя субъекта-службы. Не забывайте, что имя субъекта-службы совпадает с именем соответствующего задания Stream Analytics. Как только вы начнете вводить имя субъекта, оно отобразится под текстовым полем. Выберите необходимое имя субъекта-службы, а затем щелкните **Выбрать**.

   ![Выбор имени субъекта-службы](./media/stream-analytics-managed-identities-adls/stream-analytics-service-principal-name.png)
 
8. В области **Разрешения** выберите разрешения **Запись** и **Выполнить** и установите переключатель **К этой папке и всем вложенным элементам**. Затем нажмите кнопку **ОК**.

   ![Выбор разрешений на запись и выполнение](./media/stream-analytics-managed-identities-adls/stream-analytics-select-permissions.png)
 
9. Субъект-служба находится в списке **Назначенные разрешения** в области **Доступ**, как показано ниже. Теперь вы можете вернуться назад и приступить к заданию Stream Analytics.

   ![Список доступа Stream Analytics на портале](./media/stream-analytics-managed-identities-adls/stream-analytics-access-list.png)

   Дополнительные сведения о разрешениях файловой системы Azure Data Lake Storage 1-го поколения см. в статье [Контроль доступа в Azure Data Lake Storage 1-го поколения](../data-lake-store/data-lake-store-access-control.md).

## <a name="stream-analytics-tools-for-visual-studio"></a>Средства Stream Analytics для Visual Studio

1. В JobConfig.json задайте **Использовать назначенное системой удостоверение** значение **True**.

   ![Управляемые идентификаторы конфигурации задания Stream Analytics](./media/stream-analytics-managed-identities-adls/adls-mi-jobconfig-vs.png)

2. В окне Выходные свойства приемника выходных данных ADLS 1-го поколения щелкните раскрывающийся список режим проверки подлинности и выберите пункт **управляемое удостоверение**.

   ![Управляемые удостоверения выходных данных ADLS](./media/stream-analytics-managed-identities-adls/adls-mi-output-vs.png)

3. Укажите остальные свойства и щелкните **Сохранить**.

4. Щелкните **Отправить в Azure** в редакторе запросов.

   При отправке задания инструменты выполняют два действия.

   * Автоматически создают субъект-службу для идентификации задания Stream Analytics в Azure Active Directory. Жизненным циклом нового удостоверения будет управлять Azure. При удалении задания Stream Analytics Azure автоматически удаляет связанное удостоверение (то есть субъект-службу).

   * Автоматически установите разрешения на **запись** и **выполнение** для пути префикса ADLS 1-го поколения, используемого в задании, и назначьте его этой папке и ее всем дочерним элементам.

5. Шаблоны диспетчер ресурсов можно создать со следующим свойством, используя [Stream Analytics CI.CD пакета NuGet](https://www.nuget.org/packages/Microsoft.Azure.StreamAnalytics.CICD/) версии 1.5.0 или более поздней на компьютере сборки (за пределами Visual Studio). Следуйте инструкциям по развертыванию шаблона Resource Manager, приведенным в следующем разделе, чтобы получить субъект-службу и предоставить доступ субъект-службе через PowerShell.

## <a name="resource-manager-template-deployment"></a>Развертывание шаблонов Resource Manager

1. Вы можете создать ресурс *Microsoft.StreamAnalytics/streamingjobs* с использованием управляемого удостоверения, включив в раздел ресурсов шаблона Resource Manager следующее свойство:

    ```json
    "Identity": {
      "Type": "SystemAssigned",
    },
    ```

   Это свойство указывает Azure Resource Manager, что требуется создать удостоверение для задания Azure Stream Analytics и управлять им.

   **Пример задания**
   
   ```json
   {
     "Name": "AsaJobWithIdentity",
     "Type": "Microsoft.StreamAnalytics/streamingjobs",
     "Location": "West US",
     "Identity": {
       "Type": "SystemAssigned",
     },
     "properties": {
       "sku": {
         "name": "standard"
       },
       "outputs": [
         {
           "name": "string",
           "properties":{
             "datasource": {
               "type": "Microsoft.DataLake/Accounts",
               "properties": {
                 "accountName": "myDataLakeAccountName",
                 "filePathPrefix": "cluster1/logs/{date}/{time}",
                 "dateFormat": "YYYY/MM/DD",
                 "timeFormat": "HH",
                 "authenticationMode": "Msi"
             }
           }
         }
       }
     }
   }
   ```
  
   **Пример ответа задания**

   ```json
   {
    "Name": "mySAJob",
    "Type": "Microsoft.StreamAnalytics/streamingjobs",
    "Location": "West US",
    "Identity": {
      "Type": "SystemAssigned",
        "principalId": "GUID",
        "tenantId": "GUID",
      },
      "properties": {
        "sku": {
          "name": "standard"
        },
     }
   }
   ```

   Запишите идентификатор субъекта (principalId) из ответа задания для предоставления доступа к необходимому ресурсу ADLS.

   **Идентификатор клиента** (tenantId) — это идентификатор клиента Azure Active Directory, где создан субъект-служба. Субъект-служба создается в клиенте Azure, который является доверенным для этой подписки.

   **Type** указывает тип управляемого удостоверения, как описано в типах управляемых удостоверений. Поддерживается только тип удостоверения, назначаемого системой.

2. Предоставьте доступ субъекту-службе с помощью PowerShell. Чтобы предоставить доступ субъекту-службе с помощью PowerShell, выполните следующую команду:

   ```powershell
   Set-AzDataLakeStoreItemAclEntry -AccountName <accountName> -Path <Path> -AceType User -Id <PrinicpalId> -Permissions <Permissions>
   ```

   **PrincipalId** — идентификатор объекта субъекта-службы. Он отображается на экране портала после создания субъекта-службы. В случае создания задания путем развертывания шаблона Resource Manager идентификатор объекта отображается в свойстве идентификаторов в ответе задания.

   **Пример**

   ```powershell
   PS > Set-AzDataLakeStoreItemAclEntry -AccountName "adlsmsidemo" -Path / -AceType
   User -Id 14c6fd67-d9f5-4680-a394-cd7df1f9bacf -Permissions WriteExecute
   ```

   Дополнительные сведения о команде PowerShell см. в документации по [Set-аздаталакестореитемаклентри](/powershell/module/az.datalakestore/set-azdatalakestoreitemaclentry) .

## <a name="remove-managed-identity"></a>Удалить управляемое удостоверение

Управляемое удостоверение, созданное для задания Stream Analytics, удаляется только при удалении задания. Невозможно удалить управляемое удостоверение, не удаляя задание. Если вы больше не хотите использовать управляемое удостоверение, можно изменить метод проверки подлинности для выходных данных. Управляемое удостоверение будет продолжать существовать до тех пор, пока задание не будет удалено, и будет использоваться при повторном использовании проверки подлинности с управляемым удостоверением.

## <a name="limitations"></a>Ограничения
Эта функция не поддерживает следующие компоненты:

1. **Многоклиентский доступ**. субъект-служба, созданная для данного задания Stream Analytics, будет находиться в Azure Active Directoryном клиенте, на котором было создано задание, и не может использоваться для ресурса, который находится в другом клиенте Azure Active Directory. Таким образом, можно использовать только MSI для ресурсов ADLS Gen 1, которые находятся в том же Azure Active Directory клиенте, что и задание Azure Stream Analytics. 

2. **[Пользователь, которому назначено удостоверение](../active-directory/managed-identities-azure-resources/overview.md)**:, не поддерживается. Это означает, что пользователь не может ввести собственный субъект-службу, который будет использоваться их Stream Analyticsным заданием. Субъект-служба создается Azure Stream Analytics.

## <a name="next-steps"></a>Дальнейшие действия

* [Создание выходных данных Data Lake Store с помощью Stream Analytics](../data-lake-store/data-lake-store-stream-analytics.md)
* [Локальное тестирование запросов Stream Analytics с помощью Visual Studio](stream-analytics-vs-tools-local-run.md)
* [Тестирование реальных данных в локальной среде с помощью инструментов Azure Stream Analytics для Visual Studio (предварительная версия)](stream-analytics-live-data-local-testing.md) 
