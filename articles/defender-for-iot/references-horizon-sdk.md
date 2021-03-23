---
title: Пакет SDK Horizon
description: Пакет SDK для горизонта позволяет разработчикам Azure для Интернета вещей проектировать подключаемые модули, которые декодируются сетевой трафик и могут быть обработаны автоматическим защитником для программ анализа сети IoT.
ms.date: 1/13/2021
ms.topic: article
ms.openlocfilehash: 500f1d78a6a2ab91284c993a5c56ef8e679db83f
ms.sourcegitcommit: f611b3f57027a21f7b229edf8a5b4f4c75f76331
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104782492"
---
# <a name="horizon-proprietary-protocol-dissector"></a>Межадресный протокол IP горизонта

Горизонт — это открытая среда разработки (node), которая используется для защиты устройств Интернета вещей и ICS, использующих собственные протоколы.

Эта среда предоставляет следующие решения для клиентов и технологий.

- Неограниченная полная поддержка общих, специализированных, настраиваемых протоколов или протоколов, которые отличаются от любых стандартных. 

- Новый уровень гибкости и охвата для разработки DPI.

- Инструмент, экспоненциально расширяющий видимость и контроль, без необходимости обновления защитника для версий платформы IoT.

- Обеспечение безопасности при разрешении собственной разработки без использования конфиденциальной информации.

Пакет SDK для горизонта позволяет разработчикам Azure для Интернета вещей проектировать подключаемые модули, которые декодируются сетевой трафик и могут быть обработаны автоматическим защитником для программ анализа сети IoT.

Десекторы протокола разрабатываются как внешние подключаемые модули и интегрируются с обширным диапазоном защитника для служб IoT. Например, службы, обеспечивающие функции мониторинга, оповещения и создания отчетов.

## <a name="secure-development-environment"></a>Безопасная среда разработки 

Горизонт Node позволяет разрабатывать собственные или собственные протоколы, которые не могут быть предоставлены за пределами Организации. Например, из-за юридических положений или корпоративных политик.

Разрабатывайте подключаемые модули для расотраслей без: 

- раскрывается любая собственная информация о том, как определяются протоколы.

- Совместное использование любых конфиденциальных Пкапс.

- нарушение нормативных требований.

## <a name="customization-and-localization"></a>Настройка и локализация  

