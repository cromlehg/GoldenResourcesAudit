# Аудит смартконтрактов проекта Golden Resources

###### Репозиторий с кодом контрактов https://github.com/DenisKaizer/GDR/

Коммит, для которого проводился аудит: https://github.com/DenisKaizer/GDR/tree/69662b096cfc45ba919a50848d254f96ca2ede21




## Уровни выявленных проблем

##### Низкий — не оказывает существенного влияния на возможные сценарии использования контракта, часто бывает субъективной.
##### Средний — уязвимость, которая может повлиять на желаемый результат исполнения контракта в конкретном сценарии.
##### Высокий — уязвимость, которая влияет на желаемый результат при использовании контракта или предоставляет возможность использовать контракт не регламентированным образом.
##### Критический — уязвимость, которая может вызвать нарушение работы контракта по ряду сценариев или предоставляет возможность нарушить работу контракта.




## Окружение, в котором проводился аудит

truffle 4.0.6 - фреймворк для запуска тестов
Версия компилятора: 0.4.19+commit.c4cbbb05.Emscripten.clang, в связи с использованием этой версии в truffle 4.0.6 (последней на момент аудита)




## Анализ файла contracts/GDR.sol

Файл содержит код контракта токена и базовые классы из библиотеки Zeppelin Solidity, реализующие стандарт ERC20

### Низкий уровень

###### 1. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/GDR.sol#L285

Согласно стандарту тип для decimal должен быть uint8: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md#decimals

###### 2. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/GDR.sol#L55
###### https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/GDR.sol#L56
###### https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/GDR.sol#L64
###### https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/GDR.sol#L65
###### https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/GDR.sol#L66
###### https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/GDR.sol#L151


Некоторые базовые контракты из библиотеки Zeppelin Solidity взяты не самой последней версии, в которой не прописана видимость для некоторых функций, что вызвает предупреждения компилятора `Warning: No visibility specified. Defaulting to "public"`

##### 3. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/GDR.sol#L123

В реализацию токена добавлен модификатор, который в дальнейшем не используется




## Анализ файла contracts/Crowdsale.sol

Файл содержит в себе контракт PreICO и вспомогательный контракт из библиотеки Zeppelin Solidity

### Средний уровень

###### 1. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/Crowdsale.sol#L161

Текущий механизм вычисления количества купленных токенов неточен: сначала происходит деление на большое число (priceUSD), потом умножение на большое число (rate).

Например, если купить токенов меньше, чем на 1 цент, то контракт отправит покупатлю 0 токенов, хотя decimals токена позволяют совершить такую покупку. Аналогично при покупке на 1.5 цента - токены придут только за 1 цент.

##### 2. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/Crowdsale.sol#L143

Нет проверки на нулевое значение. Также рекомендуется проверка на аномально большое изменение цены. Например, если цена эфира уменьшилась/увеличилась в 10 раз, то, скорее всего, было ошибочное введение цены.

### Низкий уровень

###### 1. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/Crowdsale.sol#L133
###### https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/Crowdsale.sol#L138

Нет проверки на пустой адрес

###### 2. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/Crowdsale.sol#L127

Возможно нужна проверка на то, что закончилось время распродажи или достигнут hardcap

###### 3. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/Crowdsale.sol#L46
###### https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/contracts/Crowdsale.sol#L142

Некорректные комментарии




## Анализ файла migrations/2_crowdsale_migrations.js

Файл содержит скрипт деплоя контрактов в сеть

### Высокий уровень

###### 1. https://github.com/DenisKaizer/GDR/blob/69662b096cfc45ba919a50848d254f96ca2ede21/migrations/2_crowdsale_migrations.js#L10

В качестве первого параметра контракт PreICO принимает unixtime в секундах, а `Date.now()` возвращает в миллисекундах: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/now
