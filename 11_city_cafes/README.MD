# Анализ рынка общественного питания Москвы

> python pandas numpy matplotlib scipy seaborn plotly math re json Map, Choropleth

_В файле cafe_moscow_new.ipynb не отображаются интерактивные карты с покраской районов по показателям_

## Условие

Инвесторы решили попробовать себя в новой области и открыть заведение общественного питания в Москве. Заказчики поставили задачу **изучить этот рынок и хотят получить рекомендации где и почему было бы выгодно открыть кофейню.**

В качестве источника имеем **датасет с заведениями общественного питания Москвы**, составленный на основе данных сервисов Яндекс.Карты и Яндекс.Бизнес на лето 2022 года.

**Цели исследования:**

1. Изучить рынок общественного питания Москвы, более детально - рынок кофеен.
2. Разработать рекомендации для заказчика.


## Итоги

### Исследования рынка

- Больше всего в Москве заведений категории кафе (28.3%), на втором месте рестораны (24.3%), на третьем месте кофейни (16.8%). Меньше всего булочных (3.05%) и столовых (3.75%).

- Больше всего посадочных мест в ресторанах (86 ед.), барах и пабах (82 ед.), кофейнях (80 ед.). Меньше всего в булочных (50 ед.), пиццериях (55 ед.).

- Доля сетевых заведений составляет более трети - 38.1%, несетевых - 61.9%. Больше всего сетевых заведений встречается среди кафе, ресторанов и кофеен. Меньше всего - среди столовых, булочных и баров / пабов.

- Более четверти заведений общественного питания Москвы находится в ЦАО, далее по популярномти САО и ЮАО. Меньше всего заведений в СЗАО и ЮЗАО. В целом заметна популярность во всех районах Москвы кафе и ресторанов. По графику виднео, что в центре города есть тенденция к популярности баров и пабов, кофеен.

- Средний рейтинг в разрезе категорий не сильно отличается. Самый высокий средний рейтинг у баров и пабов (4.4 балла), а самый низкий у заведений быстрого питания (4.1 балл). В ЦАО самый высокий средний рейтинг - 4.4 балла, самый низкий в ЮВАО - 4.1 балла. Скорее всего рейтинги зависят от плотности населения / проходимости, которая подразумевает высокую конкуренцию и как следствие более качественный сервис.

- В ТОП-15 вошли улицы с плотностью заведений общественно питания от 48 до 184 единиц. Многие из представленных улиц имеют большую протяженность. Например, проспект Мира - 8,9 км., Профсоюзная улица - 9.3 км., Ленинский проспект - 16 км.

- Есть 416 улиц, на которых есть только по 1 заведению общественного питания. В ЦАО больше всего улиц всего с одним заведением общественного питания - 138 единиц. Скорее всего это связано с тем, что в центр Москвы содержит много улиц с небольшой протяженностью. В целом такие улицы есть в каждом из районов. Чаще всего среди "одиночек" встречаются кафе, рестораны и кофейни.

- Самый высокий чек в ЦАО и ЗАО - 1000 рублей. Эти районы считаются престижными - популярные среди туристов локации, бизнес-центры, университеты и др. Самые низкие чеки в ЮВАО (450 рублей), СВАО, ЮАО (по 500 рублей) - в 2 и более раза ниже, чем в ЦАО и ЗАО. Медиана среднего чека зависит не напрямую от расстояния до центра города, а скорее от инфрастурктуры, плотности насления, плотности бизнеса, общей проходимости. 

### Перспективы открытия кофейни

Кофейня - очень популярный формат общепита в Москве, он стоит на третьем месте по популярности после кафе и ресторанов. Всего в Москве 1413 кофеен, что составляет 16.8% всех заведений общественного питания.

**Конкуренция**
- Почти треть всех кофеен Москвы находится в ЦАО - 428 ед. (30.29%). На втором месте САО - 193 ед. (13.66%). Меньше всего кофеен в СЗАО - 62 ед.(4.39%). **Лучше выбрать место с хорошей проходимостью, концентрацией бизнес-центров, учебных заведений, культурных объектов и др.** Обычно кофейни удобны для быстрого перекуса, проведения встреч в течение рабочего дня, чтобы согреться и взбодриться в течение дня. Поэтому лучше обратить внимание на районы с высокой конкуренцией - ЦАО, САО. 

**Формат**
- Почти половина всех кофеен (49%) являются сетевыми. В ЦАО очень высокая концентрация - в нем находится каждая десятая кофейня сетевого формата. Сетевой формат означает налаженные бизнес-процессы, широкий охват и лояльных клиентов. В ТОП-5 сетевых кофеен входят Шоколадница, One Price Coffee, Cofix, КОФЕПОРТ, CofeFest. **При выборе локации следует объективно оценивать свою возможность конкурировать с сетевыми кофейнями**.

**Режим работы**
- Круглосуточный режим в работе кофеен не является популярным, по всей Москве таких заведений всего 59 ед. (5.3%). Почти половина из них находитсяв ЦАО (26 ед.). Это можно объяснить тем, что сам формат кофейни предполагает подачу кофе, десертов, перекусов, что больше применимо к дневному времяпровождению. **Лучше выбрать режим работы с раннего утра (например, с 8 ч.) и до вечера**.

**Отношение клиентов**
- Медианный рейтинг среди кофеен составляет 4.3 балла, средний - 4.27 баллов. Есть кофейни с рейтингом ниже 1.5 баллов и равным максимальным 5 баллам. Самый высокий рейтинг в ЦАО - 4.34 балла, самый низкий в ЗАО - 4.2 балла. 

**Ценовая политика**
- Самый высокий средний чек обычно бывает в ЗАО (600 руб.) и ЦАО (500 руб.), а чашка капучно в среднем стоит дорого ЮЗАО (195 руб.), ЦАО (190 руб.) и ЗАО (189 руб.). Дешевле всего ходить в кофейни в ЮВАО (средний чек всего 250 руб., а чашка капучино обойдется в 147 руб.). **Высокая медиана цен в ЗАО и ЦАО может означать, что в этих районах клиенты платежеспособны**.
