# Элементы содержимого

Некоторые объекты CE (Document и Annotation) могут обладать "элементами содержимого" - прикреплёнными файлами. Например, к документу может быть прикреплён многостраничный файл pdf, содержащий сканы этого документа, или же несколько одностраничных pdf-файлов (второй вариант предпочтительнее, т.к. может уменьшить объём передаваемого трафика, если нужны только некоторые страницы).

Элемент содержимого представлен базовым интерфейсом [ContentElement](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/ContentElement.html). Он имеет два сабинтерфейса:
* [ContentReference](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/ContentReference.html). Представляет элемент содержимого, находящийся за пределами хранилища объектов и поэтому неподконтрольный CE. Работа с ним происходит через URL - свойство ContentLocation 
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
