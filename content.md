# Элементы содержимого

Некоторые объекты CE (Document и Annotation) могут обладать "элементами содержимого" - прикреплёнными файлами. Например, к документу может быть прикреплён многостраничный файл pdf, содержащий сканы этого документа, или же несколько одностраничных pdf-файлов (второй вариант предпочтительнее, т.к. может уменьшить объём передаваемого трафика, если нужны только некоторые страницы).

Элемент содержимого представлен базовым интерфейсом [ContentElement](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/ContentElement.html). Он имеет два сабинтерфейса:
* [ContentReference](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/ContentReference.html). Представляет элемент содержимого, находящийся за пределами хранилища объектов и поэтому неподконтрольный CE. Работа с ним происходит через URL (свойство ContentLocation)
* [ContentTransfer](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/ContentTransfer.html). Представляет элемент содержимого в хранилище объектов
 
Элемент ContentTransfer имеет следующие метаданные:

* размер содержимого в байтах (content size)
* MIME-тип содержимого (content type)
* имя файла (retrieval name)
* номер элемента в последовательности элементов содержимого (sequence number)

Рассмотрим методы ContentTransfer:

метод | что делает
------------ | -------------
`void set_ContentType(java.lang.String value)`|Указать тип содержимого. Для элемента ContentTransfer CE пытается определить тип автоматически по содержимому, но рекомендуется указывать тип вручную
`java.lang.String get_ContentType()`|Получить тип содержимого
`java.lang.Integer get_ElementSequenceNumber()`|Получить номер элемента в последовательности
`java.io.InputStream accessContentStream()`|Получить поток для доступа к данным
`java.lang.Double get_ContentSize()`|Получить размер данных
`java.lang.String get_RetrievalName()`|Получить имя файла
`void set_RetrievalName(java.lang.String value)`|Указать имя файла
`void setCaptureSource(java.io.InputStream source)`|Указать поток - источник данных

## Работа с содержимым

Рассмотрим два способа получения объекта потока данных содержимого

### Через метод содержащего объекта

Объекты, обладающие содержимым, имеют метод 

`java.io.InputStream accessContentStream(int element)` - получить объект InputStream для доступа к данным, где element - номер элемента в последовательности эл-тов содержимого

Далее работаем средствами Java:

```java
InputStream stream = document.accessContentStream(0);
BufferedReader reader = new BufferedReader(new InputStreamReader(stream));

String text = "", line;

while ((line = reader.readLine()) != null)
  text += line;
  
reader.close();
```

### Через свойство ContentElements

Свойство ContentElements содержит список элементов содержимого, будь то ContentReference или ContentTransfer. Использование ContentElements 

* даёт доступ к метаданным на чтение и запись
* позволяет обойти все элементы с помощью итератора 
* позволяет не только читать, но и записать данные содержимого

Общий порядок действий такой:

1. Получить документ (или аннотацию)
2. Получить свойство ContentElements - список элементов содержимого (ContentElementList)
3. Получить нужный элемент из списка методом get() либо использовать итератор для обхода списка
4. Работать с элементом, приведя его к нужному типу (ContentReference или ContentTransfer)

Пример работы с содержимым через ContentElements:

```java
public static void main(String[] args)
{
    /*
      Соединение с CE для краткости опущено
    */

    // фильтр свойств
    PropertyFilter filter = new PropertyFilter();
    filter.addIncludeProperty(new FilterElement(null, null, null, "ContentElements", null));

    // 1. запись содержимого в файл
    // получаем документ
    Document d = Factory.Document.fetchInstance(objectStore, "/path/to/document1", filter);

    // получаем список элементов содержимого
    ContentElementList elements = d.get_ContentElements();

    // пусть нам нужен первый в списке элемент
    ContentTransfer element = (ContentTransfer)elements.get(0);

    // получаем имя файла
    String filename = element.get_RetrievalName();

    // получаем поток данных
    InputStream stream = element.accessContentStream();

    // записываем данные в файл
    Double size = writeContent(stream, filename);

    // проверяем, что было записано столько же байт, сколько составляет размер данных
    Double expected = element.get_ContentSize()
    if( size != expected )
        System.err.println("Invalid content size retrieved");

    // 2. пример работы с итератором
    // получаем другой документ и записываем все его элементы содержимого в файлы  
    d = Factory.Document.fetchInstance(objectStore, "/path/to/document2", filter);
    elements = d.get_ContentElements();
    for( Iterator i = elements.iterator(); i.hasNext(); ) {
        element = (ContentTransfer)i.next();
        writeContent(element.accessContentStream(),"element" + element.get_ElementSequenceNumber());
    }
    
    // 3. пример создания нового содержимого
    // получаем третий документ и прикрепляем к нему данные первого документа
    d = Factory.Document.fetchInstance(objectStore, "/path/to/document3", filter);
    try {
       InputStream is = new FileInputStream(filename);
       elements = d.get_ContentElements();
      
       // создаём новый элемент ContentTransfer
       element = Factory.ContentTransfer.createInstance(objectStore);
       element.setCaptureSource(is);
       element.setRetrievalName(filename);
      
       if (elements == null) {
           // если содержимого нет, создаём список содержимого
           elements = Factory.ContentElement.createList();
       } 
       
       // добавляем элемент в список
       elements.add(element)
       
       d.set_ContentElements(elements);
       d.save(RefreshMode.REFRESH);
   } catch (IOException e) {
       e.printStackTrace();
   }
}

public static Double writeContent(InputStream s, String filename) throws IOException {
    BufferedOutputStream writer = new BufferedOutputStream(new FileOutputStream(filename));
    Double size = new Double(0);
    int bufferSize;
    byte[] buffer = new byte[1024];

    while( ( bufferSize = s.read(buffer) ) != -1 ) {
        size += bufferSize;
        writer.write(buffer, 0, bufferSize);
    }
    writer.close();
    s.close();
    return size;
}
```
