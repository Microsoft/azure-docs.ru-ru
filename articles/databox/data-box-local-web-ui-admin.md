---
title: Администрирование Azure Data Box и Azure Data Box Heavy с помощью локального пользовательского веб-интерфейса
description: Описание использования локального веб-интерфейса для администрирования устройств Data Box и Data Box Heavy
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: article
ms.date: 12/18/2020
ms.author: alkohli
ms.openlocfilehash: e84f39fa5b9245d1874e60d651156e99c0885040
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97678915"
---
# <a name="use-the-local-web-ui-to-administer-your-data-box-and-data-box-heavy"></a>Использование локального веб-интерфейса для администрирования Data Box и Data Box Heavy

В этой статье описываются некоторые задачи по настройке и управлению, выполняемые на устройствах Data Box и Data Box Heavy. Вы можете управлять Data Box и Data Box Heavy устройствами через пользовательский интерфейс портал Azure и локальный веб-интерфейс для устройства. В этой статье рассматриваются задачи, выполняемые с помощью локального веб-интерфейса.

Локальный веб-интерфейс для Data Box и для Data Box Heavy используется для начальной настройки устройства. Можно также использовать локальный веб-интерфейс для выключения или перезапуска устройства, выполнения диагностических тестов, обновления программного обеспечения, просмотра журналов копирования, удаления локальных данных с устройства и создания пакета поддержки для служба поддержки Майкрософт. На Data Box Heavy устройстве с двумя независимыми узлами можно получить доступ к двум отдельным локальным веб-интерфейсам, соответствующим каждому узлу устройства.

## <a name="generate-support-package"></a>Создание пакета поддержки

Если возникают проблемы с устройством, из системных журналов можно создать пакет поддержки. Служба поддержки Майкрософт использует этот пакет для устранения неполадок.

Чтобы создать пакет поддержки, выполните следующие действия.

1. В локальном веб-ИНТЕРФЕЙСе перейдите в службу **поддержки**. При необходимости выберите **включить дампы памяти**. Затем выберите **создать пакет поддержки**.

    Дамп памяти — это содержимое памяти устройства, которое сохраняется после сбоя системы.

    Не следует выбирать параметр **включить дампы памяти** , если только поддержка не запрашивается. Сбор пакета поддержки, включающего дампы памяти, и включение в него конфиденциальных данных занимает много времени.

    ![Создание пакета поддержки 1](media/data-box-local-web-ui-admin/create-support-package-1.png)

    Создание пакета поддержки. Эта операция занимает несколько минут, если включены только системные журналы. Если вы включаете дампы памяти, это занимает гораздо больше времени.

    ![Создание пакета поддержки 2](media/data-box-local-web-ui-admin/create-support-package-2.png)

2. После завершения создания пакета поддержки выберите **скачать пакет поддержки**.

    ![Создать пакет поддержки 3](media/data-box-local-web-ui-admin/create-support-package-3.png)

3. Просмотрите и выберите расположение загрузки. Откройте папку, чтобы просмотреть содержимое.

    ![Создание пакета поддержки 4](media/data-box-local-web-ui-admin/create-support-package-4.png)

## <a name="erase-local-data-from-your-device"></a>Удаление локальных данных с устройства

Вы можете использовать локальный веб-интерфейс для удаления локальных данных с устройства перед их возвратом в центр обработки данных Azure.

> [!IMPORTANT]
> Очистка данных не может быть отменена. Перед удалением локальных данных с устройства обязательно создайте резервные копии файлов.

Чтобы стереть локальные данные с устройства, выполните следующие действия.

1. В локальном веб-ИНТЕРФЕЙСе перейдите к элементу **Очистка данных**.
2. Введите пароль устройства и нажмите кнопку **стереть данные**.

    ![Параметр стирания данных для устройства](media/data-box-local-web-ui-admin/erase-local-data-1.png)

