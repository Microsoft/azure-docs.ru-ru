---
title: Учебник по заказу Azure Data Box | Документация Майкрософт
description: Из этого руководства вы узнаете о гибридном решении Azure Data Box, которое позволяет импортировать локальные данные в Azure. Здесь также описано, как заказать Azure Data Box.
services: databox
author: v-dalc
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 03/08/2021
ms.author: alkohli
ms.openlocfilehash: aa3614aa3c4fbaec3611806406e5129379999bc3
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106067147"
---
# <a name="tutorial-order-azure-data-box"></a>Руководство по Заказ Azure Data Box

Azure Data Box — это гибридное решение, которое позволяет импортировать локальные данные в Azure быстрым, простым и надежным способом. Вы переносите свои данные на накопитель с доступной емкостью 80 ТБ, поставляемый Майкрософт, и затем отправляете устройство обратно. Затем эти данные передаются в Azure.

В этом руководстве описано, как заказать Azure Data Box. В этом руководстве вы рассмотрите следующее:  

> [!div class="checklist"]
>
> * Необходимые условия для развертывания Data Box
> * Заказ Data Box
> * Отслеживание заказа
> * Отмена заказа

## <a name="prerequisites"></a>Предварительные требования

# <a name="portal"></a>[Портал](#tab/portal)

Перед развертыванием устройства выполните следующие предварительные требования службы и устройства Data Box.

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.

Вы можете войти в Azure и выполнить команды Azure CLI одним из двух способов:

* Установить CLI и выполнить команды CLI локально.
* Выполнить команды CLI на портале Azure в Azure Cloud Shell.

Для работы с этим учебником мы используем Azure CLI в Windows PowerShell, но вы можете выбрать любой из этих вариантов.

### <a name="for-azure-cli"></a>Для Azure CLI

Перед тем как начать, убедитесь в следующем.

#### <a name="install-the-cli-locally"></a>Установка CLI локально

