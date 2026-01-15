# Apuntes Programación Multiproceso - Multihilo PSP

## Programación Multiproceso

**Clase ProcessBuilder**

- Ejecuta un programa externo
- Lanzar un comando del sistema
- Abrir otro .jar

### Llamada básica a un proceso

```java
String javaHome = System.getProperty("java.home"); // Identificar la versión de Java
String javaBin = javaHome + File.separator + "bin" + File.separator + "java"; // Completar la ruta hasta el ejecutable de la máquina virtual
String classpath = System.getProperty("java.class.path"); // Ruta para encontrar la clase a ejecutar
String clase = "clase a ejecutar"; // Identificar la clase a ejecutar, por ejemplo: es.florida.multiproceso.Sumador

List<String> command = new ArrayList<>();
command.add(javaBin);
command.add("-cp");
command.add(classpath);
command.add(clase);
command.add(String.valueOf(n1)) // Parametro 1 (opcional)
command.add(String.valueOf(n2)) // Parametro 2 (opcional)

ProcessBuilder builder = new ProcessBuilder(command);
builder.start();
```

### Comunicación entre procesos

- `inheritIO()`: Redirigir la salida a la consola de la aplicación padre

    ```java
    builder.inheritIO().start();
    ```
- `redirectOutput(File f)`: Redirige la salida de la consola de la aplicación hija a un fichero `File f`

    ```java
    builder.redirectOutput(new File("fichero.log")).start();
    ```
-`exitValue()`: Controla si el proceso acaba y con que código

    ```java
    Process process = builder.start();
    process.waitFor();
    System.out.println(process.exitValue);
    ```

**Ejemplo comunicación entre procesos**

```java
File fitxerResultat = new File(fitxerResultat);
String javaHome = System.getProperty("java.home");
String javaBin = javaHome + File.separator + "bin" + File.separator + "java";
String classpath = System.getProperty("java.class.path");
String classe = "es.florida.multiprocess.Sumador";

List<String> command = new ArrayList<>();
command.add(javaBin);
command.add("-cp");
command.add(classpath);
command.add(className);
command.add(String.valueOf(n1));
command.add(String.valueOf(n2));

ProcessBuilder builder = new ProcessBuilder(command);
builder.redirectOutput(fitxerResultat);
Process p = builder.start();
```

### Sincronización entre procesos

- Mecanismo para coordinar las actividade de dos o más procesos.
- Acceso a recursos compartidos con integridad critica (p.e escritrua en fichero, hilo en espera de resultados de otro hilo).
- Java: **monitor** para cada objeto instaciado.
    - Permite bloquear el acceso al objeto o a alguno de sus metodos.
    - Uso de la palabra `synchronized` com a modificador del metodo.
    - Cuando un hilo accede al metodo, el monitor lo bloquea y ningun otro hilo puede acceder hasta que no finalize el primer hilo.
---
<br>
<br>

## Programación Multihilo

### Clase Thread

- `Thread()`: Constructor
- `currentThread()`: Devuelve el objeto`thread` que representa el hilo de ejecución actual.
- `sleep(long)`: Pausa de `<long>` milisegundos.
- `start()`: Crea el contexto del hilo y comienza a ejecutarlo
- `run()`: Ejecución del hilo, llamado por el método `start()`.
- `interrupt()`: Indica al final que se ha interrumpido (no fuerza la interrupción).
- `setName(String)`: Asigna a un hilo un nombre determinado.
- `getName()`: Devuelve el nombre del hilo.
- `join()`: Espera a que el hilo acabe y devuleve el control al hilo principal.
- `isAlive()`: Devuelve si el hilo aún esta en ejecución.
- `setPriority(int)`: Establece la prioridad de un hilo.  Poco fiable
- `getPriority()`: Devuelve la prioridad de un hilo.  Poco fiable

## Implementación Runnable

```java
class EjecutarHilo implements Runnable {
    private String nombre;

    public EjecutarHilo(String nombre) {
        this.nombre = nombre;
    }

    @Override
    public void run() {
        // Codigo a ejecutar por el hilo
    }
}
```

## Sincronización y comunicación entre hilos

- **Condición de carrear**: Diversos hilos acceden al mismo objeto (recursos compartidos) y lo modifican sin control, produciendo resultados inesperados.

- Cuando un método acceda a una variable que esté compartida deberemos protegerlo utilizando `synchronized`.

- Se puede poner todo el método como `synchronized` o marcar un trozo de codigo más reducido.

- Un hilo se sincroniza cuando se convierte en propietario del monitor del objeto (bloqueo).

- Aplicación **thread-safe**: aplicación programada teniendo en cuenta que si se ejecutan deversos hilos se puede asegurar que será estable.

**Ejemplo sincronización**

