---
title: Запуск расширения пользовательских скриптов на виртуальных машинах Linux в Azure
description: Автоматизируйте задачи настройки виртуальных машин Linux с помощью расширения настраиваемых скриптов версии 2.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
ms.author: amjads
author: amjads1
ms.collection: linux
ms.date: 04/25/2018
ms.openlocfilehash: 094e5f4b1bf1611f2d418d3a7b8db15ec5d58878
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102563580"
---
# <a name="use-the-azure-custom-script-extension-version-2-with-linux-virtual-machines"></a>Использование расширения настраиваемых скриптов Azure версии 2 на виртуальных машинах Linux
Расширение настраиваемых скриптов версии 2 скачивает и выполняет скрипты на виртуальных машинах Azure. Это расширение можно использовать для настройки после развертывания, установки программного обеспечения и других задач настройки или управления. Сценарии можно скачать из службы хранилища Azure или другого расположения, доступного из Интернета, или передать в среду выполнения расширения. 

Расширение пользовательских сценариев интегрируется с шаблонами Azure Resource Manager. Его также можно запустить с помощью Azure CLI, PowerShell или REST API Виртуальных машин Azure.

В этой статье показано, как применять пользовательское расширение сценариев из Azure CLI и как выполнять расширения с помощью шаблона Azure Resource Manager. В этой статье также показаны шаги по устранению неполадок в системах Linux.


Имеется два расширения настраиваемых скриптов Linux:
* Версия 1 — Microsoft.OSTCExtensions.CustomScriptForLinux
* Версия 2 — Microsoft.Azure.Extensions.CustomScript

Чтобы работать с новой версией 2, используйте новые и имеющиеся развертывания. Новая версия рассчитана на то, чтобы стать готовой заменой. Таким образом, миграция выполняется так же легко, как и изменение имени и версии, при этом изменять конфигурацию расширения не требуется.


### <a name="operating-system"></a>Операционная система

Расширение пользовательских скриптов для Linux будет работать в операционной системе расширения, поддерживаемой расширением. Дополнительные сведения см. в этой [статье](../linux/endorsed-distros.md).

### <a name="script-location"></a>Расположение сценария

С помощью расширения можно получить доступ к хранилищу BLOB-объектов Azure с помощью учетных данных службы хранилища BLOB-объектов Azure. Кроме того, скрипт может находиться в любом расположении при условии, что виртуальная машина может осуществлять маршрутизацию в эту конечную точку, такую как GitHub, внутренний файловый сервер и т. д.

