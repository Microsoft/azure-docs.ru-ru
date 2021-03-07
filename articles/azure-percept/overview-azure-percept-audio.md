---
title: Общие сведения о звуковом устройстве Azure Перцепт
description: Дополнительные сведения об Azure Перцепт Audio
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: conceptual
ms.date: 02/18/2021
ms.custom: template-concept
ms.openlocfilehash: 85eb4ed0832c9384fcd05154833bc21f83f4adf2
ms.sourcegitcommit: ba676927b1a8acd7c30708144e201f63ce89021d
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2021
ms.locfileid: "102426354"
---
# <a name="introduction-to-azure-percept-audio"></a>Введение в службу Перцепт Azure Audio

Azure Перцепт Audio — это устройство, которое добавляет возможности преобразования речи в речь в Azure Перцепт DK. В нем содержится предварительно настроенный аудио-процессор и четыре линейных микрофона, позволяющие применять голосовые команды, распознавание ключевых слов и проходящие речевые функции на локальных устройствах прослушивания с помощью Azure Cognitive Services. Azure Перцепт Audio позволяет производителям устройств расширять возможности Azure Перцепт ТЕМНее, помимо возможностей для новых, активированных с помощью смарт-устройства устройств. Она интегрирована с Azure Перцепт DK, Azure Перцепт Studio и другими службами управления Azure ребра. Его можно приобрести в [Microsoft Online Store](https://go.microsoft.com/fwlink/p/?LinkId=2155270).

:::image type="content" source="./media/overview-azure-percept-audio/percept-audio.png" alt-text="Аудиоустройство Azure Перцепт.":::

## <a name="azure-percept-audio-components"></a>Компоненты звука Azure Перцепт

Служба Перцепт Audio Azure содержит следующие основные компоненты:

- Готовое к работе аудиоустройство Azure Перцепт Audio Device (SoM) с четырьмя микрофонами линейным массивом и аудио-обработкой, осуществляемой кодеком КСМОС
- Плата для разработчиков (выработка) (включает две кнопки, светоиндикаторы 3 раза, Micro USB и 3,5 мм)
- Необходимые кабели: кабель ФПК, USB Micro Type-B to USB-A
- Приветственная карта
- Механическая монтирование пластин с интегрированным подключением серии 80/20 1010

## <a name="compute-capabilities"></a>Возможности вычислений 

Служба "звук Azure Перцепт" передает звуковые входные данные через стек речи, который работает на ЦП платы перевозчика в Azure Перцепт DK в гибридной облачной службе. Таким образом, для выполнения в службе аудио Azure Перцепт требуется плата с ОС, поддерживающая стек распознавания речи. 

Обработка выполняется следующим образом. 

- Azure Перцепт Audio: захватывает и преобразует аудио, а затем отправляет их в разъемы DK и Audio.

- Azure Перцепт DK. в стеке распознавания речи выполняется многообразовая обработка и отработка отказа, которая обрабатывает входящий звук для оптимизации речи. Затем он выполняет поиск по ключевому слову.

- Cloud: обрабатывает команды и фразы естественного языка, проверку ключевых слов и переобучение. 

- Вне сети. Если устройство находится в автономном режиме, оно обнаружит ключевое слово и записывает данные телеметрии состояния подключения к Интернету. Увеличенная частота принятия для ключевых слов может наблюдаться, так как проверка ключевых слов в облаке не может быть выполнена. 

## <a name="getting-started"></a>Начало работы

- [Сборка Azure Перцепт DK](./quickstart-percept-dk-unboxing.md)
- [Завершение процесса установки Azure Перцепт DK](./quickstart-percept-dk-set-up.md)
- [Подключение устройства Azure Перцепт Audio к DevKit](./quickstart-percept-audio-setup.md)

## <a name="build-a-no-code-prototype"></a>Создание прототипа без кода

Создавайте [речевое решение без кода](./tutorial-no-code-speech.md) в [Azure перцепт Studio](https://go.microsoft.com/fwlink/?linkid=2135819) с помощью шаблонов голосового помощника перцепт для бизнес-целей, здравоохранения, инвентаризации и автомобильных сценариев.

### <a name="manage-your-no-code-speech-solution"></a>Управляйте своим речевым решением без кода

- [Настройка речевого помощника в центре Интернета вещей](./how-to-manage-voice-assistant.md)
- [Настройка голосового помощника в Azure Percept Studio](./how-to-configure-voice-assistant.md)
- [Устранение неполадок c Azure Percept Audio](./troubleshoot-audio-accessory-speech-module.md)

## <a name="additional-technical-information"></a>Дополнительные технические сведения

- [Спецификации Azure Percept Audio](./azure-percept-audio-datasheet.md)
- [Поведение кнопки и индикатора](./audio-button-led-behavior.md)

## <a name="next-steps"></a>Дальнейшие действия

Закажите аудиоустройство Azure Перцепт в [Microsoft Online Store](https://go.microsoft.com/fwlink/p/?LinkId=2155270).
