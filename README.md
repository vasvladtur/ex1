## Демонстрация некоторых случаев с которыми я столкнулся.
 Сделано на основе расширения 1С (удобно для Демонстрации).
 1С:Предприятие 8.3 (8.3.23.1782) 1С:ERP Управление предприятием 2 (2.5.12.53)
 
 Изменения где необходимо помечаются комментарием 
 ```
 //1729
 //-1729
 ```
 Реализовать можно и в расширении, но 
### Я решил:  меняем конфигурацию. (оригинальную сохраняем в безопасном месте)
 Задачи сгенерированы обычными пользователями
 Все идеи взяты из открытых источников. (Спасибо авторам)
 Новые сущности имеют префикс др_
 
1. Снимаем с поддержки

2. Создать форму списка справочника серии независящее от параметров открытия

2.1. Создать форму списка справочника серии (просто без кода, добавляем реквизиты справочника)
 делаем ее основной

2.2. Добавить в подсистему НСИ и администрирование -> НСИ -> Серии номенклатуры

3. Создать подсистему Расширение (Тестовое решение)

4. _
 

5. Создать возможность для (CRM и маркетинг)->(Скидки (наценки))->
(Создать элемент (например, 
- Тип скидки (скидка количеством),
- предоставляется на: Номенклатуру из списка, 
- нажать гиперссылка справа,
- кнопка Подобрать товары)
В этой форме релизовать: Добавить все характеристики номенклатуры при выборе номенклатуры


5.1. Решение

5.2. Добавить в форму обработки ПодборТоваровВДокументПродажи кнопку и команду

<details>

<summary>Просмотр кода</summary>

### Код формы

```
&НаСервере
Процедура ПриСозданииНаСервере(Отказ, СтандартнаяОбработка)

...
 	
//1729 Задача 5.1. Программно Добавить Команду и кнопку
// для добавления характеристик при выборе номенклатуры
	Команда = Команды.Добавить(
		"ДобавитьНоменклатуруСХарактеристиками"); //Имя команды
	Команда.Заголовок = "Добавить все Характеристики номенклатуры";
	Команда.Действие  = "др_ДобавитьНоменклатуруСХарактеристикамиПосле"; //Имя связанной процедуры
	 	
	//Добавление кнопки формы
	КнопкаФормы = Элементы.Добавить(
		"др_КнопкаДобавитьНоменклатуруСХарактеристиками", //Имя кнопки
		Тип("КнопкаФормы"),             //Тип, всегда КнопкаФормы
		КоманднаяПанель);                      //Контейнер для кнопки 
		
	КнопкаФормы.ИмяКоманды = "ДобавитьНоменклатуруСХарактеристиками"; //Связь с командой по имени
	
	КнопкаФормы.Вид = ВидКнопкиФормы.ОбычнаяКнопка; 
//1729- Программно Добавить Команду и кнопку
	

	
КонецПроцедуры

```

</details>


5.3. Открыть форму Выбора номенклатуры

<details>

<summary>Просмотр кода</summary>

### Код формы

```

 //1729 Задача 5.1. Добавить все характеристики номенклатуры при выборе номенклатуры
&НаКлиенте
Процедура др_ДобавитьНоменклатуруСХарактеристикамиПосле(Команда)
	ОповещениеОзакрытии = Новый ОписаниеОповещения("др_ДобавитьНоменклатуруСХарактеристикамиЗавершение",ЭтотОбъект);
	параметрыОткрытия = Новый Структура("РежимВыбора,МножественныйВыбор,ЗакрыватьПриВыборе",Истина,Истина,Истина);
//	открыть форму выбора справочника номенклатура
	ОткрытьФорму("Справочник.Номенклатура.ФормаВыбора",параметрыОткрытия,ЭтотОбъект,,,,ОповещениеОзакрытии );
	  
КонецПроцедуры

&НаКлиенте
Процедура др_ДобавитьНоменклатуруСХарактеристикамиЗавершение(РезультатЗакрытия, ДополнительныеПараметры) Экспорт 
    Если ЗначениеЗаполнено(РезультатЗакрытия) Тогда
		др_ДобавитьНоменклатуруСХарактеристикамиЗавершениеСервер(РезультатЗакрытия);
	КонецЕсли;
КонецПроцедуры
 
&НаСервере
Процедура др_ДобавитьНоменклатуруСХарактеристикамиЗавершениеСервер(СписокТоваров)   
	Запрос = Новый Запрос;
	Запрос.Текст = 
	"ВЫБРАТЬ
	|	Номенклатура.Ссылка КАК Номенклатура,
	|	Номенклатура.ВидНоменклатуры КАК ВидНоменклатуры,
	|	Номенклатура.ТипНоменклатуры КАК ТипНоменклатуры
	|ПОМЕСТИТЬ втНоменклатураСВидомНоменклатуры
	|ИЗ
	|	Справочник.Номенклатура КАК Номенклатура
	|ГДЕ
	|	Номенклатура.Ссылка В(&СписокТоваров)
	|;
	|
	|////////////////////////////////////////////////////////////////////////////////
	|ВЫБРАТЬ
	|	втНоменклатураСВидомНоменклатуры.Номенклатура КАК Номенклатура,
	|	ХарактеристикиНоменклатуры.Ссылка КАК Характеристика,
	|	1 КАК Количество,
	|	1 КАК КоличествоУпаковок,
	|	ХарактеристикиНоменклатуры.Ссылка КАК Артикул,
	|	Ложь КАК Обособленно,
	|	Истина КАК ХарактеристикиИспользуются,
	|	втНоменклатураСВидомНоменклатуры.ТипНоменклатуры КАК ТипНоменклатуры,
	|	Ложь КАК ПроизводитсяВПроцессе,
	|	Истина КАК ЗаказатьНаСклад
	|ИЗ
	|	втНоменклатураСВидомНоменклатуры КАК втНоменклатураСВидомНоменклатуры
	|		ВНУТРЕННЕЕ СОЕДИНЕНИЕ Справочник.ХарактеристикиНоменклатуры КАК ХарактеристикиНоменклатуры
	|		ПО (втНоменклатураСВидомНоменклатуры.ВидНоменклатуры = (ВЫРАЗИТЬ(ХарактеристикиНоменклатуры.Владелец КАК Справочник.ВидыНоменклатуры)))
	|ГДЕ
	|	НЕ ХарактеристикиНоменклатуры.ПометкаУдаления
	|
	|УПОРЯДОЧИТЬ ПО
	|	Номенклатура";
	
	Запрос.УстановитьПараметр("СписокТоваров", СписокТоваров);
	
	РезультатЗапроса = Запрос.Выполнить();
	
	ВыборкаДетальныеЗаписи = РезультатЗапроса.Выбрать();
	Пока ВыборкаДетальныеЗаписи.Следующий() Цикл                               
		НоваяСтрока = Объект.Корзина.Добавить();
		ЗаполнитьЗначенияСвойств(НоваяСтрока,ВыборкаДетальныеЗаписи);
		Элементы.Корзина.ТекущаяСтрока = НоваяСтрока.ПолучитьИдентификатор();
	КонецЦикла;

КонецПроцедуры
//-1729 

```


</details>

5.4. Обработать выбор
