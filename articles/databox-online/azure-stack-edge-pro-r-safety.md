---
title: Azure Stack пограничных руководств по безопасности R для Pro | Документация Майкрософт
description: Описывает соглашения о безопасности, рекомендации, рекомендации и объясняет, как безопасно устанавливать и использовать устройство R для Azure Stack пограничной Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 02/12/2021
ms.author: alkohli
ms.openlocfilehash: dacc9ecc28ffa482b60d1e48735fe3620b5b7558
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100363072"
---
# <a name="azure-stack-edge-pro-r-safety-instructions"></a>Инструкции по безопасности для Azure Stack пограничных Pro R

![Значок "Внимание!"](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
![Значок "Рекомендации по безопасности"](./media/azure-stack-edge-pro-r-safety/icon-safety-read-all-instructions.png)
**ОЗНАКОМЬТЕСЬ С ИНФОРМАЦИЕЙ О МЕРАХ ПРЕДОСТОРОЖНОСТИ**

Ознакомьтесь со всеми сведениями о безопасности, приведенными в этой статье, прежде чем использовать устройство R для Azure Stack ребра. Несоблюдение инструкций может привести к пожару, электрическим ударам, травмаху или повреждению свойств. Ознакомьтесь со всеми сведениями о безопасности, приведенными ниже, прежде чем использовать Azure Stack пограничных Pro R.

## <a name="safety-icon-conventions"></a>Условные обозначения сведений о безопасности

Ниже перечислены ключевые слова для подписывания предупреждений опасности.

| Значок | Описание |
|:--- |:--- |
| ![Символ опасности](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)| **Опасность:** Обозначает опасную ситуацию, которая, в противном случае, приведет к смерти или серьезной травме. <br> **Предупреждение:** Указывает на опасную ситуацию, которая может привести к смерти или серьезным травмам. <br> **Внимание!** Указывает на опасную ситуацию, которая может привести к незначительному или умеренному травме.|
|

При настройке и запуске Azure Stack пограничных устройств R с пограничными устройствами можно наблюдать за следующими значками опасности:

| Значок | Описание |
|:--- |:--- |
| ![Сначала прочесть все инструкции](./media/azure-stack-edge-pro-r-safety/icon-safety-read-all-instructions.png) | Сначала прочесть все инструкции |
| ![Значок "Примечание"](./media/azure-stack-edge-mini-r-safety/icon-safety-notice.png) **ПРИМЕЧАНИЕ.** | Указывает на важные сведения, не связанные с угрозой здоровью и жизни человека. || ![Символ опасности](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) | Символ опасности |
| ![Значок предупреждения об опасности](./media/azure-stack-edge-pro-r-safety/icon-safety-tip-hazard.png)  | Опасность опрокидывания|
| ![Значок «Большой вес»](./media/azure-stack-edge-pro-r-safety/icon-safety-heavy-weight.png)  | Большая масса опасности|
| ![Значок «Электрошок»](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) | Опасность поражения электрическим током |
| ![Значок «Нет компонентов, подлежащих самостоятельному ремонту»](./media/azure-stack-edge-pro-r-safety/icon-safety-do-not-access.png) | Нет элементов, подобслуживаемых пользователем. Не проводите техническое обслуживание устройства без должной подготовки. |
| ![Значок нескольких источников питания](./media/azure-stack-edge-pro-r-safety/icon-safety-disconnect-all-power.png)  | Несколько источников питания. Отключите все кабели питания, чтобы полностью отключить питание оборудования. |
| ![Значок точек сжатия](./media/azure-stack-edge-pro-r-safety/icon-pinching-points.png)  | Имеются точки сжатия. |
| ![Значок "горячие компоненты" или "поверхности"](./media/azure-stack-edge-pro-r-safety/icon-hot-component-surface.png)  | Указывает на горячие компоненты или поверхности. |
|

## <a name="handling-precautions-and-site-selection"></a>Обработка мер предосторожности и выбора сайта

![Значок предупреждения](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) **ВНИМАНИЕ:**

* Например, при перемещении и обработке устройства с поставкой необходимо использовать соответствующее оборудование (например, гнездо палеты) и персональное защитное оборудование (PPE).

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)![ значок опасности ](./media/azure-stack-edge-pro-r-safety/icon-safety-tip-hazard.png) **. Предупреждение:**

