# **План автоматизации тестирования**

**Цель плана автоматизации** - описание процесса автоматизации тестирования комплексного сервиса, взаимодействующего с СУБД и API Банка.

### **Исходные данные:**

|     Поле      | Валидные данные                                                 | Невалидные данные   |
|:-------------:|:----------------------------------------------------------------|:--------------------|
| `Номер карты` | специальные номера карт, которые даны для тестирования:         | пустое поле         |
|               | APPROVED карта - `1111 2222 3333 4444`                          | 0000 0000 0000 0000 |
|               | DECLINED карта - `5555 6666 7777 8888`                          | пробелы             |
|               |                                                                 | 1 цифра             |
|               |                                                                 | 15 цифр             |
|               |                                                                 | 17 цифр             |
|               |                                                                 | цифры и буквы       |
|               |                                                                 | символы             |
|    `Месяц`    | числовое значение из 2-х цифр (от 01 до 12),                    | пустое поле         |
|               | не может быть месяцем в прошлом                                 | 00                  |
|               | (01, 02, 11, 12)                                                | пробелы             |
|               |                                                                 | 13                  |
|               |                                                                 | одна цифра          |
|               |                                                                 | цифра и буква       |
|               |                                                                 | символы             |
|     `Год`     | числовое значение текущего или следующего года                  | пустое поле         |
|               | из 2-х цифр (от 22 до 99),                                      | 00     21           |
|               | не может быть годом в прошлом                                   | текущий год +6      |
|               | Срок, на который выдана карта - 5 лет (предположение)           | пробелы             |
|               | текущий год (+1, +2, +5)                                        | одна цифра          |
|               |                                                                 | цифра и буква       |
|               |                                                                 | символы             |
|  `Владелец`   | фамилия и имя на латинице,                                      | пустое поле         |
|               | состоящие из двух слов через пробел                             | пробелы             |
|               | в верхнем регистре                                              | символы             |
|               | Имя и фамилия клиента не лолжны превышать 20 знаков с пробелами | цифры               |
|               |                                                                 | ФИ кириллицей       |
|               |                                                                 | 1 буква в имени     |
|               |                                                                 | 1 буква в фамилии   |
|               |                                                                 | три слова латиницей |
|               |                                                                 | ФИ без пробела      |
|     `CVC`     | любое число,                                                    | пустое поле         |
|               | состоящее из 3-х цифр (от 100 до 999)                           | нули                |
|               |                                                                 | одна цифра          |
|               |                                                                 | две цифры           |
|               |                                                                 | четыре цифры        |
|               |                                                                 | буквы               |
|               |                                                                 | символы             |
|               |                                                                 | пробелы             |


## **Перечень автоматизируемых сценариев**

### **UI сценарии**

#### **Предусловия:**
1. Открыть страницу http://localhost:8080
2. Нажать кнопку `Купить`, открывается `Оплата по карте` ИЛИ
3. Нажать кнопку `Купить в кредит`, открывается `Кредит по данным карты`

|  №  | Отправка формы                                                                               | Результат                            |
|:---:|:---------------------------------------------------------------------------------------------|:-------------------------------------|
|     | **Позитивные сценарии**                                                                      |                                      |
|  1  | Покупка `Оплата по карте` / `Кредит по данным карты` со статусом `APPROVED` валидные данные  | `Успешно` Операция одобрена банком   |
|  2  | Покупка `Оплата по карте` / `Кредит по данным карты` со статусом `DECLINED` валидные данные  | `Ошибка` Отказ в проведении операции |
|     | **Негативны сценарии**                                                                       |                                      |
|  3  | Поле`Номер карты` - ввод невалидных данных `Оплата по карте` / `Кредит по данным карты`      | `Ошибка` Отказ в проведении операции |
|  4  | Поле `Месяц` - ввод невалидных данных `Оплата по карте` / `Кредит по данным карты`           | `Ошибка` Отказ в проведении операции |
|  5  | Поле `Год` - ввод невалидных данных `Оплата по карте` / `Кредит по данным карты`             | `Ошибка` Отказ в проведении операции |
|  6  | Поле `Владелец` - ввод невалидных данных `Оплата по карте` / `Кредит по данным карты`        | `Ошибка` Отказ в проведении операции |
|  7  | Поле `CVC` - ввод невалидных данных `Оплата по карте` / `Кредит по данным карты`             | `Ошибка` Отказ в проведении операции |
|  8  | Отправление данных с незаполненными полями `Оплата по карте` / `Кредит по данным карты`      | `Ошибка` Отказ в проведении операции |

### **API сценарии**

Симулятор банковских сервисов:

`Кредит по данным карты` `http://185.119.57.197:9999/credit`

`Оплата по карте` `http://185.119.57.197:9999/payment`

