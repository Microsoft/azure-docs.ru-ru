---
title: Использование общей папки Azure в Windows | Документация Майкрософт
description: Узнайте, как использовать файловые ресурсы Azure с Windows и Windows Server. Используйте общие файловые ресурсы Azure с SMB 3,0 в операционных системах Windows, работающих локально или на виртуальных машинах Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e64b7efdd430287a7a3a969c5bf62b0c0e2aec9c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94626900"
---
# <a name="use-an-azure-file-share-with-windows"></a>Использование общей папки Azure в Windows
[Файлы Azure](storage-files-introduction.md) — это простая в использовании облачная файловая система от корпорации Майкрософт. Общие папки Azure можно легко использовать в Windows и Windows Server. В этой статье рассматриваются рекомендации по использованию общей папки Azure в Windows и Windows Server.

Чтобы использовать общую папку Azure за пределами региона Azure, в котором она размещается, например локально или в другом регионе Azure, операционная система должна поддерживать протокол SMB 3.0. 

Общие папки Azure можно использовать в ОС Windows, работающей на виртуальной машине Azure или в локальной среде. В следующей таблице показано, какие версии операционной системы поддерживают доступ к общим папкам и в какой среде:

| Версия Windows        | Версия SMB | Возможность подключения на виртуальной машине Azure | Возможность подключения в локальной среде |
|------------------------|-------------|-----------------------|-----------------------|
| Windows Server 2019 | SMB 3.0 | Да | Да |
| Windows 10<sup>1</sup> | SMB 3.0 | Да | Да |
| Semi-Annual Channel<sup>2</sup> для Windows Server | SMB 3.0 | Да | Да |
| Windows Server 2016 | SMB 3.0 | Да | Да |
| Windows 8.1 | SMB 3.0 | Да | Да |
| Windows Server 2012 R2 | SMB 3.0 | Да | Да |
| Windows Server 2012 | SMB 3.0 | Да | Да |
| Windows 7<sup>3</sup>. | SMB 2.1 | Да | нет |
| Windows Server 2008 R2<sup>3</sup> | SMB 2.1 | Да | Нет |

