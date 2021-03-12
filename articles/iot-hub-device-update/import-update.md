---
title: Импорт нового обновления | Документация Майкрософт
description: How-To инструкции по импорту нового обновления центра Интернета вещей для центра Интернета вещей.
author: andrewbrownmsft
ms.author: andbrown
ms.date: 2/11/2021
ms.topic: how-to
ms.service: iot-hub-device-update
ms.openlocfilehash: b9d40848abdd85beeca592001b697e3c50b7cd59
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103008568"
---
# <a name="import-new-update"></a>Импорт нового обновления
Узнайте, как импортировать новое обновление в центр обновления для центра Интернета вещей. Если вы еще не сделали этого, обязательно ознакомьтесь с основными [понятиями импорта](import-concepts.md).

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

2. Создайте текстовый файл с именем **адуупдате. PSM1** в каталоге, где находится файл образа обновления или файл манифеста APT. Затем откройте командлет PowerShell [адуупдате. PSM1](https://github.com/Azure/iot-hub-device-update/tree/main/tools/AduCmdlets) , скопируйте содержимое в текстовый файл, а затем сохраните текстовый файл.

3. В PowerShell перейдите в каталог, в котором был создан командлет PowerShell, из шага 2. Далее выполните:

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
    | deviceManufacturer | Изготовитель устройства, совместимое с обновлением, например contoso. Должно соответствовать  [свойству устройства](https://docs.microsoft.com/azure/iot-hub-device-update/device-update-plug-and-play#device-properties)производителя.
    | deviceModel | Модель устройства, совместимое с обновлением, например тостер. Должно соответствовать  [свойству устройства](https://docs.microsoft.com/azure/iot-hub-device-update/device-update-plug-and-play#device-properties)модели.
    | упдатепровидер | Сущность, которая создает или непосредственно отвечает за обновление. Часто это будет название компании.
    | упдатенаме | Идентификатор класса обновлений. Класс может быть любым выбранным. Как правило, это имя устройства или модели.
    | упдатеверсион | Номер версии, отличающий это обновление от других, имеющих одного и того же поставщика и имя. Не соответствует версии отдельного программного компонента на устройстве (но может быть, если вы выбрали).
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

## <a name="import-update"></a>Импорт обновлений

[!NOTE]
В приведенных ниже инструкциях показано, как импортировать обновление с помощью пользовательского интерфейса портал Azure. Вы также можете использовать [обновление устройства для API центра Интернета вещей](https://github.com/Azure/iot-hub-device-update/tree/main/docs/publish-api-reference) , чтобы импортировать обновление. 

1. Войдите в [портал Azure](https://portal.azure.com) и перейдите в центр Интернета вещей с помощью обновления устройства.

2. В левой части страницы выберите "обновления устройства" в разделе "автоматическое управление устройствами".

   :::image type="content" source="media/import-update/import-updates-3.png" alt-text="Импорт обновлений" lightbox="media/import-update/import-updates-3.png":::

3. В верхней части экрана вы увидите несколько вкладок. Откройте вкладку "Обновления".

   :::image type="content" source="media/import-update/updates-tab.png" alt-text="Обновления" lightbox="media/import-update/updates-tab.png":::

4. Выберите "+ Импорт нового обновления" под заголовком "готово к развертыванию".

   :::image type="content" source="media/import-update/import-new-update-2.png" alt-text="Импорт нового обновления" lightbox="media/import-update/import-new-update-2.png":::

5. Щелкните значок папки или текстовое поле в разделе "Выберите файл манифеста импорта". На экране появится диалоговое окно выбора файлов. Выберите созданный ранее манифест импорта с помощью командлета PowerShell. Затем щелкните значок папки или текстовое поле в разделе "Выберите один или несколько файлов обновлений". На экране появится диалоговое окно выбора файлов. Выберите файлы обновлений.

   :::image type="content" source="media/import-update/select-update-files.png" alt-text="Выбор файлов обновления" lightbox="media/import-update/select-update-files.png":::

6. Щелкните значок папки или текстовое поле в разделе "Выберите контейнер хранилища". Затем выберите соответствующую учетную запись хранения. Контейнер хранилища используется для временного размещения файлов обновления.

   :::image type="content" source="media/import-update/storage-account.png" alt-text="Учетная запись хранения" lightbox="media/import-update/storage-account.png":::

7. Если контейнер уже создан, его можно использовать повторно. (В противном случае выберите "+ Контейнер", чтобы создать новый контейнер хранилища для обновлений.)  Выделите контейнер для использования и нажмите кнопку "Выбрать".

   :::image type="content" source="media/import-update/container.png" alt-text="Выбор контейнера" lightbox="media/import-update/container.png":::

8. Нажмите кнопку "Отправить", чтобы запустить процесс импорта.

   :::image type="content" source="media/import-update/publish-update.png" alt-text="Публикация обновления" lightbox="media/import-update/publish-update.png":::

9. Начнется процесс импорта, и на экране будет переключаться к разделу "журнал импорта". Выберите "Обновить" для просмотра хода выполнения до завершения процесса импорта (в зависимости от размера обновления это может завершиться через несколько минут, но может занять больше времени).

   :::image type="content" source="media/import-update/update-publishing-sequence-2.png" alt-text="Обновление последовательности импорта" lightbox="media/import-update/update-publishing-sequence-2.png":::

10. Если столбец "Состояние" сигнализирует, что импорт успешно выполнен, выберите заголовок "Готово к развертыванию". Импортированное обновление должно появиться в списке.

   :::image type="content" source="media/import-update/update-ready.png" alt-text="Состояние задания" lightbox="media/import-update/update-ready.png":::

## <a name="next-steps"></a>Next Steps

[Создание групп](create-update-group.md)

[Сведения о концепциях импорта](import-concepts.md)
