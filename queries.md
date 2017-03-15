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

## Синтаксис CE SQL

TODO

## Дополнительная информация

* [Query syntax](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.0/com.ibm.p8.ce.dev.ce.doc/query_sql_syntax_rel_queries.htm)
* [Working with queries](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.ce.doc/query_procedures.htm)
