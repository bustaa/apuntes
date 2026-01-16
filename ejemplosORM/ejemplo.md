PASO 0: Preparar la base de datos

Crear la base de datos productos en MySQL.

Crear la tabla precios:

CREATE DATABASE IF NOT EXISTS productos;
USE productos;

CREATE TABLE precios (
    id INT(3) NOT NULL AUTO_INCREMENT,
    item VARCHAR(100) NOT NULL,
    precio INT(5) NOT NULL,
    precioOferta INT(5) NOT NULL,
    saldo VARCHAR(2) NOT NULL,
    PRIMARY KEY (id)
);

PASO 1: Estructura del proyecto

En IntelliJ:

src/
 └─ main/
    ├─ java/
    │   └─ com/ejemplo/hibernate/
    │       ├─ Precio.java
    │       └─ MainApp.java
    └─ resources/
        ├─ hibernate.cfg.xml
        └─ com/ejemplo/hibernate/Precio.hbm.xml


com/ejemplo/hibernate → tu package Java

resources/com/ejemplo/hibernate → tus archivos XML de mapeo

PASO 2: Hibernate.cfg.xml
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>

        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/productos</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">TU_PASSWORD</property>

        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.hbm2ddl.auto">update</property>

        <mapping resource="com/ejemplo/hibernate/Precio.hbm.xml"/>

    </session-factory>
</hibernate-configuration>

PASO 3: Precio.hbm.xml
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.ejemplo.hibernate">

    <class name="Precio" table="precios">
        <id name="id" type="int">
            <generator class="identity"/>
        </id>

        <property name="item"/>
        <property name="precio"/>
        <property name="precioOferta"/>
        <property name="saldo"/>
    </class>

</hibernate-mapping>

PASO 4: Precio.java
package com.ejemplo.hibernate;

public class Precio {

    private int id;
    private String item;
    private int precio;
    private int precioOferta;
    private String saldo;

    // Constructor vacío
    public Precio() {}

    // Constructor con todos los campos (sin id)
    public Precio(String item, int precio, int precioOferta, String saldo) {
        this.item = item;
        this.precio = precio;
        this.precioOferta = precioOferta;
        this.saldo = saldo;
    }

    // Getters y Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getItem() { return item; }
    public void setItem(String item) { this.item = item; }

    public int getPrecio() { return precio; }
    public void setPrecio(int precio) { this.precio = precio; }

    public int getPrecioOferta() { return precioOferta; }
    public void setPrecioOferta(int precioOferta) { this.precioOferta = precioOferta; }

    public String getSaldo() { return saldo; }
    public void setSaldo(String saldo) { this.saldo = saldo; }

    @Override
    public String toString() {
        return "Precio{" +
                "id=" + id +
                ", item='" + item + '\'' +
                ", precio=" + precio +
                ", precioOferta=" + precioOferta +
                ", saldo='" + saldo + '\'' +
                '}';
    }
}

PASO 5: MainApp.java (la aplicación completa)

Aquí implementamos todas las funcionalidades que pide el ejercicio:

package com.ejemplo.hibernate;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;

import java.util.List;
import java.util.Scanner;

public class MainApp {

    private static SessionFactory sessionFactory;

    public static void main(String[] args) {

        // Configuración de Hibernate
        Configuration configuration = new Configuration()
                .configure("hibernate.cfg.xml")
                .addResource("com/ejemplo/hibernate/Precio.hbm.xml");

        ServiceRegistry registry = new StandardServiceRegistryBuilder()
                .applySettings(configuration.getProperties())
                .build();

        sessionFactory = configuration.buildSessionFactory(registry);

        Scanner sc = new Scanner(System.in);

        // 1️⃣ Insertar los 3 registros iniciales
        insertarIniciales();

        boolean salir = false;
        while (!salir) {
            System.out.println("\n--- Menú ---");
            System.out.println("1. Añadir nuevo registro");
            System.out.println("2. Listar todos los precios (id y item)");
            System.out.println("3. Ver detalle por id");
            System.out.println("4. Actualizar registro por id");
            System.out.println("5. Borrar registro por id");
            System.out.println("6. Salir");
            System.out.print("Elige una opción: ");
            int opcion = sc.nextInt();
            sc.nextLine(); // limpiar buffer

            switch (opcion) {
                case 1 -> agregarRegistro(sc);
                case 2 -> listarRegistros();
                case 3 -> detallePorId(sc);
                case 4 -> actualizarPorId(sc);
                case 5 -> borrarPorId(sc);
                case 6 -> salir = true;
                default -> System.out.println("Opción inválida");
            }
        }

        sc.close();
        sessionFactory.close();
        System.out.println("✅ Aplicación finalizada");
    }

