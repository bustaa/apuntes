## 📁 Estructura de ficheros en el servidor

El servidor debe tener en su directorio de ejecución:

Usuarios_autorizados.txt

Contrasenyas_autorizadas.txt

Contenido_a_enviar.txt

---

## ⚠️ IMPORTANTE
Las líneas de usuarios y contraseñas deben ser correlativas, por ejemplo:

Usuarios_autorizados.txt

juan
ana
pedro


Contrasenyas_autorizadas.txt

1234
abcd
qwerty

## 🖥️ Servidor multihilo (ServerSocket)

Servidor.java
```java
import java.io.*;
import java.net.*;
import java.util.*;

public class Servidor {

    public static final int PUERTO = 5000;

    public static void main(String[] args) {
        System.out.println("Servidor arrancado...");

        try (ServerSocket serverSocket = new ServerSocket(PUERTO)) {

            while (true) {
                Socket cliente = serverSocket.accept();
                System.out.println("Cliente conectado");
                new HiloCliente(cliente).start();
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

HiloCliente.java
```java
import java.io.*;
import java.net.*;
import java.util.*;

public class HiloCliente extends Thread {

    private Socket socket;
    private List<String> usuarios;
    private List<String> contrasenas;

    public HiloCliente(Socket socket) {
        this.socket = socket;
        cargarUsuarios();
    }

    private void cargarUsuarios() {
        usuarios = new ArrayList<>();
        contrasenas = new ArrayList<>();

        try (BufferedReader brUsuarios = new BufferedReader(new FileReader("Usuarios_autorizados.txt"));
             BufferedReader brContras = new BufferedReader(new FileReader("Contrasenyas_autorizadas.txt"))) {

            String linea;
            while ((linea = brUsuarios.readLine()) != null) {
                usuarios.add(linea);
            }

            while ((linea = brContras.readLine()) != null) {
                contrasenas.add(linea);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void run() {
        try (
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true)
        ) {

            // 1. Recibir usuario
            String usuario = in.readLine();
            int indice = usuarios.indexOf(usuario);

            if (indice == -1) {
                out.println("ERROR");
                socket.close();
                return;
            }

            out.println("200 OK");

            // 2. Recibir contraseña
            String contrasena = in.readLine();

            if (!contrasenas.get(indice).equals(contrasena)) {
                out.println("ERROR");
                socket.close();
                return;
            }

            out.println("200 OK");

            // 3. Recibir PREPARADO
            String preparado = in.readLine();
            if (!"PREPARADO".equals(preparado)) {
                socket.close();
                return;
            }

            // 4. Enviar número de líneas
            File fichero = new File("Contenido_a_enviar.txt");
            List<String> lineas = new ArrayList<>();

            try (BufferedReader br = new BufferedReader(new FileReader(fichero))) {
                String linea;
                while ((linea = br.readLine()) != null) {
                    lineas.add(linea);
                }
            }

            out.println(String.valueOf(lineas.size()));

            // 5. Enviar contenido línea a línea
            for (String l : lineas) {
                out.println(l);
            }

            socket.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 💻 Cliente

Cliente.java
```java
import java.io.*;
import java.net.*;
import java.util.Scanner;

public class Cliente {

    public static final String HOST = "localhost";
    public static final int PUERTO = 5000;

    public static void main(String[] args) {

        try (
            Socket socket = new Socket(HOST, PUERTO);
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            Scanner teclado = new Scanner(System.in)
        ) {

            // Usuario
            System.out.print("Usuario: ");
            String usuario = teclado.nextLine();
            out.println(usuario);

            String respuesta = in.readLine();
            if (!"200 OK".equals(respuesta)) {
                System.out.println("Usuario no autorizado");
                return;
            }

            // Contraseña
            System.out.print("Contraseña: ");
            String contrasena = teclado.nextLine();
            out.println(contrasena);

            respuesta = in.readLine();
            if (!"200 OK".equals(respuesta)) {
                System.out.println("Contraseña incorrecta");
                return;
            }

            // Preparado
            out.println("PREPARADO");

            // Número de líneas
            int numLineas = Integer.parseInt(in.readLine());

            // Recibir contenido
            System.out.println("\nContenido recibido:");
            for (int i = 0; i < numLineas; i++) {
                System.out.println(in.readLine());
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Qué cumple esta solución

* ✔ Servidor multihilo
* ✔ Comunicación cliente-servidor por Sockets TCP
* ✔ Autenticación por usuario y contraseña
* ✔ Protocolo seguido paso a paso
* ✔ Envío de fichero línea a línea
* ✔ Cierre correcto de conexiones