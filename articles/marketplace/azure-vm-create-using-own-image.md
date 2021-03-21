---
title: Создание предложения виртуальной машины Azure в Azure Marketplace с помощью собственного образа
description: Узнайте, как опубликовать предложение виртуальной машины в Azure Marketplace с помощью собственного образа.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: krsh
ms.author: krsh
ms.date: 03/10/2021
ms.openlocfilehash: 4711ea76af83594ec529cfda13a308fbe6646398
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103200460"
---
# <a name="how-to-create-a-virtual-machine-using-your-own-image"></a>Создание виртуальной машины с помощью собственного образа

В этой статье описывается создание и развертывание образа виртуальной машины, предоставленной пользователем.

> [!NOTE]
> Перед началом этой процедуры ознакомьтесь с [техническими требованиями](marketplace-virtual-machines.md#technical-requirements) для предложений виртуальных машин Azure, включая требования к виртуальному жесткому диску (VHD).

Чтобы вместо этого использовать утвержденный базовый образ, следуйте инструкциям в разделе [Создание образа виртуальной машины из утвержденной базы](azure-vm-create-using-approved-base.md).

## <a name="configure-the-vm"></a>Настройка виртуальной машины

В этом разделе описано обновление и подготовка виртуальной машины Azure, а также определение ее размера. Эти шаги нужны для того, чтобы подготовить виртуальную машину к развертыванию в Azure Marketplace.

### <a name="size-the-vhds"></a>Размер виртуальных жестких дисков

[!INCLUDE [Discussion of VHD sizing](includes/vhd-size.md)]

### <a name="install-the-most-current-updates"></a>Установка самых свежих обновлений

[!INCLUDE [Discussion of most current updates](includes/most-current-updates.md)]

### <a name="perform-more-security-checks"></a>Выполнение дополнительных проверок безопасности

[!INCLUDE [Discussion of addition security checks](includes/additional-security-checks.md)]

### <a name="perform-custom-configuration-and-scheduled-tasks"></a>Выполнение пользовательской настройки и запланированные задачи

[!INCLUDE [Discussion of custom configuration and scheduled tasks](includes/custom-config.md)]

### <a name="generalize-the-image"></a>Обобщение образа

Все образы в Azure Marketplace должны быть доступны для повторного использования в первоначальном виде. Для этой цели виртуальный жесткий диск операционной системы необходимо подготовить (обобщить). Эта операция удаляет с виртуальной машины все идентификаторы, относящиеся к определенному экземпляру, и программные драйверы.

## <a name="bring-your-image-into-azure"></a>Перенесите образ в Azure

Существует три способа переноса образа в Azure.

1. Отправьте виртуальный жесткий диск в общую галерею образов (SIG).
1. Отправьте виртуальный жесткий диск в учетную запись хранения Azure.
1. Извлечение виртуального жесткого диска из управляемого образа (при использовании служб Image строит службы).

Эти параметры описаны в следующих трех разделах.

### <a name="option-1-upload-the-vhd-as-shared-image-gallery"></a>Вариант 1. Отправка виртуального жесткого диска в качестве коллекции общих образов

1. Отправьте виртуальные жесткие диски в учетную запись хранения.
2. На портал Azure выполните поиск по запросу **развертывание настраиваемого шаблона**.
3. Выберите **Создать собственный шаблон в редакторе**.
4. Скопируйте следующий шаблон Azure Resource Manager (ARM).

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "sourceStorageAccountResourceId": {
          "type": "string",
          "metadata": {
            "description": "Resource ID of the source storage account that the blob vhd resides in."
          }
        },
        "sourceBlobUri": {
          "type": "string",
          "metadata": {
            "description": "Blob Uri of the vhd blob (must be in the storage account provided.)"
          }
        },
        "sourceBlobDataDisk0Uri": {
          "type": "string",
          "metadata": {
            "description": "Blob Uri of the vhd blob (must be in the storage account provided.)"
          }
        },
        "sourceBlobDataDisk1Uri": {
          "type": "string",
          "metadata": {
            "description": "Blob Uri of the vhd blob (must be in the storage account provided.)"
          }
        },
        "galleryName": {
          "type": "string",
          "metadata": {
            "description": "Name of the Shared Image Gallery."
          }
        },
        "galleryImageDefinitionName": {
          "type": "string",
          "metadata": {
            "description": "Name of the Image Definition."
          }
        },
        "galleryImageVersionName": {
          "type": "string",
          "metadata": {
            "description": "Name of the Image Version - should follow <MajorVersion>.<MinorVersion>.<Patch>."
          }
        }
      },
      "resources": [
        {
          "type": "Microsoft.Compute/galleries/images/versions",
          "name": "[concat(parameters('galleryName'), '/', parameters('galleryImageDefinitionName'), '/', parameters('galleryImageVersionName'))]",
          "apiVersion": "2020-09-30",
          "location": "[resourceGroup().location]",
          "properties": {
            "storageProfile": {
              "osDiskImage": {
                "source": {
                  "id": "[parameters('sourceStorageAccountResourceId')]",
                  "uri": "[parameters('sourceBlobUri')]"
                }
              },
    
              "dataDiskImages": [
                {
                  "lun": 0,
                  "source": {
                    "id": "[parameters('sourceStorageAccountResourceId')]",
                    "uri": "[parameters('sourceBlobDataDisk0Uri')]"
                  }
                },
                {
                  "lun": 1,
                  "source": {
                    "id": "[parameters('sourceStorageAccountResourceId')]",
                    "uri": "[parameters('sourceBlobDataDisk1Uri')]"
                  }
                }
              ]
            }
          }
        }
      ]
    }
    
    ```

5. Вставьте шаблон в редактор.

    :::image type="content" source="media/create-vm/vm-sample-code-screen.png" alt-text="Образец экрана кода для виртуальной машины.":::

1. Щелкните **Сохранить**.
1. Используйте параметры в этой таблице, чтобы заполнить поля на экране ниже.

| Параметры | Описание |
| --- | --- |
| саурцесторажеаккаунтресаурцеид | Идентификатор ресурса исходной учетной записи хранения, в которой находится виртуальный жесткий диск большого двоичного объекта.<br><br>Чтобы получить идентификатор ресурса, перейдите в **учетную запись хранения** на **портал Azure**, перейдите в раздел " **Свойства**" и скопируйте значение **ResourceId** . |
| саурцеблобури | URI большого двоичного объекта VHD диска ОС (должен быть указан в предоставленной учетной записи хранения).<br><br>Чтобы получить URL-адрес большого двоичного объекта, перейдите в **учетную запись хранения** на **портал Azure**, перейдите к своему **большому двоичному объекту** и скопируйте значение **URL-адреса** . |
| sourceBlobDataDisk0Uri | Универсальный код ресурса (URI) большого двоичного объекта виртуального жесткого диска данных (должен быть указан в предоставленной учетной записи хранения). Если у вас нет диска данных, удалите этот параметр из шаблона.<br><br>Чтобы получить URL-адрес большого двоичного объекта, перейдите в **учетную запись хранения** на **портал Azure**, перейдите к своему **большому двоичному объекту** и скопируйте значение **URL-адреса** . |
| sourceBlobDataDisk1Uri | Универсальный код ресурса (URI) дополнительного BLOB-объекта виртуального жесткого диска данных (должен быть указан в предоставленной учетной записи хранения). Если у вас нет дополнительного диска данных, удалите этот параметр из шаблона.<br><br>Чтобы получить URL-адрес большого двоичного объекта, перейдите в **учетную запись хранения** на **портал Azure**, перейдите к своему **большому двоичному объекту** и скопируйте значение **URL-адреса** . |
| галлеринаме | Имя коллекции общих образов |
| галлеримажедефинитионнаме | Имя определения образа |
| галлеримажеверсионнаме | Имя создаваемой версии образа в следующем формате: `<MajorVersion>.<MinorVersion>.<Patch>` |
|

:::image type="content" source="media/create-vm/custom-deployment-window.png" alt-text="Отображает настраиваемое окно развертывания.":::

8. Выберите **Review + create** (Просмотреть и создать). После завершения проверки выберите **создать**.

> [!TIP]
> Учетная запись издателя должна иметь доступ владельца для публикации образа SIG. При необходимости выполните следующие действия, чтобы предоставить доступ.
>
> 1. Перейдите в коллекцию общих образов (SIG).
> 2. На панели слева выберите **Управление доступом** (IAM).
> 3. Выберите **Добавить**, а затем — **добавить назначение ролей**.
> 4. В качестве **роли** выберите **владелец**.
> 5. **Чтобы назначить доступ**, выберите **пользователь, группа или субъект-служба**.
> 6. Введите адрес электронной почты Azure пользователя, который будет публиковать образ.
> 7. Щелкните **Сохранить**.<br><br>
> :::image type="content" source="media/create-vm/add-role-assignment.png" alt-text="Отобразится окно Добавление назначения роли.":::

### <a name="option-2-upload-the-vhd-to-a-storage-account"></a>Вариант 2. Отправка виртуального жесткого диска в учетную запись хранения

Настройте и подготовьте виртуальную машину, которая будет отправлена, как описано в статье [Подготовка VHD или VHDX Windows к отправке в Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md) или [Создание и передача виртуального жесткого диска Linux](../virtual-machines/linux/create-upload-generic.md).

### <a name="option-3-extract-the-vhd-from-managed-image-if-using-image-building-services"></a>Вариант 3. Извлечение виртуального жесткого диска из управляемого образа (при использовании служб Image строит службы)

Если вы используете службу создания образов, например [Pack](https://www.packer.io/), возможно, потребуется извлечь VHD из образа. Для этого нет прямого способа. Вам потребуется создать виртуальную машину и извлечь виртуальный жесткий диск с диска виртуальной машины.

## <a name="create-the-vm-on-the-azure-portal"></a>Создание виртуальной машины на портал Azure

Выполните следующие действия, чтобы создать базовый образ виртуальной машины на [портал Azure](https://ms.portal.azure.com/).

1. Войдите на [портал Azure](https://ms.portal.azure.com/).
2. Выберите **Виртуальные машины**.
3. Выберите **+ Добавить** , чтобы открыть экран **Создание виртуальной машины** .
4. Выберите изображение из раскрывающегося списка или выберите **Просмотреть все общедоступные и частные образы** , чтобы найти или просмотреть все доступные образы виртуальных машин.
5. Чтобы создать виртуальную машину **Gen 2** , перейдите на вкладку **Дополнительно** и выберите параметр **Gen 2** .

    :::image type="content" source="media/create-vm/vm-gen-option.png" alt-text="Выберите Gen 1 или Gen 2.":::

6. Выберите размер виртуальной машины для развертывания.

    :::image type="content" source="media/create-vm/create-virtual-machine-sizes.png" alt-text="Выберите рекомендуемый размер виртуальной машины для выбранного образа.":::

7. Укажите другие необходимые сведения для создания виртуальной машины.
8. Выберите **Просмотр и создание**, чтобы проверить выбранные параметры. Когда появится сообщение **Проверка пройдена** , выберите **создать**.

Azure начнет подготовку указанной вами виртуальной машины. Чтобы отслеживать ход выполнения, выберите вкладку **виртуальные машины** в меню слева. После создания состояние виртуальных машин изменится на **работает**.

## <a name="connect-to-your-vm"></a>Подключение к виртуальной машине

Обратитесь к следующей документации, чтобы подключиться к виртуальной машине [Windows](../virtual-machines/windows/connect-logon.md) или [Linux](../virtual-machines/linux/ssh-from-windows.md#connect-to-your-vm) .

[!INCLUDE [Discussion of addition security checks](includes/size-connect-generalize.md)]

## <a name="next-steps"></a>Следующие шаги

- [Протестируйте образ виртуальной машины](azure-vm-image-test.md) , чтобы убедиться, что он соответствует требованиям к публикации Azure Marketplace. Это необязательно.
- Если вы не хотите протестировать образ виртуальной машины, войдите в [Центр партнеров](https://partner.microsoft.com/) и ОПУБЛИКУЙТЕ образ SIG (параметр #1).
- Если вы присвоены параметру #2 или #3, [Создайте URI SAS](azure-vm-get-sas-uri.md).
- Если вы столкнулись с трудностями при создании нового виртуального жесткого диска на основе Azure, см. раздел [вопросы и ответы о виртуальной машине для Azure Marketplace](azure-vm-create-faq.md).