3. В окне подтверждения нажмите кнопку **Да** , чтобы продолжить. Очистка данных может занять до 50 минут.

   Обязательно создайте резервные копии локальных данных, прежде чем удалять их с устройства. Очистка данных не может быть отменена.

    ![Запрос подтверждения очистки данных](media/data-box-local-web-ui-admin/erase-local-data-2.png)

## <a name="shut-down-or-restart-your-device"></a>завершение работы или перезапуск устройства.

Вы можете выключить или перезапустить устройство с помощью локального веб-интерфейса. Перед перезапуском рекомендуется отключить общие папки на узле, а затем на устройстве. Это снизит вероятность повреждения данных. Убедитесь, что копирование данных не выполняется при завершении работы устройства.

Чтобы завершить работу устройства, выполните следующие действия.

1. В локальном пользовательском веб-интерфейсе перейдите в раздел **Завершение работы или перезагрузка**.

2. Щелкните **Завершение работы**.

    ![Завершение работы Data Box 1](media/data-box-local-web-ui-admin/shut-down-local-web-ui-1.png)

3. При появлении запроса на подтверждение нажмите кнопку **ОК** , чтобы продолжить.

    ![Завершение работы Data Box 2](media/data-box-local-web-ui-admin/shut-down-local-web-ui-2.png)

Если устройство выключено, включите его, воспользовавшись кнопкой питания на передней панели.

Чтобы перезагрузить Data Box, выполните следующие действия.

1. В локальном пользовательском веб-интерфейсе перейдите в раздел **Завершение работы или перезагрузка**.
2. Нажмите кнопку **Перезапустить**.

    ![Перезагрузка Data Box 1](media/data-box-local-web-ui-admin/restart-local-web-ui-1.png)

3. При появлении запроса на подтверждение нажмите кнопку **ОК** , чтобы продолжить.

   Устройство завершит работу, а затем перезагрузится.

## <a name="download-bom-or-manifest-files"></a>Скачивание файла метки порядка байтов или манифеста

Файлы BOM или manifest содержат список файлов, которые копируются в Data Box или Data Box Heavy. Эти файлы создаются для заказа импорта при подготовке устройства к отправке.

Прежде чем начать, выполните следующие действия, чтобы скачать файлы спецификации или файла манифеста для заказа импорта.

1. Перейдите в локальный веб-интерфейс для своего устройства. Убедитесь, что устройство завершило **Подготовка к отправке** шаге. По завершении подготовки состояние устройства отобразится как **Все готово к поставке**.

    ![Устройство готово к отправке](media/data-box-local-web-ui-admin/prepare-to-ship-3.png)

2. Выберите **скачать список файлов** , чтобы скачать список файлов, которые были скопированы на Data Box.

    <!-- ![Select Download list of files](media/data-box-portal-admin/download-list-of-files.png) -->

3. В проводнике отдельные списки файлов создаются в зависимости от протокола, используемого для подключения к устройству и используемого типа хранилища Azure.

    <!-- ![Files for storage type and connection protocol](media/data-box-portal-admin/files-storage-connection-type.png) -->
    ![Файлы для типа хранилища и протокола подключения](media/data-box-local-web-ui-admin/prepare-to-ship-5.png)

   В следующей таблице имена файлов сопоставлены с типом службы хранилища Azure и используемым протоколом подключения.

    |Имя файла  |Тип службы хранилища Azure  |Используемый протокол подключения |
    |---------|---------|---------|
    |utSAC1_202006051000_BlockBlob-BOM.txt     |Blob-блоки         |SMB/NFS         |
    |utSAC1_202006051000_PageBlob-BOM.txt     |Страничные большие двоичные объекты         |SMB/NFS         |
    |utSAC1_202006051000_AzFile-BOM.txt    |Файлы Azure         |SMB/NFS         |
    |utsac1_PageBlock_Rest-BOM.txt     |Страничные большие двоичные объекты         |REST        |
    |utsac1_BlockBlock_Rest-BOM.txt    |Blob-блоки         |REST         |

