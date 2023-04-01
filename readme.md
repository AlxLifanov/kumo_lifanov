## Постановка задачи

КУМО является шлюзом между сетью с высокой целостностью и сетью с низкой целостностью. 

## Концепция безопасности
### Активы и ущербы

|Актив|Неприемл.событие|Ущерб|
| ----------- | ----------- | ----------- |
|Внутренняя сеть|Пакет снаружи|Воздействие на PLC|
|Обновление фирмвера|Пользователь неавторизован|Потеря защищенности|
|Обновление фирмвера|Фирмвер неправильно подписан|Потеря защищенности|

### Сценарии работы

1. Данные получаются от внутреннего источника и кладутся в буферную БД.
2. Данные запрашиваются снаружи и отдаются запрашивающему.
3. Пользователь заливает новый фирмвер и обновляет прошивку.

## ЦПБ
### Цели безопасности
Ц1. Данные из внешней сети во внутреннюю не передаются.

Ц2. Внешний пользователь может только считывать данные, сгенерированные ПЛК.

Ц3. Неавторизованный пользователь не имеет права заливки фирмвера.

Ц4. Не прошедшая проверку прошивка в устройство не шьется.

### Предположения безопасности
П1. Устройство не доступно для физического доступа.

П2. Невозможно образование каналов связи между программными компонентами помимо запроектированных (=MILS реализован корректно).

### Предварительная архитектура

![](https://github.com/AlxLifanov/kumo_lifanov/blob/53298a824d546679cd7d31bbb449a2c20fb1161f/initial.png)

## Положительные сценарии
### Обновление и запрос данных
![](https://www.plantuml.com/plantuml/png/NP1D3e9038NtSufUm0kmKFpPc1XZu0JDK9G4Pt3QmNXxYn0ON4txsk-zPd8M31AVhUdqRMpJeHEuNOwhlY1BJKzX9HvYNVz99RbA9RJYWqAlI2pQFvmN0hJ1CsVbnsdXV6GmcWERxMmFcPRts6A02WNDhElEcAORLte3zUHaRRldn7UE7iZnA2NFhEn9ZNTcqbBVP3ffAJka_cvQo2Ka6PjSv_gT_0K0)

### Обновление фирмвера
![](https://www.plantuml.com/plantuml/png/VP0n3i8m34Ltdy8xS046L8GwS00sbYXDIDJWH8chzlYu8a3KWVNzvs_BNM6LUNe6K9fZnYjnJmzFnkHA-kL7ahXU-wI8yJonYzcN3RuueySDkwL1iJ1GWQ7PW8TJKuD73542tA6TeLdxJpCq03IXgFM2hnV7_Ptq2jU1JofYFKj4Exyj6rHnw1cWasBvIda1)

## Отрицательные сценарии
### Нарушение целостности буфера данных (нарушение Ц2)
![](https://www.plantuml.com/plantuml/png/NL3D3i8W3Bxp54qy-m8x6Fzk6cFS4n3MAGc3MShCtjukJcIUsdw_X6raJPmwftgPrjXUUuRh-R2uRsZG3aWBS9_QzW-gqJT8i29ib3OSI9pFX955z_214RIX46DLZruM7r849MMmtkGEg6Iz9wS96irQPLmqdqfHNF05EA3dPUmk5hcvUdBEBwlWP9qxJhapKgPcRaPA7tOzZuNfb_xbr5P8-Sdu1W00)

### Перехват авторизации (формальное нарушение Ц3)
![](https://www.plantuml.com/plantuml/png/ZP6nRiCm34HtVGM1Zcb-84EGeWZGRbqwrHrKYPRLA9GWCN3pzw5GsN2C3bsFftT7wb1OPxwSWzA6OuPNuYCvJuYvASUVMO67AuOGnpPvqUPKnOnM9Q5uuvCcXgSB3p7xyjViUS3ww4vxJrQpQ6gWo0ZQv5ZAxYsZc12ae4FLihOVbVYvNsRim9EOGkWeZH2YSyAwGybIc0qcRh3bIjiLIzvxWWtqerheSsxhqNmT-GczJln6m__fxcAq4sm3Nzgka_VJpUPoyVJOaQbYzKvJCRP3Yh4VVrrzq6K0pqeTaAnyymq0)

### Политика архитектуры
|Компонент|Делает|Отдает|Получает|Оценка|
| ----------- | ----------- | ----------- | ----------- | ----------- |
|PLC|Управление рабочим процессом|Данные о техпроцессе|Подтверждение отправки||
|Requester|Запрашивает данные|Команда запроса|Данные о техпроцессе||
|IntEngineer|Обновление прошивки|Login + FW|Нет||
|ExtEngineer|Обновление прошивки|Login + FW|Нет||
|Receiver|Ведет обмен с ПЛК|Подтверждение ПЛК, новые данные в буфер (с превышением)|Данные от ПЛК|MM|
|Buffer|Хранит события для Requester|События|Запросы только от Sender|MM|
|Sender|Ведет обмен с Requester, обрабатывает только корректные запросы. Требует верификации. Можно выделить входной файрволл отдельно, если внешний протокол обмена сложный.|События|Только запросы событий|SS|
|TLS Terminator|Обеспечивает неперехватываемость данных в канале|HTTP|HTTPS|CM|
|Input sorter|Фильтрация HTTP-посылок|Файлы - только запись в Storage, данные авторизации - только запись в Authoriser|Raw HTTP|SS|
|Authoriser|Проверка авторизованности пользователя|Если пользователь имеет право прошивки - машет флажком Checker|Login + password|SM|
|FW storage|Хранение прошивки-кандидата|Прошивку по запросу Checker|Прошивку от сортера. Рассматривается как блоб, не парсится.|MM|
|Checker|Проверяет подпись прошивки, если корректная - забирает из storage c удалением там и отдает в updater|Прошивку в Updater|Команду от авторизатора, прошивку из Storage|ML|

Разметка сделана с точки зрения целостности данных.

![](https://github.com/AlxLifanov/kumo_lifanov/blob/53298a824d546679cd7d31bbb449a2c20fb1161f/offer.png)
