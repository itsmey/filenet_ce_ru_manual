# Работа с объектами

Здесь будет рассмотрены действия с такими объектами, как Document, Folder, CustomObject

*Важно: все изменения объектов (создание, обновление, удаление) считаются локальными, пока не будет вызван метод объекта .save(RefreshMode refreshMode), выполняющий коммит изменений CE. Параметр refreshMode определяет, нужно ли обновить свойства объекта с сервера CE после сохранения. Он принимает значения REFRESH или NO_REFRESH. Использовать REFRESH рекомендуется тогда, когда после сохранения объект ещё нужен, т.е. его свойства будут использованы в рамках текущей операции.
Также существует вариант метода save с дополнительным параметром типа PropertyFilter, чтобы обновить не все, а только указанные в фильтре свойства.*

## Создание

Для создания объекта используется метод Factory.XXXX.createInstance().

```java
Document document = Factory.Document.createInstance(objectStore, null);
document.save(RefreshMode.REFRESH);
System.out.println(document.get_ClassDescription().get_SymbolicName() + " : " + document.get_Id());
```

Здесь создаётся объект класса "Document" (не путать с интерфейсом Document API CE). Новому объекту присваивается случайно сгенерированный id. Во второй строчке документ сохраняется с параметром REFRESH. Это означает, что свойства будут получены с сервера и записаны в локальный кэш. Таким оюразом, теперь можно обратиться к свойствам "ClassDescription" и "Id", вызвав соответствующие методы-геттеры.

Предположим, существует определение класса "InheritedDocument", наследующее "Document". Создать объект этого класса:

```java
Document document = Factory.Document.createInstance(objectStore, "InheritedDocument");
```

Допустим, мы хотим создать объект, после чего нам нужен лишь его Id, который мы сохраняем где-то в другом месте, чтобы когда-нибудь потом воспользоваться объектом.

```java
Id id = new Id(UUID.randomUUID().toString());
Document document = Factory.Document.createInstance(objectStore, null, id);
document.save(RefreshMode.NO_REFRESH);
//используем id, но не сам объект
```

В первой строке мы создаём случайный Id, используя java.util.UUID.
Сохранение без обновления свойств говорит о том, что далее в этом коде объект использоваться не будет.

## Получение

Как уже [было сказано](instance_methods.md), методы Factory.XXXXgetInstance() и Factory.XXXX.fetchInstance() отличаются тем, что последний записывает свойства объекта в локальный кэш. Соответственно, получать объект с помощью getInstance() целесообразно тогда, когда не требуется работать со свойствами.

Свойства могут быть извлечены и после получения объекта. Для этого предназначены  методы объекта refresh() или fetchProperties(). Они получают с сервера актуальные значения свойств и обновляют локальный кэш. Разница между методами заключается в том, что fetchProperties() бросает исключение, если объект на сервере был изменён с момента предыдущего обращения (таким образом, fetchProperties() годится только на то, чтобы подгружать свойства, которые не были взяты ранее).

Оба метода имеют перегруженные варианты с аргументом типа PropertyFilter.

Рассмотрим получение объекта по ID:

```java
//1-й способ
Document document1 = Factory.Document.fetchInstance(objectStore, id, null); // PropertyFilter = null
System.out.println(document1.get_Name() + ": " + document1.get_Id());

//2-й способ
Document document2 = Factory.Document.getInstance(objectStore, "InheritedDocument", id);
document2.refresh();
System.out.println(document2.get_Name() + ": " + document2.get_Id());
```

Объекты типа Containable можно получать не только по ID, но и по их расположению относительно корневой папки хранилища. Существуют варианты getInstance() и fetchInstance() с параметром Path типа String.
Пример:

```java
Folder folder = Factory.Folder.fetchInstance(objectStore, "/Configuration/WS_OSMI/settings", null);
```

## Редактирование свойств

Свойства - это метаданные объекта. Свойства бывают нескольких типов, а также обладают мощностью, т.е. свойство может иметь одиночное значение (single-valued) или список значений (multi-valued).

Объект типа Property представляет определённое свойство определённого объекта CE. Свойство имеет имя, по которому происходит доступ к нему, и значение некоторого типа.

Объект типа Properties представляет коллекцию свойств объекта. Следующий код выводит список всех свойств объекта и их значений, приведённых (упрощенно говоря) к типу Object:

```java
Properties properties = document.getProperties();  //получить коллекцию свойств
for(Iterator<Property> iterator = properties.iterator(); iterator.hasNext();) {
    Property property = iterator.next();
    System.out.println(String.format("%s %s", property.getPropertyName(), property.getObjectValue()));
}
```                

Для каждого возможного сочетания "тип-мощность" объекты Property и Properties имеют набор методов, возвращающих значение свойства (аргумент name - имя свойства - имеет тип String):

Тип | Property | Properties
------------ | ------------- | -------------
true/false | getBooleanListValue(), getBooleanValue() | getBooleanListValue(name), getBooleanValue(name)
строка | getStringListValue(), getStringValue() | getStringListValue(name), getStringValue(name)
массив байт | getBinaryListValue(), getBinaryValue() | getBinaryListValue(name), getBinaryValue(name)
дата/время | getDateTimeListValue(), getDateTimeValue() | getDateTimeListValue(name), getDateTimeValue(name)
ID | getIdListValue(), getIdValue() | getIdListValue(name), getIdValue(name)
32-битное целое число | getInteger32ListValue(), getInteger32Value() | getInteger32ListValue(name), getInteger32Value(name)   
64-битное число с плав. точкой | getFloat64ListValue(), getFloat64Value() | getFloat64ListValue(name), getFloat64Value(name)
EngineObject | getEngineObjectValue() | getEngineObjectValue(name)

Возвращаемые значения методов, в основном, соответствуют указанным типам (boolean, String, byte[], Date, Id, int, float, EngineObject).

Методы Property:

Метод | Что делает
------------ | -------------
`java.lang.Object getObjectValue()`|Возвращает значение свойства, приведённое к типу Object
`java.lang.String getPropertyName()` |Возвращает имя свойства
`PropertyState getState() `|Возвращает статус свойства
`boolean isDirty() `|Возвращает, было ли значение свойства изменено с момента последнего сохранения.
`boolean isSettable() `|Возвращает, может ли приложение устанавливать значение свойства.



## Удаление
