## 1️⃣ HashMap
```java
import java.util.HashMap;

public class EjemploHashMap {
    public static void main(String[] args) {
        // Creamos un HashMap
        HashMap<String, Integer> edades = new HashMap<>();

        // Agregamos elementos
        edades.put("Ana", 25);
        edades.put("Luis", 30);
        edades.put("María", 28);

        // Accedemos a un valor
        System.out.println("Edad de Luis: " + edades.get("Luis")); // 30

        // Verificamos si contiene una clave
        System.out.println("¿Contiene Ana? " + edades.containsKey("Ana")); // true

        // Eliminamos un elemento
        edades.remove("María");

        // Iteramos sobre el mapa
        for (String nombre : edades.keySet()) {
            System.out.println(nombre + " tiene " + edades.get(nombre) + " años");
        }
    }
}
```

🔹 Característica: no mantiene orden de inserción.

## 2️⃣ LinkedHashMap
```java
import java.util.LinkedHashMap;

public class EjemploLinkedHashMap {
    public static void main(String[] args) {
        // LinkedHashMap mantiene el orden de inserción
        LinkedHashMap<String, Integer> edades = new LinkedHashMap<>();

        edades.put("Ana", 25);
        edades.put("Luis", 30);
        edades.put("María", 28);

        // Iteramos y se mantiene el orden
        for (String nombre : edades.keySet()) {
            System.out.println(nombre + " tiene " + edades.get(nombre) + " años");
        }
    }
}
```

🔹 Característica: mantiene el orden de inserción.

## 3️⃣ TreeMap
```java
import java.util.TreeMap;

public class EjemploTreeMap {
    public static void main(String[] args) {
        // TreeMap ordena automáticamente las claves
        TreeMap<String, Integer> edades = new TreeMap<>();

        edades.put("Ana", 25);
        edades.put("Luis", 30);
        edades.put("María", 28);

        // Iteramos y se muestran ordenadas alfabéticamente por clave
        for (String nombre : edades.keySet()) {
            System.out.println(nombre + " tiene " + edades.get(nombre) + " años");
        }
    }
}
```

🔹 Característica: mantiene las claves ordenadas (naturalmente o por un comparador).

## 💡 Resumen rápido:

- Map	            Orden de elementos	    Uso principal
- HashMap	        No garantiza orden	    Acceso rápido
- LinkedHashMap 	Mantiene inserción	    Orden + rápido
- TreeMap	        Claves ordenadas	    Ordenamiento automático