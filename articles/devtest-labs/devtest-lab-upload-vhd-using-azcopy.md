---
title: Отправка VHD-файла в Azure Labs DevTest с помощью AzCopy | Документация Майкрософт
description: В этой статье содержатся пошаговые инструкции по использованию служебной программы командной строки AzCopy для передачи VHD-файла в учетную запись хранения лаборатории в Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 1d8ede0f78726b04ac862a00b559b8d42c3ed1cd
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88650780"
---
# <a name="upload-vhd-file-to-labs-storage-account-using-azcopy"></a>Отправка VHD-файла в учетную запись хранения лаборатории с помощью AzCopy

[!INCLUDE [devtest-lab-upload-vhd-selector](../../includes/devtest-lab-upload-vhd-selector.md)]

В Azure DevTest Labs можно использовать VHD-файлы для создания пользовательских образов, которые используются при подготовке виртуальных машин. Ниже приведены пошаговые инструкции по отправке VHD-файла в учетную запись хранения лаборатории с помощью служебной программы командной строки AzCopy. После отправки VHD-файла переходите к разделу [Дальнейшие действия](#next-steps), где перечислены несколько статей, в которых описывается создание пользовательского образа из загруженного VHD-файла. Дополнительные сведения о дисках и VHD в Azure см. в статье [Обзор компонента "Управляемые диски" Azure](../virtual-machines/managed-disks-overview.md).

> [!NOTE] 
>  
> AzCopy является служебной программой командной строки только для Windows.

## <a name="step-by-step-instructions"></a>Пошаговые инструкции

Ниже приведены пошаговые инструкции по отправке VHD-файла в Azure DevTest Labs с помощью [AzCopy](https://aka.ms/downloadazcopy). 

1. Получите имя учетной записи хранения лаборатории с помощью портала Azure.

1. Войдите на [портал Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Выберите **все службы**, а затем в списке выберите **DevTest Labs** .

1. Из списка лабораторий выберите нужную лабораторию.  

1. В колонке лаборатории выберите **Конфигурация**. 

1. В колонке **Configuration** (Конфигурация) лаборатории выберите **Custom images (VHDs)** (Пользовательские образы (VHD)).

1. В колонке **Custom images** (Пользовательские образы) выберите **+Add** (+ Добавить). 

1. В колонке **Custom images** (Пользовательские образы) выберите **VHD**.

1. В колонке **VHD** щелкните **Upload a VHD file using PowerShell** (Отправить VHD-файл с помощью PowerShell).

    ![Отправка VHD с помощью PowerShell](./media/devtest-lab-upload-vhd-using-azcopy/upload-image-using-psh.png)

1. Колонка **Upload an image using PowerShell** (Отправка образа с помощью PowerShell) отображает вызов командлета **Add-AzureVhd**. Первый параметр (*Назначение*) содержит URI для контейнера больших двоичных объектов (*отправки*) в следующем формате:

    ```
    https://<STORAGE-ACCOUNT-NAME>.blob.core.windows.net/uploads/...
    ``` 

1. Запишите полный URI, так как он понадобится вам дальше.

1. Отправьте VHD-файл с помощью AzCopy.
 
1. [Скачайте и установите последнюю версию AzCopy](https://aka.ms/downloadazcopy).

1. Откройте окно командной строки и перейдите к каталогу установки AzCopy. При необходимости можно добавить место установки AzCopy к системному пути. По умолчанию AzCopy устанавливается в следующий каталог:

    ```command-line
    %ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy
    ```

1. Используя ключ учетной записи и URI контейнера больших двоичных объектов, выполните следующую команду в командной строке. Значение *vhdFileName* должно быть заключено в кавычки. В зависимости от размера VHD-файла и скорости подключения отправка файла может быть продолжительной.   

    ```command-line
    AzCopy /Source:<sourceDirectory> /Dest:<blobContainerUri> /DestKey:<storageAccountKey> /Pattern:"<vhdFileName>" /BlobType:page
    ```

## <a name="next-steps"></a>Дальнейшие действия

- [Управление пользовательскими образами Azure DevTest Labs для создания виртуальных машин](devtest-lab-create-template.md)
- [Создание пользовательского образа из VHD-файла с помощью PowerShell](devtest-lab-create-custom-image-from-vhd-using-powershell.md)