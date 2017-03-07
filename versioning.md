# Версии

В CE реализовано версионирование документов. Основные идеи:

* каждый документ имеет номер старшей (major) и младшей (minor) версий. "Общая" версия состоит из старшей и младшей (далее пишем через точку: major.minor)
* только что созданный документ имеет версию 0.1
* старшая версия обычно инкрементируется при "выпуске" (release) документа. Обычно под выпуском подразумевается, что документ становится доступен широкой публике
* младшая версия обычно инкрементируется при изменении документа между выпусками, что может означать некие промежуточные изменения при подготовке к очередному выпуску
* каждая версия документа - это самостоятельный документ со своими метаданными, правами доступа, содержимым и т.д.
* *набором версий (version series)* называется список документов, составляющих одну "цепочку изменений". Самое последнее звено в этой цепочке называется *текущей версией (current version)*

## Состояние версии

Каждая версия находится в одном из 4-х состояний:

* **Выпущенная (released)**. Это последний выпуск документа, т.е. последняя версия вида X.0
* **Промежуточная (in-process)**. Это последняя версия вида X.Y, где Y не равен 0
* **Зарезервированная (reservation)**. Специальная версия, которая создаётся в результате чекаута
* **Замещённая (superseded)**. Любая версия, заменённая другой, более новой

В наборе версий может быть только одна выпущенная и только одна промежуточная версия.

Важно: "текущая" не является состоянием. Текущей версией может быть выпущенная или промежуточная версии.

## Чекаут, чекин

Основной способ создания новой версии - процедура check out/check in 

**Check out** - создание зарезервированной версии из текущей

* можно делать чекаут только текущей (выпущенной или промежуточной) версиии
* нельзя сделать чекаут одной и той же версии дважды
* в результате создаётся новая зарезервированная версия с номером младшей версии, увеличенным на 1

**Check in** - создание новой текущей версии из зарезервированной

* зарезервированная версия становится текущей версией
* та версия, которая была текущей, становится замещённой
* существует два типа чекина: младший и старший. При младшем типе зарезервированная версия становится промежуточной, номер не меняется. При старшем типе номер меняется с X.Y до (X+1).0, зарезервированная версия становится выпущенной

Типичный порядок действий:

1. Чекаут текущей версии
2. Работа с зарезервированной версией
3. Чекин зарезервированной версии

## Пример

Действие | Версия 1 | Версия 2 | Версия 3
------------ | ------------- | ------------- | -------------
Создание документа, внесение изменений, чекин (minor) | 0.1 **Промежуточная** | |
Чекаут, внесение изменений | 0.1 **Промежуточная** | 0.2 **Зарезервированная** | 
Чекин (minor)| 0.1 **Замещённая** |  0.2 **Промежуточная** |
Чекаут, внесение изменений| 0.1 **Замещённая** | 0.2 **Промежуточная**|0.3 **Зарезервированная**
Чекин (major)| 0.1 **Замещённая** | 0.2 **Замещённая**| 1.0 **Выпущенная**

## Реализация в CE

Технически работа с версиями представлена
* методом checkin() интерфейса Document
* методами интерфейса [Versionable](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Versionable.html). Versionable является суперинтерфейсом для Document, поэтому эти методы доступны в любом документе
* методами интерфейса [VersionSeries](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/VersionSeries.html). Чтобы получить объект VersionSeries, можно воспользоваться методом get_VersionSeries() документа или методами фабричного класса Factory.VersionSeries

Далее рассмотрим некоторые методы.

### checkin()

Метод | Что делает
------------ | -------------
`void checkin(AutoClassify autoClassify, CheckinType checkinType)`|Сделать чекин зарезервированной версии. Параметр checkinType определяет тип чекина: MINOR_VERSION - младший, MAJOR_VERSION - старший. Параметр autoClassify определяет, необходимо ли [автоматически классифицировать](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.0/com.ibm.p8.ce.admin.tasks.doc/autodocclassification/adc_understanding_auto_doc_classification.htm) документ по вложениям. Возможные значения: AUTO_CLASSIFY, DO_NOT_AUTO_CLASSIFY

Если версия документа не зарезервирована, бросается исключение E_NOT_SUPPORTED.

### Versionable

