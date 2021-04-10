---
title: Создание ключей, защищенных модулем HSM, и их передача в хранилище ключей Azure — Azure Key Vault
description: Данная статья поможет вам подготовить и создать ваши собственные ключи, защищенные аппаратным модулем безопасности, а затем передать их в хранилище ключей Azure. Также известные как собственные ключи или BYOK.
services: key-vault
author: amitbapat
manager: devtiw
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: keys
ms.topic: tutorial
ms.date: 02/24/2021
ms.author: ambapat
ms.openlocfilehash: f7761cf011a3a678bb7609e1063ac6ebec90d395
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102499192"
---
# <a name="import-hsm-protected-keys-for-key-vault-ncipher"></a>Создание и перемещение ключей, защищенных модулем HSM, для Key Vault (nCipher)

> [!WARNING]
> Метод импорта HSM-ключей, описанный в этом документе, является **устаревшим** и не будет поддерживаться в будущем. Он работает только с семейством HSM nCipher nShield со встроенным ПО 12.40.2 или 12.50 с исправлением. Настоятельно рекомендуется использовать [новый метод импорта HSM-ключей](hsm-protected-keys-byok.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Чтобы обеспечить более высокий уровень защиты при работе с хранилищем ключей Azure, можно импортировать ключи или создать их в аппаратных модулях безопасности (ключи никогда не покидают их пределы). Такой сценарий с использованием собственного ключа часто называется *BYOK*. Для защиты ключей Azure Key Vault использует семейство модулей HSM nCipher nShield (проверенных по стандарту FIPS 140-2 уровня 2).


Информация в этой статье поможет вам подготовить и создать ваши собственные ключи, защищенные аппаратным модулем безопасности, а затем передать их в хранилище ключей Azure. 

Эта функция недоступна в Azure для Китая (21Vianet).

> [!NOTE]
> Дополнительные сведения о хранилище ключей Azure см. в статье [Что такое хранилище ключей Azure?](../general/overview.md)  
> Указания по началу работы, включая создание хранилища для ключей, защищенных модулем HSM, см. в [этой статье](../general/overview.md).

Дополнительные сведения о создании и передаче ключей, защищенных аппаратным модулем безопасности, через Интернет:

* Ключ создается на автономной рабочей станции, что позволяет уменьшить область атаки.
* Ключ шифруется с использованием ключа обмена ключами и остается зашифрованным до тех пор, пока не будет передан в аппаратные модули безопасности хранилища ключей Azure. Исходную рабочую станцию покидает только зашифрованная версия ключа.
* Набор инструментов задает свойства ключа клиента, который привязывает ваш ключ к системе безопасности хранилища ключей Azure. В связи с этим после, как только аппаратные модули безопасности хранилища ключей Azure получат и расшифруют ваш ключ, использовать его смогут только они. Экспортировать ключ нельзя. Эта привязка реализуется аппаратными модулями безопасности nCipher.
* Ключ обмена ключами, используемый для шифрования вашего ключа, создается в аппаратных модулях безопасности хранилища ключей Azure и не может быть экспортирован. Аппаратные модули безопасности не допускают существования ключа обмена ключами в чистом виде за пределами аппаратных модулей безопасности. Кроме того, этот набор инструментов включает аттестацию, в процессе которой nCipher проверяет, можно ли экспортировать ключ обмена ключами и был ли он создан в подлинном модуле HSM производства nCipher.
* Набор инструментов включает аттестацию, в процессе которой nCipher проверяет, была ли система безопасности Azure Key Vault создана в подлинном модуле HSM производства nCipher. Эта аттестация является доказательством того, что корпорация Майкрософт использует подлинное оборудование.
* Корпорация Майкрософт применяет отдельные ключи шифрования ключей, а также отдельные системы безопасности в каждом географическом регионе. Такое разделение значит, что ваш ключ может использоваться только в центрах обработки данных того региона, в котором он был зашифрован. Например, ключ европейского клиента нельзя использовать в центрах обработки данных в Северной Америке или Азии.

## <a name="more-information-about-ncipher-hsms-and-microsoft-services"></a>Дополнительные сведения об аппаратных модулях безопасности nCipher и службах Майкрософт

nCipher Security, компания Entrust Datacard, — лидер на рынке HSM общего назначения, обеспечивающий надежность и целостность критически важной для бизнеса информации и приложений, а также возможности контроля над ними в ведущих организациях всего мира. Криптографические решения nCipher защищают новые технологии (облако, Интернет вещей, блокчейн, цифровые платежи), а также обеспечивают соответствие новым требованиям, используя те же проверенные технологии, которые международные организации применяют в данный момент для защит от угроз, связанных с конфиденциальными данными, сетевыми коммуникациями и корпоративной инфраструктурой. nCipher обеспечивает доверие к критически важным для бизнеса приложениям, гарантируя целостность данных и предоставляя клиентам полный контроль — сегодня, завтра и в любой момент.

Сотрудничество с компанией nCipher Security позволяет корпорации Майкрософт повышать уровень технического развития аппаратных модулей безопасности. Эти улучшения позволят вам пользоваться стандартными преимуществами размещаемых служб, не теряя контроль над ключами, а корпорации Майкрософт — управлять аппаратными модулями безопасности за вас. Поскольку Azure Key Vault является облачной службой, она легко масштабируется в соответствии с потребностями вашей организации. В то же время ваш ключ защищен в модулях HSM Майкрософт: вы получаете контроль над жизненным циклом ключа, так как создаете ключ и перемещаете его в модули HSM Майкрософт.

## <a name="implementing-bring-your-own-key-byok-for-azure-key-vault"></a>Реализация сценария BYOK для хранилища ключей Azure

Используйте сведения и процедуры этого раздела при создании собственного ключа, защищенного аппаратным модулем безопасности, и передачи этого ключа в хранилище ключей Azure — сценарий BYOK.

## <a name="prerequisites-for-byok"></a>Предварительные требования для BYOK

Список предварительных требований для реализации сценария BYOK для хранилища ключей Azure к см. в приведенной ниже таблице.

| Требование | Дополнительные сведения |
| --- | --- |
| Подписка на Azure |Для создания хранилища ключей Azure требуется подписка Azure: [зарегистрируйтесь для получения бесплатной пробной версии](https://azure.microsoft.com/pricing/free-trial/) |
| Поддержка защищенных ключей аппаратных модулей безопасности в хранилище ключей Azure уровня служб "Премиум". |Дополнительные сведения об уровнях служб и возможностях хранилища ключей Azure см. на веб-сайте [Цены на хранилище ключей Azure](https://azure.microsoft.com/pricing/details/key-vault/). |
| Модули HSM nCipher nShield, смарт-карты и программное обеспечение поддержки |Вы должны иметь доступ к аппаратному модулю безопасности nCipher и базовые знания о работе модулей HSM nCipher nShield. Список совместимых моделей или сведения о покупке аппаратного модуля безопасности, если у вас его нет, см. на странице [аппаратного модуля безопасности nCipher nShield](https://go.ncipher.com/rs/104-QOX-775/images/nCipher_nShield_Family_Brochure.pdf?_ga=2.106120835.1607422418.1590478092-577009923.1587131206). |
| Следующее оборудование и программное обеспечение:<ol><li>Автономная рабочая станция x64 с операционной системой Windows версии не ниже Windows 7 и программным обеспечением nCipher nShield версии не ниже 11.50.<br/><br/>Если рабочая станция выполняется на ОС Windows 7, [установите Microsoft .NET Framework 4.5](https://download.microsoft.com/download/b/a/4/ba4a7e71-2906-4b2d-a0e1-80cf16844f5f/dotnetfx45_full_x86_x64.exe).</li><li>Рабочая станция, подключенная к Интернету, с операционной системой Windows версии не ниже Windows 7 и установленным [Azure PowerShell](/powershell/azure/) **версии не ниже 1.1.0**.</li><li>USB-накопитель или другое переносное устройство хранения, на котором имеется не менее 16 МБ свободного места.</li></ol> |По соображениям безопасности рекомендуется не подключать первую рабочую станцию к сети. Однако это не применяется принудительно программным путем.<br/><br/>Далее эта рабочая станция называется отключенной рабочей станцией.</p></blockquote><br/>Кроме того, если ключ клиента предназначен для производственной сети, рекомендуется использовать вторую, отдельную рабочую станцию для скачивания набора средств и отправки ключа клиента. Но в целях тестирования можно использовать первую рабочую станцию.<br/><br/>Далее вторая рабочая станция называется рабочей станцией, подключенной к Интернету.</p></blockquote><br/> |

## <a name="generate-and-transfer-your-key-to-azure-key-vault-hsm"></a>Создание и передача ключа в аппаратный модуль безопасности хранилища ключей Azure

Для создания и передачи ключа в аппаратный модуль безопасности хранилища ключей Azure используются следующие пять шагов:

* [Шаг 1. Подготовка рабочей станции, подключенной к Интернету](#step-1-prepare-your-internet-connected-workstation)
* [Шаг 2. Подготовка отключенной рабочей станции](#step-2-prepare-your-disconnected-workstation)
* [Шаг 3. Создание ключа](#step-3-generate-your-key)
* [Шаг 4. Подготовка ключа к передаче](#step-4-prepare-your-key-for-transfer)
* [Шаг 5. Передача ключа в хранилище ключей Azure](#step-5-transfer-your-key-to-azure-key-vault)

## <a name="step-1-prepare-your-internet-connected-workstation"></a>Шаг 1. Подготовка рабочей станции, подключенной к Интернету

Чтобы подготовить рабочую станцию, подключенную к Интернету, выполните описанные ниже действия.

### <a name="step-11-install-azure-powershell"></a>Шаг 1.1. Установка Azure PowerShell

С компьютера, подключенного к Интернету, загрузите и установите модуль Azure PowerShell, который содержит командлеты для управления хранилищем ключей Azure. Инструкции по установке см. в статье [Установка и настройка Azure PowerShell](/powershell/azure/).

### <a name="step-12-get-your-azure-subscription-id"></a>Шаг 1.2. Получение идентификатора подписки Azure

Запустите сеанс Azure PowerShell и выполните вход в учетную запись Azure с помощью следующей команды:

```Powershell
   Connect-AzAccount
```
Во всплывающем окне браузера введите имя пользователя и пароль учетной записи Azure. Затем воспользуйтесь командой [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription):

```powershell
   Get-AzSubscription
```
В выходных данных найдите идентификатор для подписки, которая будет использоваться для хранилища ключей Azure. Этот идентификатор подписки понадобится позже.

Не закрывайте окно Azure PowerShell.

### <a name="step-13-download-the-byok-toolset-for-azure-key-vault"></a>Шаг 1.3. Загрузка набора инструментов BYOK для хранилища ключей Azure

Перейдите в Центр загрузки Майкрософт и [скачайте набор инструментов BYOK для хранилища ключей Azure](https://www.microsoft.com/download/details.aspx?id=45345) для своего географического региона или экземпляра Azure. Чтобы определить имя пакета для скачивания и его хэш SHA-256, используйте следующие сведения.

---
**США:**

KeyVault-BYOK-Tools-UnitedStates.zip

2E8C00320400430106366A4E8C67B79015524E4EC24A2D3A6DC513CA1823B0D4

---
**Европа**

KeyVault-BYOK-Tools-Europe.zip

9AAA63E2E7F20CF9BB62485868754203721D2F88D300910634A32DFA1FB19E4A

---
**Азия**

KeyVault-BYOK-Tools-AsiaPacific.zip

4BC14059BF0FEC562CA927AF621DF665328F8A13616F44C977388EC7121EF6B5

---
**Латинская Америка**

KeyVault-BYOK-Tools-LatinAmerica.zip

E7DFAFF579AFE1B9732C30D6FD80C4D03756642F25A538922DD1B01A4FACB619

---
**Япония**

KeyVault-BYOK-Tools-Japan.zip

3933C13CC6DC06651295ADC482B027AF923A76F1F6BF98B4D4B8E94632DEC7DF

---
**Корея**

KeyVault-BYOK-Tools-Korea.zip

71AB6BCFE06950097C8C18D532A9184BEF52A74BB944B8610DDDA05344ED136F

---
**Южно-Африканская Республика**

KeyVault-BYOK-Tools-SouthAfrica.zip

C41060C5C0170AAAAD896DA732E31433D14CB9FC83AC3C67766F46D98620784A

---
**ОАЭ**

KeyVault-BYOK-Tools-UAE.zip

FADE80210B06962AA0913EA411DAB977929248C65F365FD953BB9F241D5FC0D3

---
**Австралия**

KeyVault-BYOK-Tools-Australia.zip

CD0FB7365053DEF8C35116D7C92D203C64A3D3EE2452A025223EEB166901C40A

---
[**Azure для государственных организаций:**](https://azure.microsoft.com/features/gov/)

KeyVault-BYOK-Tools-USGovCloud.zip

F8DB2FC914A7360650922391D9AA79FF030FD3048B5795EC83ADC59DB018621A

---
**Министерство обороны США**

KeyVault-BYOK-Tools-USGovernmentDoD.zip

A79DD8C6DFFF1B00B91D1812280207A205442B3DDF861B79B8B991BB55C35263

---
**Канада**

KeyVault-BYOK-Tools-Canada.zip

61BE1A1F80AC79912A42DEBBCC42CF87C88C2CE249E271934630885799717C7B

---
**Германия**

KeyVault-BYOK-Tools-Germany.zip

5385E615880AAFC02AFD9841F7BADD025D7EE819894AA29ED3C71C3F844C45D6

---
**Германия (общедоступная версия)**

KeyVault-BYOK-Tools-Germany-Public.zip

54534936D0A0C99C8117DB724C34A5E50FD204CFCBD75C78972B785865364A29

---
**Индия**

KeyVault-BYOK-Tools-India.zip

49EDCEB3091CF1DF7B156D5B495A4ADE1CFBA77641134F61B0E0940121C436C8

---
**Франция:**

KeyVault-BYOK-Tools-France.zip

5C9D1F3E4125B0C09E9F60897C9AE3A8B4CB0E7D13A14F3EDBD280128F8FE7DF

---
**Великобритания:**

KeyVault-BYOK-Tools-UnitedKingdom.zip

432746BD0D3176B708672CCFF19D6144FCAA9E5EB29BB056489D3782B3B80849

---
**Швейцария**

KeyVault-BYOK-Tools-Switzerland.zip

88CF8D39899E26D456D4E0BC57E5C94913ABF1D73A89013FCE3BBD9599AD2FE9

---


Чтобы проверить целостность скачанного набора инструментов BYOK, в сеансе Azure PowerShell запустите командлет [Get-FileHash](/powershell/module/microsoft.powershell.utility/get-filehash) .

   ```powershell
   Get-FileHash KeyVault-BYOK-Tools-*.zip
   ```

Набор инструментов включает в себя следующее.

* Пакет ключей обмена ключами с именем, начинающимся с **BYOK-KEK-pkg-.**
* Пакет Security World с именем, начинающимся с **BYOK-SecurityWorld-pkg-**
* Сценарий на Python с именем **verifykeypackage.py.**
* Исполняемый файл командной строки с именем **KeyTransferRemote.exe** и связанными DLL-файлами.
* Распространяемый пакет Visual C++ с именем **vcredist_x64.exe.**

Скопируйте пакет на USB-накопитель или другое переносное устройство.

## <a name="step-2-prepare-your-disconnected-workstation"></a>Шаг 2. Подготовка отключенной рабочей станции

Чтобы подготовить рабочую станцию, не подключенную к сети (к Интернету или внутренней сети), выполните указанные ниже действия.

### <a name="step-21-prepare-the-disconnected-workstation-with-ncipher-nshield-hsm"></a>Шаг 2.1. Подготовка отключенной рабочей станции с помощью модуля HSM nCipher nShield

Установите программное обеспечение поддержки nCipher на компьютере Windows, а затем подключите к этому компьютеру модуль HSM nCipher nShield.

Убедитесь, что инструменты nCipher расположены в каталоге по вашему пути ( **%nfast_home%\bin**). Например, введите следующую команду:

  ```cmd
  set PATH=%PATH%;"%nfast_home%\bin"
  ```

Дополнительные сведения см. в руководстве пользователя, которое поставляется вместе с модулем HSM nShield.

### <a name="step-22-install-the-byok-toolset-on-the-disconnected-workstation"></a>Шаг 2.2. Установка набора инструментов BYOK на отключенную рабочую станцию

Скопируйте пакет набора инструментов BYOK с USB-накопителя или другого переносного устройства, а затем выполните указанные ниже действия.

1. Извлеките файлы из загруженного пакета в любую папку.
2. Запустите файл vcredist_x64.exe из этой папки.
3. Следуйте инструкциям по установке компонентов среды выполнения Visual C++ для Visual Studio 2013.

## <a name="step-3-generate-your-key"></a>Шаг 3. Создание ключа

Чтобы создать свой ключ, выполните на отключенной рабочей станции указанные ниже действия. Для выполнения этого шага HSM должен находиться в режиме инициализации. 

### <a name="step-31-change-the-hsm-mode-to-i"></a>Шаг 3.1. Изменение режима HSM на I

Если вы используете nCipher nShield Edge, для изменения режима сделайте следующее. 1. Используйте кнопку "Режим", чтобы выделить необходимый режим. 2. Через несколько секунд нажмите и удерживайте пару секунд кнопку "Очистить". Если режим изменится, светодиодный индикатор нового режима перестанет мигать и начнет светиться. Индикатор состояния может беспорядочно мигать в течение нескольких секунд. Затем, когда устройство будет готово, он станет мигать регулярно. В противном случае устройство останется в текущем режиме. При этом загорится соответствующий индикатор режима.

### <a name="step-32-create-a-security-world"></a>Шаг 3.2. Создание системы безопасности

Откройте командную строку и запустите новую программу системы безопасности nCipher.

   ```cmd
    new-world.exe --initialize --cipher-suite=DLf3072s256mRijndael --module=1 --acs-quorum=2/3
   ```

Эта программа создает файл **Security World** в каталоге %NFAST_KMDATA%\local\world, который соответствует папке C:\ProgramData\nCipher\Key Management Data\local. Можно использовать разные значения для кворума, но в данном примере вам будет предложено ввести три пустые карты и закрепления для каждой из них. После этого любые две карты предоставят полный доступ к системе безопасности. Эти карты станут **набором карт администратора** для новой системы безопасности.

> [!NOTE]
> Если HSM не поддерживает более новый набор тестов DLf3072s256mRijndael, вы можете заменить --cipher-suite= DLf3072s256mRijndael на --cipher-suite=DLf1024s160mRijndael.
> 
> Система безопасности, созданная с помощью программы new-world.exe, которая поставляется с программным обеспечением nCipher версии 12.50, несовместима с этой процедурой BYOK. Есть два варианта.
> 1) Понизить версию программного обеспечения nCipher до 12.40.2, чтобы создать новую систему безопасности.
> 2) Обратиться в службу поддержки nCipher и попросить их предоставить исправление для версии программного обеспечения 12.50, которое позволяет использовать версию 12.40.2 программы new-world.exe, совместимую с этой процедурой BYOK.

После этого выполните описанные ниже действия.

* Создайте резервную копию файла системы безопасности. Защитите файл системы безопасности, карты администратора и их закрепления и убедитесь, что все имеют доступ не больше, чем к одной карте.

### <a name="step-33-change-the-hsm-mode-to-o"></a>Шаг 3.3. Изменение режима HSM на "O"

Если вы используете nCipher nShield Edge, для изменения режима сделайте следующее. 1. Используйте кнопку "Режим", чтобы выделить необходимый режим. 2. Через несколько секунд нажмите и удерживайте пару секунд кнопку "Очистить". Если режим изменится, светодиодный индикатор нового режима перестанет мигать и начнет светиться. Индикатор состояния может беспорядочно мигать в течение нескольких секунд. Затем, когда устройство будет готово, он станет мигать регулярно. В противном случае устройство останется в текущем режиме. При этом загорится соответствующий индикатор режима.

### <a name="step-34-validate-the-downloaded-package"></a>Шаг 3.4. Проверка скачанного пакета

Этот шаг необязателен, но рекомендуется, поскольку позволяет проверить следующее.

* Ключ обмена ключами, включенный в набор инструментов, создан в подлинном модуле HSM nCipher nShield.
* Хэш системы безопасности, включенный в набор инструментов, создан в подлинном модуле HSM nCipher nShield.
* Ключ обмена ключами недоступен для экспорта.

> [!NOTE]
> Для проверки загруженного пакета аппаратный модуль безопасности должен быть подключен, включен и оснащен системой безопасности (которую вы только что создали).

Проверка загруженного пакета

1. Запустите сценарий verifykeypackage.py, указав одно из приведенных ниже значений в зависимости от вашего географического региона или экземпляра Azure.

   * Для Северной Америки:

      ```azurepowershell
      "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-NA-1 -w BYOK-SecurityWorld-pkg-NA-1
      ```
   * Для Европы:

      ```azurepowershell
      "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-EU-1 -w BYOK-SecurityWorld-pkg-EU-1
      ```
   * Для Азии:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-AP-1 -w BYOK-SecurityWorld-pkg-AP-1
        ```
   * Для Латинской Америки:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-LATAM-1 -w BYOK-SecurityWorld-pkg-LATAM-1
        ```
   * Для Японии:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-JPN-1 -w BYOK-SecurityWorld-pkg-JPN-1
        ```
   * Для Кореи:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-KOREA-1 -w BYOK-SecurityWorld-pkg-KOREA-1
        ```
   * Для Южно-Африканской Республики:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-SA-1 -w BYOK-SecurityWorld-pkg-SA-1
        ```
   * Для ОАЭ:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-UAE-1 -w BYOK-SecurityWorld-pkg-UAE-1
        ```
   * Для Австралии:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-AUS-1 -w BYOK-SecurityWorld-pkg-AUS-1
        ```
   * Для [Azure для государственных организаций](https://azure.microsoft.com/features/gov/), использующего экземпляр Azure для правительства США:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-USGOV-1 -w BYOK-SecurityWorld-pkg-USGOV-1
        ```
   * Для министерства обороны США:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-USDOD-1 -w BYOK-SecurityWorld-pkg-USDOD-1
        ```
   * Для Канады:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-CANADA-1 -w BYOK-SecurityWorld-pkg-CANADA-1
        ```
   * Для Германии:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-GERMANY-1 -w BYOK-SecurityWorld-pkg-GERMANY-1
        ```
   * Для Германии (общедоступная версия):

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-GERMANY-1 -w BYOK-SecurityWorld-pkg-GERMANY-1
        ```
   * Для Индии:

      ```azurepowershell
      "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-INDIA-1 -w BYOK-SecurityWorld-pkg-INDIA-1
      ```
   * Для Франции:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-FRANCE-1 -w BYOK-SecurityWorld-pkg-FRANCE-1
        ```
   * Для Соединенного Королевства:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-UK-1 -w BYOK-SecurityWorld-pkg-UK-1
        ```
   * Для Швейцарии:

        ```azurepowershell
        "%nfast_home%\python\bin\python" verifykeypackage.py -k BYOK-KEK-pkg-SUI-1 -w BYOK-SecurityWorld-pkg-SUI-1
        ```

     > [!TIP]
     > Программное обеспечение nCipher nShield включает файл python, который находится в каталоге в %NFAST_HOME%\python\bin.
     >
     >
2. Должен появиться следующий текст, показывающий, что проверка пройдена: **Результат: УСПЕШНО**

Этот сценарий проверяет цепочку до корневого ключа nShield подписавшего пользователя. Хэш этого корневого ключа встроен в скрипт, и его значение должно быть **59178a47 de508c3f 291277ee 184f46c4 f1d9c639**. Кроме того, вы можете проверить это значение отдельно на [веб-сайте nCipher](https://www.ncipher.com).

Теперь можно создать новый ключ.

### <a name="step-35-create-a-new-key"></a>Шаг 3.5. Создание ключа

Создайте ключ с помощью программы **generatekey** в nCipher nShield.

Для создания ключа выполните следующую команду:

```azurepowershell
generatekey --generate simple type=RSA size=2048 protect=module ident=contosokey plainname=contosokey nvram=no pubexp=
```

При выполнении команды следуйте приведенным ниже инструкциям.

* Параметру *protect* должно быть задано значение **module**, как показано. В результате будет создан ключ, защищенный модулем. Набор средств BYOK не поддерживает ключи, защищенные OCS.
* Замените значение *contosokey* на **ident**, а **plainname** — любым строковым значением. Чтобы минимизировать административные затраты и снизить риск возникновения ошибок, рекомендуется использовать для обоих параметров одно и то же значение. Значение **Ident** должно содержать только цифры, тире и буквы нижнего регистра.
* В данном примере значение параметра pubexp остается пустым (по умолчанию), но можно указать и конкретные значения. Дополнительные сведения см. в [документации по nCipher](https://www.entrust.com/-/media/documentation/brochures/entrust-nshield-general-purpose-hsms-br-a4.pdf).

Эта команда создает файл ключа с маркером в папке %NFAST_KMDATA%\local с именем, начинающимся с **key_simple_**, за которым следует значение параметра **ident**, указанное в команде. Пример: **key_simple_contosokey**. Этот файл содержит зашифрованный ключ.

Создайте резервную копию файла ключа с токеном в безопасном расположении.

> [!IMPORTANT]
> После передачи ключа в хранилище ключей Azure корпорация Майкрософт не сможет экспортировать ключ обратно, поэтому крайне важно сделать резервную копию ключа и системы безопасности. За инструкциями и рекомендациями по резервному копированию ключа обращайтесь в компанию [nCipher](https://www.ncipher.com/about-us/contact-us).
>


После этого ключ можно передать в хранилище ключей Azure.

## <a name="step-4-prepare-your-key-for-transfer"></a>Шаг 4. Подготовка ключа к передаче

Чтобы подготовить ключ к передаче, выполните на отключенной рабочей станции указанные ниже действия.

### <a name="step-41-create-a-copy-of-your-key-with-reduced-permissions"></a>Шаг 4.1. Создание копии ключа с ограниченными разрешениями

Откройте новую командную строку и измените текущий каталог на каталог, в который вы разархивировали ZIP-файл BYOK. Чтобы ограничить разрешения в ключе, выполните через командную строку одну из следующих команд в зависимости от вашего географического региона или экземпляра Azure.

* Для Северной Америки:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-NA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-NA-
   ```
* Для Европы:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-EU-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-EU-1
   ```
* Для Азии:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AP-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AP-1
   ```
* Для Латинской Америки:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-LATAM-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-LATAM-1
   ```
* Для Японии:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-JPN-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-JPN-1
   ```
* Для Кореи:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-KOREA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-KOREA-1
   ```
* Для Южно-Африканской Республики:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SA-1
   ```
* Для ОАЭ:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UAE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UAE-1
   ```
* Для Австралии:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AUS-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AUS-1
   ```
* Для [Azure для государственных организаций](https://azure.microsoft.com/features/gov/), использующего экземпляр Azure для правительства США:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USGOV-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USGOV-1
   ```
* Для министерства обороны США:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USDOD-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USDOD-1
   ```
* Для Канады:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-CANADA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-CANADA-1
   ```
* Для Германии:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1
   ```
* Для Германии (общедоступная версия):

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1
   ```
* Для Индии:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-INDIA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-INDIA-1
   ```
* Для Франции:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-FRANCE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-FRANCE-1
   ```
* Для Соединенного Королевства:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UK-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UK-1
   ```
* Для Швейцарии:

   ```azurepowershell
   KeyTransferRemote.exe -ModifyAcls -KeyAppName simple -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SUI-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SUI-1
   ```

При запуске этой команды замените *contosokey* тем же значением, которое вы указали при выполнении этапа **Шаг 3.5. Создание ключа** процедуры [Создание ключа](#step-3-generate-your-key) .

Появится запрос на подключение карт администратора системы безопасности.

После завершения команды появится текст **Результат: успешно\<contosokey>, а копия ключа с ограниченными разрешениями появится в файле с именем key_xferacId_**.

Вы можете проверить списки ACL, используя следующие команды и служебные программы nCipher nShield:

* aclprint.py:

   ```cmd
   "%nfast_home%\bin\preload.exe" -m 1 -A xferacld -K contosokey "%nfast_home%\python\bin\python" "%nfast_home%\python\examples\aclprint.py"
   ```
* kmfile-dump.exe:

   ```cmd
   "%nfast_home%\bin\kmfile-dump.exe" "%NFAST_KMDATA%\local\key_xferacld_contosokey"
   ```
  При запуске этих команд замените contosokey тем же значением, которое вы указали на этапе **Шаг 3.5. Создание ключа** в разделе [Создание ключа](#step-3-generate-your-key).

### <a name="step-42-encrypt-your-key-by-using-microsofts-key-exchange-key"></a>Шаг 4.2. Шифрование ключа с помощью ключа обмена ключами Майкрософт

Выполните одну из следующих команд в зависимости от вашего географического региона или экземпляра Azure:

* Для Северной Америки:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-NA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-NA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Европы:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-EU-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-EU-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Азии:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AP-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AP-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Латинской Америки:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-LATAM-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-LATAM-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Японии:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-JPN-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-JPN-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Кореи:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-KOREA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-KOREA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Южно-Африканской Республики:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для ОАЭ:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UAE-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UAE-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Австралии:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-AUS-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-AUS-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для [Azure для государственных организаций](https://azure.microsoft.com/features/gov/), использующего экземпляр Azure для правительства США:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USGOV-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USGOV-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для министерства обороны США:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-USDOD-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-USDOD-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Канады:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-CANADA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-CANADA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Германии:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Германии (общедоступная версия):

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-GERMANY-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-GERMANY-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Индии:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-INDIA-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-INDIA-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Франции:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-France-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-France-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Соединенного Королевства:

   ```azurepowershell
   KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-UK-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-UK-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
   ```
* Для Швейцарии:

  ```azurepowershell
  KeyTransferRemote.exe -Package -KeyIdentifier contosokey -ExchangeKeyPackage BYOK-KEK-pkg-SUI-1 -NewSecurityWorldPackage BYOK-SecurityWorld-pkg-SUI-1 -SubscriptionId SubscriptionID -KeyFriendlyName ContosoFirstHSMkey
  ```


При выполнении команды следуйте приведенным ниже инструкциям.

* Замените *contosokey* идентификатором, который вы использовали для создания ключа на этапе **Шаг 3.5. Создание ключа** в разделе [Создание ключа](#step-3-generate-your-key).
* Замените *SubscriptionID* на идентификатор подписки Azure, содержащей ваше хранилище ключей. Это значение было получено ранее на этапе **Шаг 1.2. Получение идентификатора подписки Azure** процедуры [Подготовка рабочей станции, подключенной к Интернету](#step-1-prepare-your-internet-connected-workstation) .
* Замените *ContosoFirstHSMKey* меткой, которая используется для имени выходного файла.

После выполнения команды отобразится сообщение **Результат: успешно**, а в текущей папке появится новый файл с именем KeyTransferPackage-*ContosoFirstHSMkey*.byok.

### <a name="step-43-copy-your-key-transfer-package-to-the-internet-connected-workstation"></a>Шаг 4.3. Копирование пакета передачи ключа на рабочую станцию, подключенную к Интернету

С помощью USB-накопителя или другого переносного устройства скопируйте выходной файл из предыдущего шага (KeyTransferPackage-ContosoFirstHSMkey.byok) на рабочую станцию, подключенную к Интернету.

## <a name="step-5-transfer-your-key-to-azure-key-vault"></a>Шаг 5. Передача ключа в хранилище ключей Azure

Наконец, на рабочей станции, подключенной к Интернету, выполните командлет [Add-AzKeyVaultKey](/powershell/module/az.keyvault/add-azkeyvaultkey), чтобы отправить пакет передачи ключа, скопированный с отключенной рабочей станции, в аппаратный модуль безопасности Azure Key Vault.

   ```powershell
        Add-AzKeyVaultKey -VaultName 'ContosoKeyVaultHSM' -Name 'ContosoFirstHSMkey' -KeyFilePath 'c:\KeyTransferPackage-ContosoFirstHSMkey.byok' -Destination 'HSM'
   ```

При успешном завершении отправки на экране появятся свойства добавленного ключа.

## <a name="next-steps"></a>Дальнейшие действия

Теперь ключ, защищенный с помощью аппаратного модуля безопасности, можно использовать в хранилище ключей. Дополнительные сведения см. на [этой странице цен и сравнения характеристик](https://azure.microsoft.com/pricing/details/key-vault/).