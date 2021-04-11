---
title: Учебник по созданию, сборке и развертыванию смарт-контрактов в службе "Блокчейн Azure"
description: Руководство по использованию комплекта SDK Блокчейн Azure для расширения Ethereum в Visual Studio Code для создания, сборки и развертывания смарт-контракта в службе Azure Блокчейн.
ms.date: 11/30/2020
ms.topic: tutorial
ms.reviewer: caleteet
ms.openlocfilehash: 4c2df952480d2c30de10838c3d0f7714fc7e6126
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105628651"
---
# <a name="tutorial-create-build-and-deploy-smart-contracts-on-azure-blockchain-service"></a>Руководство по Создание, сборка и развертывание смарт-контрактов в службе "Блокчейн Azure"

В этом руководстве показано использование комплекта SDK Блокчейн Azure для расширения Ethereum в Visual Studio Code для создания, сборки и развертывания смарт-контракта в службе "Блокчейн Azure". Вы также можете использовать комплект SDK для выполнения функции смарт-контракта путем транзакции.

С помощью комплекта SDK службы "Блокчейн Azure" для Ethereum выполняются такие задачи:

> [!div class="checklist"]
> * Создание смарт-контракта
> * Развертывание смарт-контракта
> * выполнение функции смарт-контракта путем транзакции;

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

