PASO 0: Crear la base de datos y tabla

En MySQL (phpMyAdmin, MySQL Workbench, o consola):

CREATE DATABASE IF NOT EXISTS viajes;
USE viajes;

CREATE TABLE destinos (
    id INT(3) NOT NULL AUTO_INCREMENT,
    lugar VARCHAR(100) NOT NULL,
    precio INT(5) NOT NULL,
    precioOferta INT(5) NOT NULL,
    pasaporteCOVID VARCHAR(2) NOT NULL,
    PRIMARY KEY (id)
);

PASO 1: Estructura del proyecto
src/
 └─ main/
    ├─ java/
    │   └─ com/ejemplo/viajes/
    │       ├─ Destino.java
    │       └─ MainApp.java
    └─ resources/
        ├─ hibernate.cfg.xml
        └─ com/ejemplo/viajes/Destino.hbm.xml

PASO 2: hibernate.cfg.xml
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>

        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/viajes</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">TU_PASSWORD</property>

        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        <property name="hibernate.hbm2ddl.auto">update</property>

        <mapping resource="com/ejemplo/viajes/Destino.hbm.xml"/>

    </session-factory>
</hibernate-configuration>

PASO 3: Destino.hbm.xml
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.ejemplo.viajes">

    <class name="Destino" table="destinos">
        <id name="id" type="int">
            <generator class="identity"/>
        </id>

        <property name="lugar"/>
        <property name="precio"/>
        <property name="precioOferta"/>
        <property name="pasaporteCOVID"/>
    </class>

</hibernate-mapping>

PASO 4: Destino.java
package com.ejemplo.viajes;

public class Destino {

    private int id;
    private String lugar;
    private int precio;
    private int precioOferta;
    private String pasaporteCOVID;

    public Destino() {}

    public Destino(String lugar, int precio, int precioOferta, String pasaporteCOVID) {
        this.lugar = lugar;
        this.precio = precio;
        this.precioOferta = precioOferta;
        this.pasaporteCOVID = pasaporteCOVID;
    }

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getLugar() { return lugar; }
    public void setLugar(String lugar) { this.lugar = lugar; }

    public int getPrecio() { return precio; }
    public void setPrecio(int precio) { this.precio = precio; }

    public int getPrecioOferta() { return precioOferta; }
    public void setPrecioOferta(int precioOferta) { this.precioOferta = precioOferta; }

    public String getPasaporteCOVID() { return pasaporteCOVID; }
    public void setPasaporteCOVID(String pasaporteCOVID) { this.pasaporteCOVID = pasaporteCOVID; }

    @Override
    public String toString() {
        return "Destino{" +
                "id=" + id +
                ", lugar='" + lugar + '\'' +
                ", precio=" + precio +
                ", precioOferta=" + precioOferta +
                ", pasaporteCOVID='" + pasaporteCOVID + '\'' +
                '}';
    }
}

PASO 5: MainApp.java
package com.ejemplo.viajes;

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

        Configuration configuration = new Configuration()
                .configure("hibernate.cfg.xml")
                .addResource("com/ejemplo/viajes/Destino.hbm.xml");

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
            System.out.println("1. Añadir nuevo destino");
            System.out.println("2. Listar destinos (id y lugar)");
            System.out.println("3. Ver detalle por id");
            System.out.println("4. Actualizar destino por id");
            System.out.println("5. Borrar destino por id");
            System.out.println("6. Salir");
            System.out.print("Elige una opción: ");
            int opcion = sc.nextInt();
            sc.nextLine(); // limpiar buffer

            switch (opcion) {
                case 1 -> agregarDestino(sc);
                case 2 -> listarDestinos();
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

        session.save(new Destino("Londres", 1000, 900, "Si"));
        session.save(new Destino("Tokio", 1800, 1500, "Si"));
        session.save(new Destino("Rio de Janeiro", 1500, 1100, "No"));

        session.getTransaction().commit();
        session.close();
    }

    private static void agregarDestino(Scanner sc) {
        System.out.print("Lugar: ");
        String lugar = sc.nextLine();
        System.out.print("Precio: ");
        int precio = sc.nextInt();
        System.out.print("Precio oferta: ");
        int precioOferta = sc.nextInt();
        sc.nextLine();
        System.out.print("Pasaporte COVID (Si/No): ");
        String pasaporte = sc.nextLine();

        Session session = sessionFactory.openSession();
        session.beginTransaction();

        session.save(new Destino(lugar, precio, precioOferta, pasaporte));

        session.getTransaction().commit();
        session.close();

        System.out.println("✅ Destino agregado");
    }

    private static void listarDestinos() {
        Session session = sessionFactory.openSession();
        List<Destino> lista = session.createQuery("from Destino", Destino.class).list();
        System.out.println("ID\tLugar");
        for (Destino d : lista) {
            System.out.println(d.getId() + "\t" + d.getLugar());
        }
        session.close();
    }

    private static void detallePorId(Scanner sc) {
        System.out.print("Introduce id: ");
        int id = sc.nextInt();
        sc.nextLine();

        Session session = sessionFactory.openSession();
        Destino d = session.get(Destino.class, id);

        if (d != null) {
            System.out.println(d);
        } else {
            System.out.println("❌ No existe un destino con ese id");
        }

        session.close();
    }

    private static void actualizarPorId(Scanner sc) {
        System.out.print("Introduce id: ");
        int id = sc.nextInt();
        sc.nextLine();

        Session session = sessionFactory.openSession();
        session.beginTransaction();

        Destino d = session.get(Destino.class, id);
        if (d != null) {
            System.out.print("Nuevo lugar (" + d.getLugar() + "): ");
            String lugar = sc.nextLine();
            if (!lugar.isBlank()) d.setLugar(lugar);

            System.out.print("Nuevo precio (" + d.getPrecio() + "): ");
            String precioStr = sc.nextLine();
            if (!precioStr.isBlank()) d.setPrecio(Integer.parseInt(precioStr));

            System.out.print("Nuevo precioOferta (" + d.getPrecioOferta() + "): ");
            String ofertaStr = sc.nextLine();
            if (!ofertaStr.isBlank()) d.setPrecioOferta(Integer.parseInt(ofertaStr));

            System.out.print("Nuevo pasaporteCOVID (" + d.getPasaporteCOVID() + "): ");
            String pasaporte = sc.nextLine();
            if (!pasaporte.isBlank()) d.setPasaporteCOVID(pasaporte);

            session.update(d);
            session.getTransaction().commit();
            System.out.println("✅ Destino actualizado");
        } else {
            System.out.println("❌ No existe un destino con ese id");
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

        Destino d = session.get(Destino.class, id);
        if (d != null) {
            session.delete(d);
            session.getTransaction().commit();
            System.out.println("✅ Destino borrado");
        } else {
            System.out.println("❌ No existe un destino con ese id");
            session.getTransaction().rollback();
        }

        session.close();
    }
}

✅ Esta aplicación cumple todos los requisitos

Configuración Hibernate ✔

Inserta 3 registros iniciales ✔

Permite agregar registros por teclado ✔

Lista todos los destinos (id y lugar) ✔

Muestra detalle por id ✔

Actualiza registros por id ✔

Borra registros por id ✔