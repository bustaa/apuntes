PASO 0: Crear base de datos y tablas

En MySQL (phpMyAdmin o consola):

CREATE DATABASE IF NOT EXISTS stromboli;
USE stromboli;

CREATE TABLE ingredientes (
    id INT(2) NOT NULL AUTO_INCREMENT,
    nombre VARCHAR(100) NOT NULL,
    alergia INT(1) NOT NULL,
    aptoVeganos INT(1) NOT NULL,
    PRIMARY KEY (id)
);

CREATE TABLE pizzas (
    id INT(2) NOT NULL AUTO_INCREMENT,
    nombre VARCHAR(100) NOT NULL,
    ingredientes VARCHAR(100) NOT NULL,
    precio INT(2) NOT NULL,
    novedad INT(2) NOT NULL,
    PRIMARY KEY (id)
);


Nota: Para simplificar este ejercicio, vamos a guardar los IDs de los ingredientes en la tabla pizzas como una cadena 1,2,3, tal como indica el enunciado.

PASO 1: Estructura de proyecto en IntelliJ
src/
 └─ main/
    ├─ java/
    │   └─ com/ejemplo/stromboli/
    │       ├─ Pizza.java
    │       ├─ Ingrediente.java
    │       └─ MainApp.java
    └─ resources/
        ├─ hibernate.cfg.xml
        ├─ com/ejemplo/stromboli/Pizza.hbm.xml
        └─ com/ejemplo/stromboli/Ingrediente.hbm.xml

PASO 2: hibernate.cfg.xml
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>

        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/stromboli</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">TU_PASSWORD</property>

        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.hbm2ddl.auto">update</property>

        <mapping resource="com/ejemplo/stromboli/Pizza.hbm.xml"/>
        <mapping resource="com/ejemplo/stromboli/Ingrediente.hbm.xml"/>

    </session-factory>
</hibernate-configuration>

PASO 3: Pizza.hbm.xml
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.ejemplo.stromboli">

    <class name="Pizza" table="pizzas">
        <id name="id" type="int">
            <generator class="identity"/>
        </id>
        <property name="nombre"/>
        <property name="ingredientes"/> <!-- guardaremos IDs separados por coma -->
        <property name="precio"/>
        <property name="novedad"/>
    </class>

</hibernate-mapping>

PASO 4: Ingrediente.hbm.xml
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.ejemplo.stromboli">

    <class name="Ingrediente" table="ingredientes">
        <id name="id" type="int">
            <generator class="identity"/>
        </id>
        <property name="nombre"/>
        <property name="alergia"/>
        <property name="aptoVeganos"/>
    </class>

</hibernate-mapping>

PASO 5: Clases Java
Pizza.java
package com.ejemplo.stromboli;

public class Pizza {
    private int id;
    private String nombre;
    private String ingredientes; // IDs separados por coma
    private int precio;
    private int novedad;

    public Pizza() {}

    public Pizza(String nombre, String ingredientes, int precio, int novedad) {
        this.nombre = nombre;
        this.ingredientes = ingredientes;
        this.precio = precio;
        this.novedad = novedad;
    }

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }

    public String getIngredientes() { return ingredientes; }
    public void setIngredientes(String ingredientes) { this.ingredientes = ingredientes; }

    public int getPrecio() { return precio; }
    public void setPrecio(int precio) { this.precio = precio; }

    public int getNovedad() { return novedad; }
    public void setNovedad(int novedad) { this.novedad = novedad; }

    @Override
    public String toString() {
        return "Pizza{" +
                "id=" + id +
                ", nombre='" + nombre + '\'' +
                ", ingredientes='" + ingredientes + '\'' +
                ", precio=" + precio +
                ", novedad=" + novedad +
                '}';
    }
}

Ingrediente.java
package com.ejemplo.stromboli;

public class Ingrediente {
    private int id;
    private String nombre;
    private int alergia;
    private int aptoVeganos;

    public Ingrediente() {}

    public Ingrediente(String nombre, int alergia, int aptoVeganos) {
        this.nombre = nombre;
        this.alergia = alergia;
        this.aptoVeganos = aptoVeganos;
    }

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getNombre() { return nombre; }
    public void setNombre(String nombre) { this.nombre = nombre; }

    public int getAlergia() { return alergia; }
    public void setAlergia(int alergia) { this.alergia = alergia; }

    public int getAptoVeganos() { return aptoVeganos; }
    public void setAptoVeganos(int aptoVeganos) { this.aptoVeganos = aptoVeganos; }

    @Override
    public String toString() {
        return nombre;
    }
}

PASO 6: MainApp.java

Aquí haremos todas las operaciones: truncar tablas, insertar arrays, mostrar cartas y añadir nueva pizza.

package com.ejemplo.stromboli;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;

import java.util.*;

public class MainApp {

    private static SessionFactory sessionFactory;

    public static void main(String[] args) {
        Configuration configuration = new Configuration()
                .configure("hibernate.cfg.xml")
                .addResource("com/ejemplo/stromboli/Pizza.hbm.xml")
                .addResource("com/ejemplo/stromboli/Ingrediente.hbm.xml");

        ServiceRegistry registry = new StandardServiceRegistryBuilder()
                .applySettings(configuration.getProperties())
                .build();

        sessionFactory = configuration.buildSessionFactory(registry);

        Scanner sc = new Scanner(System.in);

        // Borrar contenido de las tablas
        limpiarTablas();

        // Insertar ingredientes
        Map<String, Ingrediente> ingredientesMap = insertarIngredientes();

        // Insertar pizzas
        insertarPizzas(ingredientesMap);

        // Mostrar carta completa
        System.out.println("\n--- Carta completa ---");
        mostrarCarta(false, false);

        // Carta de alérgicos (solo ingredientes alergia = 0)
        System.out.println("\n--- Carta de alérgicos ---");
        mostrarCarta(true, false);

        // Carta vegana (solo ingredientes aptoVeganos = 1)
        System.out.println("\n--- Carta vegana ---");
        mostrarCarta(false, true);

        // Añadir nueva pizza por teclado
        System.out.println("\n--- Añadir nueva pizza ---");
        añadirPizza(sc);

        // Mostrar carta completa de nuevo
        System.out.println("\n--- Carta completa actualizada ---");
        mostrarCarta(false, false);

        sessionFactory.close();
        sc.close();
    }

