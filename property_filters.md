# Фильтры свойств

Фильтры свойств - мощный и гибкий способ управления тем, какие свойства и из каких объектов нужно извлечь при обращении к хранилищу объектов. Технически этот механизм реализован через методы с аргументами [PropertyFilter](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/PropertyFilter.html):

* `IndependentlyPersistableObject.save(RefreshMode refreshMode, PropertyFilter filter)`
* `IndependentObject.refresh(PropertyFilter filter)`
* `IndependentObject.fetchProperty(java.lang.String propertyName, PropertyFilter filter)`
* `IndependentObject.fetchProperties(PropertyFilter filter)`
* `Factory.XXXX.fetchInstance(...)` - множество разновидностей

Полный список можно посмотреть [здесь](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/class-use/PropertyFilter.html).

Фильтры свойств улучшают быстродействие двумя способами:
* уменьшив объём данных (количество извлекаемых свойств)
* применив рекурсивный способ получения свойств - происходит обработка не только базового объекта, но и тех объектов, которые являются значениями его свойств. Это сокращает число запросов к БД

Типичный алгоритм использования фильтра:

1. создать объект PropertyFilter. Так как PropertyFilter - класс, это делается не через фабрику, а с помощью ключевого слова new.
2. добавить в фильтр включаемые и/или исключаемые свойства методами addIncludeProperty() или addExcludeProperty()
3. передать объект PropertyFilter в один из методов fetchInstance(), fetchPrpperties() или других

В самом простом варианте фильтр работает так же, как метод fetchProperties() с аргументом типа массив строк. Код

```java
PropertyFilter filter = new PropertyFilter();
filter.addIncludeProperty(new FilterElement(null, null, null, "DocumentTitle DateCreated", null));
document.save(RefreshMode.REFRESH, filter);
```

эквивалентен коду

```java
document.fetchProperties(new String[] {"DocumentTitle", "DateCreated"});
document.save(RefreshMode.REFRESH, filter);
```

Основные методы PropertyFilter:

метод | что делает
------------ | -------------
`void addExcludeProperty(java.lang.String value)`|Добавить имена свойств (разделяются пробелами), которые не будут извлечены. Если задано одно и то же свойство для включения и исключения, то исключение имеет приоритет
`void addIncludeProperty(FilterElement fe)`|Добавить свойства для включения. См. [FilterElement](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/FilterElement.html)
`void addIncludeType(FilterElement fe)`|Добавить свойства определённого типа. См. вариант конструктора FIlterElement с параметром FilteredPropertyType
`void setLevelDependents(boolean levelDependents)`|Установить глобальное значение параметра levelDependents (глобальное значение используется, когда соответствующее значение не задано (null) в конструкторе FilterElement)
`void setMaxRecursion(int maxRecursion)`|Установить глобальное значение параметра maxRecursion (глобальное значение используется, когда соответствующее значение не задано (null) в конструкторе FilterElement)
`void setMaxSize(java.lang.Long maxSize)`|Установить глобальное значение параметра maxSize (глобальное значение используется, когда соответствующее значение не задано (null) в конструкторе FilterElement)
`void setPageSize(int pageSize)`|Установить глобальное значение параметра pageSize (глобальное значение используется, когда соответствующее значение не задано (null) в конструкторе FilterElement)

Класс [FilterElement](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/FilterElement.html) содержит информацию о том, что нужно включить в фильтр. Интересны только конструкторы:

* `FilterElement(java.lang.Integer maxRecursion,java.lang.Long maxSize,java.lang.Boolean levelDependents,FilteredPropertyType value,java.lang.Integer pageSize)` - извлечь свойства определённого типа. Тип указывается параметром [FilteredPropertyType](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/constants/FilteredPropertyType.html)
* FilterElement(java.lang.Integer maxRecursion,java.lang.Long maxSize,java.lang.Boolean levelDependents,java.lang.String value,java.lang.Integer pageSize) - извлечь свойства, имена которых указаны в параметре value. Если свойств несколько, имена разделяются пробелом

Параметры, управляющие глубиной фильтрации:

* **levelDependents**: A Boolean that specifies whether the recursion level to use when retrieving a dependent object is the same as that of the independent object to which it belongs (true) or one level deeper (false). 
* **maxRecursion**: A zero-based Integer that specifies the maximum allowable recursion depth to use when retrieving property relationships. This attribute determines the level of depth at which property values are included. If unspecified, the default is zero.
* **maxSize**: A Long that specifies the maximum size, in bytes, of content data that can be returned when properties that hold content data are retrieved. If the amount of content data held by retrieved properties exceeds this size, no content data is returned. If unspecified, the default is to return all content data, no matter how large. 
* **pageSize**: An Integer that specifies the iterator page size for independent object sets returned by PropertyIndependentObjectSet properties. The iterator page size determines how many elements of an independent object set are retrieved during each fetch. If the page size is unspecified, by default the server uses the value of the QueryPageDefaultSize property of the ServerCacheConfiguration object (the default for this property is 500). If the page size exceeds the value of the QueryPageMaxSize property of the ServerCacheConfiguration object (the default for this property is 1000), the server uses the value of the QueryPageMaxSize property instead.


