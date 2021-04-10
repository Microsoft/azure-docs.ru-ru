---
title: Учебник. Передача данных в учетную запись хранения с помощью Azure Stack Edge Pro с GPU | Документация Майкрософт
description: Сведения о том, как добавить локальные учетные записи хранения и учетные записи хранения Edge и подключиться к ним на устройстве Azure Stack Edge Pro с GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 03/12/2021
ms.author: alkohli
ms.openlocfilehash: d20ade0fdf6262c08a4ae961d17b1ec8a9ba39f2
ms.sourcegitcommit: 73fb48074c4c91c3511d5bcdffd6e40854fb46e5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/31/2021
ms.locfileid: "106065809"
---
# <a name="tutorial-transfer-data-via-storage-accounts-with-azure-stack-edge-pro-gpu"></a>Руководство по Передача данных через учетные записи хранения с помощью Azure Stack Edge Pro с GPU 

[!INCLUDE [applies-to-GPU-and-pro-r-and-mini-r-skus](../../includes/azure-stack-edge-applies-to-gpu-pro-r-mini-r-sku.md)]

В этом учебнике описывается, как добавить учетные записи хранения на устройство Azure Stack Edge Pro и подключиться к ним. После добавления учетных записей хранения устройство Azure Stack Edge Pro может выполнять передачу данных в Azure.

Эта процедура может занять около 30 минут.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * добавление учетной записи хранения;
> * Подключение к учетной записи хранения

 
## <a name="prerequisites"></a>Предварительные требования

Прежде чем добавлять учетные записи хранения в Azure Stack Edge Pro, убедитесь, что:

- Вы установили физическое устройство, как описано в статье [Установка Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).

- Физическое устройство активировано, как описано в статье [Активация Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).


## <a name="add-an-edge-storage-account"></a>Добавление учетной записи хранения Edge

Чтобы создать учетную запись хранения Edge, выполните следующие действия:

[!INCLUDE [Add an Edge storage account](../../includes/azure-stack-edge-gateway-add-storage-account.md)]


## <a name="connect-to-the-edge-storage-account"></a>Подключение к учетной записи хранения Edge

Теперь вы можете подключаться к REST API хранилища Edge по протоколу *HTTP* или *HTTPS*.

- Протокол *HTTPS* является безопасным и рекомендуемым способом подключения.
- Протокол *HTTP* используется при подключении через доверенные сети.

## <a name="connect-via-http"></a>Подключение по протоколу HTTP

Для подключения к REST API хранилища Edge по протоколу HTTP необходимо выполнить следующие действия:

- Добавление виртуального IP-адреса службы, согласованной с Azure, и конечной точки службы BLOB-объектов к удаленному узлу.
- Проверка подключения 

Каждое из этих действий описано в следующих разделах.

### <a name="add-device-ip-address-and-blob-service-endpoint-to-the-remote-client"></a>Добавление IP-адреса устройства и конечной точки службы BLOB-объектов к удаленному клиенту

[!INCLUDE [Add device IP and blob service endpoint to host file](../../includes/azure-stack-edge-gateway-add-device-ip-address-blob-service-endpoint.md)]


### <a name="verify-connection"></a>Проверка подключения

Чтобы проверить подключение, вам, как правило, необходима информация (может отличаться), полученная на предыдущем этапе:

- имя учетной записи хранения;
- ключ доступа к учетной записи хранения;
- конечная точка службы BLOB-объектов.

У вас уже есть имя учетной записи хранения и конечная точка службы BLOB-объектов. Вы можете получить ключ доступа к учетной записи хранения, выполнив подключение к устройству через Azure Resource Manager с помощью клиента Azure PowerShell.

Выполните действия, описанные в статье о [подключении к устройству с помощью Azure Resource Manager](azure-stack-edge-gpu-connect-resource-manager.md). Выполнив вход в API локального устройств через Azure Resource Manager, получите список учетных записей хранения на устройстве. Выполните следующий командлет:

`Get-AzureRMStorageAccount`

В списке учетных записей хранения на устройстве найдите учетную запись хранения, для которой требуется ключ доступа. Запишите имя учетной записи хранения и группы ресурсов.

Результат выполнения команды показан ниже:

```azurepowershell
PS C:\windows\system32> Get-AzureRmStorageAccount

StorageAccountName ResourceGroupName Location SkuName     Kind    AccessTier CreationTime          ProvisioningState EnableHttpsTrafficOnly
------------------ ----------------- -------- -------     ----    ---------- ------------          ----------------- ----------------------
myasetiered1       myasetiered1      DBELocal StandardLRS Storage            11/27/2019 7:10:12 PM Succeeded         False
```

Чтобы получить ключ доступа, выполните следующий командлет:

`Get-AzureRmStorageAccountAccessKey`

Результат выполнения команды показан ниже:

