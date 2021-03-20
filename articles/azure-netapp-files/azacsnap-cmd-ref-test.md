---
title: Тестирование средства создания моментальных снимков с помощью приложения Azure для Azure NetApp Files | Документация Майкрософт
description: В этой статье объясняется, как выполнить команду Test средства создания моментального снимка, совместимого с приложениями Azure, которое можно использовать с Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 12/14/2020
ms.author: phjensen
ms.openlocfilehash: 03ffa93a71ded40e033f2068ea23fc6b994a5bbb
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97632817"
---
# <a name="test-azure-application-consistent-snapshot-tool-preview"></a>Тестирование средства создания моментальных снимков с единообразием приложений Azure (Предварительная версия)

В этой статье объясняется, как выполнить команду Test инструмента для создания моментального снимка, совместимого с приложениями Azure, который можно использовать с Azure NetApp Files.

## <a name="introduction"></a>Введение

Проверка конфигурации выполняется с помощью `azacsnap -c test` команды.

## <a name="command-options"></a>Параметры команды

`-c test`Команда имеет следующие параметры.

- `--test hana`  проверяет соединение с экземпляром SAP HANA.

- `--test storage` проверяет связь с базовым интерфейсом хранилища, создавая временный моментальный снимок хранилища для всех настроенных `data` томов, а затем удаляя их. 

- `--test all` будет выполнять `hana` `storage` тесты и последовательно.

- `[--configfile <config filename>]` — Необязательный параметр, который позволяет использовать пользовательские имена файлов конфигурации.

## <a name="check-connectivity-with-sap-hana-azacsnap--c-test---test-hana"></a>Проверка подключения с помощью SAP HANA `azacsnap -c test --test hana`

Эта команда проверяет подключение HANA для всех экземпляров HANA в файле конфигурации. Он использует HDBuserstore для подключения к СИСТЕМДБ и получения сведений о SID.

Для SSL эта команда может принимать следующий необязательный аргумент:

- `--ssl=` принудительно устанавливает зашифрованное соединение с базой данных и определяет метод шифрования, используемый для взаимодействия с SAP HANA, либо `openssl` `commoncrypto` . Если этот параметр определен, эта команда должна найти два файла в одном каталоге. Эти файлы должны называться после соответствующего идентификатора безопасности. Обратитесь к статье [использование SSL для связи с SAP HANA](azacsnap-installation.md#using-ssl-for-communication-with-sap-hana).

### <a name="output-of-the-azacsnap--c-test---test-hana-command"></a>Выходные данные `azacsnap -c test --test hana` команды

```bash
azacsnap -c test --test hana
```

```output
BEGIN : Test process started for 'hana'
BEGIN : SAP HANA tests
PASSED: Successful connectivity to HANA version 2.00.032.00.1533114046
END   : Test process complete for 'hana'
```

## <a name="check-connectivity-with-storage-azacsnap--c-test---test-storage"></a>Проверка подключения к хранилищу `azacsnap -c test --test storage`

`azacsnap`Команда создаст моментальный снимок для всех томов данных, настроенных на проверку правильности доступа к томам для каждого экземпляра SAP HANA. Временный моментальный снимок создается, а затем удаляется для каждого тома данных, чтобы проверить доступ к моментальному снимку для каждой файловой системы.

### <a name="output-of-the-azacsnap--c-test---test-storage-command"></a>Выходные данные `azacsnap -c test --test storage` команды

```bash
azacsnap -c test --test storage
```

```output
BEGIN : Test process started for 'storage'
BEGIN : Storage test snapshots on 'data' volumes
BEGIN : 2 task(s) to Test Snapshots for Storage Volume Type 'data'
PASSED: Task#2/2 Storage test successful for Volume
PASSED: Task#1/2 Storage test successful for Volume
END   : Storage tests complete
END   : Test process complete for 'storage'
```

> [!NOTE]
> В случае с большим экземпляром Azure `azacsnap -c test --test storage` команда экстраполяция поколения хранилища и номера SKU Хли.  На основе этих сведений он предоставляет инструкции по настройке снимков загрузки (см. строку, начинающуюся с `Action:` Output).

```output
SID1   : Generation 4
Storage: ams07-a700s-saphan-1-01v250-client25-nprod
HLI SKU: S96
Action : Configure the 'boot' snapshots on ALL the servers.
```

## <a name="next-steps"></a>Дальнейшие действия

- [Резервное копирование моментальных снимков с помощью инструмента создания моментальных снимков для приложений Azure](azacsnap-cmd-ref-backup.md)