* Установите [Azure CLI](/cli/azure/install-azure-cli) версии 2.0.67 или более поздней. Кроме того, вы можете [выполнить установку с помощью MSI](https://aka.ms/installazurecliwindows).

**Вход в Azure**

Откройте командное окно Windows PowerShell и войдите в Azure с помощью команды [az login](/cli/azure/reference-index#az-login):

```azurecli
PS C:\Windows> az login
```

Ниже приведены выходные данные успешной операции входа:

```output
You have logged in. Now let us find all the subscriptions to which you have access.
[
   {
      "cloudName": "AzureCloud",
      "homeTenantId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      "id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "isDefault": true,
      "managedByTenants": [],
      "name": "My Subscription",
      "state": "Enabled",
      "tenantId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
      "user": {
          "name": "gusp@contoso.com",
          "type": "user"
      }
   }
]
```

**Установка расширения Azure Data Box CLI**

Прежде чем использовать команды интерфейса командной строки Azure Data Box, вам нужно установить расширение. Расширения Azure CLI предоставляют доступ к экспериментальным командам и предварительным выпускам команд, которые еще не поставлялись как часть основного CLI. Дополнительные сведения о расширениях см. в статье [Использование расширений с Azure CLI](/cli/azure/azure-cli-extensions-overview).

Чтобы установить расширение для Azure Data Box, выполните следующую команду: `az extension add --name databox`:

```azurecli

    PS C:\Windows> az extension add --name databox
```

Если расширение установлено успешно, вы увидите следующие выходные данные:

```output
    The installed extension 'databox' is experimental and not covered by customer support. Please use with discretion.
    PS C:\Windows>

    # az databox help

    PS C:\Windows> az databox -h

    Group
        az databox

    Subgroups:
        job [Experimental] : Commands to manage databox job.

    For more specific examples, use: az find "az databox"

        Please let us know how we are doing: https://aka.ms/clihats
```

#### <a name="use-azure-cloud-shell"></a>Использование Azure Cloud Shell

Вы можете применить для выполнения команд CLI размещенную в Azure интерактивную оболочку среды [Azure Cloud Shell](https://shell.azure.com/), с которой можно работать в браузере. Azure Cloud Shell поддерживает для работы со службами Azure оболочки bash и Windows PowerShell. Предварительно установленный интерфейс Azure CLI настроен для использования с вашей учетной записью. В меню в правом верхнем углу окна портала Azure нажмите кнопку Cloud Shell:

![Выбор меню Cloud Shell](../storage/common/media/storage-quickstart-create-account/cloud-shell-menu.png)

Эта кнопка запускает интерактивную оболочку, с помощью которой можно выполнять действия, описанные в этой статье.

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

[!INCLUDE [Prerequisites](../../includes/data-box-deploy-ordered-prerequisites.md)]

### <a name="for-azure-powershell"></a>Для Azure PowerShell

Перед началом работы необходимо выполнить следующее:

* Установка Windows PowerShell 6.2.4 или более поздней версии.
* Установка модуля Azure PowerShell (AZ).
* Установка модуля Azure Data Box (Az.DataBox).
* Войдите в Azure.

#### <a name="install-azure-powershell-and-modules-locally"></a>Локальная установка Azure PowerShell и модулей

**Установка или обновление Windows PowerShell**

Необходимо наличие Windows PowerShell версии 6.2.4 или более поздней. Чтобы определить установленную версию PowerShell, выполните команду: `$PSVersionTable`.

Вы увидите такой результат:

```azurepowershell
    PS C:\users\gusp> $PSVersionTable
    
    Name                           Value
    ----                           -----
    PSVersion                      6.2.3
    PSEdition                      Core
    GitCommitId                    6.2.3
    OS                             Microsoft Windows 10.0.18363
    Platform                       Win32NT
    PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
    PSRemotingProtocolVersion      2.3
    SerializationVersion           1.1.0.1
    WSManStackVersion              3.0
```

Если версия ниже 6.2.4, обновите Windows PowerShell. Сведения об установке последней версии Windows PowerShell см. в статье [Установка Azure PowerShell](/powershell/scripting/install/installing-powershell).

**Установка Azure PowerShell и модулей Data Box**

Чтобы использовать Azure PowerShell для заказа Azure Data Box, понадобится установить модули Azure PowerShell. Чтобы установить модули Azure PowerShell, сделайте следующее:

1. Установите [модуль Az для Azure PowerShell](/powershell/azure/new-azureps-module-az).
2. Затем установите Az.DataBox, используя команду `Install-Module -Name Az.DataBox`.

```azurepowershell
PS C:\PowerShell\Modules> Install-Module -Name Az.DataBox
PS C:\PowerShell\Modules> Get-InstalledModule -Name "Az.DataBox"

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
0.1.1                Az.DataBox                          PSGallery            Microsoft Azure PowerShell - DataBox ser…
```

#### <a name="sign-in-to-azure"></a>Вход в Azure

Откройте командное окно Windows PowerShell и войдите в Azure с помощью команды [Connect-AzAccount](/powershell/module/az.accounts/Connect-AzAccount):

```azurepowershell
PS C:\Windows> Connect-AzAccount
```

Ниже приведены выходные данные успешной операции входа:

```output
WARNING: To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code FSBFZMBKC to authenticate.

Account              SubscriptionName                          TenantId                             Environment
-------              ----------------                          --------                             -----------
gusp@contoso.com     MySubscription                            aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa AzureCloud

PS C:\Windows\System32>
```

Подробные сведения о входе в Azure, используя Windows PowerShell, см. в статье [Вход с помощью Azure PowerShell](/powershell/azure/authenticate-azureps).

---

## <a name="order-data-box"></a>Оформление заказа Data Box

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы заказать устройство, выполните следующие шаги на портале Azure.

1. Используйте учетные данные Microsoft Azure для входа по этому URL-адресу: [https://portal.azure.com](https://portal.azure.com).
2. Выберите **+Создать ресурс** и выполните поиск по фразе *Azure Data Box*. Выберите **Azure Data Box**.

   ![Снимок экрана нового раздела с Azure Data Box в поле поиска](media/data-box-deploy-ordered/select-data-box-import-02.png)

3. Нажмите кнопку **создания**.

   ![Снимок экрана раздела Azure Data Box с вызванным параметром создания](media/data-box-deploy-ordered/select-data-box-import-03.png)

4. Проверьте, доступен ли сервис Data Box в вашем регионе. Введите или выберите следующие сведения и нажмите кнопку **Применить**.

    |Параметр  |Значение  |
    |---------|---------|
    |Тип передачи     | Выберите **Импорт в Azure**.        |
    |Подписка     | Выберите для использования службы Data Box подписку EA, CSP или "Спонсорское предложение Azure". <br> Подписка привязана к учетной записи для выставления счетов.       |
    |Группа ресурсов | Выберите имеющуюся группу ресурсов. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. |
    |Страна или регион отправки    |    Выберите страну/регион, в которой находятся ваши данные.         |
    |Конечный регион Azure     |     Выберите регион Azure, куда вы хотите передать данные. <br> Дополнительные сведения см. в разделе о [доступности по регионам](data-box-overview.md#region-availability).            |

    [ ![Оформление заказа на импорт Azure Data Box](media/data-box-deploy-ordered/select-data-box-import-04-b.png) ](media/data-box-deploy-ordered/select-data-box-import-04-b.png#lightbox)

5. Выберите **Data Box**. Максимальная полезная емкость для одного заказа составляет 80 ТБ. Для больших размеров данных можно создать несколько заказов.

    ![Доступные размеры данных: Диск Data Box, 40 ТБ; Data Box, 100 ТБ; Data Box Heavy, 1000 ТБ; отправка собственных дисков, 1 ТБ](media/data-box-deploy-ordered/select-data-box-import-05.png)

6. На странице **Заказ** перейдите на вкладку **Основы**. Введите или выберите следующие сведения и нажмите кнопку **Next: Data destination>** (Далее: назначение данных>).

    |Параметр  |Значение  |
    |---------|---------|
    |Подписка      | Сведения о подписке заполняются автоматически на основе ранее выбранного значения.|
    |Группа ресурсов    | Группа ресурсов, выбранная ранее. |
    |Импорт имени заказа | Введите понятное имя для отслеживания заказа. <br> Имя может содержать от 3 до 24 знаков, среди которых могут быть буквы, цифры и дефисы. <br> Имя должно начинаться и заканчиваться буквой или цифрой.    |

    ![Мастер заказа на импорт Data Box, экран "Основы" с правильно заполненными данными](media/data-box-deploy-ordered/select-data-box-import-06.png)

7. На экране **Назначение данных** выберите **Назначение данных** — учетные записи хранения либо управляемые диски.

    При использовании **учетной записи хранения** в качестве целевого хранилища вы увидите следующий снимок экрана.

    ![Мастер заказа на импорт Data Box, экран "Назначение данных" с выбранными учетными записями хранения](media/data-box-deploy-ordered/select-data-box-import-07.png)

    С учетом заданного региона Azure выберите в отфильтрованном списке существующую учетную запись хранения. Data Box можно связать с 10 учетными записями хранения. Вы также можете создать новую **Учетную запись общего назначения версии 1**, **Учетную запись общего назначения версии 2** или **Учетную запись хранения BLOB-объекта**.

   > [!NOTE]
   > - Если выбрать учетные записи хранения для хранилища Azure категории "Премиум", подготовленная квота в общем ресурсе учетной записи хранения увеличится до размера данных, копируемых в общую папку. После увеличения квоты она не изменяется, например, если по какой-либо причине Data Box не может скопировать данные.
   > - Эта квота используется для выставления счетов. Когда данные будут переданы в центр обработки данных, необходимо настроить квоту в соответствии с вашими потребностями. Дополнительные сведения см. в статье [Общие сведения о выставлении счетов](../../articles/storage/files/understanding-billing.md).

    Поддерживаются учетные записи хранения в виртуальных сетях. Чтобы служба Data Box могла работать с защищенными учетными записями хранения, включите доверенные службы в настройках сетевого брандмауэра учетной записи хранения. Дополнительные сведения см. в статье о [добавлении Azure Data Box в качестве доверенной службы](../storage/common/storage-network-security.md#exceptions).

    При использовании Data Box для создания **управляемых дисков** на основе локальных виртуальных жестких дисков (VHD) необходимо будет также указать следующие сведения:

    |Параметр  |Значение  |
    |---------|---------|
    |Группы ресурсов     | Если вы планируете создать управляемые диски из локальных жестких дисков, создайте новые группы ресурсов. Вы можете использовать существующую группу ресурсов только в том случае, если она была создана ранее при создании заказа Data Box для управляемых дисков службой Data Box. <br> Укажите несколько групп ресурсов, разделив их точкой с запятой. Поддерживается максимум 10 групп ресурсов.|

    ![Мастер заказа на импорт Data Box, экран "Назначение данных" с выбранными управляемыми дисками](media/data-box-deploy-ordered/select-data-box-import-07-b.png)

    Учетная запись хранения, указанная для управляемых дисков, используется в качестве промежуточной учетной записи хранения. Служба Data Box выгружает виртуальные жесткие диски в виде страничных BLOB-объектов в промежуточную учетную запись хранения, а затем преобразовывает их в управляемые диски и перемещает в группы ресурсов. Дополнительные сведения см. в разделе [Проверка передачи данных в Azure](data-box-deploy-picked-up.md#verify-data-upload-to-azure).

   > [!NOTE]
   > Если страничный BLOB-объект не удалось преобразовать в управляемый диск, он остается в учетной записи хранения и взимается плата за хранилище.

8. По завершении выберите **Next: Security** (Далее: безопасность), чтобы продолжить.

    На экране **Безопасность** можно использовать собственные устройство и ключ шифрования и предоставить общий доступ к паролям, а также использовать двойное шифрование.

    Все параметры на экране **Безопасность** являются необязательными. Если никак не изменять эти параметры, будут применяться параметры по умолчанию.

    ![Экран "Безопасность" мастера заказа на импорт Data Box](media/data-box-deploy-ordered/select-data-box-import-security-01.png)

9. Если вы хотите использовать собственный ключ, управляемый клиентом, чтобы защитить ключ доступа разблокировки для нового ресурса, разверните элемент **Тип шифрования**.

    Настройка ключа, управляемого клиентом, для Azure Data Box необязательна. По умолчанию Data Box использует управляемый Майкрософт ключ для защиты ключа разблокировки.

    Ключ, управляемый клиентом, не влияет на шифрование данных на устройстве. Ключ используется только для шифрования ключа разблокировки устройства.

    Если вы не хотите использовать ключ, управляемый клиентом, перейдите к шагу 15.

   ![Снимок экрана "Безопасность" с параметрами типа шифрования](./media/data-box-deploy-ordered/customer-managed-key-01.png)

10. Выберите в качестве типа ключа **Управляемый клиентом ключ**. Затем выберите **Select a key vault and key** (Выбрать хранилище ключей и ключ).
   
    ![Экран безопасности, параметры для ключа, управляемого клиентом](./media/data-box-deploy-ordered/customer-managed-key-02.png)

11. В колонке **Select key from Azure Key Vault** (Выбор ключа из Azure Key Vault) подписка заполняется автоматически.

    - Для поля **Хранилище ключей** можно выбрать существующее хранилище Key Vault из раскрывающегося списка.

      ![Снимок экрана: выбор ключа в Azure Key Vault](./media/data-box-deploy-ordered/customer-managed-key-03.png)

    - Чтобы создать Key Vault, можно также выбрать **Создать**. На экране **Создать хранилище ключей** введите группу ресурсов и имя хранилища ключей. Убедитесь, что включены **Обратимое удаление** и **Защита от очистки**. Примите другие значения по умолчанию и выберите **Просмотр и создание**.

      ![Параметры создания нового хранилища Azure Key Vault](./media/data-box-deploy-ordered/customer-managed-key-04.png)

      Проверьте сведения о хранилище ключей и выберите **Создать**. Подождите несколько минут, пока не завершится создание хранилища ключей.

      ![Снимок экрана: новое хранилище Azure Key Vault](./media/data-box-deploy-ordered/customer-managed-key-05.png)

12. В колонке **Select key from Azure Key Vault** (Выбор ключа из Azure Key Vault) можно выбрать ключ в существующем хранилище ключей.

    ![Выберите существующий ключ в Azure Key Vault](./media/data-box-deploy-ordered/customer-managed-key-06.png)

    Если вы хотите создать новый ключ, выберите **Создать новый**. Необходимо использовать ключ RSA. Размер ключа может составлять 2048 бит или больше. Введите имя для нового ключа, примите остальные значения по умолчанию и нажмите кнопку **Создать**.

      ![Параметр "Создать новый ключ"](./media/data-box-deploy-ordered/customer-managed-key-07.png)

      Вы получите уведомление о создании ключа в хранилище ключей.

13. Выберите параметр **Версия** для используемого ключа, а затем щелкните **Выбрать**.

      ![Ключ, созданный в Key Vault](./media/data-box-deploy-ordered/customer-managed-key-08.png)

    Если вы хотите создать новую версию ключа, выберите **Создать новый**.

    ![Диалоговое окно для создания новой версии ключа](./media/data-box-deploy-ordered/customer-managed-key-08-a.png)

    Выберите параметры для новой версии ключа и выберите команду **Создать**.

    ![Создание новой версии ключа](./media/data-box-deploy-ordered/customer-managed-key-08-b.png)

    Параметры **Тип шифрования** на экране **Безопасность** будут содержать ваше хранилище ключей и ключ.

    ![Ключ и хранилище ключей для управляемого клиентом ключа](./media/data-box-deploy-ordered/customer-managed-key-09.png)

14. Выберите удостоверение пользователя, которое будет использоваться для управления доступом к этому ресурсу. Щелкните **Выбрать удостоверение пользователя**. На панели справа выберите подписку и управляемое удостоверение для использования. Затем щелкните **Выбрать**.

    Назначаемое пользователем управляемое удостоверение — это автономный ресурс Azure, который можно использовать для управления несколькими ресурсами. Дополнительные сведения см. в статье [Что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md)  

    Если вам нужно создать управляемое удостоверение, следуйте указаниям в статье [Создание, получение списка, удаление ролей и их назначение для управляемого удостоверения, назначаемого пользователем, с помощью портала Azure](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md).
    
    ![Выбор удостоверения пользователя](./media/data-box-deploy-ordered/customer-managed-key-10.png)

    Удостоверение пользователя отображается в параметрах **Тип шифрования**.

    ![Выбранное удостоверение пользователя отображается в параметрах "Тип шифрования"](./media/data-box-deploy-ordered/customer-managed-key-11.png)

15. Если вы не хотите использовать созданные системой пароли, которые Azure Data Box использует по умолчанию, разверните элемент **Создать собственный пароль** на экране **Безопасность**.

    Пароли, созданные системой, являются безопасными и рекомендуется использовать их, если в организации не требуется иное.

    ![Развернутый элемент "Создать собственный пароль" для заказа импорта Data Box](media/data-box-deploy-ordered/select-data-box-import-security-02.png) 

   - Чтобы использовать собственный пароль для нового устройства с использованием параметра **Set preference for the device password** (Задать предпочтения для пароля устройства), выберите **Использовать собственный пароль** и введите пароль, соответствующий требованиям безопасности.
     
     Пароль должен состоять из букв и цифр, содержать от 12 до 15 символов, включать хотя бы одну прописную и строчную буквы, один специальный знак и одну цифру. 

     - Разрешенные специальные символы: @ # - $ % ^ ! + = ; : _ ( )
     - Недопустимые символы: I i L o O 0
   
     ![Экран "Безопасность" для заказа на импорт Data Box с параметрами для использования собственного пароля устройства](media/data-box-deploy-ordered/select-data-box-import-security-03.png)

 - Чтобы использовать собственные пароли для общих ресурсов, выполните следующие действия.

   1. Рядом с **Set preference for share passwords** (Задать предпочтения для паролей общих папок), выберите **Использовать собственные пароли**, а затем — **Выбрать пароли для общих папок**.
     
       ![Экран "Безопасность" для заказа на импорт Data Box с параметрами для использования собственного пароля для общей папки](media/data-box-deploy-ordered/select-data-box-import-security-04.png)

    1. Введите пароль для каждой учетной записи хранения в заказе. Пароль будет использоваться для всех общих папок в учетной записи хранения.
    
       Пароль должен состоять из букв и цифр, содержать от 12 до 64 символов, включать хотя бы одну прописную и строчную буквы, один специальный знак и одну цифру.

       - Разрешенные специальные символы: @ # - $ % ^ ! + = ; : _ ( )
       - Недопустимые символы: I i L o O 0
     
    1. Чтобы использовать один и тот же пароль для всех учетных записей хранения, выберите **Копировать во все**. 

    1. По завершении нажмите кнопку **Сохранить**.
     
       ![Экран для ввода паролей для общих папок для заказа на импорт Data Box](media/data-box-deploy-ordered/select-data-box-import-security-05.png)

    Чтобы изменить пароли, на экране **Безопасность** выберите **View or change passwords** (Просмотр или изменение паролей).

16. Чтобы включить двойное программное шифрование, на экране **Безопасность** разверните элемент **Double-encryption (for highly secure environments)** (Двойное шифрование (для сред с высоким уровнем безопасности)) и выберите **Включить двойное шифрование для заказа**.

    ![Экран "Безопасность" для импорта Data Box, включение программного шифрования для заказа Data Box](media/data-box-deploy-ordered/select-data-box-import-security-07.png)

    Программное шифрование в Data Box выполняется в дополнение к шифрованию данных AES-256.

    > [!NOTE]
    > Включение этого параметра может привести к увеличению времени на обработку заказов и копирование данных. После создания заказа нельзя изменить этот параметр.

    По завершении выберите **Next: Contact details** (Далее: контактные сведения), чтобы продолжить.

17. На вкладке **Контактные сведения** выберите **+ Добавить адрес доставки**.

    ![Добавление адреса доставки для заказа на импорт Azure Data Box на экране "Контактные сведения"](media/data-box-deploy-ordered/select-data-box-import-08-a.png)

18. Во вкладке **Адрес доставки** укажите свое имя и фамилию, название и почтовый адрес компании и действительный номер телефона. Выберите **Проверить адрес**. Данная служба проверяет адрес доставки на доступность услуг. Если служба доступна для указанного адреса доставки, вы получите соответствующее уведомление.

    ![Снимок экрана диалогового окна добавления адреса доставки с параметрами доставки и добавления адреса доставки.](media/data-box-deploy-ordered/select-data-box-import-10.png)

    Если вы выбрали самоуправляемую доставку, после успешного размещения заказа вы получите уведомление по электронной почте. Дополнительные сведения см. в статье [об использовании самостоятельной доставки](data-box-portal-customer-managed-shipping.md).

19. После успешной проверки сведений о доставке выберите **Добавить адрес доставки**. Вы вернетесь на вкладку **Контактные сведения**.

20. После возвращения на вкладку **Контактные сведения** добавьте один или несколько адресов электронной почты. Служба отправляет уведомления по электронной почте об обновлениях состояния заказа на указанные адреса электронной почты.

    Рекомендуется использовать групповой адрес электронной почты, чтобы продолжить получать уведомления после ухода администратора группы.

    ![Раздел "Электронная почта" на вкладке "Контактные сведения" в мастере заказа](media/data-box-deploy-ordered/select-data-box-import-08-c.png)

21. Просмотрите во вкладке **Review + Order** (Просмотр и заказ) сведения, относящиеся к заказу, контактам, уведомлениям и условиям конфиденциальности. Установите флажок, соответствующий соглашению с условиями конфиденциальности.

22. Выберите **Заказ**. Создание заказа может занять несколько минут.

    ![Экран Review + Order (Просмотр и заказ) в мастере заказа](media/data-box-deploy-ordered/select-data-box-import-11.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Чтобы заказать устройство, выполните следующие действия в Azure CLI:

1. Запишите параметры заказа Data Box. Эти параметры содержат личные данные или сведения об организации, имя подписки, сведения об устройстве и доставке. Эти сведения нужно указать в качестве параметров при выполнении команды CLI, которая создает заказ на Data Box. В следующей таблице приведены значения параметров, примененные для `az databox job create`.

   | Настройка (параметр) | Описание |  Образец значения |
   |---|---|---|
   |resource-group| Используйте имеющуюся или создайте новую. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
   |name| Имя заказа, который вы создаете. | "mydataboxorder"|
   |contact-name| Имя, связанное с указанным адресом доставки. | "Gus Poland"|
   |phone| Номер телефона сотрудника или организации, которые получат этот заказ.| "14255551234"
   |location| Ближайший к вам регион Azure, из которого будет доставлено устройство.| "US West"|
   |sku| Конкретное устройство Data Box, которое вы заказываете. Допустимые значения: "DataBox", "DataBoxDisk" и "DataBoxHeavy"| "DataBox" |
   |email-list| Адреса электронной почты, связанные с этим заказом.| "gusp@contoso.com" |
   |street-address1| Физический адрес, по которому будет отправлен заказ. | "15700 NE 39th St" |
   |street-address2| Дополнительные сведения об адресе, например номер корпуса или квартиры. | "Building 123" |
   |city| Город, в который будет отправлено устройство. | "Redmond" |
   |state-or-province| Штат или регион, в который будет отправлено устройство.| "WA" |
   |country| Страна, в которую будет отправлено устройство. | "Соединенные Штаты" |
   |postal-code| Почтовый код или индекс адреса доставки.| "98052"|
   |company-name| Название компании, в которой вы работаете.| "Contoso, LTD" |
   |запись хранения Azure| Учетная запись хранения Azure, из которой вы будете импортировать данные.| "mystorageaccount"|
   |debug| Включение отладочной информации для подробного ведения журнала.  | --debug |
   |help| Отображение справочной информации по этой команде. | --help -h |
   |only-show-errors| Показывать только ошибки, блокируя предупреждения. | --only-show-errors |
   |--output -o| Формат выходных данных.  Допустимые значения: json, jsonc, none, table, tsv, yaml, yamlc. По умолчанию используется значение json. | --output "json" |
   |query| Строка запроса JMESPath. Дополнительные сведения см. в [документации по JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Включение подробного ведения журнала. | --verbose |

2. В выбранном сеансе командной строки или терминале введите [az data box job create](/cli/azure/ext/databox/databox/job#ext-databox-az-databox-job-create), чтобы создать заказ на Azure Data Box.

   ```azurecli
   az databox job create --resource-group <resource-group> --name <order-name> --location <azure-location> --sku <databox-device-type> --contact-name <contact-name> --phone <phone-number> --email-list <email-list> --street-address1 <street-address-1> --street-address2 <street-address-2> --city "contact-city" --state-or-province <state-province> --country <country> --postal-code <postal-code> --company-name <company-name> --storage-account "storage-account"
   ```

   Ниже приведен пример применения этой команды:

   ```azurecli
   az databox job create --resource-group "myresourcegroup" \
                         --name "mydataboxtest3" \
                         --location "westus" \
                         --sku "DataBox" \
                         --contact-name "Gus Poland" \
                         --phone "14255551234" \
                         --email-list "gusp@contoso.com" \
                         --street-address1 "15700 NE 39th St" \
                         --street-address2 "Bld 25" \
                         --city "Redmond" \
                         --state-or-province "WA" \
                         --country "US" \
                         --postal-code "98052" \
                         --company-name "Contoso" \
                         --storage-account mystorageaccount
   ```

   Ниже представлен результат выполнения этой команды:

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   {
     "cancellationReason": null,
     "deliveryInfo": {
        "scheduledDateTime": "0001-01-01T00:00:00+00:00"
   },
   "deliveryType": "NonScheduled",
   "details": null,
   "error": null,
   "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myresourcegroup/providers/Microsoft.DataBox/jobs/mydataboxtest3",
   "identity": {
     "type": "None"
   },
   "isCancellable": true,
   "isCancellableWithoutFee": true,
   "isDeletable": false,
   "isShippingAddressEditable": true,
   "location": "westus",
   "name": "mydataboxtest3",
   "resourceGroup": "myresourcegroup",
   "sku": {
     "displayName": null,
     "family": null,
     "name": "DataBox"
   },
   "startTime": "2020-06-10T23:28:27.354241+00:00",
   "status": "DeviceOrdered",
   "tags": {},
   "type": "Microsoft.DataBox/jobs"

   }
   PS C:\Windows>

   ```

3. Все команды Azure CLI по умолчанию будут использовать JSON в качестве выходного формата, если вы не измените его. Формат вывода можно изменить с помощью глобального параметра `--output <output-format>`. Если вы укажете формат "table", выходные данные будут более удобными для чтения.

   Вот та же команда, для которой мы немного изменили параметры форматирования:

    ```azurecli
    az databox job create --resource-group "myresourcegroup" --name "mydataboxtest4" --location "westus" --sku "DataBox" --contact-name "Gus Poland" --phone "14255551234" --email-list "gusp@contoso.com" --street-address1 "15700 NE 39th St" --street-address2 "Bld 25" --city "Redmond" --state-or-province "WA" --country "US" --postal-code "98052" --company-name "Contoso" --storage-account mystorageaccount --output "table"
   ```

   Ниже представлен результат выполнения этой команды:

   ```output

    Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
    DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name            ResourceGroup    StartTime                         Status
    --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  --------------  ---------------  --------------------------------  -------------
    NonScheduled    True             True                       False          True                         westus      mydataboxtest4  myresourcegroup  2020-06-18T03:48:00.905893+00:00  DeviceOrdered

    ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

Чтобы заказать устройство, выполните следующие действия в Azure PowerShell:

1. Перед созданием заказа на импорт необходимо получить учетную запись хранения и сохранить объект учетной записи хранения в переменной.

   ```azurepowershell
    $storAcct = Get-AzStorageAccount -Name "mystorageaccount" -ResourceGroup "myresourcegroup"
   ```

2. Запишите параметры заказа Data Box. Эти параметры содержат личные данные или сведения об организации, имя подписки, сведения об устройстве и доставке. Эти сведения нужно указать в качестве параметров при выполнении команды PowerShell, которая создает заказ на Data Box. В следующей таблице приведены значения параметров, примененные для [New-AzDataBoxJob](/powershell/module/az.databox/New-AzDataBoxJob).

    | Настройка (параметр) | Описание |  Образец значения |
    |---|---|---|
    |ResourceGroupName [обязательный]| Использование имеющейся группы ресурсов. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
    |Name [обязательный]| Имя заказа, который вы создаете. | "mydataboxorder"|
    |ContactName [обязательный]| Имя, связанное с указанным адресом доставки. | "Gus Poland"|
    |PhoneNumber [обязательный]| Номер телефона сотрудника или организации, которые получат этот заказ.| "14255551234"
    |Location [обязательный]| Ближайший к вам регион Azure, из которого будет доставлено устройство.| "WestUS"|
    |DataBoxType [обязательный]| Конкретное устройство Data Box, которое вы заказываете. Допустимые значения: "DataBox", "DataBoxDisk" и "DataBoxHeavy"| "DataBox" |
    |EmailId [обязательный]| Адреса электронной почты, связанные с этим заказом.| "gusp@contoso.com" |
    |StreetAddress1 [обязательный]| Физический адрес, по которому будет отправлен заказ. | "15700 NE 39th St" |
    |StreetAddress2| Дополнительные сведения об адресе, например номер корпуса или квартиры. | "Building 123" |
    |StreetAddress3| Сведения о третьестепенном адресе. | |
    |City [обязательный]| Город, в который будет отправлено устройство. | "Redmond" |
    |StateOrProvinceCode [обязательный]| Штат или регион, в который будет отправлено устройство.| "WA" |
    |CountryCode [обязательный]| Страна, в которую будет отправлено устройство. | "Соединенные Штаты" |
    |PostalCode [обязательный]| Почтовый код или индекс адреса доставки.| "98052"|
    |CompanyName| Название компании, в которой вы работаете.| "Contoso, LTD" |
    |StorageAccountResourceId [обязательный]| ИД учетной записи хранения Azure, из которой вы будете импортировать данные.| <AzStorageAccount>.id |

3. В выбранном сеансе командной строки или терминале используйте команду [New-AzDataBoxJob](/powershell/module/az.databox/New-AzDataBoxJob), чтобы создать заказ на Azure Data Box.

   ```azurepowershell
    PS> $storAcct = Get-AzureStorageAccount -StorageAccountName "mystorageaccount"
    PS> New-AzDataBoxJob -Location "WestUS" \
                         -StreetAddress1 "15700 NE 39th St" \
                         -PostalCode "98052" \
                         -City "Redmond" \
                         -StateOrProvinceCode "WA" \
                         -CountryCode "US" \
                         -EmailId "gusp@contoso.com" \
                         -PhoneNumber 4255551234 \
                         -ContactName "Gus Poland" \
                         -StorageAccount $storAcct.id \
                         -DataBoxType DataBox \
                         -ResourceGroupName "myresourcegroup" \
                         -Name "myDataBoxOrderPSTest"
   ```

   Ниже представлен результат выполнения этой команды:

   ```output
    jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
    ----------------     -------------------- ------------------ --------------------- -------------------- -------------
    myDataBoxOrderPSTest DataBox              DeviceOrdered      07-06-2020 05:25:30   westus               myresourcegroup
   ```

---

## <a name="track-the-order"></a>Отслеживание заказа

# <a name="portal"></a>[Портал](#tab/portal)

После размещения заказа можно отслеживать его состояние на портале Azure. Перейдите к заказу Data Box, а затем щелкните **Обзор** для просмотра состояния. Заказ на портале отображается в состоянии **Заказано**.

Если устройство недоступно, вы получите уведомление. Если устройство доступно, корпорация Майкрософт определяет устройство для поставки и подготавливает отгрузку. Во время подготовки устройства выполняются следующие действия.

* Для каждой учетной записи хранилища создаются общие ресурсы SMB, связанные с устройством.
* Для каждого общего ресурса создаются учетные данные доступа, такие как имя пользователя и пароль.
* Также создается пароль устройства, который позволяет его разблокировать.
* Устройство Data Box заблокировано для предотвращения несанкционированного доступа к устройству.

После завершения подготовки устройства на портале отображается состояние заказа — **Обработано**.

![Обработанный заказ Data Box](media/data-box-overview/data-box-order-status-processed.png)

Майкрософт подготавливает и отправляет диски с помощью регионального поставщика. Когда устройство будет отправлено, вы получите номер для отслеживания. На портале отобразится состояние заказа — **Отправлено**.

![Отправленный заказ Data Box](media/data-box-overview/data-box-order-status-dispatched.png)

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

### <a name="track-a-single-order"></a>Отслеживание одного заказа

Чтобы получить сведения об одном уже сделанном заказе на Azure Data Box, выполните команду [`az databox job show`](/cli/azure/ext/databox/databox/job#ext-databox-az-databox-job-show). Эта команда отображает различные сведения о заказе, в том числе следующие: имя, группа ресурсов, сведения для отслеживания, идентификатор подписки, контактная информация, тип доставки и ценовая категория устройства.

   ```azurecli
   az databox job show --resource-group <resource-group> --name <order-name>
   ```

   В следующей таблице приведены значения параметров для `az databox job show`.

   | Параметр | Описание |  Образец значения |
   |---|---|---|
   |resource-group [обязательный]| Имя группы ресурсов, связанной с этим заказом. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
   |name [обязательный]| Отображаемое имя для заказа. | "mydataboxorder"|
   |debug| Включение отладочной информации для подробного ведения журнала. | --debug |
   |help| Отображение справочной информации по этой команде. | --help -h |
   |only-show-errors| Показывать только ошибки, блокируя предупреждения. | --only-show-errors |
   |--output -o| Формат выходных данных.  Допустимые значения: json, jsonc, none, table, tsv, yaml, yamlc. По умолчанию используется значение json. | --output "json" |
   |query| Строка запроса JMESPath. Дополнительные сведения см. в [документации по JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Включение подробного ведения журнала. | --verbose |

   Ниже приведен пример команды, для которой выходной формат имеет значение table:

   ```azurecli
    PS C:\WINDOWS\system32> az databox job show --resource-group "myresourcegroup" \
                                                --name "mydataboxtest4" \
                                                --output "table"
   ```

   Ниже представлен результат выполнения этой команды:

   ```output
    Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
    DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name            ResourceGroup    StartTime                         Status
    --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  --------------  ---------------  --------------------------------  -------------
    NonScheduled    True             True                       False          True                         westus      mydataboxtest4  myresourcegroup  2020-06-18T03:48:00.905893+00:00  DeviceOrdered
   ```

> [!NOTE]
> Упорядочение списка может поддерживаться на уровне подписки, и в этом случае группа ресурсов становится необязательным параметром.

### <a name="list-all-orders"></a>Список всех заказов

Если вы заказали несколько устройств, с помощью команды [`az databox job list`](/cli/azure/ext/databox/databox/job#ext-databox-az-databox-job-list) можно просмотреть все заказы на Azure Data Box. Эта команда выводит список всех заказов, которые относятся к определенной группе ресурсов. В выходные данные также включаются следующие сведения: имя заказа, состояние доставки, регион Azure, тип доставки, состояние заказа. Отмененные заказы также включаются в список.
Среди прочего, эта команда отображает для каждого заказа отметки времени.

```azurecli
az databox job list --resource-group <resource-group>
```

В следующей таблице приведены значения параметров для `az databox job list`.

   | Параметр | Описание |  Образец значения |
   |---|---|---|
   |resource-group [обязательный]| Имя группы ресурсов, в которой содержатся заказы. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
   |debug| Включение отладочной информации для подробного ведения журнала. | --debug |
   |help| Отображение справочной информации по этой команде. | --help -h |
   |only-show-errors| Показывать только ошибки, блокируя предупреждения. | --only-show-errors |
   |--output -o| Формат выходных данных.  Допустимые значения: json, jsonc, none, table, tsv, yaml, yamlc. По умолчанию используется значение json. | --output "json" |
   |query| Строка запроса JMESPath. Дополнительные сведения см. в [документации по JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Включение подробного ведения журнала. | --verbose |

   Ниже приведен пример команды, для которой выходной формат имеет значение table:

   ```azurecli
    PS C:\WINDOWS\system32> az databox job list --resource-group "GDPTest" --output "table"
   ```

   Ниже представлен результат выполнения этой команды:

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   CancellationReason                                               DeliveryType    IsCancellable    IsCancellableWithoutFee    IsDeletable    IsShippingAddressEditable    Location    Name                 ResourceGroup    StartTime                         Status
   ---------------------- ----------------------------------------  --------------  ---------------  -------------------------  -------------  ---------------------------  ----------  -------------------  ---------------  --------------------------------  -------------
   OtherReason This was a test order for documentation purposes.    NonScheduled    False            False                      True           False                        westus      gdpImportTest        MyResGrp         2020-05-26T23:20:57.464075+00:00  Cancelled
   NoLongerNeeded This order was created for documentation purposes.NonScheduled    False            False                      True           False                        westus      mydataboxExportTest  MyResGrp         2020-05-27T00:04:16.640397+00:00  Cancelled
   IncorrectOrder                                                   NonScheduled    False            False                      True           False                        westus      mydataboxtest2       MyResGrp         2020-06-10T16:54:23.509181+00:00  Cancelled
                                                                    NonScheduled    True             True                       False          True                         westus      mydataboxtest3       MyResGrp         2020-06-11T22:05:49.436622+00:00  DeviceOrdered
                                                                    NonScheduled    True             True                       False          True                         westus      mydataboxtest4       MyResGrp         2020-06-18T03:48:00.905893+00:00  DeviceOrdered
   PS C:\WINDOWS\system32>
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

### <a name="track-a-single-order"></a>Отслеживание одного заказа

Чтобы получить сведения об одном уже существующем заказе на Azure Data Box, выполните команду [Get-AzDataBoxJob](/powershell/module/az.databox/Get-AzDataBoxJob). Эта команда отображает различные сведения о заказе, в том числе следующие: имя, группа ресурсов, сведения для отслеживания, идентификатор подписки, контактная информация, тип доставки и ценовая категория устройства.

> [!NOTE]
> `Get-AzDataBoxJob` используется для отображения как одного, так и нескольких заказов. Разница заключается в том, что вам понадобится указать имена всех заказов.

   ```azurepowershell
    Get-AzDataBoxJob -ResourceGroupName <String> -Name <String>
   ```

   В следующей таблице приведены значения параметров для `Get-AzDataBoxJob`.

   | Параметр | Описание |  Образец значения |
   |---|---|---|
   |ResourceGroup [обязательный]| Имя группы ресурсов, связанной с этим заказом. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
   |Name [обязательный]| Название заказа для получения данных. | "mydataboxorder"|
   |ResourceId| ИД ресурса, связанного с этим заказом. |  |

   Ниже представлен пример такой команды с результатом ее выполнения:

   ```azurepowershell
    PS C:\WINDOWS\system32> Get-AzDataBoxJob -ResourceGroupName "myResourceGroup" -Name "myDataBoxOrderPSTest"
   ```

   Ниже представлен результат выполнения этой команды:

   ```output
   jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
   ----------------     -------------------- ------------------ --------------------- -------------------- -------------
   myDataBoxOrderPSTest DataBox              DeviceOrdered      7/7/2020 12:37:16 AM  WestUS               myResourceGroup
   ```

### <a name="list-all-orders"></a>Список всех заказов

Если вы заказали несколько устройств, с помощью команды [`Get-AzDataBoxJob`](/powershell/module/az.databox/Get-AzDataBoxJob) можно просмотреть все заказы на Azure Data Box. Эта команда выводит список всех заказов, которые относятся к определенной группе ресурсов. В выходные данные также включаются следующие сведения: имя заказа, состояние доставки, регион Azure, тип доставки, состояние заказа. Отмененные заказы также включаются в список.
Среди прочего, эта команда отображает для каждого заказа отметки времени.

```azurepowershell
Get-AzDataBoxJob -ResourceGroupName <String>
```

Вот пример выполнения такой команды:

```azurepowershell
PS C:\WINDOWS\system32> Get-AzDataBoxJob -ResourceGroupName "myResourceGroup"
```

Ниже представлен результат выполнения этой команды:

```output
jobResource.Name     jobResource.Sku.Name jobResource.Status jobResource.StartTime jobResource.Location ResourceGroup
----------------     -------------------- ------------------ --------------------- -------------------- -------------
guspImportTest       DataBox              Cancelled          5/26/2020 11:20:57 PM WestUS               myResourceGroup
mydataboxExportTest  DataBox              Cancelled          5/27/2020 12:04:16 AM WestUS               myResourceGroup
mydataboximport1     DataBox              Cancelled          6/26/2020 11:00:34 PM WestUS               myResourceGroup
myDataBoxOrderPSTest DataBox              Cancelled          7/07/2020 12:37:16 AM WestUS               myResourceGroup
mydataboxtest2       DataBox              Cancelled          6/10/2020 4:54:23  PM WestUS               myResourceGroup
mydataboxtest4       DataBox              DeviceOrdered      6/18/2020 3:48:00  AM WestUS               myResourceGroup
PS C:\WINDOWS\system32>
```

---

## <a name="cancel-the-order"></a>Отмена заказа

# <a name="portal"></a>[Портал](#tab/portal)

Чтобы отменить заказ, на портале Azure перейдите к вкладке **Обзор** и выберите **Отменить** на панели команд.

После размещения заказа его можно отменить в любой момент, пока состояние заказа не будет отмечено как "Обработано".

Чтобы удалить отмененный заказ, перейдите к вкладке **Обзор** и выберите **Удалить** на панели команд.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

### <a name="cancel-an-order"></a>Отмена заказа

Чтобы отменить заказ на Azure Data Box, выполните команду [`az databox job cancel`](/cli/azure/ext/databox/databox/job#ext-databox-az-databox-job-cancel). Вам придется указать причину отмены заказа.

   ```azurecli
   az databox job cancel --resource-group <resource-group> --name <order-name> --reason <cancel-description>
   ```

   В следующей таблице приведены значения параметров для `az databox job cancel`.

   | Параметр | Описание |  Образец значения |
   |---|---|---|
   |resource-group [обязательный]| Имя группы ресурсов, связанной с удаляемым заказом. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
   |name [обязательный]| Имя удаляемого заказа. | "mydataboxorder"|
   |reason [обязательный]| Причина для отмены заказа. | "I entered erroneous information and needed to cancel the order." (Заказ пришлось отменить из-за ошибки при вводе информации). |
   |да| Не запрашивать подтверждение. | --yes (-y)| 
   |debug| Включение отладочной информации для подробного ведения журнала. | --debug |
   |help| Отображение справочной информации по этой команде. | --help -h |
   |only-show-errors| Показывать только ошибки, блокируя предупреждения. | --only-show-errors |
   |--output -o| Формат выходных данных.  Допустимые значения: json, jsonc, none, table, tsv, yaml, yamlc. По умолчанию используется значение json. | --output "json" |
   |query| Строка запроса JMESPath. Дополнительные сведения см. в [документации по JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Включение подробного ведения журнала. | --verbose |

   Ниже представлен пример такой команды с результатом ее выполнения:

   ```azurecli
   PS C:\Windows> az databox job cancel --resource-group "myresourcegroup" --name "mydataboxtest3" --reason "Our budget was slashed due to **redacted** and we can no longer afford this device."
   ```

   Ниже представлен результат выполнения этой команды:

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   Are you sure you want to perform this operation? (y/n): y
   PS C:\Windows>
   ```

### <a name="delete-an-order"></a>Удаление заказа

Если вы отменили заказ на Azure Data Box, его можно удалить с помощью команды [`az databox job delete`](/cli/azure/ext/databox/databox/job#ext-databox-az-databox-job-delete).

   ```azurecli
   az databox job delete --name [-n] <order-name> --resource-group <resource-group> [--yes] [--verbose]
   ```

   В следующей таблице приведены значения параметров для `az databox job delete`.

   | Параметр | Описание |  Образец значения |
   |---|---|---|
   |resource-group [обязательный]| Имя группы ресурсов, связанной с удаляемым заказом. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
   |name [обязательный]| Имя удаляемого заказа. | "mydataboxorder"|
   |Подписка| Имя или идентификатор (GUID) подписки Azure. | "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" |
   |да| Не запрашивать подтверждение. | --yes (-y)|
   |debug| Включение отладочной информации для подробного ведения журнала. | --debug |
   |help| Отображение справочной информации по этой команде. | --help -h |
   |only-show-errors| Показывать только ошибки, блокируя предупреждения. | --only-show-errors |
   |--output -o| Формат выходных данных.  Допустимые значения: json, jsonc, none, table, tsv, yaml, yamlc. По умолчанию используется значение json. | --output "json" |
   |query| Строка запроса JMESPath. Дополнительные сведения см. в [документации по JMESPath](http://jmespath.org/). | --query <string>|
   |verbose| Включение подробного ведения журнала. | --verbose |

Ниже представлен пример такой команды с результатом ее выполнения:

   ```azurecli
   PS C:\Windows> az databox job delete --resource-group "myresourcegroup" --name "mydataboxtest3" --yes --verbose
   ```

   Ниже представлен результат выполнения этой команды:

   ```output
   Command group 'databox job' is experimental and not covered by customer support. Please use with discretion.
   command ran in 1.142 seconds.
   PS C:\Windows>
   ```

# <a name="powershell"></a>[PowerShell](#tab/azure-ps)

### <a name="cancel-an-order"></a>Отмена заказа

Чтобы отменить заказ на Azure Data Box, выполните команду [Stop-AzDataBoxJob](/powershell/module/az.databox/stop-azdataboxjob). Вам придется указать причину отмены заказа.

```azurepowershell
Stop-AzDataBoxJob -ResourceGroup <String> -Name <String> -Reason <String>
```

В следующей таблице приведены значения параметров для `Stop-AzDataBoxJob`.

| Параметр | Описание |  Образец значения |
|---|---|---|
|ResourceGroup [обязательный]| Имя группы ресурсов, связанной с отменяемым заказом. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
|Name [обязательный]| Имя удаляемого заказа. | "mydataboxorder"|
|Reason [обязательный]| Причина для отмены заказа. | "I entered erroneous information and needed to cancel the order." (Заказ пришлось отменить из-за ошибки при вводе информации). |
|Force | Принудительное выполнение командлета без подтверждения пользователем. | -Force |

Ниже представлен пример такой команды с результатом ее выполнения:

```azurepowershell
PS C:\PowerShell\Modules> Stop-AzDataBoxJob -ResourceGroupName myResourceGroup \
                                            -Name "myDataBoxOrderPSTest" \
                                            -Reason "I entered erroneous information and had to cancel."
```

Ниже представлен результат выполнения этой команды:

```output
Confirm
"Cancelling Databox Job "myDataBoxOrderPSTest
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
PS C:\WINDOWS\system32>
```

### <a name="delete-an-order"></a>Удаление заказа

Если вы отменили заказ на Azure Data Box, его можно удалить с помощью команды [`Remove-AzDataBoxJob`](/powershell/module/az.databox/remove-azdataboxjob).

```azurepowershell
Remove-AzDataBoxJob -Name <String> -ResourceGroup <String>
```

В следующей таблице приведены значения параметров для `Remove-AzDataBoxJob`.

| Параметр | Описание |  Образец значения |
|---|---|---|
|ResourceGroup [обязательный]| Имя группы ресурсов, связанной с удаляемым заказом. Группа ресурсов — это логический контейнер для ресурсов, которые могут управляться или развертываться вместе. | "myresourcegroup"|
|Name [обязательный]| Имя удаляемого заказа. | "mydataboxorder"|
|Force | Принудительное выполнение командлета без подтверждения пользователем. | -Force |

Ниже представлен пример такой команды с результатом ее выполнения:

```azurepowershell
PS C:\Windows> Remove-AzDataBoxJob -ResourceGroup "myresourcegroup" \
                                   -Name "mydataboxtest3"
```

Ниже представлен результат выполнения этой команды:

```output
Confirm
"Removing Databox Job "mydataboxtest3
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): y
PS C:\Windows>
```

---

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве освещены следующие темы, касающиеся Azure Data Box:

> [!div class="checklist"]
>
> * Необходимые условия для развертывания Data Box
> * Оформление заказа Data Box
> * Отслеживание заказа
> * Отмена заказа

Перейдите к следующему руководству, чтобы узнать, как настроить Data Box.

> [!div class="nextstepaction"]
> [Руководство. Распаковка, подключение и разблокировка диска Azure Data Box](./data-box-deploy-set-up.md)
