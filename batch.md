# Пакетные операции

В CE есть классы [UpdatingBatch](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/UpdatingBatch.html) и [RetrievingBatch](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/RetrievingBatch.html). Они служат для пакетного изменения или получения объектов. Эти классы имеют общего родителя Batch.

"Пакетный" означает, что действия над группой объектов выполняются единовременно, в одной точке кода (подразумевается код верхнего уровня).

С помощью пакетных операций можно значительно улучшить производительность. Пакетные операции рекомендуется применять, если бизнес-логика допускает разбиение на *независимые* операции над объектами, т.е. внутри "пакета" каждая операция никак не зависит от состояний других объектов и результатов прочих операций.

* Пакетное изменение транзакционно. Это значит, что либо все операции будут успешными, либо никаких изменений не будет сделано вообще
* Пакетное получение не транзакционно. Это значит, что для каждого объекта может быть свой результат: успешное обновление свойств либо какое-то исключение

## Пакетное изменение

Рассмотрим методы UpdatingBatch:

метод | что делает
------------ | -------------
`BatchItemHandle add(IndependentlyPersistableObject object, PropertyFilter filter)`|Добавляет объект в пакет. Параметр filter определяет, какие свойства нужно извлечь. Метод возвращает элемент пакета ([BatchItemHandle](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/BatchItemHandle.html)). Имея элемент пакета, можно узнать исключение, которое произошло при работе с данным объектом (актуально для RetrievalBatch)
`static UpdatingBatch createUpdatingBatchInstance(Domain domain, RefreshMode refresh)`|Фабричный метод пакета изменений. Параметр refresh может принимать значения RefreshMode.REFRESH и RefreshMode.NO_REFRESH. Он определяет, будут ли свойства в локальном кэше переписаны актуальными после завершения операции
`boolean hasPendingExecute()`|Возвращает, есть ли в пакете элементы, ожидающие изменения
`void updateBatch()`|Запустить пакетную операцию

Порядок действий:

1. Создать или получить объекты, которые нужно обновить
2. Изменить эти объекты, не вызывая метод save()
3. Создать объект updatingBatch  - UpdatingBatch.createUpdatingBatchInstance()
4. Добавить изменённые объекты в пакет - add()
5. Запустить пакетное обновление - updateBatch()

### Пример

```java
// для создания пакета нужно знать домен
Domain myDomain = objectStore.get_Domain();

// создаём пакет
UpdatingBatch ub = UpdatingBatch.createUpdatingBatchInstance(myDomain, RefreshMode.REFRESH);

// получаем объекты
Document doc1 = Factory.Document.fetchInstance(objectStore, new Id("{F905DBD6-5A69-4252-9985-2D3DD28D7FBA}"), null);
Document doc2 = Factory.Document.fetchInstance(objectStore, new Id("{35026B90-B443-40CA-B5C3-66BEAD13E2B7}"), null);

// как-нибудь работаем с объектами
doc1.getProperties().putValue("DocumentTitle", "doc1"); 
doc2.getProperties().putValue("DocumentTitle", "doc2"); 

// добавляем в пакет
ub.add(doc1, null);  
ub.add(doc2, null);  

// запускаем операцию
ub.updateBatch();
```

## Пакетное получение