```azurepowershell
PS C:\windows\system32> Get-AzureRmStorageAccountKey

cmdlet Get-AzureRmStorageAccountKey at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
ResourceGroupName: myasetiered1
Name: myasetiered1

KeyName Value    Permissions                                                                                
------- -----    -----------                                                                                
key1    Jb2brrNjRNmArFcDWvL4ufspJjlo+Nie1uh8Mp4YUOVQNbirA1uxEdHeV8Z0dXbsG7emejFWI9hxyR1T93ZncA==        Full
key2    6VANuHzHcJV04EFeyPiWRsFWnHPkgmX1+a3bt5qOQ2qIzohyskIF/2gfNMqp9rlNC/w+mBqQ2mI42QgoJSmavg==        Full
```

Скопируйте и сохраните его. Этот ключ будет использоваться для проверки подключения с помощью Обозревателя службы хранилища Azure.

Чтобы убедиться, что подключение установлено успешно, используйте Обозреватель службы хранилища для подключения к учетной записи внешнего хранилища. Загрузите [Обозреватель службы хранилища](https://go.microsoft.com/fwlink/?LinkId=708343&clcid=0x409), если он у вас не установлен.

[!INCLUDE [Verify connection using Storage Explorer](../../includes/azure-stack-edge-gateway-verify-connection-storage-explorer.md)]


## <a name="connect-via-https"></a>Подключение по протоколу HTTPS

Для подключения к REST API хранилища BLOB-объектов Azure по протоколу HTTP необходимо выполнить следующие действия:

- получить сертификат конечной точки большого двоичного объекта;
- импортировать сертификат на клиенте или удаленном узле;
- добавить IP-адрес устройства и конечную точку службы BLOB-объектов к клиенту или удаленному узлу;
- настроить и проверить подключение.

Каждое из этих действий описано в следующих разделах.

### <a name="get-certificate"></a>Получение сертификата

Для доступа к хранилищу BLOB-объектов по протоколу HTTPS требуется SSL-сертификат для устройства. Этот сертификат также будет отправлен на устройство Azure Stack Edge Pro в качестве *PFX*-файла с прикрепленным закрытым ключом. Дополнительные сведения о создании (только в целях тестирования и разработки) и передаче этих сертификатов на устройство Azure Stack Edge Pro см. в следующих ресурсах:

- [Создание сертификата конечной точки большого двоичного объекта](azure-stack-edge-gpu-manage-certificates.md#create-certificates-optional).
- [Передача сертификата конечной точки большого двоичного объекта](azure-stack-edge-gpu-manage-certificates.md#upload-certificates).
- [Импорт сертификатов на клиенте, получающем доступ к устройству](azure-stack-edge-gpu-manage-certificates.md#import-certificates-on-the-client-accessing-the-device).

### <a name="import-certificate"></a>Импорт сертификата

Если для подключения к учетным записям хранения на устройстве используется Обозреватель службы хранилища Azure, в него также необходимо импортировать сертификат в формате PEM. В среде Windows *CER*-файл в кодировке Base-64 аналогичен формату PEM.

Выполните указанные ниже действия, чтобы импортировать сертификаты в Обозревателе службы хранилища Azure.

1. Убедитесь, что Обозреватель службы хранилища Azure предназначен для API Azure Stack. Выберите **Изменить > Разработка для API Azure Stack**. При появлении запроса перезапустите Обозреватель службы хранилища, чтобы изменения вступили в силу.

2. Чтобы импортировать сертификаты SSL, выберите **Изменить > SSL-сертификаты > Импорт сертификатов**.

  
   ![Импорт сертификатов в Обозревателе службы хранилища](./media/azure-stack-edge-gpu-deploy-add-storage-accounts/import-cert-storage-explorer-1.png) 

3. Найдите и укажите цепочку подписывания и сертификаты больших двоичных объектов. Цепочка подписывания и сертификат большого двоичного объекта должны иметь формат PEM, который аналогичен формату в кодировке Base64 в системе Windows. Появится сообщение о том, что сертификаты успешно импортированы.


### <a name="add-device-ip-address-and-blob-service-endpoint"></a>Добавление IP-адреса устройства и конечной точки службы BLOB-объектов

Выполните те же действия для [добавления IP-адреса устройства и конечной точки службы BLOB-объектов, что и при подключении по протоколу *HTTP*](#add-device-ip-address-and-blob-service-endpoint-to-the-remote-client).

### <a name="configure-and-verify-connection"></a>Настройка и проверка подключения

Выполните шаги для [настройки и проверки подключения по протоколу *HTTP*](#verify-connection). Единственное отличие — не устанавливайте флажок для *использования HTTP*.

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике были рассмотрены такие темы относительно Azure Stack Edge Pro:

> [!div class="checklist"]
> * добавление учетной записи хранения;
> * подключение к учетной записи хранения

Чтобы узнать о преобразовании данных с помощью Azure Stack Edge Pro, перейдите к следующему учебнику:

> [!div class="nextstepaction"]
> [Преобразование данных с помощью Azure Stack Edge Pro](./azure-stack-edge-j-series-deploy-configure-compute.md)
