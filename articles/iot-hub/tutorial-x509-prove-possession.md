---
title: Руководство. Подтверждение владения сертификатами ЦС в Центре Интернета вещей Azure | Документация Майкрософт
description: Руководство. Подтверждение владения сертификатом ЦС для Центра Интернета вещей Azure
author: v-gpettibone
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: tutorial
ms.date: 02/26/2021
ms.author: robinsh
ms.custom:
- mvc
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: b7740fa1f6a54dcfcc1181dddedcdd5fdb50402c
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378233"
---
# <a name="tutorial-proving-possession-of-a-ca-certificate"></a>Руководство. Подтверждение владения сертификатом ЦС

После отправки сертификата корневого центра сертификации (ЦС) или подчиненного ЦС в центр Интернета вещей вам потребуется подтвердить, что вы владеете сертификатом:

1. На портале Azure перейдите к нужному Центру Интернета вещей и выберите **Параметры > Сертификаты**.

2. Выберите **Добавить**, чтобы добавить сертификат ЦС.

3. Введите отображаемое имя в поле **Имя сертификата** и выберите PEM-файл сертификата для добавления.

4. Щелкните **Сохранить**. Сертификат появится в списке сертификатов с состоянием **Не проверено**. Процесс проверки позволит вам подтвердить, что именно вы владеете сертификатом.

5. Выберите сертификат, чтобы открыть диалоговое окно **Сведения о сертификате**.

6. В диалоговом окне выберите **Создать код проверки**.

  :::image type="content" source="media/tutorial-x509-prove-possession/certificate-details.png" alt-text="{Диалоговое окно &quot;Сведения о сертификате&quot;}":::

7. Скопируйте код проверки в буфер обмена. Этот код проверки необходимо задать в качестве субъекта сертификата. Например, если код проверки имеет вид 75B86466DA34D2B04C0C4C9557A119687ADAE7D4732BDDB3, добавьте его в качестве субъекта сертификата, как показано на следующем шаге.

8. Существует три способа создания сертификата проверки:

    * Если вы используете скрипт PowerShell, предоставленный корпорацией Майкрософт, выполните командлет `New-CACertsVerificationCert "75B86466DA34D2B04C0C4C9557A119687ADAE7D4732BDDB3"`, чтобы создать сертификат с именем `VerifyCert4.cer`. Дополнительные сведения см. в статье [Использование предоставленных Майкрософт скриптов для создания тестовых сертификатов](tutorial-x509-scripts.md).

    * Если вы используете скрипт Bash, предоставленный корпорацией Майкрософт, выполните командлет `./certGen.sh create_verification_certificate "75B86466DA34D2B04C0C4C9557A119687ADAE7D4732BDDB3"`, чтобы создать сертификат с именем `verification-code.cert.pem`. Дополнительные сведения см. в статье [Использование предоставленных Майкрософт скриптов для создания тестовых сертификатов](tutorial-x509-scripts.md).

    * Если вы используете OpenSSL для создания сертификатов, сначала создайте закрытый ключ, а затем запрос на подписывание сертификата (CSR):

      ```bash
      $ openssl genpkey -out pop.key -algorithm RSA -pkeyopt rsa_keygen_bits:2048

      $ openssl req -new -key pop.key -out pop.csr

      -----
      Country Name (2 letter code) [XX]:.
      State or Province Name (full name) []:.
      Locality Name (eg, city) [Default City]:.
      Organization Name (eg, company) [Default Company Ltd]:.
      Organizational Unit Name (eg, section) []:.
      Common Name (eg, your name or your server hostname) []:75B86466DA34D2B04C0C4C9557A119687ADAE7D4732BDDB3
      Email Address []:

      Please enter the following 'extra' attributes
      to be sent with your certificate request
      A challenge password []:
      An optional company name []:
 
      ```

      Затем создайте сертификат с помощью файла конфигурации корневого ЦС (как показано ниже) или файла конфигурации подчиненного ЦС и CSR.

      ```bash
      openssl ca -config rootca.conf -in pop.csr -out pop.crt -extensions client_ext

      ```

    Дополнительные сведения см. в статье [Использование OpenSSL для создания тестовых сертификатов](tutorial-x509-openssl.md).

10. Выберите новый сертификат в представлении **Сведения о сертификате**.

11. После отправки сертификата щелкните **Проверить**. Состояние сертификата ЦС должно измениться на **Проверено**.