Используйте этот список, чтобы проверить файлы, отправленные в учетную запись хранения Azure, после возврата устройства Data Box в центр обработки данных Azure. Ниже приведен пример файла манифеста.

> [!NOTE]
> На Data Box Heavy два набора файлов (файлов BOM) представлены в соответствии с двумя узлами на устройстве.

```xml
<file size="52689" crc64="0x95a62e3f2095181e">\databox\media\data-box-deploy-copy-data\prepare-to-ship2.png</file>
<file size="22117" crc64="0x9b160c2c43ab6869">\databox\media\data-box-deploy-copy-data\connect-shares-file-explorer2.png</file>
<file size="57159" crc64="0x1caa82004e0053a4">\databox\media\data-box-deploy-copy-data\verify-used-space-dashboard.png</file>
<file size="24777" crc64="0x3e0db0cd1ad438e0">\databox\media\data-box-deploy-copy-data\prepare-to-ship5.png</file>
<file size="162006" crc64="0x9ceacb612ecb59d6">\databox\media\data-box-cable-options\cabling-dhcp-data-only.png</file>
<file size="155066" crc64="0x051a08d36980f5bc">\databox\media\data-box-cable-options\cabling-2-port-setup.png</file>
<file size="150399" crc64="0x66c5894ff328c0b1">\databox\media\data-box-cable-options\cabling-with-switch-static-ip.png</file>
<file size="158082" crc64="0xbd4b4c5103a783ea">\databox\media\data-box-cable-options\cabling-mgmt-only.png</file>
<file size="148456" crc64="0xa461ad24c8e4344a">\databox\media\data-box-cable-options\cabling-with-static-ip.png</file>
<file size="40417" crc64="0x637f59dd10d032b3">\databox\media\data-box-portal-admin\delete-order1.png</file>
<file size="33704" crc64="0x388546569ea9a29f">\databox\media\data-box-portal-admin\clone-order1.png</file>
<file size="5757" crc64="0x9979df75ee9be91e">\databox\media\data-box-safety\japan.png</file>
<file size="998" crc64="0xc10c5a1863c5f88f">\databox\media\data-box-safety\overload_tip_hazard_icon.png</file>
<file size="5870" crc64="0x4aec2377bb16136d">\databox\media\data-box-safety\south-korea.png</file>
<file size="16572" crc64="0x05b13500a1385a87">\databox\media\data-box-safety\taiwan.png</file>
<file size="999" crc64="0x3f3f1c5c596a4920">\databox\media\data-box-safety\warning_icon.png</file>
<file size="1054" crc64="0x24911140d7487311">\databox\media\data-box-safety\read_safety_and_health_information_icon.png</file>
<file size="1258" crc64="0xc00a2d5480f4fcec">\databox\media\data-box-safety\heavy_weight_hazard_icon.png</file>
<file size="1672" crc64="0x4ae5cfa67c0e895a">\databox\media\data-box-safety\no_user_serviceable_parts_icon.png</file>
<file size="3577" crc64="0x99e3d9df341b62eb">\databox\media\data-box-safety\battery_disposal_icon.png</file>
<file size="993" crc64="0x5a1a78a399840a17">\databox\media\data-box-safety\tip_hazard_icon.png</file>
<file size="1028" crc64="0xffe332400278f013">\databox\media\data-box-safety\electrical_shock_hazard_icon.png</file>
<file size="58699" crc64="0x2c411d5202c78a95">\databox\media\data-box-deploy-ordered\data-box-ordered.png</file>
<file size="46816" crc64="0x31e48aa9ca76bd05">\databox\media\data-box-deploy-ordered\search-azure-data-box1.png</file>
<file size="24160" crc64="0x978fc0c6e0c4c16d">\databox\media\data-box-deploy-ordered\select-data-box-option1.png</file>
<file size="115954" crc64="0x0b42449312086227">\databox\media\data-box-disk-deploy-copy-data\data-box-disk-validation-tool-output.png</file>
<file size="6093" crc64="0xadb61d0d7c6d4deb">\databox\data-box-cable-options.md</file>
<file size="6499" crc64="0x080add29add367d9">\databox\data-box-deploy-copy-data-via-nfs.md</file>
<file size="11089" crc64="0xc3ce6b13a4fe3001">\databox\data-box-deploy-copy-data-via-rest.md</file>
<file size="9126" crc64="0x820856b5a54321ad">\databox\data-box-overview.md</file>
<file size="10963" crc64="0x5e9a14f9f4784fd8">\databox\data-box-safety.md</file>
<file size="5941" crc64="0x8631d62fbc038760">\databox\data-box-security.md</file>
<file size="12536" crc64="0x8c8ff93e73d665ec">\databox\data-box-system-requirements-rest.md</file>
<file size="3220" crc64="0x7257a263c434839a">\databox\data-box-system-requirements.md</file>
<file size="2823" crc64="0x63db1ada6fcdc672">\databox\index.yml</file>
<file size="4364" crc64="0x62b5710f58f00b8b">\databox\data-box-local-web-ui-admin.md</file>
<file size="3603" crc64="0x7e34c25d5606693f">\databox\TOC.yml</file>
```