* См. подробнее об [использовании Подключение к сети консорциума службы "Блокчейн Azure" с помощью Visual Studio Code](connect-vscode.md)
* [Visual Studio Code](https://code.visualstudio.com/Download)
* [расширение "Комплект SDK Блокчейна Azure для Ethereum"](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain);
* [Node.js 10.15.x или более поздней версии](https://nodejs.org/download).
* [Git 2.10.x или более поздней версии](https://git-scm.com).
* [Truffle 5.0.0](https://www.trufflesuite.com/docs/truffle/getting-started/installation).
* [Ganache CLI 6.0.0](https://github.com/trufflesuite/ganache-cli).

В Windows для модуля node-gyp должен быть установлен компилятор C++. Вы можете использовать средства MSBuild:

* Если установлен Visual Studio 2017, настройте NPM для использования средств MSBuild с помощью команды `npm config set msvs_version 2017 -g`.
* Если установлен Visual Studio 2019, задайте путь к средствам сборки MS для NPM. Например `npm config set msbuild_path "C:\Program Files (x86)\Microsoft Visual Studio\2019\Community\MSBuild\Current\Bin\MSBuild.exe"`.
* В противном случае установите автономные средства сборки VS, используя `npm install --global windows-build-tools` в командной оболочке с повышенными правами *от имени администратора*.

Дополнительные сведения о node-gyp см. в репозитории [node-gyp на сайте GitHub](https://github.com/nodejs/node-gyp).

## <a name="create-a-smart-contract"></a>Создание смарт-контракта

Комплект SDK службы "Блокчейн Azure" для Ethereum использует шаблоны проектов и средства Truffle для создания, сборки и развертывания контрактов. Перед началом работы изучите следующее [краткое руководство: Подключение к сети консорциума Блокчейн Azure с помощью Visual Studio Code](connect-vscode.md). Краткое руководство содержит инструкции по установке и настройке комплекта SDK службы "Блокчейн Azure" для Ethereum.

1. В палитре команд VS Code выберите **Блокчейн: Создать проект Solidity**.
1. Выберите **Create basic project** (Создать базовый проект).
1. Создайте папку с именем `HelloBlockchain` и выберите **Select new project path** (Выбрать путь к новому проекту).

Комплект SDK службы "Блокчейн Azure" создаст и инициализирует проект Solidity. Базовый проект содержит пример смарт-контракта **HelloBlockchain** и все необходимые файлы для сборки и развертывания в участнике консорциума в службе "Блокчейн Azure". Создание проекта может занять несколько минут. Вы можете отслеживать ход выполнения на панели терминала VS Code, выбрав выходные данные для Azure Блокчейн.

Структура проекта выглядит так, как показано в следующем примере.

   ![Проект Solidity](./media/send-transaction/solidity-project.png)

## <a name="build-a-smart-contract"></a>Сборка смарт-контракта

Смарт-контракты находятся в каталоге проекта **contracts**. Нужно выполнить сборку смарт-контрактов перед их развертыванием в блокчейне. Используйте команду **Build Contracts** (Выполнить сборку контрактов), чтобы компилировать все смарт-контракты в проекте.

1. В боковой панели обозревателя VS Code разверните папку **contracts** своего проекта.
1. Щелкните правой кнопкой мыши файл **HelloBlockchain.sol** и в меню выберите пункт **Build Contracts** (Выполнить сборку контрактов).

    ![Выберите меню сборки контрактов ](./media/send-transaction/build-contracts.png)

Комплект SDK службы "Блокчейн Azure" использует Truffle для компиляции смарт-контрактов.

![Выходные данные компилятора Truffle](./media/send-transaction/compile-output.png)

## <a name="deploy-a-smart-contract"></a>Развертывание смарт-контракта

Truffle использует скрипты миграции для развертывания контрактов в сети Ethereum. Скрипты миграции — это файлы JavaScript, которые находятся в каталоге проекта **migrations**.

1. Чтобы развернуть смарт-контракт, щелкните правой кнопкой мыши файл **HelloBlockchain.sol** и в меню выберите пункт **Deploy Contracts** (Развернуть контракты).
1. Выберите сеть консорциума службы "Блокчейн Azure"в палитре команд. Сеть блокчейн-консорциума добавлена в файл конфигурации проекта Truffle при создании проекта.
1. Выберите **Generate mnemonic** (Создать мнемонический код). Задайте имя файлу с мнемоническим кодом и сохраните его в папке проекта. Например, `myblockchainmember.env`. Файл с мнемоническим кодом используется для создания закрытого ключа Ethereum для вашего участника блокчейна.

Комплект SDK службы "Блокчейн Azure" использует Truffle, чтобы выполнить скрипт миграции для развертывания контрактов в блокчейне.

![Успешное развертывание контракта](./media/send-transaction/deploy-contract.png)

## <a name="call-a-contract-function&quot;></a>Вызов функции контракта
Функция **SendRequest** контракта **HelloBlockchain** изменяет переменную состояния **RequestMessage**. Изменение состояния сети блокчейна выполняется через транзакцию. Можно создать скрипт для выполнения функции **SendRequest** путем транзакции.

1. В корне проекта Truffle создайте файл с именем `sendrequest.js`. Добавьте в файл следующий код Web3 JavaScript.

    ```javascript
    var HelloBlockchain = artifacts.require(&quot;HelloBlockchain");
        
    module.exports = function(done) {
      console.log("Getting the deployed version of the HelloBlockchain smart contract")
      HelloBlockchain.deployed().then(function(instance) {
        console.log("Calling SendRequest function for contract ", instance.address);
        return instance.SendRequest("Hello, blockchain!");
      }).then(function(result) {
        console.log("Transaction hash: ", result.tx);
        console.log("Request complete");
        done();
      }).catch(function(e) {
        console.log(e);
        done();
      });
    };
    ```

1. При создании проекта с помощью комплекта SDK службы "Блокчейн Azure" создается файл конфигурации Truffle со сведениями о конечной точке сети блокчейн-консорциума. Откройте в проекте файл **truffle-config.js**. В этом файле конфигурации указаны две сети — одна с именем development и вторая с именем консорциума.
1. В терминале VS Code используйте Truffle, чтобы выполнить скрипт для своей сети блокчейн-консорциума. В строке меню терминала выберите вкладку **Терминал**, а в раскрывающемся списке — **PowerShell**.

    ```PowerShell
    truffle exec sendrequest.js --network <blockchain network>
    ```

    Замените \<blockchain network\> именем сети блокчейн, указанным в файле **truffle-config.js**.

Truffle выполнит скрипт для сети блокчейн.

![Выходные данные: транзакция была отправлена](./media/send-transaction/execute-transaction.png)

При выполнении функции контракта через транзакцию транзакция не обрабатывается до тех пор, пока не будет создан блок. Функции, предназначенные для выполнения путем транзакции, вместо значения возвращают идентификатор транзакции.

## <a name="query-contract-state"></a>Запрос состояния контракта

Функции смарт-контракта могут возвращать текущее значение переменных состояния. Добавим функцию для возврата значения переменной состояния.

1. В файле **HelloBlockchain.sol** добавьте функцию **getMessage** в смарт-контракт **HelloBlockchain**.

    ``` solidity
    function getMessage() public view returns (string memory)
    {
        if (State == StateType.Request)
            return RequestMessage;
        else
            return ResponseMessage;
    }
    ```

    Функция возвращает сообщение, хранящееся в переменной состояния, о текущем состоянии контракта.

1. Щелкните правой кнопкой мыши файл **HelloBlockchain.sol** и в меню выберите **Build Contracts** (Выполнить сборку контрактов), чтобы внести изменения в смарт-контракт.
1. Чтобы развернуть его, щелкните правой кнопкой мыши файл **HelloBlockchain.sol** и в меню выберите пункт **Deploy Contracts** (Развернуть контракты). При появлении запроса выберите сеть консорциума службы "Блокчейн Azure"в палитре команд.
1. Затем создайте скрипт, предназначенный для вызова функции **getMessage**. В корне проекта Truffle создайте файл с именем `getmessage.js`. Добавьте в файл следующий код Web3 JavaScript.

    ```javascript
    var HelloBlockchain = artifacts.require("HelloBlockchain");
    
    module.exports = function(done) {
      console.log("Getting the deployed version of the HelloBlockchain smart contract")
      HelloBlockchain.deployed().then(function(instance) {
        console.log("Calling getMessage function for contract ", instance.address);
        return instance.getMessage();
      }).then(function(result) {
        console.log("Request message value: ", result);
        console.log("Request complete");
        done();
      }).catch(function(e) {
        console.log(e);
        done();
      });
    };
    ```

1. В терминале VS Code используйте Truffle, чтобы выполнить скрипт для своей сети блокчейн. В строке меню терминала выберите вкладку **Терминал**, а в раскрывающемся списке — **PowerShell**.

    ```bash
    truffle exec getmessage.js --network <blockchain network>
    ```

    Замените \<blockchain network\> именем сети блокчейн, указанным в файле **truffle-config.js**.

Скрипт выполняет запрос к смарт-контракту, вызывая функцию getMessage. Возвращается текущее значение переменной состояния **RequestMessage**.

![Выходные данные запроса getmessage, в которых показано текущее значение переменной состояния RequestMessage](./media/send-transaction/execute-get.png)

Обратите внимание, что возвращается не значение **Hello, blockchain!**, а заполнитель. При изменении и развертывании контракт получает новый адрес, а переменным состояния присваиваются значения в конструкторе смарт-контрактов. Скрипт миграции Truffle **2_deploy_contracts.js** развертывает смарт-контракт и передает значение заполнителя в качестве аргумента. Конструктор задает для переменной состояния **RequestMessage** значение заполнителя, которое и возвращается.

1. Чтобы задать значение для переменной состояния **RequestMessage** и запросить его, выполните еще раз скрипты **sendrequest.js** и **getmessage.js**.

    ![Выходные данные скриптов sendrequest и getmessage: транзакция RequestMessage отправлена](./media/send-transaction/execute-set-get.png)

    Скрипт **sendrequest.js** задает переменной состояния **RequestMessage** значение **Hello, blockchain!**, а **getmessage.js** запрашивает значение переменной состояния контракта **RequestMessage** и возвращает **Hello, blockchain!**.
## <a name="clean-up-resources"></a>Очистка ресурсов

Если ресурсы больше не нужны, вы можете удалить их. Для этого удалите группу ресурсов `myResourceGroup`, которую вы создали при выполнении предварительных требований *краткого руководства по созданию участника блокчейна*.

Чтобы удалить группу ресурсов, сделайте следующее:

1. На портале Azure перейдите к **группам ресурсов** в области навигации слева и выберите группу ресурсов, которую необходимо удалить.
1. Выберите **Удалить группу ресурсов**. Подтвердите удаление, введя имя группы ресурсов и выбрав **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Из этого руководства вы узнали, как создать проект Solidity с помощью комплекта SDK службы "Блокчейн Azure". Вы создали и развернули смарт-контракт, а также вызывали функцию через транзакцию в сети блокчейн-консорциума, размещенной в службе "Блокчейн Azure".

> [!div class="nextstepaction"]
> [Разработка приложений блокчейна с помощью службы "Блокчейн Azure"](develop.md)
