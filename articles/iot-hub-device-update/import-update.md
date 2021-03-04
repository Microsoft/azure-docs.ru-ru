---
title: Импорт нового обновления | Документация Майкрософт
description: How-To инструкции по импорту нового обновления центра Интернета вещей для центра Интернета вещей.
author: andrewbrownmsft
ms.author: andbrown
ms.date: 2/11/2021
ms.topic: how-to
ms.service: iot-hub-device-update
ms.openlocfilehash: c83221743e0566d783c38c40aaf92111a0cd80f7
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102030738"
---
# <a name="import-new-update"></a>Импорт нового обновления
Узнайте, как импортировать новое обновление в центр обновления для центра Интернета вещей.

## <a name="prerequisites"></a>Предварительные требования

* [Доступ к центру Интернета вещей с включенным обновлением устройства для центра Интернета вещей](create-device-update-account.md). Для центра Интернета вещей рекомендуется использовать уровень S1 (стандартный) или выше. 
* Устройство IoT (или симулятор), подготовленное для обновления устройства в центре Интернета вещей.
   * При использовании реального устройства вам потребуется файл образа обновления для обновления образа или [файл манифеста APT](device-update-apt-manifest.md) для обновления пакета.
* [PowerShell 5](https://docs.microsoft.com/powershell/scripting/install/installing-powershell) или более поздней версии.
* Поддерживаемые браузеры:
  * [Microsoft Edge](https://www.microsoft.com/edge)
  * Google Chrome;

> [!NOTE]
> Некоторые данные, отправленные в эту службу, могут обрабатываться в регионе за пределами региона, в котором был создан этот экземпляр.

## <a name="create-device-update-import-manifest"></a>Создание манифеста импорта обновления устройства

1. Убедитесь, что файл образа обновления или файл манифеста APT находятся в каталоге, доступном из PowerShell.

2. Клонирование [обновления устройства в репозитории центра Интернета вещей](https://github.com/azure/iot-hub-device-update)или скачивание его в виде ZIP-файла в расположение, доступное из PowerShell (после скачивания ZIP-файла щелкните правой кнопкой мыши для `Properties`  >  `General` перехода на вкладку > проверьте `Unblock` раздел, `Security` чтобы избежать появления предупреждений о безопасности PowerShell).

3. В PowerShell перейдите в `tools/AduCmdlets` каталог и запустите:

    ```powershell
    Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
    Import-Module .\AduUpdate.psm1
    ```

4. Выполните следующие команды, заменив значения параметров выборки, чтобы создать манифест импорта — JSON-файл, описывающий обновление:
    ```powershell
    $compat = New-AduUpdateCompatibility -DeviceManufacturer 'deviceManufacturer' -DeviceModel 'deviceModel'

    $importManifest = New-AduImportManifest -Provider 'updateProvider' -Name 'updateName' -Version 'updateVersion' `
                                            -UpdateType 'updateType' -InstalledCriteria 'installedCriteria' `
                                            -Compatibility $compat -Files 'updateFilePath(s)'

    $importManifest | Out-File '.\importManifest.json' -Encoding UTF8
    ```

    Для краткого справочника ниже приведены некоторые примеры значений для перечисленных выше параметров. Для получения дополнительных сведений можно также просмотреть полную [схему манифеста импорта](import-schema.md) .

    | Параметр | Описание |
    | --------- | ----------- |
    | deviceManufacturer | Изготовитель устройства, совместимое с обновлением, например contoso
    | deviceModel | Модель устройства, совместимое с обновлением, например тостер
    | упдатепровидер | Элемент Provider удостоверения обновления, например fabrikam
    | упдатенаме | Имя часть идентификатора обновления, например Имажеупдате
    | упдатеверсион | Версия обновления, например 2,0
    | updateType | <ul><li>Укажите `microsoft/swupdate:1` для обновления образа</li><li>Укажите `microsoft/apt:1` для обновления пакета</li></ul>
    | инсталледкритериа | <ul><li>Укажите значение Свверсион для `microsoft/swupdate:1` типа обновления</li><li>Укажите рекомендуемое значение для `microsoft/apt:1` типа обновления.
    | Упдатефилепас | Путь к файлам обновления на компьютере


## <a name="review-generated-import-manifest"></a>Проверить созданный манифест импорта

Пример
```json
{
  "updateId": {
    "provider": "Microsoft",
    "name": "Toaster",
    "version": "2.0"
  },
  "updateType": "microsoft/swupdate:1",
  "installedCriteria": "5",
  "compatibility": [
    {
      "deviceManufacturer": "Fabrikam",
      "deviceModel": "Toaster"
    },
    {
      "deviceManufacturer": "Contoso",
      "deviceModel": "Toaster"
    }
  ],
  "files": [
    {
      "filename": "file1.json",
      "sizeInBytes": 7,
      "hashes": {
        "sha256": "K2mn97qWmKSaSaM9SFdhC0QIEJ/wluXV7CoTlM8zMUo="
      }
    },
    {
      "filename": "file2.zip",
      "sizeInBytes": 11,
      "hashes": {
        "sha256": "gbG9pxCr9RMH2Pv57vBxKjm89uhUstD06wvQSioLMgU="
      }
    }
  ],
  "createdDateTime": "2020-10-08T03:32:52.477Z",
  "manifestVersion": "2.0"
}
```

## <a name="import-update"></a>Импорт обновления

1. Войдите в [портал Azure](https://portal.azure.com) и перейдите в центр Интернета вещей с помощью обновления устройства.

2. В левой части страницы выберите "обновления устройства" в разделе "автоматическое управление устройствами".

   :::image type="content" source="media/import-update/import-updates-3.png" alt-text="Импорт обновлений" lightbox="media/import-update/import-updates-3.png":::

3. В верхней части экрана вы увидите несколько вкладок. Перейдите на вкладку обновления.

   :::image type="content" source="media/import-update/updates-tab.png" alt-text="Обновления" lightbox="media/import-update/updates-tab.png":::

4. Выберите "+ Импорт нового обновления" под заголовком "готово к развертыванию".

   :::image type="content" source="media/import-update/import-new-update-2.png" alt-text="Импорт нового обновления" lightbox="media/import-update/import-new-update-2.png":::

5. Выберите значок папки или текстовое поле в разделе "выберите файл импорта манифеста". Вы увидите диалоговое окно выбора файлов. Выберите созданный ранее манифест импорта с помощью командлета PowerShell. Затем выберите значок папки или текстовое поле в разделе "выберите один или несколько файлов обновлений". Вы увидите диалоговое окно выбора файлов. Выберите файлы обновлений.

   :::image type="content" source="media/import-update/select-update-files.png" alt-text="Выбор файлов обновления" lightbox="media/import-update/select-update-files.png":::

6. Выберите значок папки или текстовое поле в разделе "выберите контейнер хранилища". Затем выберите соответствующую учетную запись хранения. Контейнер хранилища используется для временного размещения файлов обновления.

   :::image type="content" source="media/import-update/storage-account.png" alt-text="Учетная запись хранения" lightbox="media/import-update/storage-account.png":::

7. Если вы уже создали контейнер, его можно использовать повторно. (В противном случае выберите "+ Container", чтобы создать новый контейнер хранилища для обновлений.)  Выберите контейнер, который вы хотите использовать, и нажмите кнопку "выбрать".

   :::image type="content" source="media/import-update/container.png" alt-text="Выбор контейнера" lightbox="media/import-update/container.png":::

8. Нажмите кнопку "Отправить", чтобы начать процесс импорта.

   :::image type="content" source="media/import-update/publish-update.png" alt-text="Публикация обновления" lightbox="media/import-update/publish-update.png":::

9. Начнется процесс импорта, и на экране будет переключаться к разделу "журнал импорта". Выберите "Обновить" для просмотра хода выполнения до завершения процесса импорта (в зависимости от размера обновления это может завершиться через несколько минут, но может занять больше времени).

   :::image type="content" source="media/import-update/update-publishing-sequence-2.png" alt-text="Обновление последовательности импорта" lightbox="media/import-update/update-publishing-sequence-2.png":::

10. Если в столбце состояние указано, что импорт выполнен успешно, выберите заголовок "готово к развертыванию". Импортированное обновление появится в списке сейчас.

   :::image type="content" source="media/import-update/update-ready.png" alt-text="Состояние задания" lightbox="media/import-update/update-ready.png":::

## <a name="next-steps"></a>Next Steps

[Создание групп](create-update-group.md)

[Сведения о концепциях импорта](import-concepts.md)