* Разместите оборудование на плоской, жесткой и стабильной поверхности, чтобы избежать возможного Совета или крушинг опасности.
* Не следует помещает более двух устройств в стек.
* Перед наложением убедитесь, что система защищена.
* Не перемещаете и не перемещая устройства с накоплением.
* Не добавляйте в стек устройства, которые подключены к внешним кабелям.
* Убедитесь, что шнуры питания не крушед или не повреждены во время операций размещения.
* Устройства не предназначены для использования в качестве таблиц и рабочих областей.

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
 ![ : значок электрической нагрузки ](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png)
 ![ нет значок "обслуживаемые компоненты ](./media/azure-stack-edge-pro-r-safety/icon-safety-do-not-access.png) **" внимание!**

* Проверьте состояние устройства *в момент получения*. Если корпус устройства поврежден, обратитесь в [Служба поддержки Майкрософт](azure-stack-edge-contact-microsoft-support.md) , чтобы получить замену. Не пытайтесь работать с поврежденным устройством.
* Если вы подозреваете, что устройство работает неправильно, [обратитесь в Служба поддержки Майкрософт](azure-stack-edge-contact-microsoft-support.md) , чтобы получить замену. Не пытайтесь проводить техническое обслуживание самостоятельно.
* Устройство содержит компоненты, которые не подлежат обслуживанию пользователем. Внутри корпуса присутствует опасный уровень напряжения, тока и электроэнергии. Не открывайте корпус. Верните устройство в Майкрософт для проведения обслуживания.

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) ![ . значок "жирный вес ](./media/azure-stack-edge-pro-r-safety/icon-safety-heavy-weight.png) **" предупреждение:**

* Удаление модуля питания ИБП приводит к включению отключенных частей в ИБП. Не вставляйте внешние объекты в секции модуля питания.
* Не пытайтесь приподнять устройство Azure Stack пограничной Pro R. Корпус может поднимать от 52 кг до 93 кг (115 фунта и 205 фунта); Используйте механическую помощь или другую подходящую помощь при перемещении и поднятии оборудования. Соблюдение требований к местным требованиям к работоспособности и безопасности при перемещении и поднятии оборудования.
* Не пытайтесь приподнять оборудования без соответствующей механической помощи. Имейте в виду, что попытка поднять этот вес может вызвать серьезные травмах.

![Значок предупреждения](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)  **ВНИМАНИЕ:**

* Система разработана для использования в управляемой среде. Выберите сайт, который:
  * Вентилатед и вдали от источников тепла, включая прямые солнечные и аппаратные радиочастоту.
  * Не предоставляется в влажность или дождя.
  * Размещается в пространстве, в котором уменьшается вибрация и снижается физическая нагрузка.  Система разработана для ударов и вибрации в соответствии с MIL-STD-810G.
  * Изолировано от сильных электромагнитных полей, созданных электрическими устройствами.
  * Предоставляется с надлежащими заземленными розетками.
  * Предоставляется с достаточным объемом для доступа к шнурам питания, так как он выступает в качестве основного отключения питания продукта.
* Кабели Ethernet не предоставляются вместе с продуктом. Чтобы уменьшить электромагнитные помехи, рекомендуется использовать кабель экранированной витой пары (STP 6).
* Настройте оборудование в рабочей области, чтобы обеспечить адекватное использование воздуха по всему оборудованию. Убедитесь, что передняя и задняя крышки полностью удалены во время работы устройства.
* Кабели Ethernet не предоставляются вместе с продуктом. Чтобы уменьшить электромагнитные помехи, рекомендуется использовать кабель с экранированием (STP) CAT 6.
* Устанавливайте оборудование в области с контролем температуры на основе контаминантс и допускайте адекватное беспроводное электроциркуляцию по всему оборудованию.
* Не отключайте оборудование от источников жидкостей и чрезмерно Хумид сред.
* Не разрешайте системе никаких ликвидных или ни одного внешнего объекта. Не помещайте напитки или другие контейнеры жидкостей в систему или рядом с ней.

## <a name="heater-precautions"></a>Хеатер меры предосторожности

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) ![ горячие компоненты или поверхности значок 1 ](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) **Предупреждение:** 

