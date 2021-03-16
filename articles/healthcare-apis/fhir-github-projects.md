---
title: Связанные проекты GitHub для API Azure для FHIR
description: Список всех репозиториев с открытым исходным кодом (GitHub) для API Azure для FHIR.
services: healthcare-apis
author: ginalee-dotcom
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 02/01/2021
ms.author: ginle
ms.openlocfilehash: 9977765183bd567377592631d65f3e548bef4b34
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103548024"
---
# <a name="related-github-projects"></a>Связанные проекты GitHub

У нас есть множество проектов с открытым исходным кодом на сайте GitHub, которые предоставляют исходный код и инструкции по развертыванию служб для различных целей. Вы всегда можете посетить наш репозиторий GitHub, чтобы узнать и поэкспериментировать с нашими функциями и продуктами. 

## <a name="fhir-server"></a>Сервер FHIR
* [Microsoft/fhir-Server](https://github.com/microsoft/fhir-server/): сервер fhir с открытым исходным кодом, который является базисом для API Azure для fhir
* Последние выпуски см. в [заметках о выпуске](https://github.com/microsoft/fhir-server/releases)
* [Microsoft/fhir-Server-Samples](https://github.com/microsoft/fhir-server-samples): образец среды

## <a name="data-conversion--anonymization"></a>Преобразование данных & анонимность

#### <a name="fhir-converter"></a>Преобразователь FHIR
* [Microsoft/FHIR-Converter](https://github.com/microsoft/FHIR-Converter): Программа преобразования для преобразования устаревших форматов данных в FHIR
* Интеграция с API Azure для FHIR, а также FHIR Server для Azure в форме $convert-Data.
* Текущие улучшения в OSS и непрерывная интеграция с серверами FHIR
 
#### <a name="fhir-converter---vs-code-extension"></a>VS Code расширение конвертера FHIR
* [Microsoft/FHIR-Tools-for-анонимность](https://github.com/microsoft/FHIR-Tools-for-Anonymization): набор средств для помощи в анонимном использовании данных (в формате FHIR)
* Интеграция с API Azure для FHIR, а также сервером FHIR для Azure в форме "неопознанный экспорт"

#### <a name="fhir-tools-for-anonymization"></a>Средства FHIR для анонимного
* [Microsoft/vscode-азурехеалскареапис-Tools](https://github.com/microsoft/vscode-azurehealthcareapis-tools): расширение VS Code, содержащее набор средств для работы с API здравоохранения Azure
* Выпущено для Visual Studio Marketplace
* Используется для создания шаблонов жидкостей, используемых в преобразователе FHIR

## <a name="iot-connector"></a>Соединитель IoT

#### <a name="integration-with-iot-hub-and-iot-central"></a>Интеграция с центром Интернета вещей и IoT Central
* [Microsoft/иомт-fhir](https://github.com/microsoft/iomt-fhir): интеграция с центром Интернета вещей или IOT Central в fhir с нормализацией данных и fhir преобразование нормализованных данных
* Нормализация. сведения о данных устройства извлекаются в общий формат для дальнейшей обработки
* Преобразование FHIR: нормализованные и сгруппированные данные сопоставлены с FHIR. Наблюдение создаются или обновляются в соответствии с настроенными шаблонами, которые связаны с устройством и пациентом.
* [Средства, помогающие создать карту обсуждений](https://github.com/microsoft/iomt-fhir/tree/master/tools/data-mapper). Визуализируйте конфигурацию сопоставления для нормализации входных данных устройства и преобразуйте их в ресурсы FHIR. Разработчики могут использовать это средство для изменения и проверки сопоставлений, сопоставления устройств и сопоставления FHIR, а также экспорта для отправки в соединитель IoT в портал Azure.

#### <a name="healthkit-and-fhir-integration"></a>Интеграция HealthKit и FHIR
* [Microsoft/healthkit-on-fhir](https://github.com/microsoft/healthkit-on-fhir). Библиотека SWIFT, которая автоматизирует экспорт данных Apple Healthkit на сервер fhir

 