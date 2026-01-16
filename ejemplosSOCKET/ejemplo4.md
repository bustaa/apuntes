## 📁 Ficheros necesarios en el servidor
`usuarios_autorizados.txt`

Formato:
```
usuario;contraseña
```

Ejemplo:
```
juan;1234
ana;abcd
pedro;qwerty
```
`contenido.txt`

Formato:
```
usuario;contenido
```

Ejemplo:
```
juan;Este es el contenido privado de Juan
ana;Contenido exclusivo de Ana
pedro;Datos confidenciales de Pedro
```

## 🖥️ Servidor multihilo
Servidor.java
```java
import java.net.*;
import java.io.*;

public class Servidor {

    public static final int PUERTO = 8000;

    public static void main(String[] args) {

        System.out.println("Servidor iniciado...");

        try (ServerSocket serverSocket = new ServerSocket(PUERTO)) {

            while (true) {
                Socket cliente = serverSocket.accept();
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
    private Map<String, String> usuarios;

    public HiloCliente(Socket socket) {
        this.socket = socket;
        cargarUsuarios();
    }

    private void cargarUsuarios() {
        usuarios = new HashMap<>();

        try (BufferedReader br = new BufferedReader(
                new FileReader("usuarios_autorizados.txt"))) {

            String linea;
            while ((linea = br.readLine()) != null) {
                String[] partes = linea.split(";");
                usuarios.put(partes[0], partes[1]);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private String obtenerContenido(String usuario) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader("contenido.txt"));
        String linea;

        while ((linea = br.readLine()) != null) {
            String[] partes = linea.split(";");
            if (partes[0].equals(usuario)) {
                br.close();
                return partes[1];
            }
        }

        br.close();
        return null;
    }

    public void run() {

        try (
            BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(
                    socket.getOutputStream(), true)
        ) {

            // 1. Usuario y contraseña
            String usuario = in.readLine();
            String contrasena = in.readLine();

            if (!usuarios.containsKey(usuario) ||
                !usuarios.get(usuario).equals(contrasena)) {

                out.println("ERROR");
                socket.close();
                return;
            }

            // 2. Número aleatorio
            int numero = new Random().nextInt(10);
            out.println(numero);

            // 3. Respuesta del cliente
            String respuesta = in.readLine();
            if (!String.valueOf(numero).equals(respuesta)) {
                out.println("ERROR");
                socket.close();
                return;
            }

            // 4. Enviar contenido
            String contenido = obtenerContenido(usuario);
            out.println(contenido);

            // 5. Esperar RECIBIDO
            in.readLine();
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
    public static final int PUERTO = 8000;

    public static void main(String[] args) {

        try (
            Socket socket = new Socket(HOST, PUERTO);
            BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(
                    socket.getOutputStream(), true);
            Scanner teclado = new Scanner(System.in)
        ) {

            // Usuario y contraseña
            System.out.print("Usuario: ");
            out.println(teclado.nextLine());

            System.out.print("Contraseña: ");
            out.println(teclado.nextLine());

            // Respuesta servidor
            String respuesta = in.readLine();
            if ("ERROR".equals(respuesta)) {
                System.out.println("Usuario o contraseña incorrectos");
                return;
            }

            // Verificación humana
            System.out.println("Escriba el número: " + respuesta);
            out.println(teclado.nextLine());

            // Contenido o ERROR
            respuesta = in.readLine();
            if ("ERROR".equals(respuesta)) {
                System.out.println("Verificación incorrecta");
                return;
            }

            // Mostrar contenido
            System.out.println("Acceso concedido:");
            System.out.println(respuesta);

            // Confirmación
            out.println("RECIBIDO");

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Qué cumple esta solución

* ✔ Servidor multihilo
* ✔ Autenticación por usuario y contraseña
* ✔ Verificación tipo captcha numérico
* ✔ Acceso al contenido según usuario
* ✔ Lectura desde ficheros
* ✔ Protocolo seguido paso a paso