* Автоматическая хеатерная операция во время включения системы может создать сенсорную угрозу из-за высоких температур на обложке сборки хеатер. Не затрагивайте эту поверхность, пока система включена. Подождите 10-минутный период охлаждения после выключения системы.

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) ![ . значок "точки сжатия" Icon 1 ](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) **Предупреждение:** 

* При включении системы автоматическое срабатывание задней дверцы Пленум может привести к созданию опасности для точки сжатия. Не изменяйте эту область при включении системы.

## <a name="electrical-precautions"></a>Меры предосторожности в отношении электрооборудования

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) ![ значок электроудара ](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) **. Предупреждение:**

* Обеспечьте безопасное соединение с землей для кабеля электропитания. Кабель с чередованием (AC) имеет три провода заземления (вилка с заземлением). Эта вилка подходит только к заземленной розетке переменного тока. Не пренебрегайте заземляющим контактом.
* Так как штекер кабеля электропитания является основным средством отключения, убедитесь, что розетки расположены вблизи устройства и в удобном для вас месте.
* Отключите шнуры питания (выполнив подключение через разъем, а не шнур) и Разъедините все кабели, если выполняются следующие условия.

  * Кабель питания и вилка потерты или повреждены любым другим образом.
  * На кабельную обмотку попала жидкость.
  * Устройство оказалось под воздействием дождя или повышенной влажности.
  * Устройство упало, и корпус поврежден.
  * По вашему мнению, устройство требует обслуживания или ремонта.
* Перед перемещением устройства, или если вам кажется, что оно неисправно, полностью отключите его от электросети.
* Чтобы предотвратить высокий уровень утечки, если один транзитный случай имеет более одного непрерывного источника питания (ИБП), рекомендуется, чтобы каждый ИБП был подключен к независимому каналу филиала. Однако в случае, если используется единица распределения питания или другое устройство, где безопасность каждого ИБП полагается на один датчик заземления PDU, терминал заземления на наружной стороне каждого ИБП также должен использоваться с дополнительным теплопроводником.

  > [!NOTE]
  > Если используется блок распределения, который уже имеет дополнительный раскладку заземления, использование дополнительного терминала заземления на ИБП не требуется.

* Используйте подходящий источник питания с защитой от электрической перегрузки, чтобы обеспечить соответствие следующим требованиям:

  * Напряжение: от 100 до 240 вольт AC
  * Текущий: 20 а, максимум на шнур питания. Предоставляются кабели питания.
  * Частота: от 50 до 60 Гц

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
 ![ : значок электрической нагрузки ](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png)
 ![ несколько источников питания ](./media/azure-stack-edge-pro-r-safety/icon-safety-disconnect-all-power.png) **предупреждение.**

* Для систем без бесперебойного питания (ИБП) Отключите все кабели питания от сети для полного отключения питания от оборудования.
* Для систем с ИБП отключите все кабели питания от сети и используйте выключатель питания для отключения системы. ИБП содержит опасные напряжения сети AC и DC.
* Если система включает ИБП, ИБП поставляются с экранированным входным кабелем питания. Необходимо использовать экранированный входной кабель питания, не заменять или изменять шнур.

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
 ![ значок электроудара ](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) **. Предупреждение:**

