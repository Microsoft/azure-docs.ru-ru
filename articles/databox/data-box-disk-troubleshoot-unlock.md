---
title: Диск Azure Data Box устранении неполадок при разблокировании диска | Документация Майкрософт
description: Сведения о рабочих процессах для устранения неполадок, связанных с использованием средства Unlock с Диск Azure Data Box. См. статью Диск Data Box Unlock Errors Tools.
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: troubleshooting
ms.date: 08/05/2020
ms.author: alkohli
ms.openlocfilehash: 866cf01243983863292ada0b086f8f5b2f94e412
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87925567"
---
# <a name="troubleshoot-disk-unlocking-issues-in-azure-data-box-disk"></a>Устранение неполадок, связанных с разблокировкой диска, в Диск Azure Data Box

Эта статья относится к Microsoft Azure Диск Data Box и описание рабочих процессов, используемых для устранения неполадок, возникающих при использовании средства Unlock. 


<!--## Query activity logs

Use the activity logs to find who unlocked and accessed the disks. Your Data Box Disk arrive on your premises in a locked state. You can use the device credentials available in the Azure portal for your order to unlock them.  

To figure out who accessed the **Device credentials** blade, you can query the Activity logs.  Any action that involves accessing **Device details > Credentials** blade is logged into the activity logs as `ListCredentials` action.

![Query Activity logs](media/data-box-logs/query-activity-log-1.png)-->


## <a name="data-box-disk-unlock-tool-errors"></a>Ошибки средства разблокировки диска Data Box


| Сообщение об ошибке/Режимы работы средства      | Рекомендации                                                                             |
|-------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| Текущая платформа .NET не поддерживается. Поддерживаемые версии — 4.5 и более поздние.<br><br>Средство завершит свою работу с сообщением.  | Платформа .NET версии 4.5 не установлена. Установите .NET 4.5 или более поздней версии на главном компьютере, на котором запущено средство разблокировки диска Data Box.                                                                            |
| Не удалось разблокировать или проверить какие-либо тома. Обратитесь в службу технической поддержки Майкрософт.  <br><br>Средство не может разблокировать или проверить заблокированный диск. | Средству не удалось разблокировать ни один из заблокированных дисков с помощью поставляемого ключа доступа. Обратитесь в службу поддержки Майкрософт, чтобы узнать, что делать дальше.                                                |
| Следующие тома разблокированы и проверены. <br>Том букв диска: E:<br>Не удалось разблокировать все тома со следующими ключами доступа: werwerqomnf, qwerwerqwdfda <br><br>Средство разблокирует некоторые диски и перечислит успешные и неудачные буквы дисков.| Частично удалось. Не удалось разблокировать некоторые диски с помощью поставляемого ключа доступа. Обратитесь в службу поддержки Майкрософт, чтобы узнать, что делать дальше. |
| Не удалось найти заблокированные тома. Проверьте, что диск, полученный от Microsoft, подключен правильно и находится в заблокированном состоянии.          | Средству не удается найти заблокированные диски. Либо диски уже разблокированы или не обнаружены. Убедитесь, что диски подключены и заблокированы. <br> <br>Эта ошибка может также возникать, если вы отформатируете диски. Если вы отформатируете диски, они стали непригодными для использования. Обратитесь в службу поддержки Майкрософт, чтобы узнать, что делать дальше.                                                          |
| Неустранимая ошибка: недопустимый параметр<br>Имя параметра: invalid_arg<br>Использование.<br>DataBoxDiskUnlock /Ключи доступа:<passkey_list_separated_by_semicolon><br><br>Пример: DataBoxDiskUnlock /Ключи доступа:passkey1; passkey2; passkey3<br>Пример: DataBoxDiskUnlock /SystemCheck<br>Пример: DataBoxDiskUnlock /Help<br><br>/PassKeys: Получите этот ключ доступа из заказа диска Azure DataBox Disk. Ключ доступа разблокирует диски.<br>/Help: Этот параметр предоставляет справку об использовании командлета и примеры.<br>/SystemCheck: Этот параметр проверяет, соответствует ли система требованиям для запуска средства.<br><br>Нажмите любую клавишу, чтобы выйти. | Недопустимый ввод параметра. Допустимые параметры:/Системчекк,/Пасскэй и/Хелп.|


## <a name="unlock-issues-for-disks-when-using-a-windows-client"></a>Разблокируйте проблемы для дисков при использовании клиента Windows

В этом разделе подробно описаны некоторые основные проблемы, с которыми столкнулось во время развертывания Диск Data Box при использовании клиента Windows для копирования данных.

### <a name="issue-could-not-unlock-drive-from-bitlocker"></a>Ошибка: не удалось разблокировать диск из BitLocker
 
**Причина** 

Вы использовали пароль в диалоговом окне BitLocker и пытались разблокировать диск с помощью диалогового окна разблокировки в BitLocker. Это не будет работать.

**Решение**

Чтобы разблокировать Диски Data Box, необходимо использовать средство разблокирования Дисков Data Box и предоставить пароль с портала Azure. Дополнительные сведения см. в статье [учебник. Распаковка, подключение и разблокировка диск Azure Data Box](data-box-disk-deploy-set-up.md#connect-to-disks-and-get-the-passkey).
 
### <a name="issue-could-not-unlock-or-verify-some-volumes-contact-microsoft-support"></a>Ошибка: не удалось разблокировать или проверить некоторые тома. Обратитесь в службу технической поддержки Майкрософт.
 
**Причина**

Вы можете увидеть следующую ошибку в журнале ошибок и не иметь возможности разблокировать или проверить некоторые тома.

`Exception System.IO.FileNotFoundException: Could not load file or assembly 'Microsoft.Management.Infrastructure, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35' or one of its dependencies. The system cannot find the file specified.`
 
Это означает, что в вашем клиенте Windows, скорее всего, отсутствует соответствующая версия Windows PowerShell.

**Решение**

Вы можете установить [Windows PowerShell версии 5.0](https://www.microsoft.com/download/details.aspx?id=54616) и повторить операцию.
 
Если тома по-прежнему не удается разблокировать, скопируйте журналы из папки, которая содержит средство разблокировки Диска Data Box, и [обратитесь в службу поддержки Майкрософт](data-box-disk-contact-microsoft-support.md).

## <a name="next-steps"></a>Дальнейшие действия

- Сведения об [устранении неполадок при проверке](data-box-disk-troubleshoot.md).
