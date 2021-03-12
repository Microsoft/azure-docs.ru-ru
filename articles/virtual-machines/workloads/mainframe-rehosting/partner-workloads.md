---
title: Рабочие нагрузки мэйнфрейма, поддерживаемые в Azure | Документация Майкрософт
description: Используйте эмулятор мэйнфрейма и другие службы от партнеров Майкрософт для повторного размещения рабочих нагрузок мэйнфреймов, таких как системы на основе IBM Z, с помощью Microsoft Azure.
services: virtual-machines
documentationcenter: ''
author: njray
manager: edprice
editor: edprice
ms.author: larryme
ms.date: 05/09/2020
ms.topic: article
ms.service: multiple
ms.openlocfilehash: 679cc68a0f43d6bff74fef85d705544db2f2585b
ms.sourcegitcommit: 7edadd4bf8f354abca0b253b3af98836212edd93
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/10/2021
ms.locfileid: "102558259"
---
# <a name="mainframe-workloads-supported-on-azure"></a>Рабочие нагрузки мэйнфрейма, поддерживаемые в Azure

Перечисленные здесь решения помогут вам перенести рабочую нагрузку на мэйнфреймы в Microsoft Azure. Некоторые рабочие нагрузки можно перенести без особых трудностей. Другие рабочие нагрузки, зависящие от устаревшего системного программного обеспечения, могут быть переразмещены. 

Дополнительные сведения о эмуляции мэйнфреймов и службах см. в [центре миграции мэйнфреймов Azure](https://azure.microsoft.com/migration/mainframe/).

## <a name="migrate-mainframe-closer-to-azure"></a>Перенос мэйнфреймов ближе к Azure

- [ASNA Visual RPG для компилятора .NET](https://asna.com/us/products/visual-rpg) для платформа .NET Framework с помощью подключаемого модуля Visual Studio.
- Среда разработки [COBOL для АСИСКО AMT](https://www.asysco.com/cobol/) (Unisys, мэйнфреймов IBM и другие диалекты COBOL, например Micro Focus COBOL).
- [АСИСКО AMT](https://www.asysco.com/amt-go/) — Облачная архитектура развертывания для высокопроизводительных рабочих нагрузок.
- [Преобразование АСИСКО AMT](https://www.asysco.com/amt-transform/) для преобразования данных, кода, сценариев, безопасности, интерфейсов и других артефактов мэйнфреймов.
- [Fujitsu неткобол](https://www.fujitsu.com/global/products/software/developer-tool/netcobol/) средства разработки и интеграции.
- [Micro Focus](https://www.microfocus.com/products/visual-cobol/) средства разработки и интеграции Visual COBOL.
- Устаревший компилятор [Micro Focus](https://www.microfocus.com/campaign/download/pli-modernization/) для платформы .NET, поддерживающий синтаксис PL/i в мэйнфрейме, типы данных и поведение.
- Микрофокусировка платформы интеграции мэйнфреймов [Enterprise Server](https://www.microfocus.com/products/enterprise-suite/enterprise-server/) .
- [Современные системы КТУ (COBOL-to-Universal)](https://modernsystems.com/automatic-cobol-to-java-conversion/) средства разработки и интеграции.
- [NTT данных](https://us.nttdata.com/en/digital/application-development-and-modernization) — средства разработки и интеграции COBOL для корпоративных приложений.
- [NTT Open PL/i](https://us.nttdata.com/en/digital/application-development-and-modernization) устаревший компилятор для платформы .NET, поддерживающий синтаксис PL/i в мэйнфрейме, типы данных и поведение.
- Средства разработки и интеграции для [компилятора РАИНКОДЕ COBOL](https://www.raincode.com/products/cobol/) .
- [Компилятор РАИНКОДЕ PL/i](https://www.raincode.com/products/pli/) для платформы .NET поддерживает синтаксис PL/i мэйнфрейма, типы данных и поведение.
- [Компилятор РАИНКОДЕ ASM370](https://www.raincode.com/technical-landscape/asm370/) для синтаксиса 370 и Хласм для ассемблера мэйнфреймов.

## <a name="deploy-an-emulation-environment-for-online-and-batch-processing"></a>Развертывание среды эмуляции для интерактивной и пакетной обработки

- Архитектура развертывания [АСИСКО AMT Go](https://www.asysco.com/amt-go/) с поддержкой CICS, мгновенных сообщений, Tip, хвтип и других распространенных сред мэйнфреймов.
- Микрофокусировка платформы интеграции мэйнфреймов [Enterprise Server](https://www.microfocus.com/products/enterprise-suite/enterprise-server/) .
- [NTT данных в среде разработки](https://us.nttdata.com/en/-/media/assets/white-paper/apps-mainframe-re-hosting-development-environment-whitepaper.pdf) для повторного размещения собственной среды обработки транзакций.
- [Среда пакетной обработки данных NTT](https://us.nttdata.com/en/-/media/assets/white-paper/apps-mainframe-re-hosting-development-environment-whitepaper.pdf) (BPE), включая возможности транзакций жкл.
- [РАИНКОДЕ CICS](https://www.raincode.com/technical-landscape/cics/) Emulator для платформ .NET и Azure.
- [РАИНКОДЕ жкл](https://www.raincode.com/products/jcl/) , совместимый с подключаемым модулем жкл.

## <a name="code-conversion"></a>Преобразование кода

- Технология преобразования системы [асиско](https://www.asysco.com/azure-cloud/) , охватывающая исходный код, данные, пакет, планирование, мониторы TP, интерфейсы, безопасность, управление и многое другое.
- [Асиско службы AMT](https://www.asysco.com/migration-services/) обеспечивают сквозные службы для миграции проектов, включая инвентаризацию и анализ, обучение в проектировании, а также переход на живой адрес и поддержку после миграции.
- Инструменты [Blu Age](https://www.bluage.com/) для работы с устаревшими бизнес-приложениями и базами данных.
- [Хеирлум вычислительные](https://www.heirloomcomputing.com/tag/convert-cobol-to-java/) службы для преобразования в Java COBOL, CICS и всам мэйнфреймов.
- Программный контейнер программно управляемого [мэйнфрейма LzLabs](https://www.lzlabs.com/) для переноса приложений мэйнфреймов на компьютеры Linux или в частные, общедоступные и гибридные облачные среды.

## <a name="modernization-services"></a>Модернизации Services

Партнеры корпорации Майкрософт с глобальными системными интеграторами (ГСИС), которые могут помочь крупным организациям проектировать, создавать решения и управлять ими. 

- [Центр миграции мэйнфреймов Azure](https://azure.microsoft.com/migration/mainframe/)