* Для систем, оснащенных ИБП, питание AC и (или) DC всегда будет иметь потенциальный риск исходящего напряжения переменного тока на выходе из батарей или служебной программы. Чтобы избежать повреждений оборудования или личной травмы, всегда следует предположить, что на выходе ИБП может произойти напряжение, даже при отключении от основного источника питания.
* Для систем, оснащенных ИБП, все неподключенные внешние подключения ИБП — это женщина. Не удаляйте вариант или ничего не вставляйте в эти или любые соединители ИБП.

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
 ![ значок электроудара ](./media/azure-stack-edge-pro-r-safety/icon-safety-electric-shock.png) **. Предупреждение:**

* Не пытайтесь изменить или использовать шнуры питания сети, отличные от устройств, поставляемых с оборудованием.

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png)
 ![ нет значок для обслуживаемых частей предупреждение ](./media/azure-stack-edge-pro-r-safety/icon-safety-do-not-access.png) **:**

* Это оборудование содержит литий-и/или литиевые фосфатеные батареи. Не пытайтесь обслуживать оборудование. Батареи в этом оборудовании не обслуживаются пользователями. Риск развертывания, если батарея заменена неверным типом.
* Удаление модуля батареи ИБП приводит к отключенным частям в ИБП. Не вставляйте внешние объекты в секции модуля аккумулятора.

## <a name="regulatory-information"></a>Сведения о соответствии нормам

В этом разделе содержатся сведения о соответствии нормативным требованиям для устройства R Azure Stack ребра, номер модели соответствия: Azure Stack ребра Pro R.

Устройство Azure Stack ребра Pro R погранично предназначено для использования с НРТЛ в списке (UL, CSA, ETL и т. д.) и IEC/EN 60950-1 или IEC/EN 62368-1 (помечено как CE) для оборудования информационных технологий.

Устройство предназначено для функционирования в следующих средах:

| Среда | Характеристики |
|:--- |:--- |
|Спецификации температуры | <ul><li>Температура хранения: – 33 &deg; C – 63 &deg; c (– 28 &deg; f-145 &deg; f) </li><li>Непрерывная работа: 5 &deg; C – 43 &deg; c (41 &deg; F – 110 &deg; f)</li><li>Максимальное значение градиента температуры (операционная система и хранилище): 20 &deg; C/h (68 &deg; F/h)</li></ul> |
|Спецификации относительной влажности | <ul><li>Хранилище: от 5% до 95% RH с 33 &deg; C (91 &deg; F) максимальная точка росы. Атмосфера должна быть не в любой момент времени.</li><li>Работа: от 5% до 85% относительной влажности с 29 &deg; C (84,2 &deg; F) максимальной росы точки</li></ul> |
| Максимальные характеристики высоты | <ul><li>Эксплуатация (без UPS): 15 000 фт (4 572 метров)</li><li>Эксплуатация (с ИБП): 10 000 фт (3 048 метров)</li><li>Хранилище: 40 000 фт (12 192 метров)</li></ul> |

<!--|Standard operating temperature specifications | <ul>Continuous operation (for altitude less than 950 m or 3117 ft): +5&deg;C–45&deg;C (41&deg;F–113&deg;F) with no direct sunlight on the equipment</ui>
|Expanded operating temperature specifications | <ul><li>Continuous operation: 5&deg;C to 45&deg;C at 5% to 85% RH with 29&deg;C dew point.</li><li></= 1% of annual operating hours: –5&deg;C to 55&deg;C at 5% to 90% RH with 29&deg;C dew point. |
| Expanded operating temperature restrictions | <ul><li>Do not perform cold start below -15&deg;C according to IEC 60945.</li><li>The operating temperature specified is for a maximum altitude of 950 meters.</li></ul> |
| Gaseous contamination specifications | <ul><li>Copper coupon corrosion rate: < 300 &Aring;/month per Class G1 as defined by ANSI/ISA71.04-1985.</li><li>Silver coupon corrosion rate: < 200 &Aring;/month as defined by AHSRAE TC9.9.</li></ul>|