Метод | Что делает
------------ | -------------
`Versionable cancelCheckout()`|Отмена чекаута. Зарезервированный объект удаляется, текущая версия становится доступной для чекаута. Метод выполняется для текущего объекта и возвращает зарезервированный объект, ожидающий удаления. Для окончательно удаления нужно вызвать метод save() этого объекта
`void checkout(ReservationType type, Id reservationId, java.lang.String reservationClass, Properties reservationProperties)`|Выполннить чекаут, т.е. создать зарезервированную версию объекта. reservationType - тип резервации: COLLABORATIVE - пользователи с соотв. правами могут работать с зарез-й версией, EXCLUSIVE - только данный пользователь работает с версией, OBJECT_STORE_DEFAULT - тип, принятый в хранилище (настраивается методом set_DefaultReservationType() интерфейса ObjectStore). reservationId - ID зарез-й версии, null для автомат. генерации. reservationClass - класс заерз-й версии или null, чтобы оставить текущий класс. reservationProperties - коллекция свойств для присвоения новому объекту
`void demoteVersion()`|Понижает версию с состояния выпущенной до промежуточной. Соответственно меняется номер, например, если было продвижение 1.1 -> 2.0, то после понижения 2.0 станет 1.2
`void freeze()`|Заблокировать версию от любых изменений. Это действие невозможно отменить, но можно сделать чекаут
`Versionable get_CurrentVersion()`|Возвращает текущую версию того набора версий, к которому принадлежит объект
`java.util.Date get_DateCheckedIn()`|Возвращает дату чекина
`FolderSet get_FoldersFiledIn()`|Возвращает коллекцию папок (объектов Folder), содержащих объект
`java.lang.Boolean get_IsCurrentVersion()`|Возвращает, является ли данная версия текущей
`java.lang.Boolean get_IsFrozenVersion()`|Возвращает, является ли данная версия заблокированной
`java.lang.Boolean get_IsReserved()`|Возвращает, был ли сделан чекаут данной версии, т.е. существует ли у даннной версии зарезервированная. Свойство IsReserved принимает значение true после чекаута. Оно не позволяет делать чекаут больше одного раза
`java.lang.Boolean get_IsVersioningEnabled()`|Возвращает, можно ли создавать версии этого объекта
`java.lang.Integer get_MajorVersionNumber()`|Возвращает номер старшей версии
`java.lang.Integer get_MinorVersionNumber()`|Возвращает номер младшей версии
`IndependentObject get_Reservation()`|Возвращает зарезервированную версию. Используется после чекаута
`ReservationType get_ReservationType()`|Возвращает тип резервации (см. параметр reservationType в методе checkout()
`Folder get_SecurityFolder()`|Возвращает папку, от которой объект наследует права доступа
`VersionableSet get_Versions()`|Возвращает коллекцию версий набора, к которому принадлежит объект
`VersionSeries get_VersionSeries()`|Возвращает набор версий, к которому принадлежит объект
`VersionStatus get_VersionStatus()`|Возвращает состояние версии. IN_PROCESS - промежуточная, RESERVATION - зарезервированная, RELEASED - выпущенная, SUPERSEDED - замещенная
`void promoteVersion()`|Повышает промежуточную версию до выпущенной. Также меняется номер, например, 1.2 после повышения станет 2.0
`void set_DateCheckedIn(java.util.Date value)`|Устанавливает дату чекина
`void set_SecurityFolder(Folder value)`|Устанавливает папку, от которой объект наследует права доступа

### VersionSeries

Метод | Что делает
------------ | -------------
`Versionable cancelCheckout()`|Отмена чекаута текущей версии. См. Versionable
`void checkout(ReservationType type, Id reservationId, java.lang.String reservationClass, Properties reservationProperties)`|Чекаут текущей версии. См. Versionable
`Versionable get_CurrentVersion()`|Возвращает текущую версию
`java.lang.Boolean get_IsReserved()`|Возвращает, был ли сделан чекаут текущей версии
`java.lang.Boolean get_IsVersioningEnabled()`|Возвращает, можно ли создавать новые версии
`Versionable get_ReleasedVersion()`|Возвращает выпущенную версию
`IndependentObject get_Reservation()`|Возвращает зарезервированную версию
`VersionableSet get_Versions()`|Возвращает коллекцию версий

### Примеры кода

#### Создание версии

```java
// 1. С помощью Versionable
Document doc = Factory.Document.getInstance(os, ClassNames.DOCUMENT, new Id("{D6DCDE1F-EF67-4A2E-9CDB-391999BCE8E5}") );

// делаем чекаут
doc.checkout(ReservationType.OBJECT_STORE_DEFAULT, null, null, null);
// зараезервир-я версия не создана, пока изменения не будут закоммичены на сервер. делаем коммит
doc.save(RefreshMode.REFRESH);          

// получаем зарезервированную версию
Document reservation = (Document) doc.get_Reservation();

/* здесь работаем с reservation
   ...
*/

// делаем чекин младшего типа без автоклассификации
reservation.checkin(AutoClassify.DO_NOT_AUTO_CLASSIFY, CheckinType.MINOR_VERSION);
// после сохранения reservation указывает на текущую версию
reservation.save(RefreshMode.REFRESH);

// 2. С помощью VersionSeries
// получаем набор версий
VersionSeries verSeries = doc.get_VersionSeries();

// делаем чекаут и сохраняем. эти операции для набора эквивалентны операциям на текущей версии
verSeries.checkout(ReservationType.OBJECT_STORE_DEFAULT, null, null, null);
verSeries.save(RefreshMode.REFRESH);

// получаем зарезервированную версию
reservation = (Document) verSeries.get_Reservation();

/* здесь работаем с reservation
   ...
*/
 
// делаем чекин старшего типа без автоклассификации
reservation.checkin(AutoClassify.DO_NOT_AUTO_CLASSIFY, CheckinType.MAJOR_VERSION);
reservation.save(RefreshMode.REFRESH);
 
// получаем текущую версию. version и reservation ссылаются на один и тот же объект CE
version = verSeries.get_CurrentVersion();
System.out.println("Состояние: " + version.get_VersionStatus().toString() +
   "\n Номер: " + version.get_MajorVersionNumber() +"."+ version.get_MinorVersionNumber() );
```

#### Удаление

В данном примере удаляются те версии в наборе, у которых старший номер равен 3. Также демонстрируется работа с набором версий при помощи итератора.

```java
static PropertyFilter pf = new PropertyFilter();  
pf.addIncludeProperty(new FilterElement(null, null, null, "VersionSeries Id", null) );
Document doc = Factory.Document.fetchInstance(os, new Id("{91CD21FA-5F65-4D5B-AE3E-ECE529C7AC88}"), pf);   

// получаем коллекцию версий
VersionSeries vs = doc.get_VersionSeries();
VersionableSet vss = vs.get_Versions();

// обходим коллекцию версий
Iterator vssiter = vss.iterator();
while (vssiter.hasNext()) {
   Versionable ver = (Versionable)vssiter.next();
   System.out.println("Major = " + ver.get_MajorVersionNumber() + "; Minor = " + ver.get_MinorVersionNumber());
   if (ver.get_MajorVersionNumber().intValue() == 3){
      // приводим к типу Document, чтобы иметь возможность вызвать методы delete() и save()
      Document verdoc = (Document) ver;
      verdoc.delete();
      verdoc.save(RefreshMode.REFRESH);
   }
}
```

При удалении объекта VersionSeries удаляются также все версии, которые содержатся в наборе:

```java
PropertyFilter pf = new PropertyFilter();
pf.addIncludeProperty(new FilterElement(null, null, null, PropertyNames.VERSION_SERIES, null)); 
Document doc = Factory.Document.fetchInstance(os, "{3488C44F-D4BB-455F-AEED-553E9EADCC4E}", pf );

VersionSeries verSeries = doc.get_VersionSeries();

verSeries.delete();
verSeries.save(RefreshMode.REFRESH);
```

#### Отмена чекаута

Если к моменту вызова метода cancelCheckout() чекаута не было, бросается исключение API_NOT_A_RESERVATION.

Важно! Метод нужно вызывать не у зарезервированного объекта, а у того, который резервируется (т.е. у текущей версии). Далее вызывается save() зарезервированной версии.

```java
Document doc = Factory.Document.getInstance(os, ClassNames.DOCUMENT, 
      new Id("{D6DCDE1F-EF67-4A2E-9CDB-391999BCE8E5}") );
doc.checkout(ReservationType.EXCLUSIVE, null, null, null);
doc.save(RefreshMode.REFRESH);

Document reservation = (Document) doc.get_Reservation();

doc.cancelCheckout(); 
reservation.save(RefreshMode.REFRESH);
```

#### Повышение / понижение

Методы promoteVersion() и demoteVersion() позволяют изменить состояние и номер версии, минуя чекаут/чекин и не трогая свойства, не касающиеся версионности. Эти методы генерируют исключение E_NOT_SUPPORTED, если не выполняется ряд условий - версия должна быть текущей и иметь соответствующее состояние, зарезервированная версия не должна существовать, пользователь должен обладать необходимыми правами доступа.

```java
Document doc = Factory.Document.getInstance(os, ClassNames.DOCUMENT, new Id("{9B289B8A-DDD9-42DC-9D51-6B485509B68A}") );
doc.promoteVersion(); // или demoteVersion()
doc.save(RefreshMode.REFRESH);
```

#### Блокирование свойств

Пример применения метода freeze().

```java
Document doc = Factory.Document.getInstance(os, ClassNames.DOCUMENT, new Id("{9B289B8A-DDD9-42DC-9D51-6B485509B68A}"));
doc.freeze();
doc.save(RefreshMode.REFRESH);
```

Теперь свойства "заморожены". Единственный способ изменить объект - создать новую версию.

## Дополнительные материалы

* [Примеры кода](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.ce.doc/version_procedures.htm)
* [Versioning](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.sysoverview.doc/p8sov009.htm)
* [Versioning properties](https://www.ibm.com/support/knowledgecenter/SSGLW6_5.2.0/com.ibm.p8.ce.admin.tasks.doc/docsandfolders/df_versioning_properties.htm)
* [Versioning actions](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.admin.tasks.doc/docsandfolders/df_versioning_actions.htm)