    private static void insertarIniciales() {
        Session session = sessionFactory.openSession();
        session.beginTransaction();

        session.save(new Precio("Silla", 100, 40, "Si"));
        session.save(new Precio("Mesa", 600, 550, "No"));
        session.save(new Precio("Armario", 450, 420, "No"));

        session.getTransaction().commit();
        session.close();
    }

    private static void agregarRegistro(Scanner sc) {
        System.out.print("Item: ");
        String item = sc.nextLine();
        System.out.print("Precio: ");
        int precio = sc.nextInt();
        System.out.print("Precio oferta: ");
        int precioOferta = sc.nextInt();
        sc.nextLine();
        System.out.print("Saldo (Si/No): ");
        String saldo = sc.nextLine();

        Session session = sessionFactory.openSession();
        session.beginTransaction();

        session.save(new Precio(item, precio, precioOferta, saldo));

        session.getTransaction().commit();
        session.close();

        System.out.println("✅ Registro agregado");
    }

    private static void listarRegistros() {
        Session session = sessionFactory.openSession();
        List<Precio> lista = session.createQuery("from Precio", Precio.class).list();
        System.out.println("ID\tItem");
        for (Precio p : lista) {
            System.out.println(p.getId() + "\t" + p.getItem());
        }
        session.close();
    }

    private static void detallePorId(Scanner sc) {
        System.out.print("Introduce id: ");
        int id = sc.nextInt();
        sc.nextLine();

        Session session = sessionFactory.openSession();
        Precio p = session.get(Precio.class, id);

        if (p != null) {
            System.out.println(p);
        } else {
            System.out.println("❌ No existe un registro con ese id");
        }

        session.close();
    }

    private static void actualizarPorId(Scanner sc) {
        System.out.print("Introduce id: ");
        int id = sc.nextInt();
        sc.nextLine();

        Session session = sessionFactory.openSession();
        session.beginTransaction();

        Precio p = session.get(Precio.class, id);
        if (p != null) {
            System.out.print("Nuevo item (" + p.getItem() + "): ");
            String item = sc.nextLine();
            if (!item.isBlank()) p.setItem(item);

            System.out.print("Nuevo precio (" + p.getPrecio() + "): ");
            String precioStr = sc.nextLine();
            if (!precioStr.isBlank()) p.setPrecio(Integer.parseInt(precioStr));

            System.out.print("Nuevo precioOferta (" + p.getPrecioOferta() + "): ");
            String ofertaStr = sc.nextLine();
            if (!ofertaStr.isBlank()) p.setPrecioOferta(Integer.parseInt(ofertaStr));

            System.out.print("Nuevo saldo (" + p.getSaldo() + "): ");
            String saldo = sc.nextLine();
            if (!saldo.isBlank()) p.setSaldo(saldo);

            session.update(p);
            session.getTransaction().commit();
            System.out.println("✅ Registro actualizado");
        } else {
            System.out.println("❌ No existe un registro con ese id");
            session.getTransaction().rollback();
        }

        session.close();
    }

    private static void borrarPorId(Scanner sc) {
        System.out.print("Introduce id: ");
        int id = sc.nextInt();
        sc.nextLine();

        Session session = sessionFactory.openSession();
        session.beginTransaction();

        Precio p = session.get(Precio.class, id);
        if (p != null) {
            session.delete(p);
            session.getTransaction().commit();
            System.out.println("✅ Registro borrado");
        } else {
            System.out.println("❌ No existe un registro con ese id");
            session.getTransaction().rollback();
        }

        session.close();
    }
}

✅ Esta aplicación ya cumple todos los requisitos del ejercicio

Configuración Hibernate ✔

Inserta 3 registros iniciales ✔

Permite ingresar registros por teclado ✔

Muestra todos los registros con id y item ✔

Muestra detalle por id ✔

Actualiza registros por id ✔

Borra registros por id ✔

Si quieres, puedo hacer una versión simplificada usando solo anotaciones JPA, sin archivos .hbm.xml, que es más moderna y más fácil de mantener.

¿Quieres que haga esa versión?