> [!NOTE]
> Outside the standard operating temperature (10&deg;C to 35&deg;C):
> - The system can operate continuously in temperatures as low as 5&deg;C and as high as 45&deg;C. For temperatures between 35&deg;C and 45&deg;C, de-rate maximum allowable temperature by 1&deg;C per 175 m above 950 m (1&deg;F per 319 ft).
> - The system can operate down to –5&deg;C or up to 55&deg;C for a maximum of 1% of its annual operating hours. For temperatures between 45&deg;C and 55&deg;C, de-rate maximum allowable temperature by 1&deg;C per 125 m above 950 m (1&deg;F per 228 ft).
>
> When operating in the expanded temperature range:
> - System performance may be impacted.
> - Ambient temperature warnings may be reported in the System Event Log.     
>
> Maximum corrosive contaminant levels measured at &lt;/= 50% relative humidity. 


A device that has a UPS installed is designed to operate in the following environments:

| Environment | Specifications |
|:---  |:--- |
|Temperature specifications | <ul><li>Storage temperature: –40&deg;C–70&deg;C (–40&deg;F-158&deg;F) per Mil-Std 810G Method 501.5, Proc 1</li><li>Continuous operation (for altitude less than 950 m or 3117 ft): 5&deg;C–45&deg;C (41&deg;F–113&deg;F), with no direct sunlight on the equipment</li><li>Excursion temperature: 50&deg;C per Mil-Std 810G</li><li>Maximum temperature gradient (operating and storage): 20&deg;C/h (68&deg;F/h)</li></ul> |
|Relative humidity specifications | <ul><li>Storage: 5% to 95% RH with 33&deg;C (91&deg;F) maximum dew point. Atmosphere must be non-condensing at all times.</li><li>Operating: 5% to 85% relative humidity with 29&deg;C (84.2&deg;F) maximum dew point</li></ul>|
|Maximum altitude specifications | <ul><li>Storage: Mil-Std 810G method 500.5, Proc. I, 40,000 ft for 1 hour after stabilization </li><li>Operating: Mil-Std 810G method 500.5, Proc. II, air carriage, 15,000 ft for 1 hour after stabilization</li></ul>|
|Standard operating temperature specifications |<ul><li>Continuous operation (for altitude less than 950 m or 3117 ft)</li><li>+5&deg;C–45&deg;C (41&deg;F–113&deg;F) with no direct sunlight on the equipment</li></ul>|
|Expanded operating temperature specifications | <ul><li>Continuous operation: 5&deg;C to 45&deg;C at 5% to 85% RH with 29&deg;C dew point.</li><li>&lt;/= 1% of annual operating hours: 0&deg;C to 50&deg;C at 5% to 90% RH with 29&deg;C dew point.</li></ul>|
|Expanded operating temperature restrictions | <ul><li>Do not perform cold start below -15C Per IEC 60945</li><li>The operating temperature specified is for a maximum altitude of 950 meters</li></ul> |
|Gaseous contamination specifications | <ul><li>Copper coupon corrosion rate: < 300 &Aring;/month per Class G1 as defined by ANSI/ISA71.04-1985.</li><li>Silver coupon corrosion rate: < 200 &Aring;/month as defined by AHSRAE TC9.9.|-->

<!--
> [!NOTE]
> Outside the standard operating temperature (10&deg;C to 35&deg;C):
> - The system can operate continuously in temperatures as low as 5&deg;C and as high as 45&deg;C. For temperatures between 35&deg;C and 45&deg;C, de-rate maximum allowable temperature by 1&deg;C per 175 m above 950 m (1&deg;F per 319 ft).
> - The system can operate down to 0&deg;C or up to 50&deg;C for a maximum of 1% of its annual operating hours. For temperatures between 45&deg;C and 55&deg;C, de-rate maximum allowable temperature by 1&deg;C per 125 m above 950 m (1&deg;F per 228 ft).
> 
> When operating in the expanded temperature range:
> - System performance may be impacted.
> - Ambient temperature warnings may be reported in the System Event Log.
>
> Maximum corrosive contaminant levels measured at &lt;/= 50% relative humidity. --> 


