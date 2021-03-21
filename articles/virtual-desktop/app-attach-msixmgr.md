---
title: Использование средства МСИКСМГР — Azure
description: Использование средства МСИКСМГР для виртуальных рабочих столов Windows.
author: Heidilohr
ms.topic: how-to
ms.date: 02/23/2021
ms.author: helohr
manager: lizross
ms.openlocfilehash: 4b34fb0d3bb2d49255007b9722a0a636c1441b8c
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101746282"
---
# <a name="using-the-msixmgr-tool"></a>Использование средства МСИКСМГР

Средство МСИКСМГР предназначено для расширения MSIX приложений в образы MSIX. Средство принимает приложение MSIX (. MSIX) и разворачивает его в файл VHD, VHDx или CIM. Полученный образ MSIX хранится в учетной записи хранения Azure, используемой развертыванием виртуальных рабочих столов Windows. В этой статье показано, как использовать средство МСИКСМГР.

>[!NOTE]
>Чтобы обеспечить совместимость, убедитесь, что Цимс, где хранятся образы MSIX, создаются в версии ОС, которую вы используете в пулах узлов виртуальных рабочих столов Windows. МСИКСМГР может создавать файлы CIM, но эти файлы можно использовать только с пулом узлов под Windows 10 20H2.

## <a name="requirements"></a>Требования

Прежде чем выполнять инструкции в этой статье, необходимо выполнить следующие действия.

- [Загрузка средства МСИКСМГР](https://aka.ms/msixmgr)
- Получите приложение, упакованное в MSIX (. Файл MSIX)
- Получите административные разрешения на компьютере, где вы создадите образ MSIX.

## <a name="create-an-msix-image"></a>Создание образа MSIX

Расширение — это процесс создания упакованного приложения MSIX (. MSIX) и расраспаковку в образ MSIX (. VHD (x) или. CIM-файл).

Чтобы развернуть файл MSIX, выполните следующие действия.

1. [Скачайте средство мсиксмгр](https://aka.ms/msixmgr) , если вы еще этого не сделали.

2. Распакуйте MSIXMGR.zip в локальную папку.

3. Откройте командную строку в режиме с повышенными правами.

4. Найдите локальную папку на шаге 2.

5. Выполните следующую команду в командной строке, чтобы создать образ MSIX.

    ```cmd
    msixmgr.exe -Unpack -packagePath <path to package> -destination <output folder> [-applyacls] [-create] [-vhdSize <size in MB>] [-filetype <CIM | VHD | VHDX>] [-rootDirectory <rootDirectory>]
    ```

    Не забудьте заменить значения заполнителей соответствующими значениями. Пример:

    ```cmd
    msixmgr.exe -Unpack -packagePath "C:\Users\%username%\Desktop\packageName_3.51.1.0_x64__81q6ced8g4aa0.msix" -destination "c:\temp\packageName.vhdx" -applyacls -create -vhdSize 200 -filetype "vhdx" -rootDirectory apps
    ```

6. Теперь, когда вы создали образ, перейдите в папку назначения и убедитесь, что вы успешно создали образ MSIX (. VHDX).

## <a name="create-an-msix-image-in-a-cim-file"></a>Создание образа MSIX в CIM-файле

Можно также использовать команду на [шаге 5](#create-an-msix-image) для создания файлов CIM и VHDX путем замены типа файла и пути назначения.

Например, вот как можно использовать эту команду для создания файла CIM:

```cmd
msixmgr.exe -Unpack -packagePath "C:\Users\ssa\Desktop\packageName_3.51.1.0_x64__81q6ced8g4aa0.msix" -destination "c:\temp\packageName.cim" -applyacls -create -vhdSize 200 -filetype "cim" -rootDirectory apps
```

Вот как можно использовать эту команду для создания VHDX:

```cmd
msixmgr.exe -Unpack -packagePath "C:\Users\ssa\Desktop\packageName_3.51.1.0_x64__81q6ced8g4aa0.msix" -destination "c:\temp\packageName.vhdx" -applyacls -create -vhdSize 200 -filetype "vhdx" -rootDirectory apps
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о подключении приложения MSIX см [. в разделе что такое присоединение приложения MSIX?](what-is-app-attach.md)

Чтобы узнать, как настроить присоединение к приложению, ознакомьтесь со следующими статьями:

- [Настройка технологии подключения приложений MSIX с использованием портала Azure](app-attach-azure-portal.md)
- [Настройка подключения приложения MSIX с помощью PowerShell](app-attach-powershell.md)
- [Создание скриптов PowerShell для подключения к приложению MSIX](app-attach.md)
- [Подготовка образа MSIX для виртуального рабочего стола Windows](app-attach-image-prep.md)
- [Настройка общей папки для подключения к приложению MSIX](app-attach-file-share.md)

Если у вас возникли вопросы о подключении приложения MSIX, см. раздел [часто задаваемые вопросы о подключении к приложению](app-attach-faq.md) и [Глоссарий прикрепления приложений](app-attach-glossary.md).