### <a name="internet-connectivity"></a>Подключение к Интернету
Чтобы скачать скрипт из внешнего источника, например с сайта GitHub или из службы хранилища Azure, откройте дополнительные порты брандмауэра или группы безопасности сети. Например, если скрипт находится в службе хранилища Azure, вы можете разрешить доступ с помощью тегов службы Azure NSG для [хранилища](../../virtual-network/network-security-groups-overview.md#service-tags).

Если скрипт расположен на локальном сервере, вам по-прежнему может потребоваться открыть дополнительные порты брандмауэра или группы безопасности сети.

### <a name="tips-and-tricks"></a>Советы и рекомендации
* Высокий процент сбоев этого расширения связан с синтаксическими ошибками в скрипте. Чтобы упростить поиск точки сбоя, протестируйте запуски скрипта без ошибок и включите в скрипт дополнительные возможности ведения журнала.
* Пишите идемпотентные скрипты, чтобы их случайные повторные запуски не приводили к изменениям системы.
* Выполняемые скрипты не должны запрашивать ввод данных пользователем.
* На выполнение скрипта отводится 90 минут. Более продолжительное действие вызовет сбой подготовки расширения.
* Не следует помещать перезагрузки в скрипт, так как будут возникать проблемы с другими устанавливаемыми расширениями и после перезагрузки расширение прекратит работать. 
* Не рекомендуется запускать сценарий, который приведет к сбою или обновлению агента виртуальной машины. Это может привести к тому, что расширение находится в состоянии перехода, а время ожидания истечет.
* Если у вас есть сценарий, который приведет к перезагрузке, установите приложения и запустите сценарии и т. д. Следует запланировать перезагрузку с помощью задания cron или таких средств, как DSC или Chef, расширения Puppet.
* Расширение будет запускать скрипт только один раз. Чтобы запускать скрипт при каждой загрузке, можно воспользоваться [образом cloud-init](../linux/using-cloud-init.md) и модулем [скриптов при загрузке](https://cloudinit.readthedocs.io/en/latest/topics/modules.html#scripts-per-boot). Кроме того, с помощью скрипта можно создать системную единицу службы.
* К виртуальной машине можно применить только одну версию расширения. Чтобы запустить второй пользовательский скрипт, можно обновить существующее расширение с помощью новой конфигурации. Кроме того, можно удалить расширение пользовательских скриптов и повторно применить его с обновленным сценарием.
* Чтобы запланировать время выполнения скрипта, необходимо создать задание Cron с помощью расширения. 
* Во время выполнения скрипта вы увидите на портале Microsoft Azure или в CLI только переходное состояние расширения. Если требуются более частые обновления состояния выполняющегося скрипта, следует создать собственное решение.
* Расширение пользовательских сценариев изначально не поддерживает прокси-серверы, однако можно использовать средство для обмена файлами, поддерживающее прокси-серверы в сценарии, например, *фигурные*. 
* Следует учитывать настраиваемые расположения каталогов, которые могут использоваться скриптами или командами, и иметь в распоряжении логику для обработки таких случаев.
*  При развертывании пользовательского скрипта в рабочем экземпляре VMSS рекомендуется выполнить развертывание с помощью шаблона JSON и сохранить учетную запись хранения скрипта, где вы можете управлять маркером SAS. 


## <a name="extension-schema"></a>Схема расширения

В конфигурации расширения пользовательских сценариев указываются такие параметры, как расположение сценария и команда для его выполнения. Эту конфигурацию можно хранить в файлах конфигурации, указать в командной строке или в шаблоне Azure Resource Manager. 

Конфиденциальные данные можно хранить в зашифрованном виде в защищенной конфигурации, которая расшифровывается только внутри виртуальной машины. Защищенную конфигурацию можно использовать для команд, которые содержат секретные данные, например пароли.

Эти элементы следует рассматривать в качестве конфиденциальных данных. Их нужно задать в защищенной конфигурации параметров расширений. Данные защищенных параметров расширения виртуальной машины Azure зашифрованы. Они расшифровываются только на целевой виртуальной машине.

```json
{
  "name": "config-app",
  "type": "Extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2019-03-01",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "skipDos2Unix":false,
      "timestamp":123456789          
    },
    "protectedSettings": {
       "commandToExecute": "<command-to-execute>",
       "script": "<base64-script-to-execute>",
       "storageAccountName": "<storage-account-name>",
       "storageAccountKey": "<storage-account-key>",
       "fileUris": ["https://.."],
       "managedIdentity" : "<managed-identity-identifier>"
    }
  }
}
```

>[!NOTE]
> Свойство managedIdentity **не должны** использоваться в сочетании со свойствами storageAccountName или storageAccountKey

### <a name="property-values"></a>Значения свойств

| Имя | Значение и пример | Тип данных | 
| ---- | ---- | ---- |
| версия_API | 2019-03-01 | Дата |
| publisher | Microsoft.Compute.Extensions | строка |
| type | CustomScript | строка |
| typeHandlerVersion | 2.1 | INT |
| fileUris (пример) | `https://github.com/MyProject/Archive/MyPythonScript.py` | массиве |
| commandToExecute (пример) | MyPythonScript.py Python \<my-param1> | строка |
| скрипт | IyEvYmluL3NoCmVjaG8gIlVwZGF0aW5nIHBhY2thZ2VzIC4uLiIKYXB0IHVwZGF0ZQphcHQgdXBncmFkZSAteQo= | строка |
| skipDos2Unix (например) | false | Логическое |
| метка времени (например) | 123456789 | 32-разрядное целое число |
| storageAccountName (пример) | examplestorageacct | строка |
| storageAccountKey (пример) | TmJK/1N3AbAZ3q/+hOXoi/l73zOqsaxXDhqa9Y83/v5UpXQp2DQIBuv2Tifp60cE/OaHsJZmQZ7teQfczQj8hg== | строка |
| managedIdentity (пример) | {} или {"clientId": "31b403aa-c364-4240-A7FF-d85fb6cd7232"} или {"objectId": "12dd289c-0583-46e5-b9b4-115d5c19ef4b" } | Объект JSON |

### <a name="property-value-details"></a>Сведения о значениях свойств
* `apiVersion`: Наиболее актуальные apiVersion можно найти с помощью [Обозреватель ресурсов](https://resources.azure.com/) или из Azure CLI с помощью следующей команды. `az provider list -o json`
* `skipDos2Unix` — (необязательное, логическое) пропустите преобразование dos2unix URL-адресов файла на основе сценария или скрипта.
* `timestamp` — (необязательный, 32-битное целое число) используется только для повторного запуска скрипта при изменении значения этого поля.  Любое целочисленное значение допустимо, оно только должно отличаться от предыдущего значения.
* `commandToExecute`: (**обязательное**, если скрипт не задан, строка) выполняемый скрипт точки входа. Используйте это поле, если команда содержит секретные данные, например пароли.
* `script` — (**необходимо**, если не задан параметр необязательный, строка) скрипт в кодировке Base64 (и при необходимости сжатый), выполненный с помощью /bin/sh.
* `fileUris`: (необязательное, строковый массив) URL-адреса файлов для скачивания.
* `storageAccountName`: (необязательное, строка) имя учетной записи хранения. Если указаны учетные данные хранилища, все значения `fileUris` должны иметь формат URL-адресов для BLOB-объектов Azure.
* `storageAccountKey`: (необязательное, строка) ключ доступа для учетной записи хранения.
* `managedIdentity`: (необязательный, объект JSON) [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md) для скачивания файлов
  * `clientId`: (необязательный, строка) идентификатор клиента управляемого удостоверения
  * `objectId`: (необязательный, строка) идентификатор объекта управляемого удостоверения


Указанные ниже значения можно задавать либо в открытых, либо в защищенных параметрах. Расширение отклонит любую конфигурацию, в которой приведенные ниже значения будут указаны в открытых и защищенных параметрах одновременно.
* `commandToExecute`
* `script`
* `fileUris`

Открытые параметры могут быть полезны для отладки, но настоятельно рекомендуется использовать защищенные параметры.

Открытые параметры отправляются в виде открытого текста на виртуальную машину, где будет выполняться скрипт.  Защищенные параметры шифруются с помощью ключа, который известен только Azure и виртуальной машине. Параметры сохраняются на виртуальной машине в том виде, в каком они были отправлены, т. е. если параметры были зашифрованы, они сохраняются на виртуальной машине зашифрованными. Сертификат, используемый для расшифровки зашифрованных значений, хранится на виртуальной машине и применяется для расшифровки параметров (при необходимости) во время выполнения.

#### <a name="property-skipdos2unix"></a>Свойство skipDos2Unix

Значение по умолчанию равно false, что означает преобразование dos2unix **выполнено**.

Предыдущая версия CustomScript Microsoft.OSTCExtensions.CustomScriptForLinux автоматически преобразовывает файлы DOS в UNIX, меняя `\r\n`на `\n`. Это преобразование по-прежнему имеется и включено по умолчанию. Это преобразование применяется ко всем файлам, скачанным из fileUris, или к параметрам скрипта, основанного на любом из приведенных ниже критериев.

* Если используется расширение `.sh`, `.txt`, `.py` или `.pl`, оно будет преобразовано. Параметр скрипта всегда будет соответствовать этим критериям, потому что предполагается, что это скрипт, выполняемый с помощью/bin/sh, и сохраняется как файл script.sh на виртуальной машине.
* Если файл начинается с `#!`.

Преобразование dos2unix можно пропустить, настроив для skipDos2Unix значение true.

```json
{
  "fileUris": ["<url>"],
  "commandToExecute": "<command-to-execute>",
  "skipDos2Unix": true
}
```

####  <a name="property-script"></a>Свойство script

CustomScript поддерживает выполнение определяемого пользователем скрипта. Параметры script позволяют объединить команды commandToExecute и fileUris. Вместо того, чтобы настраивать файл для скачивания из хранилища Azure или GitHub gist, вы можете просто кодировать скрипт в качестве параметра. Скрипт можно использовать для замены commandToExecute и fileUris.

Скрипт **должен** быть в кодировке base64.  Скрипт также можно **при необходимости** сжать в формате GZIP. Параметр script может использоваться в общедоступных или защищенных настройках. Максимальный размер данных параметра script — 256 КБ. Если скрипт превышает этот размер, он не будет выполнен.

Например, рассмотрим следующий скрипт, сохраненный в файле /script.sh/.

```sh
#!/bin/sh
echo "Updating packages ..."
apt update
apt upgrade -y
```

Правильный вариант скрипта CustomScript будет создан с использованием выходных данных следующей команды.

```sh
cat script.sh | base64 -w0
```

```json
{
  "script": "IyEvYmluL3NoCmVjaG8gIlVwZGF0aW5nIHBhY2thZ2VzIC4uLiIKYXB0IHVwZGF0ZQphcHQgdXBncmFkZSAteQo="
}
```

Скрипт можно в дальнейшем при необходимости сжать в формат GZIP, чтобы еще больше уменьшить размер (в большинстве случаев). (CustomScript автоматически обнаруживает использование сжатия в формате GZIP.)

```sh
cat script | gzip -9 | base64 -w 0
```

Чтобы выполнить скрипт CustomScript, используется приведенный ниже алгоритм.

 1. Проверьте, не превышает ли размер скрипта 256 КБ.
 1. Декодируйте значение скрипта из кодировки base64.
 1. _Попытайтесь_ сжать в формат GUNZIP декодированное значение.
 1. Запишите декодированное (и при необходимости распакованное) значение на диск (/var/lib/waagent/custom-script/#/script.sh)
 1. Выполните скрипт, используя _/bin/sh -c /var/lib/waagent/custom-script/#/script.sh.

####  <a name="property-managedidentity"></a>Свойство: managedIdentity
> [!NOTE]
> Это свойство **должно** быть задано только в защищенных параметрах.

CustomScript (начиная с версии 2,1) поддерживает [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md) для загрузки файлов с URL-адресов, указанных в параметре "fileUris". Это предоставляет CustomScript доступ к частным BLOB-объектам или контейнерам службы хранилища Azure без необходимости передавать секреты, такие как маркеры SAS или ключи учетной записи хранения.

Чтобы использовать эту функцию, пользователь должен добавить [назначенное системой](../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity) или [назначенное пользователем](../../app-service/overview-managed-identity.md?tabs=dotnet#add-a-user-assigned-identity) удостоверение на виртуальную машину или VMSS, где ожидается выполнение CustomScript, и [предоставить управляемому удостоверению доступ к контейнеру службы хранилища Azure или BLOB-объектов](../../active-directory/managed-identities-azure-resources/tutorial-vm-windows-access-storage.md#grant-access).

Чтобы использовать назначенное системой удостоверение на целевой виртуальной машине или VMSS, задайте в поле "managedidentity" пустой объект JSON. 

> Пример
>
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.sh"],
>   "commandToExecute": "sh script1.sh",
>   "managedIdentity" : {}
> }
> ```

Чтобы использовать назначенное пользователем удостоверение на целевой виртуальной машине или VMSS, настройте в поле "managedidentity" идентификатор клиента или идентификатор объекта управляемого удостоверения.

> Примеры:
>
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.sh"],
>   "commandToExecute": "sh script1.sh",
>   "managedIdentity" : { "clientId": "31b403aa-c364-4240-a7ff-d85fb6cd7232" }
> }
> ```
> ```json
> {
>   "fileUris": ["https://mystorage.blob.core.windows.net/privatecontainer/script1.sh"],
>   "commandToExecute": "sh script1.sh",
>   "managedIdentity" : { "objectId": "12dd289c-0583-46e5-b9b4-115d5c19ef4b" }
> }
> ```

> [!NOTE]
> Свойство managedIdentity **не должны** использоваться в сочетании со свойствами storageAccountName или storageAccountKey

## <a name="template-deployment"></a>Развертывание шаблона
Расширения виртуальной машины Azure можно развернуть с помощью шаблонов Azure Resource Manager. Для запуска расширения пользовательских сценариев во время развертывания шаблона Azure Resource Manager в нем можно использовать схему JSON, описанную в предыдущем разделе. Пример шаблона, который содержит расширение пользовательских сценариев, можно найти на сайте [GitHub](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-linux).


```json
{
  "name": "config-app",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2019-03-01",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
      },
    "protectedSettings": {
      "commandToExecute": "sh hello.sh <param2>",
      "fileUris": ["https://github.com/MyProject/Archive/hello.sh"
      ]  
    }
  }
}
```

>[!NOTE]
>В именах свойств учитывается регистр. Чтобы избежать проблем с развертыванием, используйте имена, как показано ниже.

## <a name="azure-cli"></a>Azure CLI
При использовании Azure CLI для выполнения расширения пользовательских сценариев создайте один или несколько файлов конфигурации. Как минимум, требуется commandToExecute.

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --protected-settings ./script-config.json
```

При необходимости можно указать параметры в команде в виде форматированной строки JSON. Это позволяет задать конфигурацию прямо во время выполнения, не используя отдельный файл конфигурации.

```azurecli
az vm extension set \
  --resource-group exttest \
  --vm-name exttest \
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --protected-settings '{"fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"],"commandToExecute": "./config-music.sh"}'
```

### <a name="azure-cli-examples"></a>Примеры использования интерфейса командной строки Azure

#### <a name="public-configuration-with-script-file"></a>Открытая конфигурация с файлом сценария

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"],
  "commandToExecute": "./config-music.sh"
}
```

Команда интерфейса командной строки Azure:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings ./script-config.json
```

#### <a name="public-configuration-with-no-script-file"></a>Открытая конфигурация без файла сценария

```json
{
  "commandToExecute": "apt-get -y update && apt-get install -y apache2"
}
```

Команда интерфейса командной строки Azure:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings ./script-config.json
```

#### <a name="public-and-protected-configuration-files"></a>Открытый защищенный файл конфигурации

Используйте открытый файл конфигурации, чтобы задать URI файла сценария. Чтобы задать команду для выполнения, используйте защищенный файл конфигурации.

Файл открытой конфигурации:

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"]
}
```

Файл защищенной конфигурации:  

```json
{
  "commandToExecute": "./config-music.sh <param1>"
}
```

Команда интерфейса командной строки Azure:

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \ 
  --name customScript \
  --publisher Microsoft.Azure.Extensions \
  --settings ./script-config.json \
  --protected-settings ./protected-config.json
```

## <a name="troubleshooting"></a>Диагностика
Расширение пользовательских сценариев при выполнении создает или загружает сценарий в каталог, как показано в примере ниже. Выходные данные команды также сохраняются в этот каталог в файлах `stdout` и `stderr`.

```bash
/var/lib/waagent/custom-script/download/0/
```

Чтобы приступить к устранению неполадок, сначала просмотрите журнал агента Linux, убедитесь, что расширение запускалось, проверьте:

```bash
/var/log/waagent.log 
```

Необходимо обратить внимание на выполнение расширения. Оно будет выглядеть примерно так:

```output
2018/04/26 17:47:22.110231 INFO [Microsoft.Azure.Extensions.customScript-2.0.6] [Enable] current handler state is: notinstalled
2018/04/26 17:47:22.306407 INFO Event: name=Microsoft.Azure.Extensions.customScript, op=Download, message=Download succeeded, duration=167
2018/04/26 17:47:22.339958 INFO [Microsoft.Azure.Extensions.customScript-2.0.6] Initialize extension directory
2018/04/26 17:47:22.368293 INFO [Microsoft.Azure.Extensions.customScript-2.0.6] Update settings file: 0.settings
2018/04/26 17:47:22.394482 INFO [Microsoft.Azure.Extensions.customScript-2.0.6] Install extension [bin/custom-script-shim install]
2018/04/26 17:47:23.432774 INFO Event: name=Microsoft.Azure.Extensions.customScript, op=Install, message=Launch command succeeded: bin/custom-script-shim install, duration=1007
2018/04/26 17:47:23.476151 INFO [Microsoft.Azure.Extensions.customScript-2.0.6] Enable extension [bin/custom-script-shim enable]
2018/04/26 17:47:24.516444 INFO Event: name=Microsoft.Azure.Extensions.customScript, op=Enable, message=Launch command succeeded: bin/custom-sc
```

Некоторые замечания
1. Enable — означает момент запуска команды.
2. Download — относится к скачиванию пакета расширения CustomScript из Azure, а не файлов скрипта, указанных в fileUris.


Расширение сценариев Azure создает журнал, который можно найти здесь:

```bash
/var/log/azure/custom-script/handler.log
```

Следует обратить внимание на отдельное выполнение, оно будет выглядеть примерно так:

```output
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event=start
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event=pre-check
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="comparing seqnum" path=mrseq
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="seqnum saved" path=mrseq
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="reading configuration"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="read configuration"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="validating json schema"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="json schema valid"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="parsing configuration json"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="parsed configuration json"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="validating configuration logically"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="validated configuration"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="creating output directory" path=/var/lib/waagent/custom-script/download/0
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="created output directory"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 files=1
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 file=0 event="download start"
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 file=0 event="download complete" output=/var/lib/waagent/custom-script/download/0
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="executing command" output=/var/lib/waagent/custom-script/download/0
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="executing protected commandToExecute" output=/var/lib/waagent/custom-script/download/0
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event="executed command" output=/var/lib/waagent/custom-script/download/0
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event=enabled
time=2018-04-26T17:47:23Z version=v2.0.6/git@1008306-clean operation=enable seq=0 event=end
```

Здесь отображаются следующие компоненты:
* команда Enable, запускающая этот журнал;
* параметры, переданные в расширение;
* расширение, скачивающее файл, и результат скачивания;
* выполняемая команда и результат выполнения.

Можно также получить состояние выполнения расширения пользовательских скриптов, включая фактические аргументы, переданные в качестве с `commandToExecute` помощью Azure CLI:

```azurecli
az vm extension list -g myResourceGroup --vm-name myVM
```

Выходные данные выглядят так:

```output
[
  {
    "autoUpgradeMinorVersion": true,
    "forceUpdateTag": null,
    "id": "/subscriptions/subscriptionid/resourceGroups/rgname/providers/Microsoft.Compute/virtualMachines/vmname/extensions/customscript",
    "resourceGroup": "rgname",
    "settings": {
      "commandToExecute": "sh script.sh > ",
      "fileUris": [
        "https://catalogartifact.azureedge.net/publicartifacts/scripts/script.sh",
        "https://catalogartifact.azureedge.net/publicartifacts/scripts/script.sh"
      ]
    },
    "tags": null,
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "typeHandlerVersion": "2.0",
    "virtualMachineExtensionType": "CustomScript"
  },
  {
    "autoUpgradeMinorVersion": true,
    "forceUpdateTag": null,
    "id": "/subscriptions/subscriptionid/resourceGroups/rgname/providers/Microsoft.Compute/virtualMachines/vmname/extensions/OmsAgentForLinux",
    "instanceView": null,
    "location": "eastus",
    "name": "OmsAgentForLinux",
    "protectedSettings": null,
    "provisioningState": "Succeeded",
    "publisher": "Microsoft.EnterpriseCloud.Monitoring",
    "resourceGroup": "rgname",
    "settings": {
      "workspaceId": "workspaceid"
    },
    "tags": null,
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "typeHandlerVersion": "1.0",
    "virtualMachineExtensionType": "OmsAgentForLinux"
  }
]
```

## <a name="next-steps"></a>Дальнейшие действия
Код, текущие проблемы и версии доступны в [репозитории расширения CustomScript](https://github.com/Azure/custom-script-extension-linux).