> ![Значок уведомления — 2 Примечание ](./media/azure-stack-edge-pro-r-safety/icon-safety-notice.png) **.** &nbsp; изменения или изменения, внесенные в оборудование, не утвержденные корпорацией Майкрософт, могут аннулировать права пользователя на эксплуатацию оборудования.

#### <a name="canada-and-usa"></a>Для КАНАДЫ и США:

> ![Значок уведомления — 2 Примечание ](./media/azure-stack-edge-pro-r-safety/icon-safety-notice.png) **.** &nbsp; это оборудование было протестировано и найдено в соответствии с ограничениями для класса a Digital Device, согласно части 15 правил FCC. Ограничения разработаны для того, чтобы обеспечить надежную защиту от негативного влияния при эксплуатации оборудования в коммерческой среде. Это оборудование генерирует, использует и излучает радиочастотную энергию. Поэтому ненадлежащая установка и использование могут привести к помехам в радиосвязи. Работа этого оборудования в жилых помещениях, вероятнее всего, может вызвать вредоносные помехи. в этом случае пользователю потребуется устранить помехи с учетом их собственных расходов.

Это оборудование соответствует требованиям части 15 правил ФКС, а также промышленным стандартам Канады в области оборудования радиопередачи, не требующего лицензирования. Эксплуатация допускается при следующих двух условиях: 1) устройство не создает вредных помех; 2) устройство принимает любые помехи, включая помехи, которые могут привести к нарушению работы устройства.

![Предупреждение о соответствии нормативным требованиям 1](./media/azure-stack-edge-mini-r-safety/regulatory-information-1.png)


МОЖЕТ ICES-3 (A)/NMB-3 (A) корпорация Майкрософт, один Microsoft Way, Redmond, WA 98052, США США: (800) 426-9400 Канада: (800) 933-4750

#### <a name="european-union"></a>ЕВРОПЕЙСКИЙ СОЮЗ:

Запросите копию Декларации ЕС. Отправьте сообщение электронной почты с запросом кода на адрес [CSI_Compliance@microsoft.com](mailto:CSI_Compliance@microsoft.com).

![Значок предупреждения ](./media/azure-stack-edge-pro-r-safety/icon-safety-warning.png) **предупреждение!**

Устройство относится к оборудованию класса А. В домашних условиях этот продукт может вызывать радиопомехи, и в этом случае пользователю потребуется принять адекватные меры.

Утилизация использованных элементов питания, электрического и электронного оборудования:

![Значок предупреждения 14](./media/azure-stack-edge-mini-r-safety/icon-ewaste-disposal.png)

Такой символ, указанный на самом устройстве, его элементе питания или упаковке, означает, что устройство и любые его элементы питания не должны утилизироваться вместе с бытовыми отходами. Пользователь обязан передать устройство и его элементы питания в уполномоченный пункт сбора для утилизации элементов питания, электрического и электронного оборудования. Раздельный сбор и утилизация отходов помогают экономить природные ресурсы и предотвращать негативное влияние на здоровье людей и окружающую среду из-за наличия опасных веществ в составе элементов питания, электрического и электронного оборудования, а также их неправильной утилизации. За дополнительными сведениями о местах сбора использованных элементов питания и электрического оборудования обратитесь в местные органы власти, местную службу утилизации бытовых отходов или магазин, где вы приобрели устройство. Свяжитесь с erecycle@microsoft.com, чтобы получить дополнительную информацию по утилизации отходов электрического и электронного оборудования.

Устройство содержит кнопочный элемент питания.

Microsoft Ирландия Сандифорд, средство EST Дублин D18 KX32 ИРЛ телефон номер: + 353 1 295 3826 номер факса: + 353 1 706 4110


## <a name="next-steps"></a>Дальнейшие действия

- [Подготовка к развертыванию Azure Stack пограничных Pro R](azure-stack-edge-pro-r-deploy-prep.md)