Пакет SDK поддерживает различные параметры настройки, включая следующие.

  - Текст для кодов функций. 

  - Полный текст локализации для предупреждений, событий и параметров протокола. Дополнительные сведения см. в разделе [Создание файлов сопоставления (JSON)](#create-mapping-files-json).

  :::image type="content" source="media/references-horizon-sdk/localization.png" alt-text="Просмотр полностью локализованных оповещений.":::

## <a name="horizon-architecture"></a>Архитектура горизонта

Модель архитектуры включает три уровня продукта.

:::image type="content" source="media/references-horizon-sdk/architecture.png" alt-text="https://lh6.googleusercontent.com/YFePqJv_6jbI_oy3lCQv-hHB1Qly9a3QQ05uMnI8UdTwhOuxpNAedj_55wseYEQQG2lue8egZS-mlnQZPWfFU1dF4wzGQSJIlUqeXEHg9CG4M7ASCZroKgbghv-OaNoxr3AIZtIh":::

## <a name="defender-for-iot-platform-layer"></a>Защитник для уровня платформы IoT

Обеспечивает немедленную интеграцию и мониторинг пользовательских подключаемых модулей расслеживаний в режиме реального времени в защитнике для платформы IoT без необходимости обновления защитника для версии платформы IoT.

## <a name="defender-for-iot-services-layer"></a>Защитник для уровня служб IoT

Каждая служба разработана в виде конвейера, отделенного от конкретного протокола, что обеспечивает более эффективную и независимую разработку.

Каждая служба разработана в виде конвейера, отделенного от конкретного протокола. Службы прослушивают трафик в конвейере. Они взаимодействуют с данными подключаемого модуля и трафиком, захваченным датчиками, для индексирования развернутых протоколов и анализа полезных данных трафика, а также обеспечивают более эффективную и независимую разработку.

## <a name="custom-dissector-layer"></a>Пользовательский слой рассекторов 

Позволяет создавать подключаемые модули с помощью защитника для собственного пакета SDK для Интернета вещей (включая реализацию C++ и конфигурацию JSON): 

- Определение способа определения протокола

- Определите, как сопоставлять поля, которые необходимо извлечь из трафика, и извлеките их. 

- Определение способа интеграции с защитником для служб IoT

  :::image type="content" source="media/references-horizon-sdk/layers.png" alt-text="Встроенные слои.":::

Защитник для Интернета вещей предоставляет базовые разсекторы для распространенных протоколов. Вы можете создать рассекторы поверх этих протоколов.

## <a name="before-you-begin"></a>Перед началом

## <a name="what-this-sdk-contains"></a>Что содержит этот пакет SDK 

Этот набор содержит файлы заголовков, необходимые для разработки. Процесс разработки требует выполнения основных шагов и дополнительных дополнительных действий, описанных в этом пакете SDK.

Обратитесь <support@cyberx-labs.com> за информацией о получении файлов заголовков и других ресурсов.

## <a name="about-the-environment-and-setup"></a>Сведения о среде и настройке 

### <a name="requirements"></a>Требования 

- Предпочтительная среда разработки — Linux. При разработке в среде Windows рекомендуется использовать виртуальную машину с системой Linux.

- Для процесса компиляции используйте GCC 7.4.0 или более позднюю версию. Используйте любой стандартный класс из stdlib, поддерживаемый в C++ 17.

- Защитник для IoT версии 3,0 и выше.

### <a name="process"></a>Процесс

1. [Скачайте](https://www.eclipse.org/) интегрированную среду разработки Eclipse для разработчиков C/C++. Вы можете использовать любую другую интегрированную среду разработки. В этом документе описывается настройка с помощью интегрированной среды разработки Eclipse.

1. После запуска среды IDE Eclipse и настройки рабочей области (где будут храниться проекты) нажмите клавиши **CTRL + n** и создайте проект в виде проекта C++.

1. На следующем экране задайте имя протокола, который требуется разработать, и выберите тип проекта в виде `Shared Library` и `AND Linux GCC` .

1. Измените свойства проекта в разделе " **параметры сборки C/C++**  >    >  "**Параметры средства**  >  **компилятора GCC C++ компилятор**  >    >  **независимый код для различных делений**.

1. Вставьте примеры кодов, полученные с помощью пакета SDK, и скомпилируйте их.

1. Добавьте артефакты (библиотеку, config.jsи метаданные) в файл tar. gz и измените расширение файла на \<XXX> . hdp, где — \<XXX> это имя подключаемого модуля.

### <a name="research"></a>Исследования 

Перед началом убедитесь, что вы:

- Прочтите спецификацию протокола, если она доступна.

- Определите, какие поля протокола вы планируете извлечь.

- Планируют поставленные цели.

## <a name="about-plugin-files"></a>О файлах подключаемых модулей 

В процессе разработки определяются три файла.

### <a name="json-configuration-file-required"></a>Файл конфигурации JSON (обязательно) 

Этот файл должен определять идентификатор и объявления, зависимости, требования к интеграции, параметры проверки и определения сопоставления для преобразования значений в имена, числа в текст. Дополнительные сведения см. по следующим ссылкам:

- [Подготовка файла конфигурации (JSON)](#prepare-the-configuration-file-json)

- [Подготовка проверки кода реализации](#prepare-implementation-code-validations)

- [Извлечение метаданных устройства](#extract-device-metadata)

- [Подключение к службе индексирования (базовый план)](#connect-to-an-indexing-service-baseline)

### <a name="implementation-code-c-required"></a>Код реализации: C++ (обязательно)

Код реализации (CPP) анализирует необработанный трафик и сопоставляет его со значениями, такими как службы, классы и коды функций. Он извлекает поля слоя и сопоставляет их с именами индексов из файлов конфигурации JSON. Поля, извлекаемые из CPP, определяются в файле конфигурации. Дополнительные сведения см. [в разделе Подготовка кода реализации (C++)](#prepare-the-implementation-code-c).

### <a name="mapping-files-optional"></a>Файлы сопоставления (необязательно)

Вы можете настроить выходной текст подключаемого модуля в соответствии с потребностями вашей корпоративной среды.

:::image type="content" source="media/references-horizon-sdk/localization.png" alt-text="преобразования":::

Можно определить и обновить файлы сопоставления для обновления текста без изменения кода. Каждый файл может сопоставлять одно или несколько полей:

  - Сопоставление значений полей с именами, например 1: сброс, 2: начало, 3: останавливается.

  - Сопоставление текста для поддержки нескольких языков.

Дополнительные сведения см. в разделе [Создание файлов сопоставления (JSON)](#create-mapping-files-json).

## <a name="create-a-dissector-plugin-overview"></a>Создание подключаемого модуля рассектора (обзор)

1. Ознакомьтесь с разделом [о среде и настройке](#about-the-environment-and-setup) .

2.  [Подготовьте код реализации (C++)](#prepare-the-implementation-code-c). Скопируйте **template.jsв** файл и измените его в соответствии с вашими потребностями. Не изменяйте ключи. 

3. [Подготовьте файл конфигурации (JSON)](#prepare-the-configuration-file-json). Скопируйте файл **template. cpp** и реализуйте метод переопределения. Дополнительные сведения см. в разделе [горизонт::p ротокол:: басепарсер](#horizonprotocolbaseparser) for details.

4. [Подготовка проверки кода реализации](#prepare-implementation-code-validations).

## <a name="prepare-the-implementation-code-c"></a>Подготовка кода реализации (C++)

CPP-файл — это средство синтаксического анализа, ответственное за:

- Проверка заголовка и полезных данных пакета (например, длина заголовка или структура полезных данных).

- Извлечение данных из заголовка и полезных данных в определенные поля.

- Извлечение настроенных полей с помощью JSON File.

### <a name="what-to-do"></a>Предпринимаемые действия

Скопируйте файл template **. cpp** и реализуйте метод переопределения. Дополнительные сведения см. в разделе [горизонт::p ротокол:: басепарсер](#horizonprotocolbaseparser).

### <a name="basic-c-template-sample"></a>Пример шаблона Basic C++ 

В этом разделе представлен основной шаблон протокола с стандартными функциями для примера защитника для протокола горизонта Интернета вещей.

```C++
#include “plugin/plugin.h”
namespace {
 class CyberxHorizonSDK: public horizon::protocol::BaseParser
  public:
   std::vector<uint64_t> processDissectAs(const std::map<std::string,
                                                         std::vector<std::string>> &filters) const override {
     return std::vector<uint64_t>();
   }
   horizon::protocol::ParserResult processLayer(horizon::protocol::management::IProcessingUtils &ctx,
                                                horizon::general::IDataBuffer &data) override {
     return horizon::protocol::ParserResult();
   }
 };
}

extern "C" {
  std::shared_ptr<horizon::protocol::BaseParser> create_parser() {
    return std::make_shared<CyberxHorizonSDK>();
  }
}

```

### <a name="basic-c-template-description"></a>Описание шаблона Basic C++  

В этом разделе представлен основной шаблон протокола с описанием стандартных функций для примера защитника для протокола горизонта Интернета вещей. 

### <a name="include-pluginpluginh"></a>#include "подключаемый модуль/подключаемый модуль. h"

Определение, которое использует подключаемый модуль. Файл заголовка содержит все необходимое для завершения разработки.

### <a name="horizonprotocolbaseparser"></a>Горизонт::p ротокол:: Басепарсер

Интерфейс связи между инфраструктурой горизонта и уровнем подключаемого модуля. Дополнительные сведения см. в разделе [архитектура горизонта](#horizon-architecture) для обзора слоев.

Процесслайер — это метод, используемый для обработки данных.

- Первым параметром в коде функции является служебная программа обработки, используемая для извлечения данных, которые были ранее обработаны, создания новых полей и слоев.

- Вторым параметром в коде функции являются текущие данные, передаваемые из предыдущего средства синтаксического анализа.

```C++
horizon::protocol::ParserResult processLayer(horizon::protocol::management::IProcessingUtils &ctx,
                                               horizon::general::IDataBuffer &data) override {

```

### <a name="create_parser"></a>create_parser

Используйте для создания экземпляра средства синтаксического анализа.

:::image type="content" source="media/references-horizon-sdk/code.png" alt-text="https://lh5.googleusercontent.com/bRNtyLpBA3LvDXttSPbxdBK7sHiHXzGXGhLiX3hJ7zCuFhbVsbBhgJlKI6Fd_yniueQqWbClg5EojDwEZSZ219X1Z7osoa849iE9X8enHnUb5to5dzOx2bQ612XOpWh5xqg0c4vR":::

## <a name="protocol-function-code-sample"></a>Пример кода функции протокола 

В этом разделе приводится пример извлечения номера кода (2 байта) и длины сообщения (4 байта).

Это делается в соответствии с порядок следования байтов, предоставленным в файле конфигурации JSON, что означает, что протокол *невелик порядок следования байтов* и датчик выполняется на компьютере с небольшим порядок следования байтов, он будет преобразован.

Для хранения данных также создается слой. Используйте *фиелдсманажер* из служебной программы обработки для создания новых полей. Поле может иметь только один из следующих типов: *строка*, *число*, *необработанные данные*, *массив* (определенного типа) или *сложный*. Этот слой может содержать число, необработанные данные или строку с ИДЕНТИФИКАТОРом.

В приведенном ниже примере извлекаются следующие два поля:

- `function_code_number`

- `headerLength`

Будет создан новый слой, а извлеченное поле будет скопировано в него.

В приведенном ниже примере описывается конкретная функция, которая является основной логикой, реализованной для обработки подключаемого модуля.

```C++
namespace {
 class CyberxHorizonProtocol: public horizon::protocol::BaseParser {
  public:
   
   horizon::protocol::ParserResult processLayer(horizon::protocol::management::IProcessingUtils &ctx,
                                                horizon::general::IDataBuffer &data) override {
     uint16_t codeNumber = data.readUInt16();
     uint32_t headerLength = data.readUInt32();
 
     auto &layer = ctx.createNewLayer();
 
     ctx.getFieldsManager().create(layer,HORIZON_FIELD("code_number"),codeNumber;    
     ctx.getFieldsManager().create(layer,HORIZON_FIELD("header_length"),headerLength);     
     return horizon::protocol::SuccessResult();
   }
 

```

### <a name="related-json-field"></a>Связанное поле JSON 

:::image type="content" source="media/references-horizon-sdk/json.png" alt-text="Связанное поле JSON.":::

## <a name="prepare-the-configuration-file-json"></a>Подготовка файла конфигурации (JSON)

Пакет SDK для горизонта использует стандартный нотация объектов JavaScript (JSON), упрощенный формат хранения и переноса данных и не требует использования собственных языков сценариев.

В этом разделе описываются минимальные объявления конфигурации JSON, связанная структура и приводится пример файла конфигурации, который определяет протокол. Этот протокол автоматически интегрируется со службой обнаружения устройств.

## <a name="file-structure"></a>Структура файла

В приведенном ниже примере описывается структура файла.

:::image type="content" source="media/references-horizon-sdk/structure.png" alt-text="Пример структуры файла.":::

### <a name="what-to-do"></a>Предпринимаемые действия

Скопируйте файл шаблона `config.json` и измените его в соответствии с вашими потребностями. Не изменяйте ключ. Ключи помечены красным цветом в [примере файла конфигурации JSON](#sample-json-configuration-file). 

### <a name="file-naming-requirements"></a>Требования к именованию файлов 

Файл конфигурации JSON должен быть сохранен как `config.json` .

### <a name="json-configuration-file-fields"></a>Поля файла конфигурации JSON

В этом разделе описываются поля конфигурации JSON, которые вы будете определять. Не изменяйте *метки* полей.

### <a name="basic-parameters"></a>Основные параметры

В этом разделе описываются основные параметры.

| Метка параметра | Описание | Тип |
|--|--|--|
| **Идентификатор** | Имя протокола. Удалите значение по умолчанию и добавьте имя протокола в том виде, в каком оно отображается. | Строка |
| **архитектуре** | Определяет, как кодируются многобайтовые данные. Используйте только слово "маленький" или "крупный". Берется из спецификации протокола или записи трафика | Строка |
| **sanity_failure_codes** | Это коды, возвращаемые средством синтаксического анализа при конфликте работоспособности в отношении идентификатора кода. См. раздел Проверка волшебных номеров в разделе C++. | Строка |
| **malformed_codes** | Это коды, которые были правильно идентифицированы, но обнаружена ошибка. Например, если длина поля слишком короткая или длинная, или значение недопустимо. | Строка |
| **dissect_as** | Массив, определяющий, куда должен поступать конкретный трафик протокола. | TCP/UDP, порт и т. д. |
| **поля** | Объявление полей, которые будут извлечены из трафика. Каждое поле имеет собственный идентификатор (Name) и тип (числовой, строковый, необработанный, массив, сложный). Например, [функция](https://docs.google.com/document/d/14nm8cyoGiaE0ODOYQd_xjULxVz9U_bjfPKkcDhOFr5Q/edit#bookmark=id.6s1zcxa9184k) поля, извлеченная в файле средства синтаксического анализа реализации. Поля, записанные в файле конфигурации, являются единственными, которые можно добавить к слою. |  |

### <a name="other-advanced-fields"></a>Другие дополнительные поля 

В этом разделе описываются другие поля.

| Метка параметра | Описание |
|-----------------|--------|
| **разрешить списки** | Можно индексировать значения протокола и отображать их в отчетах интеллектуального анализа данных. Эти отчеты соответствуют базовым показателям сети. :::image type="content" source="media/references-horizon-sdk/data-mining.png" alt-text="Образец представления интеллектуального анализа данных."::: <br /> Дополнительные сведения см. [в разделе Подключение к службе индексирования (базовый план)](#connect-to-an-indexing-service-baseline) . |
| **встроенного по** | Можно извлечь сведения о встроенном по, определить значения индексов и активировать предупреждения встроенного по для протокола подключаемого модуля. Дополнительные сведения см. в разделе [Извлечение данных встроенного по](#extract-firmware-data) . |
| **value_mapping** | Вы можете настроить выходной текст подключаемого модуля в соответствии с потребностями вашей корпоративной среды, определив и обновив файлы сопоставления. Например, сопоставьте с языковыми файлами. Изменения можно легко реализовать в тексте без изменения или воздействия на код. Дополнительные сведения см. в разделе [Создание файлов сопоставления (JSON)](#create-mapping-files-json) для получения дополнительных сведений. |

## <a name="sample-json-configuration-file"></a>Пример файла конфигурации JSON

```json
{
  "id":"CyberX Horizon Protocol",
  "endianess": "big",
  "sanity_failure_codes": {
    "wrong magic": 0
  },
  "malformed_codes": {
    "not enough bytes": 0
  },
  "exports_dissect_as": {
  },
  "dissect_as": {
    "UDP": {
      "port": ["12345"]
    }
  },
  "fields": [
{
      "id": "function",
      "type": "numeric"
    },
    {
      "id": "sub_function",
      "type": "numeric"
    },
    {
      "id": "name",
      "type": "string"
    },
    {
      "id": "model",
      "type": "string"
    },
    {
      "id": "version",
      "type": "numeric"
    }
  ]
}


```

## <a name="prepare-implementation-code-validations"></a>Подготовка проверки кода реализации

В этом разделе описываются функции проверки кода в C++ и приводится пример кода. Доступны два уровня проверки:

- Работоспособности.

- Неправильно сформированный код.

Для создания работающего подключаемого модуля не нужно создавать код проверки. Если вы не подготовите код проверки, то можете ознакомиться с отчетами по интеллектуальному анализу данных в качестве свидетельства об успешной обработке.

Значения полей могут быть сопоставлены с текстом в файлах сопоставления и легко обновляться без влияния на обработку.

## <a name="sanity-code-validations"></a>Проверки кода работоспособности 

Это подтверждает, что переданный пакет соответствует параметрам проверки протокола, что помогает в определении протокола в пределах трафика.

Например, используйте первые 8 байт в качестве *магического числа*. В случае сбоя работоспособности возвращается ответ на сбой работоспособности.

Пример:

```C++
  horizon::protocol::ParserResult 
processLayer(horizon::protocol::management::IProcessingUtils &ctx,
                                               horizon::general::IDataBuffer 
&data) override {

    uint64_t magic = data.readUInt64();

    if (magic != 0xBEEFFEEB) {

      return horizon::protocol::SanityFailureResult(0);

    }
```

Если были развернуты другие соответствующие подключаемые модули, пакет будет проверен для них.

## <a name="malformed-code-validations"></a>Неверно сформированные проверки кода 

Неверно сформированные проверки используются после того, как протокол прошел проверку.

При сбое обработки пакетов на основе протокола возвращается ответ на сбой.

:::image type="content" source="media/references-horizon-sdk/failure.png" alt-text="неправильно сформированный код":::

## <a name="c-sample-with-validations"></a>Пример C++ с проверками

В соответствии с функцией выполняется процесс, как показано в примере ниже.

### <a name="function-20"></a>Функция 20 

- Он обрабатывается как встроенное по.

- Поля считываются в соответствии с функцией.

- Поля добавляются в слой.

### <a name="function-10"></a>Функция 10 

- Функция содержит еще одну подфункцию, которая представляет собой более конкретную операцию.

- Подфункция считывается и добавляется в слой.

По завершении обработки обработка завершается. Возвращаемое значение указывает, успешно ли обрабатывался слой реотраслей. Если это так, слой станет пригодным для использования.

```C++
#include "plugin/plugin.h"

#define FUNCTION_FIRMWARE_RESPONSE 20

#define FUNCTION_SUBFUNCTION_REQUEST 10

namespace {

class CyberxHorizonSDK: public horizon::protocol::BaseParser {

 public:

  std::vector<uint64_t> processDissectAs(const std::map<std::string,

                                                        std::vector<std::string>> &filters) const override {

    return std::vector<uint64_t>();

  }

  horizon::protocol::ParserResult processLayer(horizon::protocol::management::IProcessingUtils &ctx,

                                               horizon::general::IDataBuffer &data) override {

    uint64_t magic = data.readUInt64();

    if (magic != 0xBEEFFEEB) {

      return horizon::protocol::SanityFailureResult(0);

    }

    uint16_t function = data.readUInt16();

    uint32_t length = data.readUInt32();

    if (length > data.getRemaningData()) {

      return horizon::protocol::MalformedResult(0);

    }

    auto &layer = ctx.createNewLayer();

    ctx.getFieldsManager().create(layer, HORIZON_FIELD("function"), function);

    switch (function) {

      case FUNCTION_FIRMWARE_RESPONSE: {

        uint8_t modelLength = data.readUInt8();

        std::string model = data.readString(modelLength);

        uint16_t firmwareVersion = data.readUInt16();

        uint8_t nameLength = data.readUInt8();

        std::string name = data.readString(nameLength);

        ctx.getFieldsManager().create(layer, HORIZON_FIELD("model"), model);

        ctx.getFieldsManager().create(layer, HORIZON_FIELD("version"), firmwareVersion);

        ctx.getFieldsManager().create(layer, HORIZON_FIELD("name"), name);

      }

      break;

      case FUNCTION_SUBFUNCTION_REQUEST: {

       uint8_t subFunction = data.readUInt8();

       ctx.getFieldsManager().create(layer, HORIZON_FIELD("sub_function"), subFunction);

      }

      break;

    }

    return horizon::protocol::SuccessResult();

  }

};

}

extern "C" {

 std::shared_ptr<horizon::protocol::BaseParser> create_parser() {

   return std::make_shared<CyberxHorizonSDK>();

 }

}
```

## <a name="extract-device-metadata"></a>Извлечение метаданных устройства

Вы можете извлечь следующие метаданные для ресурсов:

  - `Is_distributed_control_system` — Указывает, является ли протокол частью распределенной системы управления. (пример: протокол СКАДА должен иметь значение false)

  - `Has_protocol_address` — Указывает, имеется ли адрес протокола; конкретный адрес для текущего протокола, например идентификатор единицы MODBUS

  - `Is_scada_protocol` — Указывает, относится ли протокол к сетям.

  - `Is_router_potential` — Указывает, используется ли протокол главным образом маршрутизаторами. Например, LLDP, CDP или STP.

Чтобы добиться этого, необходимо обновить файл конфигурации JSON с помощью свойства Metadata.

## <a name="json-sample-with-metadata"></a>Пример JSON с метаданными 

```json

{
 "id":"CyberX Horizon Protocol",
 "endianess": "big",
 "metadata": {
   "is_distributed_control_system": false,
   "has_protocol_address": false,
   "is_scada_protocol": true,
   "is_router_potenial": false
 },
 "sanity_failure_codes": {
   "wrong magic": 0
 },
 "malformed_codes": {
   "not enough bytes": 0
 },
 "exports_dissect_as": {
 },
 "dissect_as": {
   "UDP": {
     "port": ["12345"]
   }
 },
 "fields": [
   {
     "id": "function",
     "type": "numeric"
},
   {
     "id": "sub_function",
     "type": "numeric"
   },
   {
     "id": "name",
     "type": "string"
   },
   {
     "id": "model",
     "type": "string"
   },
   {
     "id": "version",
     "type": "numeric"
   }
 ],
}

```

## <a name="extract-programming-code"></a>Извлечь программный код 

При возникновении события программирования можно извлечь содержимое кода. Извлеченное содержимое позволяет:

- Сравнение содержимого файла кода в различных событиях программирования.

- Активировать оповещение при несанкционированном программировании.  

- Активация события для получения файла программного кода.

  :::image type="content" source="media/references-horizon-sdk/change.png" alt-text="Журнал изменений в программировании.":::

  :::image type="content" source="media/references-horizon-sdk/view.png" alt-text="Чтобы просмотреть программирование, нажмите кнопку.":::

  :::image type="content" source="media/references-horizon-sdk/unauthorized.png" alt-text="Оповещение о незаконном программировании.":::

Чтобы добиться этого, необходимо обновить файл конфигурации JSON с помощью `code_extraction` Свойства. 

### <a name="json-configuration-fields"></a>Поля конфигурации JSON 

В этом разделе описываются поля конфигурации JSON. 

- **Method**

  Указывает способ получения файлов программных событий. 

  ВСЕ (каждое действие программирования приведет к получению всех файлов кода, даже если имеются файлы без изменений).

- **file_type**  

  Указывает тип содержимого кода.  

  ТЕКСТ (каждый файл кода содержит текстовую информацию).

- **code_data_field**
  
  Указывает поле реализации, используемое для предоставления содержимого кода.

  Полями.

- **code_name_field**

  Указывает поле реализации, используемое для предоставления имени файла кода.

  Полями.

- **size_limit**

  Указывает ограничение размера каждого содержимого файла кода в БАЙТах, если размер файла кода превышает установленный предел. Если это поле не указано, значение по умолчанию будет 15 000 000, равное 15 МБ.

  Нумерация.

- **метаданных**

  Указывает дополнительные сведения для файла кода.

  Массив, содержащий объекты с двумя свойствами:

    - Name (строка) — указывает, что ключ метаданных Кссенсе в настоящее время поддерживает только ключ имени пользователя.
 
    - value (поле) — указывает поле реализации, используемое для предоставления данных метаданных.

## <a name="json-sample-with-programming-code"></a>Пример JSON с программным кодом

```json
{
  "id":"CyberXHorizonProtocol",
  "endianess": "big",
  "metadata": {
    "is_distributed_control_system": false,
    "has_protocol_address": false,
    "is_scada_protocol": true,
    "is_router_potenial": false
  },
  "sanity_failure_codes": {
    "wrong magic": 0
  },
  "malformed_codes": {
    "not enough bytes": 0
  },
  "exports_dissect_as": {
  },
  "dissect_as": {
    "UDP": {
      "port": ["12345"]
    }
  },
  "fields": [
    {
      "id": "function",
      "type": "numeric"
    },
    {
      "id": "sub_function",
      "type": "numeric"
    },
    {
      "id": "name",
      "type": "string"
    },
    {
      "id": "model",
      "type": "string"
    },
    {
      "id": "version",
      "type": "numeric"
    },
    {
      "id": "script",
      "type": "string"
    },
    {
      "id": "script_name",
      "type": "string"
    }, 
      "id": "username",
      "type": "string"
    }
  ],
"whitelists": [
    {
      "name": "Functions",
      "alert_title": "New Activity Detected - CyberX Horizon 
                                                Protocol Function",
      "alert_text": "There was an attempt by the source to 
                        invoke a new function on the destination",
       "fields": [
        {
          "name": "Source",
          "value": "IPv4.src"
        },
        {
          "name": "Destination",
          "value": "IPv4.dst"
        },
        {
          "name": "Function",
          "value": "CyberXHorizonProtocol.function"
        }
      ]
    },
"firmware": {
    "alert_text": "Firmware was changed on a network asset. 
                  This may be a planned activity, 
                  for example an authorized maintenance procedure",
    "index_by": [
      {
        "name": "Device",
        "value": "IPv4.src",
        "owner": true
      }
    ],
    "firmware_fields": [,
      {
        "name": "Model",
        "value": "CyberXHorizonProtocol.model",
        "firmware_index": "model"
      },
      {
        "name": "Revision",
        "value": "CyberXHorizonProtocol.version",
        “Firmware_index”: “firmware_version”
      },
      {
        "name": "Name",
        "value": "CyberXHorizonProtocol.name"
      }
    ]
  },
"code_extraction": {
    "method": "ALL",
    "file_type": "TEXT",
    "code_data_field": "script",
    "code_name_field": "script_name",
    "size_limit": 15000000,
    "metadata": [
      {
        "name": "username",
        "value": "username"
      }
    ]
  }
}

```
## <a name="custom-horizon-alerts"></a>Пользовательские оповещения горизонта

Код функции некоторых протоколов может указывать на ошибку. Например, если протокол управляет контейнером с конкретным химическим, который должен храниться всегда в определенной температуре. В этом случае может быть код функции, указывающий на ошибку в термометре. Например, если код функции равен 25, вы можете активировать оповещение в веб-консоли, которое указывает на проблему с контейнером. В этом случае можно определить глубокие предупреждения пакетов.

Добавьте параметр **Alerts** в `config.json` подключаемый модуль.

```json
“alerts”: [{
    “id”: 1,
    “message”: “Problem with thermometer at station {IPv4.src}”,
    “title”: “Thermometer problem”,
    “expression”: “{CyberXHorizonProtocol.function} == 25”
}]

```

## <a name="json-configuration-fields"></a>Поля конфигурации JSON

В этом разделе описываются поля конфигурации JSON. 

| Имя поля | Описание | Возможные значения |
|--|--|--|
| **Идентификатор** | Представляет один идентификатор предупреждения. Он должен быть уникальным в этом контексте. | Числовое значение 0-10000 |
| **message** | Сведения, отображаемые пользователю. Это поле позволяет использовать разные поля. | Используйте любое поле из протокола или любого протокола более низкого уровня. |
| **title** | Заголовок оповещения |  |
| **expression** | Если вы хотите, чтобы это оповещение отображалось. | Используйте любое числовое поле, обнаруженное на более низких уровнях, или текущий слой.</br></br> Каждое поле должно быть программой-оболочкой `{}` , чтобы пакет SDK мог определить его как поле, поддерживаемые логические операторы</br> = =-Равно</br> <=-меньше или равно</br> >=-больше или равно</br> > — больше</br> <-меньше</br> ~ =-Не равно |

## <a name="more-about-expressions"></a>Дополнительные сведения о выражениях

Каждый раз, когда пакет SDK для горизонта вызывает выражение и принимает *значение true*, в датчике будет активировано предупреждение.

В одном предупреждении можно включать несколько выражений. Например,

`{CyberXHorizonProtocol.function} == 25 and {IPv4.src} == 269488144`.

Это выражение проверяет код функции только в том случае, если пакет IPv4 Исх имеет значение 10.10.10.10. Это необработанное представление IP-адреса в числовом представлении.

`and` `or` Для соединения выражений можно использовать или.

## <a name="json-sample-custom-horizon-alerts"></a>Примеры настраиваемых оповещений горизонта JSON

```json
 "id":"CyberX Horizon Protocol",
 "endianess": "big",
 "metadata": {
   "is_distributed_control_system": false,
   "has_protocol_address": false,
   "is_scada_protocol": true,
   "is_router_potenial": false
 },
 "sanity_failure_codes": {
   "wrong magic": 0
 },
 "malformed_codes": {
   "not enough bytes": 0
 },
 "exports_dissect_as": {
 },
 "dissect_as": {
   "UDP": {
     "port": ["12345"]
   }
 },
 …………………………………….
 “alerts”: [{
“id”: 1,
“message”: “Problem with thermometer at station {IPv4.src}”,
“title”: “Thermometer problem”,
“expression”: “{CyberXHorizonProtocol.function} == 25”

```

## <a name="connect-to-an-indexing-service-baseline"></a>Подключение к службе индексирования (базовый план)

Можно индексировать значения протокола и отображать их в отчетах интеллектуального анализа данных.

:::image type="content" source="media/references-horizon-sdk/data-mining.png" alt-text="Представление параметра интеллектуального анализа данных.":::

Эти значения впоследствии могут быть сопоставлены с конкретными текстовыми, например для номеров сопоставления в виде текста или добавления информации на любом языке.

:::image type="content" source="media/references-horizon-sdk/localization.png" alt-text="преобразования":::

Дополнительные сведения см. в разделе [Создание файлов сопоставления (JSON)](#create-mapping-files-json) для получения дополнительных сведений.

Можно также использовать значения из ранее проанализированных протоколов, чтобы извлечь дополнительные сведения.

Например, для значения, основанного на TCP, можно использовать значения из уровня IPv4. На этом уровне можно извлекать значения, такие как источник пакета и назначение.

Чтобы добиться этого, необходимо обновить файл конфигурации JSON с помощью `whitelists` Свойства.

## <a name="allow-list-data-mining-fields"></a>Разрешить поля списка (интеллектуального анализа данных)

Доступны следующие поля списка разрешений:

- Имя — имя, используемое для индексирования.

- alert_title — краткий уникальный заголовок, объясняющий событие.

- alert_text — дополнительные сведения

Можно добавить несколько списков разрешений, обеспечивая полную гибкость индексирования.

## <a name="json-sample-with-indexing"></a>Пример JSON с индексацией 

```json
{
  "id":"CyberXHorizonProtocol",
  "endianess": "big",
  "metadata": {
    "is_distributed_control_system": false,
    "has_protocol_address": false,
    "is_scada_protocol": true,
    "is_router_potenial": false
  },
  "sanity_failure_codes": {
    "wrong magic": 0
  },
  "malformed_codes": {
    "not enough bytes": 0
  },
  "exports_dissect_as": {
  },
  "dissect_as": {
    "UDP": {
      "port": ["12345"]
    }
  },
  "fields": [
    {
      "id": "function",
      "type": "numeric"
    },
    {
      "id": "sub_function",
      "type": "numeric"
    },
    {
      "id": "name",
      "type": "string"
    },
    {
      "id": "model",
      "type": "string"
    },
    {
      "id": "version",
      "type": "numeric"
    }
  ],
"whitelists": [
    {
      "name": "Functions",
      "alert_title": "New Activity Detected - CyberX Horizon Protocol Function",
      "alert_text": "There was an attempt by the source to invoke a new function on the destination",
       "fields": [
        {
          "name": "Source",
          "value": "IPv4.src"
        },
        {
          "name": "Destination",
          "value": "IPv4.dst"
        },
        {
          "name": "Function",
          "value": "CyberXHorizonProtocol.function"
        }
      ]
    }

```
## <a name="extract-firmware-data"></a>Извлечение данных встроенного по

Можно извлечь сведения о встроенном по, определить значения индексов и активировать предупреждения встроенного по для протокола подключаемого модуля. Например,

- Извлеките модель встроенного по или версию. Эти сведения можно использовать для обнаружения CVE.

- Активировать оповещение при обнаружении новой версии встроенного по.

Чтобы добиться этого, необходимо обновить файл конфигурации JSON с помощью свойства встроенного по.

## <a name="firmware-fields"></a>Поля встроенного по

В этом разделе описываются поля конфигурации встроенного по JSON.

- **name**
  
  Указывает, как поле отображается в консоли датчика.

- **value**

 Указывает поле реализации, используемое для предоставления данных. 

- **firmware_index:**

  Выберите один из следующих вариантов.  
  - **model**: модель устройства. Включает обнаружение CVE.
  - **последовательный**: серийный номер устройства. Серийный номер не всегда доступен для всех протоколов. Это значение уникально для каждого устройства.
  - **стойка**: указывает идентификатор стойки, если устройство входит в стойку.
  - **слот**— идентификатор слота, если устройство входит в стойку.  
  - **module_address**: используется для представления иерархии, если модуль можно представить за другим устройством. Применяется вместо этого, если используется сочетание отсека стойки, которое является более простым представлением.  
  - **firmware_version**: указывает версию устройства. Включает обнаружение CVE.
  - **alert_text**: Указывает текст, описывающий отклонения встроенного по, например изменения версии.  
  - **index_by**: указывает поля, используемые для обнаружения и индексации устройства. В приведенном ниже примере устройство идентифицируется по его IP-адресу. В некоторых протоколах можно использовать более сложный индекс. Например, если другое устройство подключено и вы умеете извлекать его внутренний путь. Например, идентификатор единицы MODBUS можно использовать как часть индекса в качестве составного сочетания IP-адреса и идентификатора единицы измерения.
  - **firmware_fields**: указывает, какие поля являются полями метаданных устройства. В этом примере используются следующие действия: модель, редакция и имя. Каждый протокол может определять собственные данные встроенного по.

## <a name="json-sample-with-firmware"></a>Пример JSON с встроенным по 

```json
{
  "id":"CyberXHorizonProtocol",
  "endianess": "big",
  "metadata": {
    "is_distributed_control_system": false,
    "has_protocol_address": false,
    "is_scada_protocol": true,
    "is_router_potenial": false
  },
  "sanity_failure_codes": {
    "wrong magic": 0
  },
  "malformed_codes": {
    "not enough bytes": 0
  },
  "exports_dissect_as": {
  },
  "dissect_as": {
    "UDP": {
      "port": ["12345"]
    }
  },
  "fields": [
    {
      "id": "function",
      "type": "numeric"
    },
    {
      "id": "sub_function",
      "type": "numeric"
    },
    {
      "id": "name",
      "type": "string"
    },
    {
      "id": "model",
      "type": "string"
    },
    {
      "id": "version",
      "type": "numeric"
    }
  ],
"whitelists": [
    {
      "name": "Functions",
      "alert_title": "New Activity Detected - CyberX Horizon Protocol Function",
      "alert_text": "There was an attempt by the source to invoke a new function on the destination",
       "fields": [
        {
          "name": "Source",
          "value": "IPv4.src"
        },
        {
          "name": "Destination",
          "value": "IPv4.dst"
        },
        {
          "name": "Function",
          "value": "CyberXHorizonProtocol.function"
        }
      ]
    },
"firmware": {
    "alert_text": "Firmware was changed on a network asset.   
                  This may be a planned activity, for example an authorized maintenance procedure",
    "index_by": [
      {
        "name": "Device",
        "value": "IPv4.src",
        "owner": true
      }
    ],
    "firmware_fields": [,
      {
        "name": "Model",
        "value": "CyberXHorizonProtocol.model",
        "firmware_index": "model"
      },
      {
        "name": "Revision",
        "value": "CyberXHorizonProtocol.version",
        “Firmware_index”: “firmware_version”
      },
      {
        "name": "Name",
        "value": "CyberXHorizonProtocol.name"
      }
    ]
  }
}

```
## <a name="extract-device-attributes"></a>Извлечение атрибутов устройства 

Вы можете улучшить доступ к данным на устройстве при инвентаризации, интеллектуальном анализе данных и других отчетах.

- Имя

- Тип

- Vendor

- Операционная система

Чтобы добиться этого, необходимо обновить файл конфигурации JSON, используя свойство **Properties** . 

Это можно сделать после написания базового подключаемого модуля и извлечения обязательных полей.

## <a name="properties-fields"></a>Поля свойств

В этом разделе описываются поля конфигурации свойств JSON. 

**config_file** 

Содержит имя файла, определяющего способ обработки каждого ключа в `key` полях. Сам файл конфигурации должен быть в формате JSON и включен в папку протокола подключаемого модуля.

Каждый ключ в JSON определяет набор действий, которые должны выполняться при извлечении этого ключа из пакета.

Каждый ключ может иметь следующее:

- **Данные пакетов** — указывает свойства, которые будут обновлены на основе данных, извлеченных из пакета (поле реализации, используемое для предоставления этих данных).

- **Статические данные** — указывает предопределенный набор `property-value` действий, которые должны быть обновлены.

В этом файле можно настроить следующие свойства: 

- **Имя** — указывает имя устройства.

- **Тип** — указывает тип устройства.

- **Поставщик** — указывает поставщика устройства.

- **Описание** — указывает модель встроенного по устройства (более низкий приоритет, чем "модель").

- **Операционная система — указывает** операционную систему устройства.

### <a name="fields"></a>Поля

| Поле | Описание |
|--|--|
| ключ | Указывает ключ. |
| значение | Указывает поле реализации, используемое для предоставления данных. |
| is_static_key | Указывает, `key` является ли поле производным от пакета или его значением. |

### <a name="working-with-static-keys-only"></a>Работа только с статическими ключами

Если вы работаете со статическими ключами, вам не нужно настраивать `config.file` . Можно настроить только JSON File.

## <a name="json-sample-with-properties"></a>Пример JSON со свойствами 

```json
{
  "id":"CyberXHorizonProtocol",
  "endianess": "big",
  "metadata": {
    "is_distributed_control_system": false,
    "has_protocol_address": false,
    "is_scada_protocol": true,
    "is_router_potenial": false
  },
  "sanity_failure_codes": {
    "wrong magic": 0
  },
  "malformed_codes": {
    "not enough bytes": 0
  },
  "exports_dissect_as": {
  },
  "dissect_as": {
    "UDP": {
      "port": ["12345"]
    }
  },
  "fields": [
    {
      "id": "function",
      "type": "numeric"
    },
  
      "id": "sub_function",
      "type": "numeric"
    },
    {
      "id": "name",
      "type": "string"
    },
    {
      "id": "model",
      "type": "string"
    },
    {
      "id": "version",
      "type": "numeric"
    },
    {
      "id": "vendor",
      "type": "string"
    }
  ],
"whitelists": [
    {
      "name": "Functions",
      "alert_title": "New Activity Detected - CyberX Horizon Protocol Function",
      "alert_text": "There was an attempt by the source to invoke a new function on the destination",
       "fields": [
        {
          "name": "Source",
          "value": "IPv4.src"
        },
        {
          "name": "Destination",
          "value": "IPv4.dst"
        },
        {
          "name": "Function",
          "value": "CyberXHorizonProtocol.function"
        }
      ]
    },
"firmware": {
    "alert_text": "Firmware was changed on a network asset. 
                  This may be a planned activity, for example an authorized maintenance procedure",
    "index_by": [
      {
        "name": "Device",
        "value": "IPv4.src",
        "owner": true
      }
    ],
    "firmware_fields": [,
      {
        "name": "Model",
        "value": "CyberXHorizonProtocol.model",
        "firmware_index": "model"
      },
      {
        "name": "Revision",
        "value": "CyberXHorizonProtocol.version",
        “Firmware_index”: “firmware_version”
      },
      {
        "name": "Name",
        "value": "CyberXHorizonProtocol.name"
      }
    ]
  }
"properties": {
    "config_file": "config_file_example",
"fields": [
  {
    "key": "vendor",
    "value": "CyberXHorizonProtocol.vendor",
    "is_static_key": true
  },
{
    "key": "name",
    "value": "CyberXHorizonProtocol.vendor",
    "is_static_key": true
  },

]
  }
}

```

## <a name="config_file_exapmle-json"></a>CONFIG_FILE_EXAPMLE JSON 

```json
{
"someKey": {
  "staticData": {
    "model": "FlashSystem", 
    "vendor": "IBM", 
    "type": "Storage"}
  }
  "packetData": [
    "name”  
  ]
}

```

## <a name="create-mapping-files-json"></a>Создание файлов сопоставления (JSON)

Вы можете настроить выходной текст подключаемого модуля в соответствии с потребностями вашей корпоративной среды, определив файлы сопоставления и обновления. Изменения можно легко реализовать в тексте без изменения или воздействия на код. Каждый файл может сопоставлять одно или несколько полей.

- Сопоставление значений полей с именами, например 1: сброс, 2: начало, 3: останавливается.

- Сопоставление текста для поддержки нескольких языков.

Можно определить два типа файлов сопоставлений.

  - [Файл простого сопоставления](#simple-mapping-file).

  - [Файл сопоставления зависимостей](#dependency-mapping-file).

    :::image type="content" source="media/references-horizon-sdk/localization.png" alt-text="Ether NET":::

    :::image type="content" source="media/references-horizon-sdk/unhandled.png" alt-text="Представление необработанных оповещений.":::

    :::image type="content" source="media/references-horizon-sdk/policy-violation.png" alt-text="Список известных нарушений политики.":::

## <a name="file-naming-and-storage-requirements"></a>Требования к именованию файлов и хранилищу 

Файлы сопоставления должны быть сохранены в папке метаданных.

Имя файла должно соответствовать ИДЕНТИФИКАТОРу файла конфигурации JSON.

:::image type="content" source="media/references-horizon-sdk/json-config.png" alt-text="Пример файла конфигурации JSON.":::

## <a name="simple-mapping-file"></a>Файл простого сопоставления 

В следующем примере представлен базовый JSON-файл в качестве значения ключа.

При создании списка разрешений он содержит одно или несколько сопоставленных полей. Значение будет преобразовано из числа, строки или любого типа в в форматированный текст, представленный в файле сопоставления.

```json
{
  “10”: “Read”,
  “20”: “Firmware Data”,
  “3”: “Write”
}

```

## <a name="dependency-mapping-file"></a>Файл сопоставления зависимостей 

Чтобы указать, что файл является файлом зависимостей, добавьте ключевое слово `dependency` в конфигурацию сопоставления.

```json
dependency": { "field": "CyberXHorizonProtocol.function"  }}]
  }],
  "firmware": {
    "alert_text": "Firmware was changed on a network asset. This may be a planned activity, for example an authorized maintenance procedure",
    "index_by": [{ "name": "Device", "value": "IPv4.src", "owner": true }],
    "firmware_fields": [{ "name": "Model", "value":

```

Файл содержит сопоставление между полем зависимости и полем функции. Например, между функцией и подфункцией. Подфункция изменяется в соответствии с предоставленной функцией.

В ранее настроенном списке разрешений нет конфигурации зависимостей, как показано ниже.

```json
"whitelists": [
{
"name": "Functions",
"alert_title": "New Activity Detected - CyberX Horizon Protocol Function",
"alert_text": "There was an attempt by the source to invoke a new function on the destination",
"fields": [
{
"name": "Source",
"value": "IPv4.src"
},
{
"name": "Destination",
"value": "IPv4.dst"
},
{
"name": "Function",
"value": "CyberXHorizonProtocol.function"
}
]
}

```

Зависимость может основываться на определенном значении или поле. В приведенном ниже примере он основан на поле. Если он основан на значении, определите извлекаемое значение из файла сопоставления.

В приведенном ниже примере зависимость будет соответствовать значению поля.

Например, в подфункции с пятью значение изменяется в зависимости от функции.

  - Если это функция Read, то пять означает чтение памяти.

  - Если это функция записи, то для чтения из файла используется то же значение.

    ```json
    {
      “10”: {
         “5”:  “Memory”,
         “6”: “File”,
         “7” “Register”
       },
      “3”: {
       “5”: “File”,
       “7”: “Memory”,
       “6”, “Register”
      }
    }

    ```

### <a name="sample-file"></a>Образец файла

```json
{
  "id":"CyberXHorizonProtocol",
  "endianess": "big",
  "metadata": {"is_distributed_control_system": false, "has_protocol_address": false, "is_scada_protocol": true, "is_router_potenial": false},
  "sanity_failure_codes": { "wrong magic": 0 },
  "malformed_codes": { "not enough bytes": 0  },
  "exports_dissect_as": {  },
  "dissect_as": { "UDP": { "port": ["12345"] }},
  "fields": [{ "id": "function", "type": "numeric" }, { "id": "sub_function", "type": "numeric" },
     {"id": "name", "type": "string" }, { "id": "model", "type": "string" }, { "id": "version", "type": "numeric" }],
  "whitelists": [{
      "name": "Functions",
      "alert_title": "New Activity Detected - CyberX Horizon Protocol Function",
      "alert_text": "There was an attempt by the source to invoke a new function on the destination",
      "fields": [{ "name": "Source", "value": "IPv4.src" }, { "name": "Destination", "value": "IPv4.dst" },
        { "name": "Function", "value": "CyberXHorizonProtocol.function" },
        { "name": "Sub function", "value": "CyberXHorizonProtocol.sub_function", "dependency": { "field": "CyberXHorizonProtocol.function"  }}]
  }],
  "firmware": {
    "alert_text": "Firmware was changed on a network asset. This may be a planned activity, for example an authorized maintenance procedure",
    "index_by": [{ "name": "Device", "value": "IPv4.src", "owner": true }],
    "firmware_fields": [{ "name": "Model", "value": "CyberXHorizonProtocol.model", "firmware_index": "model" },
       { "name": "Revision", "value": "CyberXHorizonProtocol.version", "firmware_index": "firmware_version" },
       { "name": "Name", "value": "CyberXHorizonProtocol.name" }]
  },
  "value_mapping": {
      "CyberXHorizonProtocol.function": {
        "file": "function-mapping"
      },
      "CyberXHorizonProtocol.sub_function": {
        "dependency": true,
        "file": "sub_function-mapping"
      }
  }
}

```

## <a name="json-sample-with-mapping"></a>Пример JSON с сопоставлением

```json
{
  "id":"CyberXHorizonProtocol",
  "endianess": "big",
  "metadata": {
    "is_distributed_control_system": false,
    "has_protocol_address": false,
    "is_scada_protocol": true,
    "is_router_potenial": false
  },
  "sanity_failure_codes": {
    "wrong magic": 0
  },
  "malformed_codes": {
    "not enough bytes": 0
  },
  "exports_dissect_as": {
  },
  "dissect_as": {
    "UDP": {
      "port": ["12345"]
    }
  },
  "fields": [
    {
      "id": "function",
      "type": "numeric"
    },
    {
      "id": "sub_function",
      "type": "numeric"
    },
    {
      "id": "name",
      "type": "string"
    },
    {
      "id": "model",
      "type": "string"
    },
    {
      "id": "version",
      "type": "numeric"
    }
  ],
"whitelists": [
    {
      "name": "Functions",
      "alert_title": "New Activity Detected - CyberX Horizon Protocol Function",
      "alert_text": "There was an attempt by the source to invoke a new function on the destination",
       "fields": [
        {
          "name": "Source",
          "value": "IPv4.src"
        },
        {
          "name": "Destination",
          "value": "IPv4.dst"
        },
        {
          "name": "Function",
          "value": "CyberXHorizonProtocol.function"
        },
        {
          “name”: “Sub function”,
          “value”: “CyberXHorizonProtocol.sub_function”,
          “dependency”: {
              “field”: “CyberXHorizonProtocol.function”
        }
      ]
    },
"firmware": {
    "alert_text": "Firmware was changed on a network asset. This may be a planned activity, for example an authorized maintenance procedure",
    "index_by": [
      {
        "name": "Device",
        "value": "IPv4.src",
        "owner": true
      }
    ],
    "firmware_fields": [,
      {
        "name": "Model",
        "value": "CyberXHorizonProtocol.model",
        "firmware_index": "model"
      },
      {
        "name": "Revision",
        "value": "CyberXHorizonProtocol.version",
        “Firmware_index”: “firmware_version”
      },
      {
        "name": "Name",
        "value": "CyberXHorizonProtocol.name"
      }
    ]
  },
"value_mapping": {
    "CyberXHorizonProtocol.function": {
      "file": "function-mapping"
    },
    "CyberXHorizonProtocol.sub_function": {
      "dependency": true,
      "file": "sub_function-mapping"
    }
}

```
## <a name="package-upload-and-monitor-the-plugin"></a>Упаковка, отправка и мониторинг подключаемого модуля 

В этом разделе даются инструкции

  - Упакуйте подключаемый модуль.

  - Отправьте подключаемый модуль.

  - Отслеживайте и отлаживать подключаемый модуль, чтобы оценить, насколько хорошо он работает.

Чтобы упаковать подключаемый модуль, выполните следующие действия.

1. Добавление **артефакта** (может быть, библиотеки, config.jsили метаданных) в `tar.gz` файл.

1. Измените расширение файла на \<XXX.hdp> , где — \<XXX> это имя подключаемого модуля.

Чтобы войти в консоль горизонта, выполните следующие действия.

1.  Войдите в CLI датчика в качестве администратора, Циберкс или пользователя поддержки.

2.  В файле: `/var/cyberx/properties/horizon.properties` измените свойство **UI. Enabled** на **true** ( `horizon.properties:ui.enabled=true` ).

3.  Войдите в консоль датчика.

4.  В главном меню выберите параметр **горизонт** .

    :::image type="content" source="media/references-horizon-sdk/horizon.png" alt-text="Выберите параметр горизонт в левой части области.":::

    Откроется консоль горизонт.

    :::image type="content" source="media/references-horizon-sdk/plugins.png" alt-text="Представление консоли горизонта и всех ее подключаемых модулей.":::

## <a name="plugins-pane"></a>Область подключаемых модулей

В области подключаемый модуль перечислены:

  - Подключаемые модули инфраструктуры. подключаемые модули инфраструктуры, установленные по умолчанию с защитником для Интернета вещей.

  - Подключаемые модули приложений. устанавливаемые по умолчанию подключаемые модули приложений с защитником для IoT и другими подключаемыми модулями, разработанными защитником для IoT или внешними разработчиками
    
Включение и отключение подключаемых модулей, отправленных с помощью переключателя.

:::image type="content" source="media/references-horizon-sdk/toggle.png" alt-text="Переключатель CIP.":::

### <a name="uploading-a-plugin"></a>Отправка подключаемого модуля

После создания и упаковки подключаемого модуля его можно передать в защитник для датчика IoT. Чтобы обеспечить полное покрытие сети, необходимо передать подключаемый модуль на каждый датчик в Организации.

Для отправки:

1.  Войдите в систему датчика.


2. Щелкните **Отправить**.

    :::image type="content" source="media/references-horizon-sdk/upload.png" alt-text="Отправьте подключаемые модули.":::

3. Перейдите к подключаемому модулю и перетащите его в диалоговое окно подключаемый модуль. Убедитесь, что префикс имеет значение `.hdp` . Подключаемый модуль загружается.

## <a name="plugin-status-overview"></a>Обзор состояния подключаемого модуля 

Окно **обзора** консоли горизонта содержит сведения о загруженном подключаемом модуле и позволяет отключать и включать их.

:::image type="content" source="media/references-horizon-sdk/overview.png" alt-text="Обзор консоли горизонта.":::

| Поле | Описание |
|--|--|
| Приложение | Имя отправленного подключаемого модуля. |
| :::image type="content" source="media/references-horizon-sdk/switch.png" alt-text="Параметр включения и отключения."::: | Включение **или** **Отключение** подключаемого модуля. Защитник для Интернета вещей не обрабатывает трафик протокола, определенный в подключаемом модуле, при отключении подключаемого модуля. |
| время; | Время, когда данные были проанализированы в последний раз. Обновляется каждые 5 секунд. |
| PPS | Число пакетов в секунду. |
| Пропускная способность | Средняя пропускная способность, обнаруженная в течение последних 5 секунд. |
| малформс | Неверно сформированные проверки используются после того, как протокол прошел проверку. При сбое обработки пакетов на основе протокола возвращается ответ на сбой.   <br><br>В этом столбце указывается число малформ ошибок за последние 5 секунд. Дополнительные сведения см. в статье [синтаксические проверки](#malformed-code-validations) правильности кода для получения подробных сведений. |
| Предупреждения | Пакеты соответствуют структуре и спецификации, но непредвиденное поведение зависит от конфигурации предупреждений подключаемого модуля. |
| ошибки | Число пакетов, не прошедших проверки основных протоколов. Проверяет соответствие пакета определениям протоколов. Отображаемое здесь число указывает на количество ошибок, обнаруженных за последние 5 секунд. Дополнительные сведения см. в статье [Проверка кода работоспособности](#sanity-code-validations) для получения подробных сведений. |
| :::image type="content" source="media/references-horizon-sdk/monitor.png" alt-text="Значок &quot;монитор&quot;."::: | Проверьте сведения о малформ и предупреждениях, обнаруженных для подключаемого модуля. |

## <a name="plugin-details"></a>Сведения о подключаемом модуле

Вы можете отслеживать поведение подключаемого модуля в режиме реального времени, анализируя количество *малформ* и *предупреждений* , обнаруженных для подключаемого модуля. Доступен параметр для закрепления экрана и экспорта для дальнейшего изучения

:::image type="content" source="media/references-horizon-sdk/snmp.png" alt-text="Экран монитора SNMP.":::

Мониторинг:

В разделе Обзор выберите кнопку монитор для подключаемого модуля.

Next Steps

Настройка [API горизонта](references-horizon-api.md)
