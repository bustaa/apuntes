## 📁 Ficheros y estructura en el servidor
`Usuarios_autorizados.txt`

Formato:

```
usuario;contraseña
```

Ejemplo:
```
juan;1234
ana;abcd
```

### 📂 Carpetas de usuarios (se crean automáticamente):

```
./juan/
./ana/
```

Dentro se guardarán los ficheros con timestamp.

## 🖥️ Servidor multihilo
Servidor.java
```java
import java.net.*;
import java.io.*;

public class Servidor {

    public static final int PUERTO = 10000;

    public static void main(String[] args) {

        System.out.println("Servidor iniciado...");

        try (ServerSocket serverSocket = new ServerSocket(PUERTO)) {

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
import java.text.SimpleDateFormat;
import java.util.*;

public class Hilo extends Thread {

    private Socket socket;

    public Hilo(Socket socket) {
        this.socket = socket;
    }

    private boolean comprobarUsuario(String usuario, String contrasena)
            throws IOException {

        BufferedReader br = new BufferedReader(
                new FileReader("Usuarios_autorizados.txt"));
        String linea;

        while ((linea = br.readLine()) != null) {
            String[] partes = linea.split(";");
            if (partes[0].equals(usuario) &&
                partes[1].equals(contrasena)) {
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

            // 1. LOGIN o REGISTRO
            String mensaje = in.readLine();

            if (mensaje.startsWith("REGISTRO")) {
                String[] partes = mensaje.split(";");
                BufferedWriter bw = new BufferedWriter(
                        new FileWriter("Usuarios_autorizados.txt", true));
                bw.newLine();
                bw.write(partes[1] + ";" + partes[2]);
                bw.close();
            }

            // 2. Recibir login usuario;contraseña
            String[] cred = in.readLine().split(";");
            String usuario = cred[0];
            String contrasena = cred[1];

            if (!comprobarUsuario(usuario, contrasena)) {
                out.println("401");
                socket.close();
                return;
            }

            out.println("204");

            // 3. Recibir número de líneas
            int numLineas = Integer.parseInt(in.readLine());

            // Crear carpeta del usuario
            File dir = new File("./" + usuario);
            if (!dir.exists()) dir.mkdir();

            // Crear fichero con timestamp
            String timestamp = new SimpleDateFormat(
                    "yyyyMMddHHmmss").format(new Date());

            File fichero = new File(
                    dir, usuario + "_" + timestamp + ".txt");

            BufferedWriter bw = new BufferedWriter(
                    new FileWriter(fichero));

            out.println("PREPARADO");

            // 4. Recibir líneas
            for (int i = 0; i < numLineas; i++) {
                String linea = in.readLine();
                bw.write(linea);
                bw.newLine();
                System.out.println("Recibido: " + linea);
            }

            bw.close();
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
    public static final int PUERTO = 10000;

    public static void main(String[] args) {

        Scanner teclado = new Scanner(System.in);

        try (
            Socket socket = new Socket(HOST, PUERTO);
            BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(
                    socket.getOutputStream(), true)
        ) {

            // Opción login o registro
            System.out.print("1. Login | 2. Registro: ");
            String opcion = teclado.nextLine();

            if (opcion.equals("1")) {
                out.println("LOGIN");
            } else {
                System.out.print("Nuevo usuario: ");
                String u = teclado.nextLine();
                System.out.print("Contraseña: ");
                String p = teclado.nextLine();
                out.println("REGISTRO;" + u + ";" + p);
            }

            // Login
            System.out.print("Usuario: ");
            String usuario = teclado.nextLine();
            System.out.print("Contraseña: ");
            String contrasena = teclado.nextLine();

            out.println(usuario + ";" + contrasena);

            String respuesta = in.readLine();
            if ("401".equals(respuesta)) {
                System.out.println("Acceso denegado");
                return;
            }

            // Enviar número de líneas
            System.out.print("Número de líneas a enviar: ");
            int num = Integer.parseInt(teclado.nextLine());
            out.println(num);

            // Esperar PREPARADO
            if (!"PREPARADO".equals(in.readLine())) {
                return;
            }

            // Enviar líneas
            for (int i = 0; i < num; i++) {
                System.out.print("Línea " + (i + 1) + ": ");
                out.println(teclado.nextLine());
            }

            socket.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Qué cumple esta solución

- ✔ Registro de nuevos usuarios
- ✔ Login con validación
- ✔ Servidor multihilo
- ✔ Guardado de datos en carpeta personal
- ✔ Fichero con timestamp correcto
- ✔ Protocolo seguido línea a línea
- ✔ Código claro y de nivel examen
