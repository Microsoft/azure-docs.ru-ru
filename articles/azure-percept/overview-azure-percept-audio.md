---
title: Общие сведения о звуковом устройстве Azure Перцепт
description: Дополнительные сведения об Azure Перцепт Audio
author: elqu20
ms.author: v-elqu
ms.service: azure-percept
ms.topic: conceptual
ms.date: 03/23/2021
ms.custom: template-concept
ms.openlocfilehash: 7c80159607947dbe7ed17181beac610fa0c202c5
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104956100"
---
# <a name="introduction-to-azure-percept-audio"></a>Введение в службу Перцепт Azure Audio

Azure Перцепт Audio — это устройство, которое добавляет возможности преобразования речи в речь в [Azure ПЕРЦЕПТ DK](./overview-azure-percept-dk.md). В нем содержится предварительно настроенный аудио-процессор и четыре линейных микрофона, позволяющие использовать голосовые команды, функции распознавания ключевых слов и в работе с Azure Cognitive Services. Она интегрирована с Azure Перцепт DK, [Azure Перцепт Studio](https://go.microsoft.com/fwlink/?linkid=2135819)и другими службами управления Azure ребра. Службу Azure Перцепт Audio можно приобрести в [Microsoft Online Store](https://go.microsoft.com/fwlink/p/?LinkId=2155270).

> [!div class="nextstepaction"]
> [Купить](https://go.microsoft.com/fwlink/p/?LinkId=2155270)

</br>

> [!VIDEO https://www.youtube.com/embed/Qj8NGn-7s5A]

## <a name="azure-percept-audio-components"></a>Компоненты звука Azure Перцепт

Служба Перцепт Audio Azure содержит следующие основные компоненты:

- Готовое к работе аудиоустройство Azure Перцепт Audio (SoM) с линейным массивом с четырьмя микрофоном и аудио-обработкой через кодек КСМОС
- Доска для разработчиков (на задней панели): две кнопки, индикаторы 3 раза, микропорт Micro-USB и звуковой разъем 3,5 mm
- Необходимые кабели: кабель ФПК, USB Micro Type-B to USB-A
- Приветственная карта
- Механическая монтирование пластин с интегрированным подключением серии 80/20 1010

## <a name="compute-capabilities"></a>Возможности вычислений 

Azure Перцепт Audio передает звуковые входные данные через стек распознавания речи, который работает на центральном процессоре платы перевозчика Azure Перцепт DK в гибридной облачной службе. Таким образом, для выполнения в службе аудио Azure Перцепт требуется плата с ОС, поддерживающая стек распознавания речи. 

Звуковая обработка выполняется следующим образом: 

- Azure Перцепт Audio: захватывает и преобразует аудио, а затем отправляет их в разъемы DK и Audio.

- Azure Перцепт DK. в стеке распознавания речи выполняется многообразовая обработка и отработка отказа, которая обрабатывает входящий звук для оптимизации речи. После обработки он выполняет поиск по ключевым словам.

- Cloud: обрабатывает команды и фразы естественного языка, проверку ключевых слов и переобучение. 

- Вне сети. Если устройство находится в автономном режиме, оно обнаружит ключевое слово и записывает данные телеметрии состояния подключения к Интернету. Увеличенная частота принятия для ключевых слов может наблюдаться, так как проверка ключевых слов в облаке не может быть выполнена. 

## <a name="getting-started"></a>Начало работы

- [Сборка Azure Перцепт DK](./quickstart-percept-dk-unboxing.md)
- [Подключение устройства Azure Перцепт Audio к DevKit](./quickstart-percept-audio-setup.md)
- [Завершение процесса установки Azure Перцепт DK](./quickstart-percept-dk-set-up.md)

## <a name="build-a-no-code-prototype"></a>Создание прототипа без кода

Создавайте [речевое решение без кода](./tutorial-no-code-speech.md) в [Azure перцепт Studio](https://go.microsoft.com/fwlink/?linkid=2135819) с помощью шаблонов голосового помощника перцепт для бизнес-целей, здравоохранения, инвентаризации и автомобильных сценариев.

### <a name="manage-your-no-code-speech-solution"></a>Управляйте своим речевым решением без кода

- [Настройка голосового помощника в Azure Percept Studio](./how-to-manage-voice-assistant.md)
- [Настройка речевого помощника в центре Интернета вещей](./how-to-configure-voice-assistant.md)
- [Устранение неполадок c Azure Percept Audio](./troubleshoot-audio-accessory-speech-module.md)

## <a name="additional-technical-information"></a>Дополнительные технические сведения

- [Спецификации Azure Percept Audio](./azure-percept-audio-datasheet.md)
- [Поведение кнопки и индикатора](./audio-button-led-behavior.md)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Купить аудиоустройство Azure Перцепт в Microsoft Online Store](https://go.microsoft.com/fwlink/p/?LinkId=2155270)