Этот файл содержит список всех файлов, которые были скопированы в Data Box или Data Box Heavy. В этом файле значение *crc64* соотносится с контрольной суммой, созданной для соответствующего файла.

## <a name="view-available-capacity-of-the-device"></a>Просмотр доступной емкости устройства

Чтобы просмотреть доступную и используемую емкость устройства, используйте панель мониторинга устройства.

1. В локальном пользовательском веб-интерфейсе перейдите в раздел **Просмотреть панель мониторинга**.
2. В разделе **Подключиться и копировать** отображается свободное и используемое пространство на устройстве.

    ![Просмотр доступной емкости](media/data-box-local-web-ui-admin/verify-used-space-dashboard.png)

## <a name="skip-checksum-validation"></a>Пропуск проверки контрольной суммы

Контрольные суммы для данных создаются по умолчанию при подготовке к отправке. В некоторых редких случаях в зависимости от типа данных (файлы небольшого размера) производительность может быть медленной. В таких случаях вы можете пропустить создание контрольных сумм.

Вычисление контрольной суммы во время подготовки к отправке выполняется только для заказов на импорт, а не для заказов на экспорт.

Настоятельно рекомендуется не отключать контрольную сумму, если только производительность не окажется существенной.

1. В правом верхнем углу локального веб-интерфейса устройства перейдите в раздел **Параметры**.

    ![Отключение контрольных сумм](media/data-box-local-web-ui-admin/disable-checksum.png)

2. **Отключение** проверки контрольных сумм
3. Нажмите кнопку **Применить**.

> [!NOTE]
> Параметр пропустить вычисление контрольной суммы доступен только в том случае, если Azure Data Box разблокирована. Этот параметр не отображается, если устройство заблокировано.

## <a name="enable-smb-signing"></a>Включить подписывание SMB

Подписывание SMB — это функция, с помощью которой можно подписать связь с помощью SMB на уровне пакетов. Эта подписывание предотвращает атаки, изменяющие пакеты SMB при передаче.

