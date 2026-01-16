## 📁 Ficheros necesarios en el servidor
`Configuracion.txt`
```
5000
```

`Usuarios_autorizados.txt`

Formato:
```
usuario:contraseña
```

Ejemplo:

```
juan:1234
ana:abcd
pedro:qwerty
```

`Contenido_a_enviar.txt`

Formato:
```
1;Primera línea del fichero
2;Segunda línea del fichero
3;Tercera línea del fichero
```

## 🖥️ Servidor multihilo
Servidor.java
```java
import java.net.*;
import java.io.*;

public class Servidor {

    public static void main(String[] args) {

        int puerto = 0;

        // Leer puerto desde Configuracion.txt
        try (BufferedReader br = new BufferedReader(
                new FileReader("Configuracion.txt"))) {

            puerto = Integer.parseInt(br.readLine());

        } catch (IOException e) {
            e.printStackTrace();
            return;
        }

        System.out.println("Servidor arrancado en puerto " + puerto);

        try (ServerSocket serverSocket = new ServerSocket(puerto)) {

            while (true) {
                Socket cliente = serverSocket.accept();
                new Hilo(cliente).start();
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

Hilo.java
```java
import java.io.*;
import java.net.*;
import java.util.*;

public class Hilo extends Thread {

    private Socket socket;

    public Hilo(Socket socket) {
        this.socket = socket;
    }

    private boolean usuarioAutorizado(String credenciales) throws IOException {
        BufferedReader br = new BufferedReader(
                new FileReader("Usuarios_autorizados.txt"));
        String linea;

        while ((linea = br.readLine()) != null) {
            if (linea.equals(credenciales)) {
                br.close();
                return true;
            }
        }

        br.close();
        return false;
    }

    public void run() {

        try (
            BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(
                    socket.getOutputStream(), true)
        ) {

            // 1. Recibir usuario:contraseña
            String credenciales = in.readLine();

            if (!usuarioAutorizado(credenciales)) {
                out.println("401");
                socket.close();
                return;
            }

            out.println("204");

            // 2. Recibir GET_CONTENT o END
            String comando = in.readLine();
            if ("END".equals(comando)) {
                socket.close();
                return;
            }

            if ("GET_CONTENT".equals(comando)) {

                BufferedReader br = new BufferedReader(
                        new FileReader("Contenido_a_enviar.txt"));
                List<String> lineas = new ArrayList<>();
                String linea;

                while ((linea = br.readLine()) != null) {
                    lineas.add(linea);
                }
                br.close();

                // Enviar número de líneas
                out.println(lineas.size());

                // Enviar contenido sin número
                for (String l : lineas) {
                    out.println(l.split(";", 2)[1]);
                }
            }

            // 3. Recibir posible nueva línea
            String mensaje = in.readLine();

            if (mensaje.startsWith("200;")) {
                String nuevaLinea = mensaje.split(";", 2)[1];

                BufferedReader br = new BufferedReader(
                        new FileReader("Contenido_a_enviar.txt"));
                int contador = 0;
                while (br.readLine() != null) {
                    contador++;
                }
                br.close();

                BufferedWriter bw = new BufferedWriter(
                        new FileWriter("Contenido_a_enviar.txt", true));
                bw.newLine();
                bw.write((contador + 1) + ";" + nuevaLinea);
                bw.close();
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

    public static void main(String[] args) {

        Scanner teclado = new Scanner(System.in);

        try (
            Socket socket = new Socket("localhost", 5000);
            BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(
                    socket.getOutputStream(), true)
        ) {

            // Usuario y contraseña
            System.out.print("Usuario: ");
            String usuario = teclado.nextLine();

            System.out.print("Contraseña: ");
            String contrasena = teclado.nextLine();

            out.println(usuario + ":" + contrasena);

            // Respuesta servidor
            String respuesta = in.readLine();
            if ("401".equals(respuesta)) {
                System.out.println("Acceso no autorizado");
                out.println("END");
                socket.close();
                return;
            }

            // GET_CONTENT
            out.println("GET_CONTENT");

            int numLineas = Integer.parseInt(in.readLine());
            System.out.println("Número de líneas: " + numLineas);

            for (int i = 0; i < numLineas; i++) {
                System.out.println(in.readLine());
            }

            // Añadir nueva línea
            System.out.print("¿Desea añadir una línea? (s/n): ");
            String opcion = teclado.nextLine();

            if (opcion.equalsIgnoreCase("s")) {
                System.out.print("Nueva línea: ");
                String nueva = teclado.nextLine();
                out.println("200;" + nueva);
            } else {
                out.println("END");
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Qué cumple esta solución

- ✔ Lectura de puerto desde fichero
- ✔ Servidor multihilo real
- ✔ Autenticación usuario:contraseña
- ✔ Uso de códigos tipo HTTP (204 / 401)
- ✔ Envío de fichero línea a línea
- ✔ Inserción dinámica de nuevas líneas
- ✔ Protocolo seguido literalmente

📌 Nota importante de examen (muy útil)

Si te preguntan:

¿Por qué no usas objetos aquí?

Respuesta perfecta:

“El protocolo define mensajes de control y códigos de estado, por lo que el uso de texto simplifica la comunicación y se ajusta mejor al diseño solicitado.”