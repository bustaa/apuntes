# MongoDB + Java

## Dependencias

- Maven:

```xml
    <dependencies>
        <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongodb-driver-bom</artifactId>
            <version>5.6.2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        <dependency>
            <groupId>org.mongodb</groupId>
            <artifactId>mongodb-driver-sync</artifactId>
            <version>5.6.2</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
```

## Conexión

```java
    import com.mongodb.client.MongoClient;
    import com.mongodb.client.MongoClients;
    import com.mongodb.client.MongoDatabase;

    try (MongoClient mongoClient = MongoClients.create("mongodb://localhost:27017/")) {
        MongoDatabase db = mongoClient.getDatabase("Main");
        //
    }
```

## CRUD

- Insertar documento
```java
    MongoCollection<Document> col = db.getCollection("canciones");
    Document doc = new Document("titulo", "Imagine")
                        .append("autor", "John Lennon")
                        .append("anyo", 1971);
    col.insertOne(doc);
```

- Insertar varios
```java
    col.insertMany(List.of(doc1, doc2, doc3));
```

- Leer/find
```java
    FindIterable<Document> it = col.find(eq("anyo", 1971));
    for (Document d : it) {
        System.out.println(d.toJson());
    }
```

- Actualizar
```java
    col.updateOne(eq("titulo", "Imagine"), set("formato", "MP3"));
    col.updateMany(eq("formato", "WAV"), set("formato", "OGG"));
```

- Borrar
```java
    col.deleteOne(eq("titulo", "Imagine"));
    col.deleteMany(eq("formato", "OGG"));
``` 

## Insertar datos desde JSON

- JSON normal:
```java
    String json = Files.readString(Path.of("personas.json"));

    List<Document> docs = Document.parse(json);

    col.insertOne(doc);
```

- JSON con array de objetos:
```java
    String json = Files.readString(Path.of("personas.json"));

    Document wrapper = Document.parse("{ \"data\": " + json + " }");

    var docs = wrapper.getList("data", Document.class);

    col.insertMany(docs);
```

- JSON insertar con filtro:
```java
    String json = Files.readString(Path.of("data.json"));

    Document original = Document.parse(json);

    Document filtrado = new Document()
            .append("nombre", original.getString("nombre"))
            .append("edad", original.getInteger("edad"))
            .append("ciudad", original.getString("ciudad"));

    collection.insertOne(filtrado);
```

- JSON con Arrays insertar con filtro:
```java
    Document wrapper = Document.parse("{ \"data\": " + json + " }");
    List<Document> originales = wrapper.getList("data", Document.class);

    List<Document> filtrados = new ArrayList<>();

    for (Document d : originales) {
        Document f = new Document();
        f.append("nombre", d.getString("nombre"));
        f.append("edad", d.getInteger("edad"));
        filtrados.add(f);
    }

    collection.insertMany(filtrados);
```

## Filtros

### Filtros de comparación:
- `eq()`: vale igual que a un valor especifico
- `gt()`: vale mayor que a un valor especifico
- `gte()`: vale mayor que o igual a un valor especifico
- `lt()`: vale menos que un valor especifico
- `lte()`: vale menos que o igual a un valor especifico
- `ne()`: vale distinto a un valor especifico
- `in()`: cualquiera de los valores especificados en un array
- `nin()`: ninguno de los valores especificados en un array
- `empty()`: todos los documentos

---
Ejemplos:

`eq()`:
```java
Bson equalComparison = eq("qty", 5);
collection.find(equalComparison).forEach(doc -> System.out.println(doc.toJson()));
```
Salida:
```java
{ "_id": 1, "color": "red", "qty": 5, "vendor": ["A"] }
{ "_id": 6, "color": "pink", "qty": 5, "vendor": ["C"] }
```
---
`gte()`:
```java
Bson gteComparison = gte("qty", 10);
collection.find(gteComparison).forEach(doc -> System.out.println(doc.toJson()));
```