```java
class Contador {
    private int valor = 0;

    public synchronized void incrementar() {
        valor++;
    }

    public int getValor() {
        return valor;
    }
}

public class Main {
    public static void main(String[] args) {
        Contador contador = new Contador();

        Thread hilo1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                contador.incrementar();
            }
        });

        Thread hilo2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) {
                contador.incrementar();
            }
        });

        hilo1.start();
        hilo2.start();

        try {
            hilo1.join();
            hilo2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.pritnln(contador.getValor());
    }
}
```

## Uso de ExecutorService

- Permite ejecutar tareas en segundo plano sin tener que crear hilos manualmente.

1. Crear un **pool de hilos** (grupo de trabajadores):

    ```java
    ExecutorService executor = Executors.newFixedThreadPool(4);
    ```

    - "Tengo 4 trabajadores dispobiles para ejecutar tareas"
    - También se puede usar:
        - `newSingleThreadExecutor()`: solo un trabajador
        - `newCachedThreadPool()`: crea hilos según necesidad

2.  Las tareas se envían con `submit()` o `execute()`

## Ejemplo sin ExecutorService

```java
public class Tarea implements Runnable {
    private String nombre;

    public Tarea(String nombre) {
        this.nombre = nombre;
    }

    @Override 
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(nombre + " -> part " + (i + 1));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        Thread t1 = new Thread(new Tarea("Tarea 1"));
        Thread t2 = new Thread(new Tarea("Tarea 2"));
        Thread t3 = new Thread(new Tarea("Tarea 3"));
    }
}
```

## Ejemplo con ExecutorService

```java
public class Tarea implements Runnable {
    private String nombre;

    public Tarea(String nombre) {
        this.nombre = nombre;
    }

    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(nom + " -> part " + (i + 1));
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        String[] arrayTareas = {"Tarea 1", "Tarea 2", "Tarea 3"};
        ExecutorService executor = Executors.newFixedThreadPool(10);
        for (String tarea : arrayTareas) {
            executor.execute(new Tarea(tarea));
        }
        executor.shutdown();
    }
}
```

## Ejemplo Pizzeria sin ExecutorService

```java
class Pizzeria {
    private final List<String> comanda;

    public Pizzeria(List<String> comanda) {
        this.comanda = comanda;
    }

    // método sincronizado para que solo un hilo acceda a la lista a la vez
    public synchronized String tomarPizza() {
        if (!comanda.isEmpty()) {
            return comanda.remove(0);
        } else {
            return null;
        }
    }
}

class Trabajador extends Thread {
    private final Pizzeria pizzeria;

    public Trabajador(Pizzeria pizzeria) {
        this.pizzeria = pizzeria;
    }

    @Override
    public void run() {
        String pizza;
        while((pizza = pizzeria.tomarPizza()) != null) {
            System.out.println(Thread.currentThread().getName() + " prepara: " + pizza);
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        List<String> comanda = new ArrayList<>();
        comanda.add("Margarita");
        comanda.add("Pepperoni");
        comanda.add("Hawaiana");
        comanda.add("Cuatro Quesos");
        comanda.add("Vegana");
        comanda.add("Barbacoa");
        comanda.add("Carbonara");
        comanda.add("Napolitana");

        Pizzeria pizzeria = new Pizzeria(comanda);

        Trabajador[] trabajadores = new Trabajador[8];
        for (int i = 0; i < 8; i++) {
            trabajadores[i] = new Trabajador(pizzeria);
            trabajadores[i].start();
        }

        for (Trabajador t : trabajadores) {
            t.join();
        }

        System.out.println("Todas las pizzas están listas!");
    }
}
```

## Ejemplo Pizzeria con ExecutorService

```java
class PizzeriaES {
    private final List<String> comanda;

    public PizzeriaES(List<String> comanda) {
        this.comanda = comanda;
    }

    public synchronized String tomarPizza() {
        if (!comanda.isEmpty()) {
            return comanda.remove(0);
        } else {
            return null;
        }
    }
}

public class MainExecutorExecute {
    public static void main(String[] args) throws InterruptedException {
        List<String> comanda = new ArrayList<>();
        comanda.add("Margarita");
        comanda.add("Pepperoni");
        comanda.add("Hawaiana");
        comanda.add("Cuatro Quesos");
        comanda.add("Vegana");
        comanda.add("Barbacoa");
        comanda.add("Carbonara");
        comanda.add("Napolitana");

        PizzeriaES pizzeria = new PizzeriaES(comanda);

        ExecutorService executor = Executors.newFixedThreadPool(8);

        for (int i = 0; i < 8; i++) {
            executor.execute(() -> { // aquí usamos execute en lugar de submit
                String pizza;
                while ((pizza = pizzeria.tomarPizza()) != null) {
                    System.out.println(Thread.currentThread().getName() + " prepara: " + pizza);
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }

        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);

        System.out.println("Todas las pizzas están listas!");
    }
}
```