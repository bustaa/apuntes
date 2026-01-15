# ORM – Hibernate (DAM) · Chuleta Examen Práctico

---

## 1. ORM (Object Relational Mapping)
Técnica que permite mapear objetos Java a tablas de una base de datos relacional.
Hibernate es un framework ORM para Java.

---

## 2. Desfasamiento objeto–relacional
Problema al usar POO con BDD relacionales:
- Java: objetos, herencia, referencias
- MySQL: tablas, filas, tipos primitivos
Hibernate reduce este desfasamiento.

---

## 3. Arquitectura Hibernate (flujo obligatorio)

Configuration  
↓  
SessionFactory (una sola vez, objeto pesado)  
↓  
Session (muchas veces)  
↓  
Transaction  
↓  
CRUD  
↓  
commit / rollback  
↓  
close  

---

## 4. Estados de los objetos
- Transitorio: objeto Java normal, no existe en BDD
- Persistente: asociado a una Session
- Detached: sesión cerrada, Hibernate no lo controla

---

## 5. Configuración Hibernate (hibernate.cfg.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
  <session-factory>

    <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
    <property name="connection.url">jdbc:mysql://localhost:3306/bbdd</property>
    <property name="connection.username">root</property>
    <property name="connection.password"></property>

    <mapping resource="paquete/clase.hbm.xml"/>

  </session-factory>
</hibernate-configuration>
```

---

## 6. Fichero de mapeo (.hbm.xml)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="paquete">

  <class name="Cancion" table="canciones">

    <id name="id" column="id">
      <generator class="increment"/>
    </id>

    <property name="titulo" column="titulo" type="string"/>
    <property name="artista" column="artista" type="string"/>
    <property name="anyo" column="anyo" type="int"/>
    <property name="formato" column="formato" type="string"/>

  </class>
</hibernate-mapping>

```

---

## 7. Crear SessionFactory

```java
Configuration configuration = new Configuration().configure("hibernate.cfg.xml");
configuration.addClass(Cancion.class);

ServiceRegistry registry =
    new StandardServiceRegistryBuilder()
        .applySettings(configuration.getProperties())
        .build();

SessionFactory sessionFactory = configuration.buildSessionFactory(registry);

```

Alternativa si falla:
```java
configuration.addFile("./src/paquete/cancion.hbm.xml");
```

---

## 8. Estructura OBLIGATORIA de cualquier operación

```java
Session session = sessionFactory.openSession();
session.beginTransaction();

// operaciones CRUD

session.getTransaction().commit();
session.close();
```

---

## 9. CRUD

### CREATE

```java
Cancion c = new Cancion("Titulo", "Artista", 2024, "MP3");
session.save(c);
```

### READ

Por id:
```java
Cancion c = session.get(Cancion.class, 1);
```

Lista:
```java
List<Cancion> lista = session.createQuery("FROM Cancion").list();
```

### UPDATE

```java
Cancion c = session.get(Cancion.class, 5);
c.setFormato("FLAC");
session.update(c);
```

### DELETE

Por id:
```java
Cancion c = new Cancion();
c.setId(5);
session.delete(c);
```

Borrar todo:
```java
Query q = session.createQuery("DELETE FROM canciones");
q.executeUpdate();
```

---

## 10. HQL (Hibernate Query Language)

* Se trabaja con objetos
* Se usa el nombre de la clase
* No se usa el nombre de la tabla

---

## 11. Diferencia get/load

* get(): deveulve null si no existe
* load(): lanza excepción al acceder

---

## 12. Errores típicos de examen

* Usar nombre de tabla en HQL
* No usar transacción
* Olvidar commit
* No cerrar la sesión
* Crear varias SessionFactory

---

## 13. Regla de oro

Session -> Trnasaction -> CRUD -> Commit -> Close


---

🧠 **Consejo final**  
No improvises. Copia estructura, cambia nombres, revisa `commit()` y `close()`.

Suerte 🍀
