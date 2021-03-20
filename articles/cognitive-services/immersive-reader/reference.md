---
title: Справочник по пакету SDK для иммерсивного чтения
titleSuffix: Azure Cognitive Services
description: Пакет SDK для иммерсивное средство чтения содержит библиотеку JavaScript, которая позволяет интегрировать иммерсивное средство чтения в приложение.
services: cognitive-services
author: metanMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: reference
ms.date: 06/20/2019
ms.author: metang
ms.openlocfilehash: f2f5c8193454a3b7fa6be1cea7a1236b613d6c8f
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92636533"
---
# <a name="immersive-reader-javascript-sdk-reference-v11"></a>Справочник по SDK для Java Reader (v 1.1)

Пакет SDK для иммерсивное средство чтения содержит библиотеку JavaScript, которая позволяет интегрировать иммерсивное средство чтения в приложение.

## <a name="functions"></a>Функции

Пакет SDK предоставляет функции:

- [`ImmersiveReader.launchAsync(token, subdomain, content, options)`](#launchasync)

- [`ImmersiveReader.close()`](#close)

- [`ImmersiveReader.renderButtons(options)`](#renderbuttons)

<br>

## <a name="launchasync"></a>лаунчасинк

Запускает иммерсивное средство чтения в `iframe` веб-приложении. Обратите внимание, что размер содержимого ограничен 50 МБ.

```typescript
launchAsync(token: string, subdomain: string, content: Content, options?: Options): Promise<LaunchResponse>;
```

#### <a name="launchasync-parameters"></a>Параметры Лаунчасинк

| Имя | Type | Описание |
| ---- | ---- |------------ |
| `token` | строка | Маркер проверки подлинности Azure AD. Дополнительные сведения см. [в статье Создание иммерсивного ресурса для чтения](./how-to-create-immersive-reader.md) . |
| `subdomain` | строка | Пользовательский поддомен для иммерсивного ресурса чтения в Azure. Дополнительные сведения см. [в статье Создание иммерсивного ресурса для чтения](./how-to-create-immersive-reader.md) . |
| `content` | [Содержимое](#content) | Объект, содержащий содержимое, которое должно отображаться в иммерсивное средство чтения. |
| `options` | [Параметры](#options) | Параметры для настройки определенного поведения иммерсивное средство чтения. Необязательный параметр. |

#### <a name="returns"></a>Возвращаемое значение

Возвращает объект `Promise<LaunchResponse>` , который разрешается при загрузке иммерсивное средство чтения. `Promise`Разрешается в [`LaunchResponse`](#launchresponse) объект.

#### <a name="exceptions"></a>Исключения

Возвращенный `Promise` объект будет отклонен с [`Error`](#error) объектом, если не удается загрузить иммерсивное средство чтения. Дополнительные сведения см. в [кодах ошибок](#error-codes).

<br>

## <a name="close"></a>close

Закрывает иммерсивное средство чтения.

Примером использования этой функции является случай, если кнопка выход скрыта с помощью параметра ```hideExitButton: true``` в [параметрах](#options). Затем с помощью другой кнопки (например, стрелки обратного заголовка мобильного устройства) можно вызвать эту ```close``` функцию при ее нажатии.

```typescript
close(): void;
```

<br>

## <a name="immersive-reader-launch-button"></a>Кнопка запуска иммерсивного модуля чтения

Пакет SDK предоставляет стиль по умолчанию для кнопки запуска иммерсивное средство чтения. `immersive-reader-button`Для включения этого стиля используйте атрибут класса. Дополнительные сведения см. в разделе [как настроить кнопку иммерсивного модуля чтения](./how-to-customize-launch-button.md) .

```html
<div class='immersive-reader-button'></div>
```

#### <a name="optional-attributes"></a>Необязательные атрибуты

Используйте следующие атрибуты, чтобы настроить внешний вид и поведение кнопки.

| Атрибут | Описание |
| --------- | ----------- |
| `data-button-style` | Задает стиль кнопки. Возможные значения: `icon`, `text` или `iconAndText`. По умолчанию — `icon`. |
| `data-locale` | Задает языковой стандарт. Например, `en-US` или `fr-FR`. По умолчанию используется английский язык `en` . |
| `data-icon-px-size` | Задает размер значка в пикселях. По умолчанию используется 20px. |

<br>

## <a name="renderbuttons"></a>рендербуттонс

```renderButtons```Функция не нужна, если вы используете руководство по [настройке кнопки иммерсивного модуля чтения](./how-to-customize-launch-button.md) .

Эта функция выменяет стили и обновляет элементы кнопки чтения в документе. Если указан ```options.elements``` , то кнопки будут подготавливаться к просмотру в каждом элементе, указанном в ```options.elements``` . Использование ```options.elements``` параметра полезно при наличии в документе нескольких разделов, на которых можно запустить иммерсивное средство чтения, и требуется упрощенный способ отображения нескольких кнопок с одинаковым стилем или отображения кнопок с простым и согласованным шаблоном разработки. Чтобы использовать эту функцию с параметром [рендербуттонс Options](#renderbuttons-options) , вызовите ```ImmersiveReader.renderButtons(options: RenderButtonsOptions);``` для загрузки страницы, как показано в приведенном ниже фрагменте кода. В противном случае кнопки будут отображены в элементах документа, имеющих класс ```immersive-reader-button``` , как показано в статье [как настроить кнопку иммерсивного модуля чтения](./how-to-customize-launch-button.md) .

```typescript
// This snippet assumes there are two empty div elements in
// the page HTML, button1 and button2.
const btn1: HTMLDivElement = document.getElementById('button1');
const btn2: HTMLDivElement = document.getElementById('button2');
const btns: HTMLDivElement[] = [btn1, btn2];
ImmersiveReader.renderButtons({elements: btns});
```

Дополнительные параметры отрисовки см. в описании дополнительных [атрибутов](#optional-attributes) . Чтобы использовать эти параметры, добавьте любой атрибут параметра в каждый ```HTMLDivElement``` из HTML-страниц страницы.

```typescript
renderButtons(options?: RenderButtonsOptions): void;
```

#### <a name="renderbuttons-parameters"></a>Параметры Рендербуттонс

| Имя | Type | Описание |
| ---- | ---- |------------ |
| `options` | [Параметры Рендербуттонс](#renderbuttons-options) | Параметры для настройки определенного поведения функции Рендербуттонс. Необязательный параметр. |

### <a name="renderbuttons-options"></a>Параметры Рендербуттонс

Параметры для отрисовки иммерсивного кнопок чтения.

```typescript
{
    elements: HTMLDivElement[];
}
```

#### <a name="renderbuttons-options-parameters"></a>Параметры параметров Рендербуттонс

| Параметр | Type | Описание |
| ------- | ---- | ----------- |
| элементы | Хтмлдивелемент [] | Элементы для отрисовки впечатляющих кнопок чтения в. |

##### `elements`
```Parameters
Type: HTMLDivElement[]
Required: false
```

<br>

## <a name="launchresponse"></a>лаунчреспонсе

Содержит ответ от вызова `ImmersiveReader.launchAsync` . Обратите внимание, что доступ к ссылке `iframe` , содержащей иммерсивное средство чтения, можно получить с помощью `container.firstChild` .

```typescript
{
    container: HTMLDivElement;
    sessionId: string;
}
```

#### <a name="launchresponse-parameters"></a>Параметры Лаунчреспонсе

| Параметр | Type | Описание |
| ------- | ---- | ----------- |
| контейнер | хтмлдивелемент | Элемент HTML, содержащий иммерсивное устройство чтения IFRAME. |
| sessionID | Строка | Глобальный уникальный идентификатор для этого сеанса, используемый для отладки. |
 
## <a name="error"></a>Ошибка

Содержит сведения об ошибке.

```typescript
{
    code: string;
    message: string;
}
```

#### <a name="error-parameters"></a>Параметры ошибок

| Параметр | Type | Описание |
| ------- | ---- | ----------- |
| code | Строка | Один из набора кодов ошибок. См. статью о [кодах ошибок](#error-codes). |
| message | Строка | Понятное представление ошибки. |

#### <a name="error-codes"></a>Коды ошибок

| Код | Описание |
| ---- | ----------- |
| BadArgument | Указан недопустимый аргумент, см `message` . параметр [ошибки](#error). |
| Время ожидания | Не удалось загрузить иммерсивное средство чтения в течение указанного времени ожидания. |
| TokenExpired | Срок действия заданного маркера истек. |
| Ожидает повтора | Превышено ограничение скорости вызовов. |

<br>

## <a name="types"></a>Типы

### <a name="content"></a>Содержимое

Содержит содержимое, отображаемое в иммерсивное средство чтения.

```typescript
{
    title?: string;
    chunks: Chunk[];
}
```

#### <a name="content-parameters"></a>Параметры содержимого

| Имя | Type | Описание |
| ---- | ---- |------------ |
| title | Строка | Текст заголовка, отображаемый в верхней части иммерсивное средство чтения (необязательно) |
| блоки | [Фрагмент []](#chunk) | Массив фрагментов |

##### `title`
```Parameters
Type: String
Required: false
Default value: "Immersive Reader" 
```

##### `chunks`
```Parameters
Type: Chunk[]
Required: true
Default value: null 
```

<br>

### <a name="chunk"></a>Chunk

Один блок данных, который будет передан в содержимое иммерсивное средство чтения.

```typescript
{
    content: string;
    lang?: string;
    mimeType?: string;
}
```

#### <a name="chunk-parameters"></a>Параметры блока

| Имя | Type | Описание |
| ---- | ---- |------------ |
| содержимое | Строка | Строка, содержащая содержимое, отправленное в иммерсивное средство чтения. |
| lang | Строка | Язык текста, значение находится в формате языкового тега IETF BCP 47, например en, ES-ES. Язык будет автоматически обнаружен, если он не указан. См. сведения о [поддерживаемых языках](#supported-languages). |
| mimeType | строка | Поддерживаются обычный текст, Масмл, HTML & форматов Microsoft Word DOCX. Дополнительные сведения см. в разделе [Поддерживаемые типы MIME](#supported-mime-types) . |

##### `content`
```Parameters
Type: String
Required: true
Default value: null 
```

##### `lang`
```Parameters
Type: String
Required: false
Default value: Automatically detected 
```

##### `mimeType`
```Parameters
Type: String
Required: false
Default value: "text/plain"
```

#### <a name="supported-mime-types"></a>Поддерживаемые типы MIME

| Тип MIME | Описание |
| --------- | ----------- |
| text/plain | Обычный текст. |
| text/html | Содержимое в виде HTML. [Дополнительные сведения](#html-support)|
| Application/масмл + XML | Язык математической разметки (Масмл). [Подробнее](./how-to/display-math.md).
| приложение/vnd.openxmlformats-officedocument.wordprocessingml.docумент | Документ в формате Microsoft Word. docx.


<br>

## <a name="options"></a>Параметры

Содержит свойства, которые настраивают определенное поведение иммерсивное средство чтения.

```typescript
{
    uiLang?: string;
    timeout?: number;
    uiZIndex?: number;
    useWebview?: boolean;
    onExit?: () => any;
    allowFullscreen?: boolean;
    hideExitButton?: boolean;
    cookiePolicy?: CookiePolicy;
    disableFirstRun?: boolean;
    readAloudOptions?: ReadAloudOptions;
    translationOptions?: TranslationOptions;
    displayOptions?: DisplayOptions;
    preferences?: string;
    onPreferencesChanged?: (value: string) => any;
    customDomain?: string;
}
```

#### <a name="options-parameters"></a>Параметры параметров

| Имя | Type | Описание |
| ---- | ---- |------------ |
| уиланг | Строка | Язык пользовательского интерфейса, значение находится в формате языкового тега IETF BCP 47, например en, ES-ES. По умолчанию используется язык браузера, если он не указан. |
| timeout | Число | Длительность (в миллисекундах) до сбоя [лаунчасинк](#launchasync) с ошибкой времени ожидания (значение по умолчанию — 15000 МС). Это время ожидания применяется только к начальному запуску страницы читатель, где успешно отображается при открытии страницы читатель и запуске счетчика. Коррекция времени ожидания не требуется. |
| уизиндекс | Число | Z-индекс создаваемого IFRAME (по умолчанию — 1000). |
| усевебвиев | Логическое| Используйте тег WebView вместо iframe для совместимости с приложениями Chrome (значение по умолчанию — false). |
| onExit | Компонент | Выполняется при выходе из иммерсивного модуля чтения. |
| алловфуллскрин | Логическое | Возможность полноэкранного режима (значение по умолчанию — true). |
| хидикситбуттон | Логическое | Следует ли скрывать стрелку кнопки выхода в иммерсивное средство чтения (значение по умолчанию — false). Это должно быть справедливо, только если существует альтернативный механизм для выхода из иммерсивного средства чтения (например, стрелка обратной панели инструментов для мобильных устройств). |
| кукиеполици | [кукиеполици](#cookiepolicy-options) | Параметр для использования файлов cookie иммерсивного модуля чтения (по умолчанию — *кукиеполици. Disable*). Ведущее приложение отвечает за получение любого необходимого согласия пользователя в соответствии с политикой соответствия файлов cookie ЕС. См. раздел [Параметры политики для файлов cookie](#cookiepolicy-options). |
| дисаблефирструн | Логическое | Отключите первый интерфейс запуска. |
| реадалаудоптионс | [реадалаудоптионс](#readaloudoptions) | Параметры для настройки чтения вслух. |
| транслатионоптионс | [транслатионоптионс](#translationoptions) | Параметры для настройки перевода. |
| displayOptions | [DisplayOptions](#displayoptions) | Параметры для настройки размера текста, шрифта и т. д. |
| чувствительн | Строка | Строка, возвращаемая из Онпреференцесчанжед, представляющая предпочтения пользователя в иммерсивное средство чтения. Дополнительные [сведения см.](#settings-parameters) в статьях параметры и [инструкции по хранению пользовательских настроек](./how-to-store-user-preferences.md) . |
| онпреференцесчанжед | Компонент | Выполняется при изменении параметров пользователя. Дополнительные сведения см. [в статье как сохранить настройки пользователя](./how-to-store-user-preferences.md) . |
| customDomain | Строка | Зарезервировано для внутреннего использования. Личный домен, в котором размещается иммерсивное средство чтения webapp (по умолчанию — NULL). |

##### `uiLang`
```Parameters
Type: String
Required: false
Default value: User's browser language 
```

##### `timeout`
```Parameters
Type: Number
Required: false
Default value: 15000
```

##### `uiZIndex`
```Parameters
Type: Number
Required: false
Default value: 1000
```

##### `onExit`
```Parameters
Type: Function
Required: false
Default value: null
```

##### `preferences`

> [!CAUTION]
> **Важно!** Не пытайтесь программно изменить значения `-preferences` строки, отправленной в иммерсивное приложение чтения и из него, так как это может привести к непредвиденному поведению, что приводит к снижению производительности пользователей. Ведущие приложения никогда не должны назначать строку или управлять ей пользовательским значением `-preferences` . При использовании `-preferences` параметра String используйте только точное значение, возвращенное из `-onPreferencesChanged` параметра обратного вызова.

```Parameters
Type: String
Required: false
Default value: null
```

##### `onPreferencesChanged`
```Parameters
Type: Function
Required: false
Default value: null
```

##### `customDomain`
```Parameters
Type: String
Required: false
Default value: null
```

<br>

## <a name="readaloudoptions"></a>реадалаудоптионс

```typescript
type ReadAloudOptions = {
    voice?: string;
    speed?: number;
    autoplay?: boolean;
};
```

#### <a name="readaloudoptions-parameters"></a>Параметры Реадалаудоптионс

| Имя | Type | Описание |
| ---- | ---- |------------ |
| voice | Строка | Voice: "женщина" или "папа". Обратите внимание, что не все языки поддерживают оба пола. |
| Скорость | Число | Скорость воспроизведения должна составлять от 0,5 до 2,5 включительно. |
| Автозапуска | Логическое | Автоматически начать чтение вслух при загрузке иммерсивного модуля чтения. |

##### `voice`
```Parameters
Type: String
Required: false
Default value: "Female" or "Male" (determined by language) 
Values available: "Female", "Male"
```

##### `speed`
```Parameters
Type: Number
Required: false
Default value: 1
Values available: 0.5, 0.75, 1, 1.25, 1.5, 1.75, 2, 2.25, 2.5
```

> [!NOTE]
> В связи с ограничениями браузера автозапуск не поддерживается в Safari.

<br>

## <a name="translationoptions"></a>транслатионоптионс

```typescript
type TranslationOptions = {
    language: string;
    autoEnableDocumentTranslation?: boolean;
    autoEnableWordTranslation?: boolean;
};
```

#### <a name="translationoptions-parameters"></a>Параметры Транслатионоптионс

| Имя | Type | Описание |
| ---- | ---- |------------ |
| Язык | Строка | Задает язык перевода, значение находится в формате языкового тега IETF BCP 47, например fr-FR, es-MX, zh-Ханс-CN. Требуется для автоматического включения перевода слов или документов. |
| аутоенабледокументтранслатион | Логическое | Автоматическое преобразование всего документа. |
| аутоенаблевордтранслатион | Логическое | Автоматическое включение перевода слов. |

##### `language`
```Parameters
Type: String
Required: true
Default value: null 
Values available: See the Supported Languages section
```

<br>

## <a name="displayoptions"></a>DisplayOptions

```typescript
type DisplayOptions = {
    textSize?: number;
    increaseSpacing?: boolean;
    fontFamily?: string;
};
```

#### <a name="displayoptions-parameters"></a>Параметры DisplayOptions

| Имя | Type | Описание |
| ---- | ---- |------------ |
| textSize | Число | Задает размер выбранного текста. |
| инкреасеспаЦинг | Логическое | Задает, включается ли или выключается текстовый зазор. |
| fontFamily | Строка | Задает выбранный шрифт ("Calibri", "Комиксанс" или "Ситка"). |

##### `textSize`
```Parameters
Type: Number
Required: false
Default value: 20, 36 or 42 (Determined by screen size)
Values available: 14, 20, 28, 36, 42, 48, 56, 64, 72, 84, 96
```

##### `fontFamily`
```Parameters
Type: String
Required: false
Default value: "Calibri"
Values available: "Calibri", "Sitka", "ComicSans"
```

<br>

## <a name="cookiepolicy-options"></a>Параметры Кукиеполици

```typescript
enum CookiePolicy { Disable, Enable }
```

**Указанные ниже параметры предназначены только для информационных целей**. В "cookie" в иммерсивное средство чтения сохраняет свои параметры или предпочтения пользователя. Этот  параметр кукиеполици **отключает** использование файлов cookie по умолчанию, чтобы соответствовать законодательству соответствия файлов cookie в ЕС. Если вы хотите повторно включить файлы cookie и восстановить стандартные функции для пользователей в иммерсивное средство чтения, необходимо убедиться, что веб-сайт или приложение получат соответствующее согласие от пользователя, чтобы включить файлы cookie. Затем, чтобы повторно включить файлы cookie в иммерсивное средство чтения, необходимо явно задать для параметра *кукиеполици* значение *кукиеполици. Enable* при запуске иммерсивное средство чтения. В таблице ниже описано, какие параметры сохраняют иммерсивное средство чтения в своем файле cookie при включенном параметре *кукиеполици* .

#### <a name="settings-parameters"></a>Параметры параметров

| Параметр | Type | Описание |
| ------- | ---- | ----------- |
| textSize | Число | Задает размер выбранного текста. |
| fontFamily | Строка | Задает выбранный шрифт ("Calibri", "Комиксанс" или "Ситка"). |
| текстспаЦинг | Число | Задает, включается ли или выключается текстовый зазор. |
| formattingEnabled | Логическое | Задает включение или отключение форматирования HTML. |
| тема | Строка | Задает выбранную тему (например, "светлая", "темная"...). |
| силлабификатионенаблед | Логическое | Задает, включен ли параметр силлабификатион. |
| наунхигхлигхтинженаблед | Логическое | , который задает включение или выключение выделения существительное. |
| наунхигхлигхтингколор | Строка | Задает выбранный цвет для выделения существительных. |
| вербхигхлигхтинженаблед | Логическое | Задает включение или выключение выделения глагола. |
| вербхигхлигхтингколор | Строка | Задает цвет выделения для выбранного глагола. |
| аджективехигхлигхтинженаблед | Логическое | Задает включение или выключение выделения прилагательного. |
| аджективехигхлигхтингколор | Строка | Задает выбранный цвет выделения прилагательных. |
| адвербхигхлигхтинженаблед | Логическое | Задает включение или выключение выделения модификаторов. |
| адвербхигхлигхтингколор | Строка | Задает выбранный цвет выделения модификаторов. |
| пиктуредиктионаренаблед | Логическое | Задает включение или выключение словаря рисунков. |
| послабелсенаблед | Логическое | Задает, включается ли или выключается подпись нижнего знака для каждой выделенной части речи.  |

<br>

## <a name="supported-languages"></a>Поддерживаемые языки

Функция перевода иммерсивное средство чтения поддерживает множество языков. Дополнительные сведения см. в разделе [Поддержка языков](./language-support.md) .

<br>

## <a name="html-support"></a>Поддержка HTML

Если форматирование включено, следующее содержимое будет подготовлено к просмотру в виде HTML в иммерсивное средство чтения.

| HTML | Поддерживаемое содержимое |
| --------- | ----------- |
| Стили шрифтов | Полужирный, курсив, подчеркнутый, код, Зачеркнутый, надстрочный, нижний индекс |
| Неупорядоченные списки | Диск, круг, квадрат |
| Упорядоченные списки | Десятичная, верхняя-альфа, Нижняя-альфа, верхняя-латиница, Lower-Roman |

Неподдерживаемые теги будут подготовлены к просмотру сравнимо. Изображения и таблицы в настоящее время не поддерживаются.

<br>

## <a name="browser-support"></a>Поддержка браузеров

Используйте последние версии следующих браузеров, чтобы получить лучшие впечатления от работы с иммерсивное средство чтения.

* Microsoft Edge
* Internet Explorer 11;
* Google Chrome
* Mozilla Firefox;
* Apple Safari.

<br>

## <a name="next-steps"></a>Дальнейшие действия

* Изучите [пакет SDK иммерсивного средства чтения на сайте GitHub](https://github.com/microsoft/immersive-reader-sdk).
* [Краткое руководство. Создание веб-приложения, запускающего иммерсивное средство чтения (C#)](./quickstarts/client-libraries.md?pivots=programming-language-csharp)