<sup>1</sup> Windows 10, версии 1507, 1607, 1803, 1809, 1903, 1909 и 2004.  
<sup>2</sup> Windows Server, версии 1809, 1903, 1909, 2004.  
<sup>3</sup>Обычная поддержка Майкрософт для Windows 7 и Windows Server 2008 R2 закончилась. Дополнительную поддержку обновлений для системы безопасности можно приобрести только с помощью программы [Расширенные обновления для системы безопасности (ESU)](https://support.microsoft.com/help/4497181/lifecycle-faq-extended-security-updates). Мы настоятельно рекомендуем отказаться от этих операционных систем.

> [!Note]  
> Мы всегда рекомендуем использовать последнюю версию KB для своей версии Windows.

## <a name="prerequisites"></a>Предварительные требования 

Открытый порт 445. Для протокола SMB требуется, чтобы TCP-порт 445 был открыт. В противном случае установить подключение не получится. Вы можете проверить, блокирует ли брандмауэр порт 445 с помощью `Test-NetConnection` командлета. Дополнительные сведения о способах обхода заблокированного порта 445 см. в разделе [Причина 1. порт 445 заблокирована](storage-troubleshoot-windows-file-connection-problems.md#cause-1-port-445-is-blocked) в нашем руководство по устранению неполадок Windows.

## <a name="using-an-azure-file-share-with-windows"></a>Использование общей папки Azure в Windows
Чтобы использовать общую папку Azure в Windows, вы должны либо подключить ее, то есть присвоить ей букву диска или путь точки подключения, либо получить доступ к ней через [UNC-путь](/windows/win32/fileio/naming-a-file). 

В этой статье предполагается, что для доступа к общей папке используется ключ учетной записи хранения. Это ключ администратора для учетной записи хранения, включая разрешения администратора для всех файлов и папок в общей папке, к которой вы получаете доступ, и для всех общих папок и других ресурсов хранения (большие двоичные объекты, очереди, таблицы и т. д.), которые размещаются в этой учетной записи. Если этого недостаточно для вашей рабочей нагрузки, можно использовать [Синхронизацию файлов Azure](storage-sync-files-planning.md) или [проверку подлинности на основе удостоверений через SMB](storage-files-active-directory-overview.md).

Общим вариантом для переноса бизнес-приложений, которые ожидают наличия подключенной к Azure общей папки SMB, является использование общей папки Azure в качестве альтернативы для запуска выделенного файлового сервера Windows на виртуальной машине Azure. Одна из важных рекомендаций для успешного перевода бизнес-приложения на использование общей папки Azure заключается в том, что многие бизнес-приложения работают в контексте выделенной учетной записи службы с ограниченными системными правами, а не административной учетной записью виртуальной машины. Поэтому нужно обязательно подключить или сохранить учетные данные для общей папки Azure из контекста учетной записи службы, а не вашей учетной записи администратора.

### <a name="mount-the-azure-file-share"></a>Подключение файлового ресурса Azure

Портал Azure предоставляет скрипт, который можно использовать для подключения общей папки непосредственно к узлу. Мы рекомендуем использовать указанный скрипт.

Чтобы получить этот скрипт, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com/).
1. Перейдите к учетной записи хранения, содержащей общую папку, которую вы хотите подключить.
1. Выберите **Общие папки**.
1. Выберите общую папку, которую вы хотите подключить.

    :::image type="content" source="media/storage-how-to-use-files-windows/select-file-shares.png" alt-text="Например":::

1. Выберите **Подключиться**.

    :::image type="content" source="media/storage-how-to-use-files-windows/file-share-connect-icon.png" alt-text="Снимок экрана с значком подключения для файлового ресурса.":::

1. Выберите букву диска для подключения общей папки.
1. Скопируйте предоставленный скрипт.

    :::image type="content" source="media/storage-how-to-use-files-windows/files-portal-mounting-cmdlet-resize.png" alt-text="Пример текста":::

1. Вставьте сценарий в оболочку на узле, на котором вы хотите подключить общую папку, и запустите ее.

Теперь вы подключили файловый ресурс Azure.

### <a name="mount-the-azure-file-share-with-file-explorer"></a>Подключение общей папки Azure с помощью проводника
> [!Note]  
> Имейте в виду, что следующие инструкции приведены для Windows 10 и могут немного отличаться для более поздних версий. 

1. Откройте проводник. Это можно сделать, открыв меню "Пуск" или нажав клавиши WIN+E.

1. Перейдите к **этому компьютеру** в левой части окна. Меню, доступные на ленте, изменятся. В меню "Компьютер" выберите раздел **Подключение сетевого диска**.
    
    ![Снимок экрана раскрывающегося меню "Подключение сетевого диска"](./media/storage-how-to-use-files-windows/1_MountOnWindows10.png)

1. Выберите букву диска и введите UNC-путь, формат UNC-пути — `\\<storageAccountName>.file.core.windows.net\<fileShareName>`. Например: `\\anexampleaccountname.file.core.windows.net\example-share-name`.
    
    ![Снимок экрана диалогового окна "Подключение сетевого диска"](./media/storage-how-to-use-files-windows/2_MountOnWindows10.png)

1. Используйте имя учетной записи хранения с префиксом `AZURE\` как имя пользователя и ключ учетной записи хранения в качестве пароля.
    
    ![Снимок экрана диалогового окна сетевых учетных данных](./media/storage-how-to-use-files-windows/3_MountOnWindows10.png)

1. Выберите общую папку Azure.
    
    ![Общая папка Azure теперь подключена](./media/storage-how-to-use-files-windows/4_MountOnWindows10.png)

1. Когда вы будете готовы отключить общую папку Azure, это можно сделать, щелкнув правой кнопкой мыши запись для общей папки в разделе проводника **Сетевые расположения** и выбрав параметр **Отключить**.

### <a name="accessing-share-snapshots-from-windows"></a>Доступ к моментальным снимкам общих файловых ресурсов из Windows
Если вы уже создали моментальные снимки общего файлового ресурса вручную или автоматически с помощью скрипта или такой службы, как Azure Backup, вы можете просмотреть предыдущие версии общего ресурса, каталога или определенного файла из подключенного общего файлового ресурса в Windows. Моментальный снимок общего ресурса можно создать с помощью [Azure PowerShell](storage-how-to-use-files-powershell.md), [Azure CLI](storage-how-to-use-files-cli.md)или [портал Azure](storage-how-to-use-files-portal.md).

#### <a name="list-previous-versions"></a>Вывод списка предыдущих версий
Перейдите к элементу или родительскому элементу, который требуется восстановить. Дважды щелкните для перехода к нужному каталогу. Щелкните правой кнопкой мыши и выберите в меню пункт **Свойства**.

![Контекстное меню для выбранного каталога](./media/storage-how-to-use-files-windows/snapshot-windows-previous-versions.png)

Выберите пункт **Предыдущие версии**, чтобы просмотреть список моментальных снимков общих ресурсов для этого каталога. Вывод списка может занять несколько секунд в зависимости от скорости сети и количества моментальных снимков в каталоге.

![Вкладка "Предыдущие версии"](./media/storage-how-to-use-files-windows/snapshot-windows-list.png)

Чтобы открыть тот или иной моментальный снимок, нажмите кнопку **Открыть**. 

![Открытый моментальный снимок](./media/storage-how-to-use-files-windows/snapshot-browse-windows.png)

#### <a name="restore-from-a-previous-version"></a>Восстановление из предыдущей версии
Нажмите кнопку **Восстановить**, чтобы рекурсивно скопировать содержимое всего каталога на момент создания моментального снимка общего ресурса в исходном расположении.

 ![Кнопка "Восстановить" в предупреждающем сообщении](./media/storage-how-to-use-files-windows/snapshot-windows-restore.png) 

## <a name="securing-windowswindows-server"></a>Обеспечение безопасности Windows или Windows Server
Чтобы подключить общую папку Azure в Windows, порт 445 должен быть доступен. Многие организации блокируют порт 445 из-за угроз безопасности, присущих SMB 1. SMB 1, также известный как CIFS (Common Internet File System), является устаревшим протоколом файловой системы, включенным в Windows и Windows Server. SMB 1 — это устаревший, неэффективный и, самое главное, небезопасный протокол. Хорошей новостью является то, что компонент "Файлы Azure" не поддерживает SMB 1, а все поддерживаемые версии Windows и Windows Server позволяют удалить или отключить этот протокол. Мы всегда [настоятельно рекомендуем](https://aka.ms/stopusingsmb1) удалять или отключать клиент и сервер SMB 1 в Windows перед использованием общей папки Azure в рабочей среде.

В следующей таблице представлена ​​подробная информация о состоянии SMB 1 для каждой версии Windows:

| Версия Windows                           | Состояние SMB 1 по умолчанию | Метод удаления или отключения       | 
|-------------------------------------------|----------------------|-----------------------------|
| Windows Server 2019                       | Выключено             | Удаление с помощью функции Windows |
| Windows Server версии 1709 или выше            | Выключено             | Удаление с помощью функции Windows |
| Windows 10 версии 1709 или выше                | Выключено             | Удаление с помощью функции Windows |
| Windows Server 2016                       | Активировано              | Удаление с помощью функции Windows |
| Windows 10 версии 1507, 1607 и 1703 | Активировано              | Удаление с помощью функции Windows |
| Windows Server 2012 R2                    | Активировано              | Удаление с помощью функции Windows | 
| Windows 8.1                               | Активировано              | Удаление с помощью функции Windows | 
| Windows Server 2012                       | Активировано              | Отключение с помощью реестра       | 
| Windows Server 2008 R2                    | Активировано              | Отключение с помощью реестра       |
| Windows 7                                 | Активировано              | Отключение с помощью реестра       | 

### <a name="auditing-smb-1-usage"></a>Аудит использования SMB 1
> Применяется к Windows Server 2019, Semi-Annual Channel для Windows Server (версии 1709 и 1803), Windows Server 2016, Windows 10 (версии 1507, 1607, 1703, 1709 и 1803), Windows Server 2012 R2 и Windows 8.1.

Прежде чем удалять SMB 1 в вашей среде, можете провести аудит использования SMB 1, чтобы узнать, повлияет ли это изменение на работу каких-либо клиентов. Если какие-либо запросы к папкам SMB выполняются с помощью SMB 1, событие аудита будет регистрироваться в журнале событий в разделе `Applications and Services Logs > Microsoft > Windows > SMBServer > Audit`. 

> [!Note]  
> Чтобы включить поддержку аудита в Windows Server 2012 R2 и Windows 8.1, требуется пакет обновлений не ниже указанного в [KB4022720](https://support.microsoft.com/help/4022720/windows-8-1-windows-server-2012-r2-update-kb4022720).

Чтобы включить аудит, выполните следующий командлет в сеансе PowerShell с повышенными правами:

```powershell
Set-SmbServerConfiguration –AuditSmb1Access $true
```

### <a name="removing-smb-1-from-windows-server"></a>Удаление SMB 1 из Windows Server
> Применяется к Windows Server 2019, Semi-Annual Channel для Windows Server (версии 1709 и 1803), Windows Server 2016 и Windows Server 2012 R2.

Чтобы удалить SMB 1 из экземпляра Windows Server, выполните следующий командлет в сеансе PowerShell с повышенными правами:

```powershell
Remove-WindowsFeature -Name FS-SMB1
```

Чтобы завершить процесс удаления, перезагрузите сервер. 

> [!Note]  
> Начиная с Windows 10 и Windows Server версии 1709, SMB 1 не установлен по умолчанию и имеет отдельные функции Windows для клиента SMB 1 и сервера SMB 1. Мы всегда рекомендуем удалять сервер SMB 1 (`FS-SMB1-SERVER`) и клиент SMB 1 (`FS-SMB1-CLIENT`).

### <a name="removing-smb-1-from-windows-client"></a>Удаление SMB 1 из клиента Windows
> Применяется к Windows 10 (версии 1507, 1607, 1703, 1709 и 1803) и Windows 8.1.

Чтобы удалить SMB 1 из клиента Windows, выполните следующий командлет в сеансе PowerShell с повышенными правами:

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
```

Чтобы завершить процесс удаления, перезагрузите компьютер.

### <a name="disabling-smb-1-on-legacy-versions-of-windowswindows-server"></a>Отключение SMB 1 в устаревших версиях Windows и Windows Server
> Применяется к Windows Server 2012, Windows Server 2008 R2 и Windows 7.

SMB 1 нельзя полностью удалить в устаревших версиях Windows и Windows Server, но его можно отключить через реестр. Чтобы отключить SMB 1, создайте новый ключ реестра `SMB1` типа `DWORD` со значением `0` в `HKEY_LOCAL_MACHINE > SYSTEM > CurrentControlSet > Services > LanmanServer > Parameters`.

Вы можете легко выполнить это с помощью следующего командлета PowerShell:

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" SMB1 -Type DWORD -Value 0 –Force
```

После создания этого раздела реестра необходимо перезагрузить сервер, чтобы отключить SMB 1.

### <a name="smb-resources"></a>Ресурсы по SMB
- [Запись блога о прекращении использования SMB 1](https://blogs.technet.microsoft.com/filecab/2016/09/16/stop-using-smb1/)
- [Центр координации продуктов с SMB 1](https://blogs.technet.microsoft.com/filecab/2017/06/01/smb1-product-clearinghouse/)
- [Запись блога об обнаружении SMB 1 в своей среде с помощью DSCEA](/archive/blogs/ralphkyttle/discover-smb1-in-your-environment-with-dscea)
- [Запись блога об отключении SMB 1 в групповой политике](/archive/blogs/secguide/disabling-smbv1-through-group-policy)

## <a name="next-steps"></a>Дальнейшие действия
Дополнительные сведения о службе файлов Azure см. по следующим ссылкам.
- [Планирование развертывания службы файлов Azure](storage-files-planning.md)
- [Часто задаваемые вопросы](./storage-files-faq.md)
- [Troubleshoot Azure File storage problems in Windows](storage-troubleshoot-windows-file-connection-problems.md) (Устранение неполадок хранилища файлов Azure в Windows)