Salida:
```java
{ "_id": 2, "color": "purple", "qty": 10, "vendor": ["C", "D"] }
{ "_id": 5, "color": "yellow", "qty": 11, "vendor": ["A", "B"] }
```
---
`empty()`:
```java
Bson emptyComparison = empty();
collection.find(emptyComparison).forEach(doc -> System.out.println(doc.toJson()));
```

Salida (todos los documentos en la colección):
```java
{ "_id": 1, "color": "red", "qty": 5, "vendor": ["A"] }
{ "_id": 2, "color": "purple", "qty": 10, "vendor": ["C", "D"] }
{ "_id": 3, "color": "blue", "qty": 8, "vendor": ["B", "A"] }
...
```
---

### Filtros logicos

- `and()`: documentos con las condiciones de todos los filtros. (AND)
- `or()`: documentos con las condiciones de cualquiera de los filtros. (OR)
- `not()`: documentos que no coincide con el filtro.
- `nor()`: documentos que no coincide con los dos filtros. (NOR)

Ejemplo:
```java
Bson orComparison = or(gt("qty", 8), eq("color", "pink"));
collection.find(orComparison).forEach(doc -> System.out.println(doc.toJson()));
```

Salida:
```java
{ "_id": 2, "color": "purple", "qty": 10, "vendor": ["C", "D"] }
{ "_id": 5, "color": "yellow", "qty": 11, "vendor": ["A", "B"] }
{ "_id": 6, "color": "pink", "qty": 5, "vendor": ["C"] }
```
---

### Filtros Arrays

- `all()`: si el  array tiene todos los elemntos especificados en la query.
- `elemMatch()`: si un elemnto del array coincide con el campo especificado.
- `size()`: si el array tiene un numero especificado de elementos.

Ejemplo:
```java
List<String> search = Arrays.asList("A", "D");
Bson allComparison = all("vendor", search);
collection.find(allComparison).forEach(doc -> System.out.println(doc.toJson()));
```

Salida:
```java
{ "_id": 8, "color": "orange", "qty": 7, "vendor": ["A", "D"] }
```
---

### Filtros elementos

- `exists()`: documentos que tienen el campo especificado.
- `type()`: si un campo tiene el tipo especificado.

Ejemplo:
```java
Bson existsComparison = and(exists("qty"), nin("qty", 5, 8));
collection.find(existsComparison).forEach(doc -> System.out.println(doc.toJson()));
```

Salida:
```java
{ "_id": 2, "color": "purple", "qty": 10, "vendor": ["C", "D"] }
{ "_id": 4, "color": "white", "qty": 6, "vendor": ["D"]}
{ "_id": 5, "color": "yellow", "qty": 11, "vendor": ["A", "B"] }
{ "_id": 8, "color": "orange", "qty": 7, "vendor": ["A", "D"] }
```
---

### Filtros evaluación

- `mod()`: cuando un modulo operacional en un campo produce un valor especificado.
- `regex()`: cuando un valor contiene un expresión regular especificada.
- `text()`: cuando contiene una expresión de texto especificada.
- `where()`: contine una expresión JavaScript especificada.

Ejemplo:
```java
Bson regexComparison = regex("color", "^p");
collection.find(regexComparison).forEach(doc -> System.out.println(doc.toJson()));
```

Salida:
```java
{ "_id": 2, "color": "purple", "qty": 10, "vendor": ["C", "D"] }
{ "_id": 6, "color": "pink", "qty": 5, "vendor": ["C"] }
```
---

### Filtros bitwise

- `bitsAllSet()`: cuando se establecen los bits especificados en un campo.
- `bitsAllClear()`: cuando los bits especificados de un campo se limpian.
- `bitsAnySet()`: cuando al menos uno de los bits especificados de un campo se establece.
- `bitsAnyClear()`: cuando al menos uno de los bits especificados de un campo se limpia.

Ejemplo:
```java
Bson bitsComparison = bitsAllSet("a", 34);
collection.find(bitsComparison).forEach(doc -> System.out.println(doc.toJson()));
```

Salida:
```java
{ "_id": 9, "a": 54, "binaryValue": "00110110" }
{ "_id": 12, "a": 102, "binaryValue": "01100110" }
```
