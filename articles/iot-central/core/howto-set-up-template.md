---
title: Определение нового типа устройства Интернета вещей в Azure IoT Central | Документация Майкрософт
description: В этой статье показано, как создать новый шаблон устройства Azure IoT в приложении IoT Central Azure с помощью построителя решений. Вы определите данные телеметрии, состояние, свойства и команды для своего типа.
author: dominicbetts
ms.author: dobett
ms.date: 12/06/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom:
- contperf-fy21q1
- device-developer
ms.openlocfilehash: 22e948a0100f23dbddef8fc138576bb4b9372c77
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363208"
---
# <a name="define-a-new-iot-device-type-in-your-azure-iot-central-application"></a>Определение типа нового устройства Интернета вещей в приложении Azure IoT Central

*Эта статья предназначена для создателей решений и разработчиков устройств.*

Шаблон устройства — это схема, которая определяет характеристики и поведение типа устройства, которое подключается к [приложению Azure IOT Central](concepts-app-templates.md).

Например, сборщик может создать шаблон устройства для подключенного вентилятора со следующими возможностями и компонентами:

- отправка данных телеметрии температуры;
- отправка свойства расположения;
- отправка событий ошибки двигателя вентилятора;
- отправка состояния работы вентилятора;
- Предоставляет свойство скорости вентилятора с возможностью записи
- предоставление команды для перезагрузки устройства;
- Предоставляет общее представление об устройстве с помощью представления

