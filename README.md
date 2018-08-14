# Решение задания 1 — найди ошибки

Решение тестового задания "Найди ошибки" для [14-й Школы разработки интерфейсов](https://academy.yandex.ru/events/frontend/shri_msk-2018-2).

## Исправления
1) Webpack не компилирует JS из-за ошибки при импорте в *index.js*. Можно добавить фигурные скобки при импорте либо default при экспорте:
	```diff
	- import initMap from  "./map";
	+ import { initMap } from  "./map";
	```

2) Ничего не видно. Открыв Chrome DevTools, видим, что карта нулевой высоты, а должна занимать весь экран. Для `html, body, #map` надо добавить `height: 100%`.
3) Теперь карта отображается, но объектов на ней нет. Смотрим, как добавляются объекты. В *map.js* создаётся карта и ObjectManager, в который добавляются объекты. Судя по документации, забыли добавить сам ObjectManager на карту. Сделаем это в обработчике промиса:

    ```diff
	loadList().then(data  => {
		objectManager.add(data);
	+	myMap.geoObjects.add(objectManager);
	});
	```

4) Объектов на карте всё равно нет. Меняем масштаб и видим: как обычно, широта и долгота перепутаны местами, и дроны управляются откуда-то из Ирана. Поправим *mappers.js*:
	```diff
	- coordinates: [obj.long, obj.lat]
	+ coordinates: [obj.lat, obj.long]
	```
5) Теперь объекты отображаются, но при клике по ним баллун не появляется, а в консоли ошибки вызова каких-то методов. Находим причину в *details.js*: в вызове templateLayoutFactory.createClass во втором параметре переопределения родительских методов указаны как стрелочные функции, а в них this привязан к окружению, в котором была создана функция. Даже при вызове в качестве метода объекта this всё равно не привязывается к нему. Для правильной привязки this надо заменить стрелочные функции на обычные.
6) Баллун появился, но на графике диапазон по вертикальной оси (от -1 до 0) не позволяет увидеть точки. Судя по генератору тестовых данных, максимальное значение для этого графика – 10. То есть, станция может одновременно обслуживать до 10 дронов. Укажем в *chart.js*:
	```diff
	- yAxes: [{ ticks: { beginAtZero:  true, max:  0 } }]
	+ yAxes: [{ ticks: { beginAtZero:  true, max:  10 } }]
	```
7) Иконка кластера не показывает, есть ли в нём неисправный объект. В документации к API есть [пример](https://tech.yandex.ru/maps/jsbox/2.1/clusterer_pie_chart)  кластеров в виде круговых диаграмм. Судя по всему, здесь предполагается его использование, в *map.js* уже задан нужный layout для кластера:
	```
	clusterIconLayout: 'default#pieChart'
	```
	...который затем по ошибке перезаписывается строкой, удалив которую, получаем требуемое поведение:
	```diff
	- objectManager.clusters.options.set('preset', 'islands#greenClusterIcons');
	```
8) Файл *popup.js* удалён, так как он нигде не используется и, похоже, содержит устаревший код формирования баллуна.
9) **Codestyle**: решено отформатировать весь JS и CSS с помошью Prettier со страндартными настройками, а также добавить автоформатирование на commit hook.

## Тесты

Единственные тесты, которые мне здесь представляются возможными – визуальные: скармливаем двум версиям приложения (стабильной и той, которую тестируем) идентичные тестовые данные. Затем делаем ряд скриншотов и сравниваем их.

## Мысли

Возможно, в генераторе тестовых данных *utils/generate-data.js* есть логическая ошибка, в результате которой количество соединений для базовой станции в прошлом может быть максимум 8, а в настоящем максимум 10. В прошлом (строка 15) ```Math.floor(Math.random() * 7) + 2```, в настоящем (строка 34) ```Math.floor(Math.random() * 7) + 4```. Поэтому значение 10 на графиках всегда бывает только последним.
