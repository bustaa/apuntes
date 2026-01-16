## 📁 Ficheros necesarios en el servidor
preguntas_secretas.txt

Formato (una línea por usuario):
```
usuario;pregunta;respuesta
```

Ejemplo:
```
usuario1;¿Nombre de tu mascota?;toby
usuario2;¿Ciudad donde naciste?;madrid
usuario3;¿Color favorito?;azul
```
---
Ficheros de contraseña por usuario

* usuario1.txt

* usuario2.txt

* usuario3.txt

Cada fichero contiene solo la contraseña, por ejemplo:

usuario1.txt
```
1234
```

## 🖥️ Servidor multihilo
Servidor.java
```java
import java.net.*;
import java.io.*;

public class Servidor {

    public static final int PUERTO = 7000;

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
    private Map<String, String[]> datos; // usuario -> {pregunta, respuesta}

    public HiloCliente(Socket socket) {
        this.socket = socket;
        cargarDatos();
    }

    private void cargarDatos() {
        datos = new HashMap<>();

        try (BufferedReader br = new BufferedReader(
                new FileReader("preguntas_secretas.txt"))) {

            String linea;
            while ((linea = br.readLine()) != null) {
                String[] partes = linea.split(";");
                datos.put(partes[0], new String[]{partes[1], partes[2]});
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void run() {

        try (
            BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(
                    socket.getOutputStream(), true)
        ) {

            // 1. Recibir usuario
            String usuario = in.readLine();

            if (!datos.containsKey(usuario)) {
                out.println("ERROR");
                socket.close();
                return;
            }

            // 2. Enviar pregunta secreta
            out.println(datos.get(usuario)[0]);

            // 3. Recibir respuesta secreta
            String respuesta = in.readLine();

            if (!datos.get(usuario)[1].equalsIgnoreCase(respuesta)) {
                out.println("ERROR");
                socket.close();
                return;
            }

            out.println("200 OK");

            // 4. Recibir nueva contraseña
            String nuevaContrasena = in.readLine();

            // 5. Sobrescribir fichero del usuario
            BufferedWriter bw = new BufferedWriter(
                    new FileWriter(usuario + ".txt"));
            bw.write(nuevaContrasena);
            bw.close();

            out.println("200 OK");
            socket.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

💻 Cliente

Cliente.java
```java
import java.io.*;
import java.net.*;
import java.util.Scanner;

public class Cliente {

    public static final String HOST = "localhost";
    public static final int PUERTO = 7000;

    public static void main(String[] args) {

        try (
            Socket socket = new Socket(HOST, PUERTO);
            BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(
                    socket.getOutputStream(), true);
            Scanner teclado = new Scanner(System.in)
        ) {

            // Usuario
            System.out.print("Usuario: ");
            String usuario = teclado.nextLine();
            out.println(usuario);

            // Pregunta o ERROR
            String respuestaServidor = in.readLine();
            if ("ERROR".equals(respuestaServidor)) {
                System.out.println("Usuario no autorizado");
                return;
            }

            // Respuesta secreta
            System.out.println("Pregunta secreta: " + respuestaServidor);
            System.out.print("Respuesta: ");
            out.println(teclado.nextLine());

            // Validación
            respuestaServidor = in.readLine();
            if (!"200 OK".equals(respuestaServidor)) {
                System.out.println("Respuesta incorrecta");
                return;
            }

            // Nueva contraseña (doble comprobación)
            String pass1, pass2;
            do {
                System.out.print("Nueva contraseña: ");
                pass1 = teclado.nextLine();
                System.out.print("Repita la contraseña: ");
                pass2 = teclado.nextLine();
            } while (!pass1.equals(pass2));

            out.println(pass1);

            // Confirmación final
            System.out.println(in.readLine());

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Qué cumple esta solución

* ✔ Servidor multihilo
* ✔ Uso de pregunta secreta
* ✔ Verificación correcta de respuesta
* ✔ Cambio real de contraseña en fichero
* ✔ Protocolo seguido paso a paso
* ✔ Comunicación clara cliente ↔ servidor