На основе этого шаблона устройства можно создавать и подключать реальные вентиляторы. Все эти вентиляторы имеют измерения, свойства и команды, которые операторы используют для отслеживания их работы и управления ими. Операторы используют [представления устройств](#add-views) и формы для взаимодействия с устройствами вентиляторов. Разработчик устройства использует шаблон, чтобы понять, как устройство взаимодействует с приложением. Дополнительные сведения см. в разделе [данные телеметрии, свойства и команды](concepts-telemetry-properties-commands.md).

> [!NOTE]
> Только администраторы и конструкторы могут создавать, изменять и удалять шаблоны устройств. На странице **Устройства** любой пользователь может создавать устройства на основе имеющихся шаблонов устройств.

В IoT Centralном приложении шаблон устройства использует модель устройства для описания возможностей устройства. Как у сборщика, у вас есть несколько вариантов создания шаблонов устройств:

- Разработайте шаблон устройства в IoT Central, а затем [реализуйте его модель устройства в коде устройства](concepts-telemetry-properties-commands.md).
- Импортируйте шаблон устройства из [каталога устройств сертификации Azure для IOT](https://aka.ms/iotdevcat). Настройте шаблон устройства в соответствие с требованиями в IoT Central.
> [!NOTE]
> IoT Central требуется полная модель со всеми ссылочными интерфейсами в том же файле, при импорте модели из репозитория моделей используйте ключевое слово "развернуто" для получения полной версии.
Например, если выбран диапазон 10.0.0.0/20 для виртуальной сети, для пространства клиентских адресов можно выбрать 10.1.0.0/24. https://devicemodels.azure.com/dtmi/com/example/thermostat-1.expanded.json

- Создание модели устройства с помощью [языка определения цифровых двойников (дтдл) версии 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md). Visual Studio Code имеет расширение, которое поддерживает создание моделей ДТДЛ. Дополнительные сведения см. в статье [Установка и использование средств разработки DTDL](../../iot-pnp/howto-use-dtdl-authoring-tools.md). Затем опубликуйте модель в репозитории общедоступной модели. Дополнительные сведения см. в разделе [хранилище моделей устройств](../../iot-pnp/concepts-model-repository.md). Реализуйте код устройства из модели и подключите реальное устройство к IoT Centralному приложению. IoT Central находит и импортирует модель устройства из общедоступного репозитория и создает шаблон устройства. Затем можно добавить любые облачные свойства, настройки и представления, необходимые IoT Central приложении к шаблону устройства.
- Создайте модель устройства с помощью ДТДЛ. Реализуйте код вашего устройства из модели. Вручную импортируйте модель устройства в приложение IoT Central, а затем добавьте свойства облака, настройки и представления, необходимые для приложения IoT Central.

> [!TIP]
> IoT Central требуется полная модель со всеми ссылочными интерфейсами в одном файле. При импорте модели из репозитория модели используйте *развернутое* ключевое слово, чтобы получить полную версию.
> Например, [https://devicemodels.azure.com/dtmi/com/example/thermostat-1.expanded.json](https://devicemodels.azure.com/dtmi/com/example/thermostat-1.expanded.json) .

Кроме того, можно добавлять шаблоны устройств в IoT Central приложение с помощью [REST API](/learn/modules/manage-iot-central-apps-with-rest-api/) или [интерфейса командной строки](howto-manage-iot-central-from-cli.md).

Некоторые [шаблоны приложений](concepts-app-templates.md) уже содержат шаблоны устройств, которые полезны в сценарии, поддерживаемом шаблоном приложения. Например, см. раздел [архитектура аналитики в магазине](../retail/store-analytics-architecture.md).

## <a name="create-a-device-template-from-the-device-catalog"></a>Создание шаблона устройства из каталога устройств

В качестве построителя вы можете быстро приступить к созданию решения с помощью сертифицированного устройства. Ознакомьтесь со списком в [каталоге устройств Azure IoT](https://catalog.azureiotsolutions.com/alldevices). IoT Central интегрируется с каталогом устройств, чтобы можно было импортировать модель устройства с любого сертифицированного устройства. Чтобы создать шаблон устройства из одного из этих устройств в IoT Central, сделайте следующее:

1. Перейдите на страницу **шаблонов устройств** в приложении IOT Central.
1. Выберите **+ создать**, а затем выберите любое из сертифицированных устройств в каталоге. IoT Central создает шаблон устройства на основе этой модели устройства.
1. Добавьте в шаблон устройства любые свойства облака, настройки или представления.
1. Щелкните **Опубликовать**, чтобы шаблон стал доступен операторам для просмотра и подключения устройств.

## <a name="create-a-device-template-from-scratch"></a>Создание шаблона устройства с нуля

Шаблон устройства содержит следующее:

- _Модель устройства_ , указывающая данные телеметрии, свойства и команды, реализуемые устройством. Эти возможности организованы в один или несколько компонентов.
- _Свойства облака_, определяющие сведения, которые хранятся в приложении IoT Central для ваших устройств. Например, свойство облака может записывать данные о последнем обслуживании устройства. Эти сведения никогда не предоставляются устройству.
- _Настройки_ позволяют построителю переопределять некоторые определения в модели устройства. Например, он может переопределить имя свойства устройства. Имена свойств отображаются в IoT Central представлениях и формах.
- _Представления и формы_ позволяют построителю создавать пользовательский интерфейс, позволяющий операторам отслеживать устройства, подключенные к приложению, и управлять ими.

Чтобы создать шаблон устройства в IoT Central, сделайте следующее.

1. Перейдите на страницу **Шаблоны устройств** в приложении IoT Central.
1. Выберите **+ новое**  >  **устройство IOT**. Затем щелкните **Next: Настройка**.
1. Введите имя шаблона, например **термостата**. Затем нажмите кнопку **Далее: Проверка** , а затем выберите **создать**.
1. IoT Central создает пустой шаблон устройства и позволяет выбрать создание настраиваемой модели с нуля или импорт модели ДТДЛ.

## <a name="manage-a-device-template"></a>Управление шаблоном устройства

Вы можете переименовать или удалить шаблон на его домашней странице.

После добавления модели устройства в шаблон ее можно опубликовать. Пока вы не опубликуете шаблон, вы не сможете подключить устройство на основе этого шаблона, чтобы операторы могли видеть его на странице **Устройства**.

## <a name="create-a-capability-model"></a>Создание модели возможностей

Чтобы создать модель устройства, можно выполнить следующие действия.

- Используйте IoT Central для создания настраиваемой модели с нуля.
- Импорт модели ДТДЛ из JSON-файла. Построитель устройств может использовать Visual Studio Code для создания модели устройства для приложения.
- Выберите одно из устройств в каталоге устройств. Этот параметр импортирует модель устройства, опубликованную производителем для этого устройства. Модель устройства, импортированная подобным образом, будет автоматически опубликована.

## <a name="manage-a-capability-model"></a>Управление моделью возможностей

После создания модели устройства можно выполнить следующие действия.

- Добавьте компоненты в модель. Модель должна иметь по крайней мере один компонент.
- Изменить метаданные модели, такие как идентификатор, пространство имен и имя.
- Удалить модель.

## <a name="create-a-component"></a>Создание компонента

Модель устройства должна иметь по крайней мере один компонент по умолчанию. Компонент — это многократно используемый набор возможностей.

Чтобы создать компонент, выполните следующие действия.

1. Перейдите к модели устройства и выберите **+ Добавить компонент**.

1. На странице **Добавление интерфейса компонента** можно выполнить следующие действия.

    - Создание пользовательского компонента с нуля.
    - Импорт существующего компонента из файла ДТДЛ. Построитель устройств может использовать Visual Studio Code для создания интерфейса компонента для устройства.

1. После создания компонента нажмите кнопку **изменить удостоверение** , чтобы изменить отображаемое имя компонента.

1. Если вы решили создать пользовательский компонент с нуля, можно добавить возможности устройства. Возможности устройства — это данные телеметрии, свойства и команды.

### <a name="telemetry"></a>Телеметрия

Телеметрия — это поток значений, отправляемых с устройства (обычно из датчика). Например, датчик может передавать температуру окружающей среды.

В следующей таблице показаны параметры конфигурации для возможностей телеметрии.

| Поле | Описание |
| ----- | ----------- |
| Отображаемое имя | Отображаемое имя для значения телеметрии, используемого в представлениях и формах. |
| Имя | Имя поля в сообщении телеметрии. IoT Central создает значение для этого поля на основе отображаемого имени, но при необходимости можно выбрать собственное значение. Это поле должно быть буквенно-цифровым. |
| Тип возможности | Телеметрия. |
| Семантический тип | Семантический тип телеметрии, например температура, состояние или событие. Выбор семантического типа определяет, какие из следующих полей будут доступны. |
| схема | Тип данных телеметрии, например double, string или vector. Доступные варианты определяются семантическим типом. Для семантических типов событий и состояний схема недоступна. |
| Уровень серьезности | Доступно только для семантического типа события. Возможны такие степени серьезности: **Ошибка**, **Сведения** или **Предупреждение**. |
| Значения состояния | Доступно только для семантического типа состояния. Определите возможные значения состояния, для каждого из которых есть отображаемое имя, название, тип перечисления и значение. |
| Unit | Единица измерения для значения телеметрии, например **миль/ч**, **%** или **&deg; C**. |
| Единица отображения | Единица отображения для использования в представлениях и формах. |
| Комментировать | Любые комментарии о возможности "Телеметрия". |
| Описание | Описание возможности "Телеметрия". |

### <a name="properties"></a>Свойства

Свойства представляют собой значения на момент во времени. Например, устройство может использовать свойство для передачи значения целевой температуры, которого пытается достичь устройство. Свойства, доступные для записи, можно задать в IoT Central.

В следующей таблице показаны параметры конфигурации для возможности "Свойства".

| Поле | Описание |
| ----- | ----------- |
| Отображаемое имя | Отображаемое имя для значения свойства, используемого в представлениях и формах. |
| Имя | Имя свойства. IoT Central создает значение для этого поля на основе отображаемого имени, но при необходимости можно выбрать собственное значение. Это поле должно быть буквенно-цифровым. |
| Тип возможности | Свойство. |
| Семантический тип | Семантический тип свойства, например температура, состояние или событие. Выбор семантического типа определяет, какие из следующих полей будут доступны. |
| схема | Тип данных свойства, например double, string или vector. Доступные варианты определяются семантическим типом. Для семантических типов событий и состояний схема недоступна. |
| Возможность записи | Если свойство недоступно для записи, устройство может передавать значения свойств в IoT Central. Если свойство доступно для записи, устройство может передавать значения свойств в IoT Central и IoT Central может передавать обновления свойств на устройство.
| Уровень серьезности | Доступно только для семантического типа события. Возможны такие степени серьезности: **Ошибка**, **Сведения** или **Предупреждение**. |
| Значения состояния | Доступно только для семантического типа состояния. Определите возможные значения состояния, для каждого из которых есть отображаемое имя, название, тип перечисления и значение. |
| Unit | Единица измерения для значения свойства, например **миль/ч**, **%** или **&deg; C**. |
| Единица отображения | Единица отображения для использования в представлениях и формах. |
| Комментировать | Любые комментарии о возможности "Свойство". |
| Описание | Описание возможности "Свойство". |

### <a name="commands"></a>Команды

Команды устройства можно вызывать в IoT Central. Команды при необходимости передают параметры на устройство и получают от него ответ. Например, можно вызвать команду для перезагрузки устройства через 10 секунд.

В следующей таблице показаны параметры конфигурации для возможности "Команда".

| Поле | Описание |
| ----- | ----------- |
| Отображаемое имя | Отображаемое имя команды, используемой в представлениях и формах. |
| Имя | Имя команды. IoT Central создает значение для этого поля на основе отображаемого имени, но при необходимости можно выбрать собственное значение. Это поле должно быть буквенно-цифровым. |
| Тип возможности | Команда |
| Комментировать | Любые комментарии о возможности "Команда". |
| Описание | Описание возможности "Команда". |
| Запрос | Если параметр включен, определение параметра запроса включает в себя: название, отображаемое имя, схему, единицу измерения и отображаемую единицу. |
| Ответ | Если параметр включен, определение ответа команды включает в себя: название, отображаемое имя, схему, единицу измерения и отображаемую единицу. |

Дополнительные сведения о том, как устройства реализуют команды, см. в разделе [данные телеметрии, свойства и команды > команды и длительные команды](concepts-telemetry-properties-commands.md#commands).

#### <a name="offline-commands"></a>Автономные команды

Вы можете выбрать команды очереди, если устройство находится в автономном режиме, включив параметр **очередь в автономном** режиме для команды в шаблоне устройства.

Этот параметр использует сообщения, отправляемые с облака на устройство центра Интернета вещей, для отправки уведомлений на устройства. Дополнительные сведения см. в статье центр Интернета вещей [Отправка сообщений из облака на устройство](../../iot-hub/iot-hub-devguide-messages-c2d.md).

Сообщения, отправляемые из облака на устройство:

- — Это односторонние уведомления для устройства из вашего решения.
- Гарантированная доставка сообщений по крайней мере. Центр Интернета вещей сохраняет сообщения, переявляющиеся из облака на устройство, в очереди каждого устройства, обеспечивая устойчивость к сбоям подключения и устройствам.
- Требовать, чтобы устройство реализовало обработчик сообщений для обработки сообщения, переданного из облака на устройство.

> [!NOTE]
> Этот параметр доступен только в веб-ИНТЕРФЕЙСе IoT Central. Этот параметр не включается, если вы экспортируете модель или компонент из шаблона устройства.

## <a name="manage-a-component"></a>Управление компонентом

Если компонент не опубликован, можно изменить возможности, определенные компонентом. Если вы хотите внести какие-либо изменения, после публикации компонента необходимо создать новую версию шаблона устройства и [версию компонента](howto-version-device-template.md). Изменения, для которых не требуется управление версиями, такие как отображаемые имена или единицы, можно выполнить в разделе **Настройка**.

Можно также экспортировать компонент в виде JSON-файла, если его нужно использовать в другой модели возможностей.

## <a name="add-cloud-properties"></a>Добавление свойств облака

Свойства облака можно использовать для хранения сведений об устройствах в IoT Central. Свойства облака никогда не отправляются на устройство. Например, свойства облака можно использовать для хранения имени клиента, установившего устройство, или даты последнего обслуживания устройства.

В следующей таблице показаны параметры конфигурации для свойства облака.

| Поле | Описание |
| ----- | ----------- |
| Отображаемое имя | Отображаемое имя для значения облачного свойства, используемого в представлениях и формах. |
| Имя | Имя свойства облака. IoT Central создает значение для этого поля на основе отображаемого имени, но при необходимости можно выбрать собственное значение. |
| Семантический тип | Семантический тип свойства, например температура, состояние или событие. Выбор семантического типа определяет, какие из следующих полей будут доступны. |
| схема | Тип данных свойства облака, например double, string или vector. Доступные варианты определяются семантическим типом. |

## <a name="add-customizations"></a>Добавление настройки

Настройки следует использовать, когда необходимо изменить импортированный компонент или добавить функции, относящиеся к IoT Central, к возможности. Можно настраивать только поля, которые не нарушают совместимость компонентов. Например, вы можете:

- Настроить отображаемое имя и единицы измерения для возможности.
- Добавить цвет по умолчанию, который используется для значений на диаграмме.
- Указать начальные, минимальные и максимальные значения для свойства.

Вы не сможете изменить имя или тип возможности. Если есть изменения, которые нельзя внести в раздел " **Настройка** ", вам потребуется версия шаблона устройства и компонента, чтобы изменить эту возможность.

### <a name="generate-default-views"></a>Создание представлений по умолчанию

Представления по умолчанию позволяют быстро визуализировать важную информацию об устройстве. Для шаблона устройства можно создать до трех представлений по умолчанию.

- **Команды**: представление с командами устройства и позволяет оператору отправлять их на устройство.
- **Обзор**: представление с телеметрией устройства, отображающее диаграммы и метрики.
- **О**: представление со сведениями об устройстве, отображающее свойства устройства.

После выбора **создать представления по умолчанию** вы увидите, что они были автоматически добавлены в раздел **представления** шаблона устройства.

## <a name="add-views"></a>Добавление представлений

Добавьте представления в шаблон устройства, чтобы разрешить операторам визуализировать устройство с помощью диаграмм и метрик. Можно использовать несколько представлений для шаблона устройства.

Чтобы добавить представление в шаблон устройства, выполните следующие действия.

1. Перейдите к шаблону устройства и выберите **представления**.
1. Щелкните **Визуализация устройства**.
1. Введите имя представления в списке **имя представления**.
1. Добавьте плитки к представлению из списка статических, свойств, свойств облака, телеметрии и плиток команд. Перетащите плитки, которые нужно добавить в представление.
1. Чтобы построить несколько значений телеметрии на одной плитке диаграммы, выберите значения телеметрии, а затем нажмите кнопку **объединить**.
1. Настройте каждую добавляемую плитку, указав для нее метод отображения данных. Для доступа к этому параметру выберите значок шестеренки или нажмите кнопку **изменить конфигурацию** на плитке диаграммы.
1. Упорядочивание и изменение размера плиток в представлении.
1. Сохраните изменения.

### <a name="configure-preview-device-to-view"></a>Настройка предварительной версии устройства для просмотра

Чтобы просмотреть и протестировать представление, выберите **настроить предварительный просмотр устройства**. Эта функция позволяет увидеть представление, когда оператор видит его после публикации. Эта функция предназначена для проверки отображения правильных данных в представлениях. Можно выбрать один из следующих вариантов.

- устройство без предварительной версии;
- реальное устройство для тестирования, настроенное для шаблона устройства;
- имеющееся устройство в приложении (с указанием идентификатора устройства).

## <a name="add-forms"></a>Добавление форм

Добавьте формы в шаблон устройства, чтобы операторы могли управлять устройством путем просмотра и настройки свойств. Операторы могут изменять только свойства облака и свойства записываемых устройств. Для шаблона устройства можно использовать несколько форм.

Чтобы добавить форму в шаблон устройства, сделайте следующее:

1. Перейдите к шаблону устройства и выберите **представления**.
1. Щелкните **Изменение устройства и облачных данных**.
1. Введите имя формы в поле **Имя формы**.
1. Выберите число столбцов, которое будет использоваться при создании макета формы.
1. Добавьте свойства в имеющийся раздел формы или выберите свойства и нажмите кнопку **Добавить раздел**. Используйте разделы для группировки свойств в форме. В раздел можно добавить заголовок.
1. Настройте каждое свойство в форме, чтобы изменить его поведение.
1. Расположите свойства в форме нужным образом.
1. Сохраните изменения.

## <a name="publish-a-device-template"></a>Публикация шаблона устройства

Перед подключением устройства, реализующего модель устройства, необходимо опубликовать шаблон устройства.

После публикации шаблона устройства можно вносить только ограниченные изменения в модель устройства. Чтобы изменить компонент, необходимо [создать и опубликовать новую версию](./howto-version-device-template.md).

Чтобы опубликовать шаблон устройства, перейдите к шаблону устройства и выберите **опубликовать**.

После публикации шаблона устройства оператор может перейти на страницу **Устройства** и добавить реальные или имитированные устройства, использующие шаблон устройства. Вы можете продолжать изменять и сохранять шаблон устройства по мере внесения изменений. Если вы хотите отправить эти изменения оператору для просмотра на странице **Устройства**, необходимо каждый раз выбирать **Опубликовать**.

## <a name="next-steps"></a>Дальнейшие действия

Если вы являетесь разработчиком устройства, рекомендуем следующий шаг — ознакомиться с [управлением версиями шаблонов устройств](./howto-version-device-template.md).