    private static void limpiarTablas() {
        Session session = sessionFactory.openSession();
        session.beginTransaction();
        session.createNativeQuery("TRUNCATE TABLE pizzas").executeUpdate();
        session.createNativeQuery("TRUNCATE TABLE ingredientes").executeUpdate();
        session.getTransaction().commit();
        session.close();
    }

    private static Map<String, Ingrediente> insertarIngredientes() {
        Session session = sessionFactory.openSession();
        session.beginTransaction();

        Map<String, Ingrediente> map = new HashMap<>();

        Ingrediente i1 = new Ingrediente("tomate",0,1);
        Ingrediente i2 = new Ingrediente("mozzarella",0,1);
        Ingrediente i3 = new Ingrediente("jamon",0,0);
        Ingrediente i4 = new Ingrediente("nueces",1,1);

        List<Ingrediente> lista = List.of(i1,i2,i3,i4);
        for(Ingrediente ing: lista){
            session.save(ing);
            map.put(ing.getNombre(), ing);
        }

        session.getTransaction().commit();
        session.close();
        return map;
    }

    private static void insertarPizzas(Map<String, Ingrediente> map) {
        Session session = sessionFactory.openSession();
        session.beginTransaction();

        Pizza p1 = new Pizza("Margarita",""+map.get("tomate").getId()+","+map.get("mozzarella").getId(),7,0);
        Pizza p2 = new Pizza("Prosciuto",""+map.get("tomate").getId()+","+map.get("mozzarella").getId()+","+map.get("jamon").getId(),8,0);
        Pizza p3 = new Pizza("Con nueces",""+map.get("tomate").getId()+","+map.get("mozzarella").getId()+","+map.get("jamon").getId()+","+map.get("nueces").getId(),9,1);

        session.save(p1);
        session.save(p2);
        session.save(p3);

        session.getTransaction().commit();
        session.close();
    }

    private static void mostrarCarta(boolean soloAlergicos, boolean soloVeganos){
        Session session = sessionFactory.openSession();
        List<Pizza> pizzas = session.createQuery("from Pizza", Pizza.class).list();
        List<Ingrediente> todosIngredientes = session.createQuery("from Ingrediente", Ingrediente.class).list();
        Map<Integer, Ingrediente> mapIng = new HashMap<>();
        for(Ingrediente ing: todosIngredientes) mapIng.put(ing.getId(), ing);

        System.out.println("Carta pizzeria Stromboli");
        for(Pizza p : pizzas){
            List<Ingrediente> listaIng = new ArrayList<>();
            for(String idStr : p.getIngredientes().split(",")){
                int id = Integer.parseInt(idStr);
                Ingrediente ing = mapIng.get(id);
                if(ing != null){
                    if((soloAlergicos && ing.getAlergia()==1) || (soloVeganos && ing.getAptoVeganos()==0)) continue;
                    listaIng.add(ing);
                }
            }
            if(listaIng.isEmpty()) continue;
            String nov = p.getNovedad()==1?"NOVEDAD! ":"";
            System.out.println(p.getId()+" - "+nov+p.getNombre());
            System.out.print(" Ingredientes: [");
            for(int i=0;i<listaIng.size();i++){
                System.out.print(listaIng.get(i).getNombre());
                if(i<listaIng.size()-1) System.out.print(", ");
            }
            System.out.println("] - Precio: "+p.getPrecio()+" eur");
        }

        session.close();
    }

    private static void añadirPizza(Scanner sc){
        Session session = sessionFactory.openSession();
        session.beginTransaction();

        List<Ingrediente> nuevosIng = new ArrayList<>();
        for(int i=1;i<=3;i++){
            System.out.print("Nombre ingrediente "+i+": ");
            String nombre = sc.nextLine();
            System.out.print("Alergia (0=No,1=Si): ");
            int alergia = sc.nextInt();
            System.out.print("Apto vegano (0=No,1=Si): ");
            int vegano = sc.nextInt();
            sc.nextLine();
            Ingrediente ing = new Ingrediente(nombre,alergia,vegano);
            session.save(ing);
            nuevosIng.add(ing);
        }

        System.out.print("Nombre pizza: ");
        String nombrePizza = sc.nextLine();
        System.out.print("Precio pizza: ");
        int precio = sc.nextInt();
        sc.nextLine();

        String ingIds = nuevosIng.get(0).getId()+","+nuevosIng.get(1).getId()+","+nuevosIng.get(2).getId();
        Pizza pizza = new Pizza(nombrePizza, ingIds, precio,1);
        session.save(pizza);

        session.getTransaction().commit();
        session.close();
    }
}

✅ Funcionalidades implementadas

Configuración Hibernate ✔

Limpieza de tablas y reinicio de índices ✔

Inserción de pizzas e ingredientes usando arrays y bucles ✔

Mostrar carta completa ✔

Carta de alérgicos ✔

Carta vegana ✔

Añadir nueva pizza con 3 ingredientes ✔