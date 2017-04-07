# Запросы SQL

В CE можно искать информацию и извлекать объекты через запросы SQL. Синтаксис CE SQL основан на стандартном SQL-92, но имеет от него некоторые отличия, связанные со спецификой работы с данными CE. 

В запросах происходит работа с типами CE как с таблицами. Названия и типы данных колонок соответствуют определениям свойств. Отдельный инстанс является элементом (строкой) таблицы. 

## Типичный порядок работы с запросами

1. Составление запроса как строки. Для этого предназначен класс SearchSQL
2. Определение области действия запроса - одно или несколько хранилищ объектов. Для этого предназначен класс SearchScope
3. Извлечение и обработка данных. Здесь может быть два пути - извлекать инстансы объектов CE или только значения указанных в запросе свойств. В превом случае используется коллекция IndependentObjectSet, во втором - RepositoryRowSet

## SearchSQL

Класс [SearchSQL](https://www.ibm.com/support/knowledgecenter/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/query/SearchSQL.html) предназначен для построения строки запроса. Работать с ним можно двумя способами. Способы взаимоисключающие, т.е. нужно выбрать какой-то один из них:

* передать строку запроса как параметр конструктора
* использовать конструктор без параметров. Сконструировать запрос с помощью методов-помощников setSelectList(), setWhereClause() и других

Преимущество второго способа в том, что снижается вероятность сделать опечатку. Недостаток в том, что порождается много громоздкого кода. Во всех дальнейших примерах используется первый способ.

## SearchScope

Класс [SearchScope](https://www.ibm.com/support/knowledgecenter/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/query/SearchScope.html) определяет область действия запроса и содержит методы для извлечения данных.

Конструкторы:

* `SearchScope(ObjectStore objectStore)` - для поиска в пределах одного хранилища объектов 
* `SearchScope(ObjectStore[] objectStores, MergeMode mergeMode)` - для поиска в нескольких хранилищах. megreMode определяет параметры слияния, т.е. как поступать в случае одинаковых данных: INTERSECTION - пересечение, UNION - объединение

Методы:

* `IndependentObjectSet fetchObjects(SearchSQL searchSQL, java.lang.Integer pageSize, PropertyFilter filter, java.lang.Boolean continuable)` - извлекает коллекцию объектов CE
* `RepositoryRowSet fetchRows(SearchSQL searchSQL, java.lang.Integer pageSize, PropertyFilter filter, java.lang.Boolean continuable)` - извлекает коллекцию наборов свойств
* `ClassDescriptionSet fetchSearchableClassDescriptions(java.lang.String[] classNames, PropertyFilter filter)` - извлекает коллекцию описаний классов

Параметры:

* searchSQL - объект SearchSQL, представляющий строку запроса
* pageSize - размер страницы, т.е. то, сколько наборов свойств извлекается на страницу. Если null, используется значение по умолчанию для сервера (ServerCacheCofiguration.QueryPageDefaultSize). Если параметр continuable равен false или null, pageSize игнорируется
* filter - объект Propertyfilter, задающий параметры фильтрации и глубину рекурсии (см. ["Фильтры свойств"](property_filters.md)). Если null, в результаты попадают все свойства, указанные в запросе
* continuable - разбивать ли на страницы. Если true, учитывается параметр pageSize. Если false или null, максимальное число записей определяется параметром сервера ServerCacheCofiguration.NonPagedQueryMaxSize или значением TOP из запроса

## IndependentObjectSet

    Метод SearchScope.fetchObjects() возвращает коллекцию объектов CE - IndependentObjectSet. Этот метод используется тогда, когда нужны не только значения свойств, но и сами объекты - например, для их последующего удаления.

Рассмотрим пример кода:

```java
String query = "SELECT TOP 100 Id, ClassDescription FROM Document WHERE IsCurrentVersion = true";

SearchSQL searchSQL = new SearchSQL(query);
SearchScope searchScope = new SearchScope(objectStore);
IndependentObjectSet objects = searchScope.fetchObjects(searchSQL, null, null, true);

Iterator iterator = objects.iterator();

while (iterator.hasNext()) {
    Document document = (Document)iterator.next();

    document.fetchProperties(new String[] {"DocumentTitle"});

    Properties properties = document.getProperties();

    Id id = properties.getIdValue("Id");
    String title = properties.getStringValue("DocumentTitle");
    ClassDescription classDescription = (ClassDescription)properties.getObjectValue("ClassDescription");

    String infoLine = String.format(
        "fetched document: id %s, title %s, class %s", id, title, classDescription.get_SymbolicName());

    System.out.println(infoLine);
}
```

В примере сформирован запрос, извлекающий свойства Id и ClassDescription. При обработке данных в цикле while нам доступен инстанс и мы "довытягиваем" свойство DocumentTitle, используя операцию fetchProperties(). 

## RepositoryRowSet

Метод SearchScope.fetchRow() возвращает коллекцию наборов свойств - RepositoryRowSet. Этот метод предпочтительнее использовать, когда нужны только данные, а сам объект не нужен. Получение набора свойств происходит немного быстрее, чем объекта.

```java
String query = "SELECT TOP 100 Id, DocumentTitle, ClassDescription FROM Document WHERE IsCurrentVersion = true";

SearchSQL searchSQL = new SearchSQL(query);
SearchScope searchScope = new SearchScope(objectStore);
RepositoryRowSet rows = searchScope.fetchRows(searchSQL, null, null, true);

Iterator iterator = rows.iterator();

while (iterator.hasNext()) {
    RepositoryRow row = (RepositoryRow)iterator.next();

    Properties properties = row.getProperties();

    Id id = properties.getIdValue("Id");
    String title = properties.getStringValue("DocumentTitle");
    ClassDescription classDescription = (ClassDescription)properties.getObjectValue("ClassDescription");

    String infoLine = String.format(
        "fetched document: id %s, title %s, class %s", id, title, classDescription.get_SymbolicName());

    System.out.println(infoLine);
}
```

Здесь свойство DocumentTitle вынесено в запрос, потому что в цикле итератора доступны только данные, но не сам объект.

## Классы и свойства, доступные для поиска

В клаузе FROM запроса должен быть идентификатор класса - GUID или имя (свойства Id или SymbolicName объекта ClassDescription). 
Как правило, доступными для поиска являются не все классы CE, а только подтипы одновременно RepositoryObject и IndependentObject.

Узнать символические имена классов, которые могут быть в клаузе FROM, поможет следующий код:

```java
SearchScope searchScope = new SearchScope(objectStore);
ClassDescriptionSet cdSet = searchScope.fetchSearchableClassDescriptions(null, null);

for (Iterator<ClassDescription> i = cdSet.iterator();i.hasNext();) {
    System.out.println(i.next().get_SymbolicName());
}
```

Также следует иметь в виду, что не все свойства можно указывать в клаузах SELECT и WHERE. Это относится, например, к свойству ContentElements (элементы содержимого), т.к. оно может существенно увеличить время выполнения запроса, и к свойствам типа Binary.

## Синтаксис CE SQL

### Простой запрос

Типичный простейший запрос на извлечение определённых данных из некоего класса по заданным условиям:

```sql
SELECT Id, DocumentTitle, DateCreated, ClassDescription 
    FROM Document 
    WHERE IsCurrentVersion = true AND DateCreated > 2017-04-10T00:00:00Z
```

Этот запрос извлекает данные - знчения свойств Id, DocumentTitle, ClassDescription - из тех инстансов класса Document, которые являются текущими версиями и созданы 10 апреля 2017 года или позднее (по времени сервера). Далее, используя методы fetchObjects() или fetchRows(), можно работать с самими инстансами (у которых в локальном кэше будут выбранные свойства) или с наборами свойств соответственно.

### Псевдонимы

Псевдонимы классов могут в некоторых случаях сократить размер строки запроса.

```sql
SELECT d.DocumentTitle, d.DateCreated
    FROM Document d    -- или FROM Document AS d
    WHERE d.IsCurrentVersion = true
```

### Оператор DISTINCT

Позволяет извлечь только отличающиеся данные. Часто свойство имеет одно и то же значение для разных объектов. DISTINCT позволяет извлечь все встречающиеся значения, среди которых не будет повторяющихся.

```sql
    SELECT DISTINCT DocumentTitle FROM Document
```

Данный запрос вернёт все варианты заголовков документов, встречающихся в хранилище. Если зафетчить объекты по этому запросу, то возможно, какой-то объект будет пропущен.

### Оператор ORDER BY

Сортирует результаты запроса. Опциональная клауза ORDER BY ставится после FROM или после WHERE, если она есть.

* ORDER BY <свойство> - по возрастанию значений свойства
* ORDER BY COALESCE(<свойство>, <значение>) - по возрастанию значений свойства. Нулевые значения свойства сортируются так, как если бы они имели значение <значение>  

Сортировка по убыванию значений: `ORDER BY ... DESC`

Пример. Сортировать по заголовку. Документы с одинаковыми заголовками сортировать по дате создания в обратном порядке. Если заголовок не указан, сортировать так, как если бы он был "Не указан":

```sql
    SELECT DocumentTitle, DateCreated FROM Document ORDER BY COALESCE(DocumentTitle, 'Не указан'), DateCreated DESC 
```

Сортировать можно только по упорядчиваемым свойствам. К ним не относятся свойства типа Binary и некоторые свойства типа String.

### Оператор JOIN

### Опрераторы сравнения

### Проверка на null

### Операторы INFOLDER, INSUBFOLDER

* INFOLDER позволяет извлекать Containable-объекты, находящиеся в определённой папке, не учитывая подпапки
* INSUBFOLDER позволяет извлекать Containable-объекты, находящиеся в определённой папке, учитывая подпапки

После ключевого слова может быть полный путь к папке или её Id. Например

```sql
    SELECT This FROM Document WHERE This INFOLDER '/some_folder'
    SELECT This FROM Document WHERE This INSUBFOLDER {4ECDE7D9-F551-4C53-A109-8D81B1DE8577}
```

Из этого примера мы узнаём следующее:
* свойство объектного типа This содержит ссылку на сам объект
* синтаксис GUID в запросах

### IsClass(), IsOfClass(), INCLUDESUBCLASSES, EXCLUDESUBCLASSES

* Функция IsClass(<псевдоним класса>, <id_класса>) позволяет находить инстансы определённого класса:

```sql
    SELECT Id FROM Document D WHERE IsClass(D,DocSubclass1) OR IsClass(D,DocSubclass2)
```

Найти инстансы только подклассов Document, но не самого класса Document:

```sql
    SELECT Id FROM Document D WHERE NOT(IsClass(D, Document))
```

* функция IsOfClass(<псевдоним класса>, <id класса>) позволяет находить инстансы определённого класса и его подклассов

Найти инстансы Correspondence и его подклассов, исключая инстнсты Email и всех его подклассов

```sql
    SELECT Title, Author, MessageText FROM Correspondence c WHERE NOT(IsOfClass(c, Email))
```

* ключевое слово INCLUDESUBCLASSES означает, что в результаты поиска попадут не только инстансы класса, указанного в клаузе FROM, но и инстансы его подклассов. Используется по умолчанию. Следующие запросы эквивалентны:

```sql
    SELECT Id FROM Document WHERE DocumentTitle = 'MyDoc'
    SELECT Id FROM Document WITH INCLUDESUBCLASSES WHERE DocumentTitle = 'MyDoc'
```

* ключевое слово EXCLUDESUBCLASSES означает, что в результаты поиска попадут только инстансы класса, указанного в клаузе FROM. Следующие запросы эквивалентны:

```sql
    SELECT Id FROM Document d WHERE DocumentTitle = 'MyDoc' AND ISCLASS (d, Document)
    SELECT Id FROM Document WITH EXCLUDESUBCLASSES WHERE DocumentTitle = 'MyDoc'
```

### ALLACCESSGRANTED()

Функция ALLACCESSGRANTED(<маска доступа>) позволяет искать объекты, на которые пользователь, выполняющий запрос, имеет как минимум те права, которые заданы маской доступа. 

Например, найти только те инстансты, на которые пользователь имеет, среди прочих, права CHANGE_STATE (1024) и DELETE (65536). Суммарная маска для этих двух прав бутет 65560:

```sql
    SELECT Id FROM MyClass WITH ALLACCESSGRANTED(66560)
```

### OBJECT()

Функция OBLECT(<идентификатор объекта>) позволяет создать как бы объектную константу, которую можно сравнивать с другим объектным значением. В качестве идентификатора может быть:

* для объектов Folder - полный путь: `OBJECT('/full/path/to/folder')`
* свойство типа Id: `OBJECT(d.SomeIdProperty)`
* константа типа Id:  `OBJECT({10F4175A-0000-CB1E-935B-62949727D02A})`

Пример:

```sql
    SELECT FolderName FROM Folder WHERE Parent = OBJECT('/root/sub1/sub1a')
```

### Сопоставление с образцом для строк

Используя LIKE, можно извлекать строковые свойства по шаблону. Например, найти все документы, у которых название начинается с "doc":

```sql
    SELECT Id, DocumentTitle FROM Document WHERE DocumentTitle LIKE 'doc%'
```

### Оператор IN

Предназначен для фильтрации по принадлежности к списку значений. Имеет несколько вариантов использования:

* <значение> IN <свойство мощности LIST>
* <значение> IN <список констант>. Список констант имеет вид `(constant1, constant2, constant3,…)`
* <значение> IN (<подзапрос>). Подзапрос - это SQL-запрос по **одному** свойству. Результаты подзапроса считаются списком, принадлежность к которому проверяется. Исключением является свойство FilterExpression, требующее собственного подхода (см. [официальную документацию](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.0/com.ibm.p8.ce.dev.ce.doc/query_sql_syntax_rel_queries.htm), раздел "IN Operator"

Примеры:

```sql
    SELECT DocumentTitle 
        FROM Document 
        WHERE DocumentTitle IN ('SomeTitle', 'YetAnotherTitle')
    SELECT c.ModelNo, c.Name 
        FROM Car c 
        WHERE c.Manufacturer IN (SELECT cm.This FROM CarManufacturer cm WHERE cm.Name = 'Mercedes-Benz')
```

### Оператор INTERSECTS

Имеет форму `<список 1> INTERSECTS <список 2>` и становится TRUE, если списки имеют хотя бы один общий элемент. К качестве списков могут быть как списки констант, так и свойства мощности LIST.

```sql
    SELECT Name, Country, Genres
        FROM Artist
        WHERE Genres INTERSECTS ('Jazz Rock', 'Blues Rock', 'Rhytm and Blues') 
```

### Функции времени

### Примеры запросов

## Дополнительная информация

* [Query syntax](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.0/com.ibm.p8.ce.dev.ce.doc/query_sql_syntax_rel_queries.htm)
* [Working with queries](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.ce.doc/query_procedures.htm)