|  №  | Отправка формы                                                                                                                                     | Результат                           |
|:---:|:---------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------|
|  1  | Отправка POST запроса `Купить` с валидным body и карты со статусом `APPROVED` на `http://185.119.57.197:9999/payment`                              | статус 200, появление  записи в БД  |
|  2  | Отправка POST запроса `Купить в кредит`  с валидным body и карты со статусом `APPROVED` на `http://185.119.57.197:9999/credit`                     | статус 200, появление  записи в БД  |
|  3  | Отправка POST запроса `Купить` с валидным body и карты со статусом `DECLINED` `http://185.119.57.197:9999/payment`                                 | статус 200, появление  записи в БД  |
|  4  | Отправка POST запроса `Купить в кредит` с валидным body и карты со статусом `DECLINED` на `http://185.119.57.197:9999/credit`                      | статус 200, появление  записи в БД  |
|  5  | Отправка POST запроса с пустым body на `http://185.119.57.197:9999/payment` / `http://185.119.57.197:9999/credit`                                  | статус 400, отсутствие записи в БД  |
|  6  | Отправка POST запроса с пустым значением у атрибута `number` в body на `http://185.119.57.197:9999/payment`  / `http://185.119.57.197:9999/credit` | статус 400, отсутствие записи в БД  |
|  7  | Отправка POST запроса с пустым значением у атрибута `month` в body на `http://185.119.57.197:9999/payment`  / `http://185.119.57.197:9999/credit`  | статус 400, отсутствие записи в БД  |
|  8  | Отправка POST запроса с пустым значением у атрибута `year` в body на `http://185.119.57.197:9999/payment`  / `http://185.119.57.197:9999/credit`   | статус 400, отсутствие записи в БД  |
|  9  | Отправка POST запроса с пустым значением у атрибута `holder` на `http://185.119.57.197:9999/payment`  / `http://185.119.57.197:9999/credit`        | статус 400, отсутствие записи в БД  |
| 10  | Отправка POST запроса с пустым значением у атрибута `cvc` на `http://185.119.57.197:9999/payment`  / `http://185.119.57.197:9999/credit`           | статус 400, отсутствие записи в БД  |

## **Перечень используемых инструментов с обоснованием выбора**

|  №  | Используемый инструмент                             | Обоснование выбора                                                                                                                        |
|:---:|:----------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------|
|  1  | Среда разработки `IntelliJ IDEA`                    | Умная и удобная среда для Java с поддержкой всех последних технологий и фреймворков                                                       |
|  2  | Язык программирования: `Java JDK11`                 | Имеет набор готового ПО для разработки и запуска приложений                                                                               |
|  3  | Система автоматической сборки: `Gradle`             | В Gradle (по сравнению с Maven) проще подключать зависимости, меньше кода, гибкость системы.                                              |
|  4  | Тестовая среда `JUnit5`                             | Тестовый фреймворк, совместимый с JVM, содержит необходимые аннотации и assert’ы                                                          |
|  5  | Для UI тестирования: `Selenide`                     | Библиотека для написания лаконичных и стабильных UI тестов с открытым исходным кодом                                                      |
|  6  | Для API тестирования: `Rest Assured`                | Библиотека для тестирования REST API, позволяет автоматизировать тестирование get и post запросов                                         |
|  7  | Плагин `Lombok`                                     | Для минимизации строк кода за счет аннотаций позволяет оптимизировать код автотестов                                                      |
|  8  | Библиотека `JavaFaker`                              | Для генерации случайных тестовых данных                                                                                                   |
|  9  | Система репортинга: `Allure`                        | Для создания отчетов о тестировании и наглядного отображения прохождения тестов и ошибок (более сложные системы репортинга будут излишни) |
| 10  | Платформа контейнеризации `Docker Desktop`          | Для развертывания контейнеров с базами данных MySQL и PostgreSQL и сервисов банка                                                         |
| 11  | Система контроля версий `Git` и веб-сервис `GitHub` | Git - возможность одновременной параллельной разработки, есть интеграция с IntelliJ IDEA. GitHub - удобный интерфейс, интегрирован с Git  |

## **Перечень и описание возможных рисков при автоматизации**

1. Отсутствие документации на приложение (какое поведение системы считать ошибочным)
2. Тесты могут не покрывать всех возможных негативных сценариев (пользователь может повести себя так, как мы не можем предвидеть)
3. Падение автотестов UI даже при незначительном изменении кода (изменения: структуры сайта, локаторов элементов) 
5. Потеря актуальности тестов API при изменение структуры запросов со стороны банка
6. Некорректность тестовых данных при работе с Faker
7. Актуализация, обновление и поддрежание тестов может занять достаточно большое количество времени

## **Интервальная оценка с учётом рисков (в часах)**

|  №  | Наименование работы                                          | Оценка с учетом рисков ~30% (в часах) |
|:---:|:-------------------------------------------------------------|:-------------------------------------:|
|  1  | Ознакомление с проектом + план автоматизации тестирования    |                   8                   |
|  2  | Написание и отладка авто-тестов                              |                  32                   |
|  3  | Настройка окружения и поведение тестирования                 |                   8                   |
|  4  | Подготовка и оформление баг-репортов                         |                  16                   |
|  5  | Формирование отчета о проведенном тестировании               |                   4                   |
|  6  | Формирование отчета о проведённой автоматизации тестирования |                   4                   |
|     | Итого                                                        |                  72                   |

## **План сдачи работ**

|  №  | Наименование работы                                 | Дата сдачи |
|:---:|:----------------------------------------------------|:----------:|
|  1  | План автоматизации тестирования                     | 21.10.2022 |
|  2  | Написание и отладка авто-тестов                     | 28.10.2022 |
|  3  | Подготовка и оформление репортов                    | 30.10.2022 |
|  4  | Отчеты по тестированию и автоматизации тестирования | 01.11.2022 |