Дополнительные сведения о подписывании SMB см. в разделе [Общие сведения о подписывании блоков сообщений сервера](https://support.microsoft.com/help/887429/overview-of-server-message-block-signing).

Чтобы включить подписывание SMB на устройстве Azure, сделайте следующее:

1. В правом верхнем углу локального веб-интерфейса устройства выберите **Параметры**.

    ![Открытие "Параметров"](media/data-box-local-web-ui-admin/data-box-settings-1.png)

2. **Включить** Подписывание SMB.

    ![Включить подписывание SMB](media/data-box-local-web-ui-admin/data-box-smb-signing-1.png)

3. Нажмите кнопку **Применить**.
4. В локальном пользовательском веб-интерфейсе перейдите в раздел **Завершение работы или перезагрузка**.
5. Нажмите кнопку **Перезапустить**.

## <a name="enable-backup-operator-privileges"></a>Включение прав оператора резервного копирования

Пользователи пользовательского веб-интерфейса имеют права оператора резервного копирования в общих ресурсах SMB по умолчанию. Если это не требуется, используйте **права доступа оператора "включить оператор** ", чтобы отключить или включить привилегии.

Дополнительные сведения см. в разделе операторы резервного копирования в [Active Directory группах безопасности](/windows/security/identity-protection/access-control/active-directory-security-groups#backup-operators).

Чтобы включить права оператора резервного копирования на устройстве Azure, сделайте следующее:

1. В правом верхнем углу локального веб-интерфейса устройства выберите **Параметры**.

   ![Открыть параметры Data Box-1](media/data-box-local-web-ui-admin/data-box-settings-1.png)

2. **Включить** Права оператора Backup.

   ![Включение прав оператора резервного копирования](media/data-box-local-web-ui-admin/data-box-backup-operator-privileges-1.png)

3. **Нажмите кнопку применить**.
4. В локальном пользовательском веб-интерфейсе перейдите в раздел **Завершение работы или перезагрузка**.
5. Нажмите кнопку **Перезапустить**.

## <a name="enable-acls-for-azure-files"></a>Включение списков управления доступом для файлов Azure

Метаданные для файлов передаются по умолчанию, когда пользователи отправляют данные через SMB в Data Box. Метаданные включают списки управления доступом (ACL), атрибуты файлов и метки времени. Если это не требуется, используйте **списки ACL для службы файлов Azure** , чтобы отключить или включить эту функцию.

<!--For more information about metadata that is transferred, see [Preserving the ACLs and metadata with Azure Data Box](./data-box-local-web-ui-admin.md#enable-backup-operator-privileges) - IN DEVELOPMENT-->

> [!Note]
> Для перемещения метаданных с файлами необходимо быть оператором резервного копирования. При использовании этой функции убедитесь, что локальные пользователи веб-интерфейса являются операторами резервного копирования. См. раздел [Включение прав оператора резервного копирования](#enable-backup-operator-privileges).

Чтобы включить перенос списков ACL для файлов Azure, сделайте следующее:

1. В правом верхнем углу локального веб-интерфейса устройства выберите **Параметры**.

    ![Откройте параметры Data Box-2.](media/data-box-local-web-ui-admin/data-box-settings-1.png)

2. **Включить** Списки управления доступом для службы файлов Azure.

     ![Включение списков управления доступом для файлов Azure](media/data-box-local-web-ui-admin/data-box-acls-for-azure-files-1.png)
  
3. Нажмите кнопку **Применить**.
4. В локальном пользовательском веб-интерфейсе перейдите в раздел **Завершение работы или перезагрузка**.
5. Нажмите кнопку **Перезапустить**.

## <a name="enable-tls-11"></a>Включение TLS 1,1

По умолчанию Azure Data Box использует протокол TLS 1,2 для шифрования, так как он более безопасен, чем TSL 1,1. Тем не менее, если вы или ваши клиенты используют браузер для доступа к данным, не поддерживающим TLS 1,2, можно включить TLS 1,1.

Дополнительные сведения о протоколе TLS см. в разделе [шлюз Azure Data Box Security](../databox-gateway/data-box-gateway-security.md).

Чтобы включить TLS 1,1 на устройстве Azure, выполните следующие действия.

1. В правом верхнем углу локального веб-интерфейса устройства выберите **Параметры**.

    ![Открыть параметры Data Box-3](media/data-box-local-web-ui-admin/data-box-settings-1.png)

2. **Включить** TLS 1,1.

    ![Включение TLS 1,1](media/data-box-local-web-ui-admin/data-box-tls-1-1.png)

3. Нажмите кнопку **Применить**.
4. В локальном пользовательском веб-интерфейсе перейдите в раздел **Завершение работы или перезагрузка**.
5. Нажмите кнопку **Перезапустить**.

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [управлять Data Box и Data Box Heavy с помощью портал Azure](data-box-portal-admin.md).
