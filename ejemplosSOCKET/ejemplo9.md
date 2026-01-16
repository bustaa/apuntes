## 📁 Fichero de usuarios

``Usuarios_autorizados.dat``:

```
juan:1234
ana:abcd
pedro:qwerty
```

## 🖥️ Servidor multihilo
``Servidor.java``
```java
import java.net.*;
import java.io.*;

public class Servidor {

    public static final int PUERTO = 5001;

    public static void main(String[] args) {
        System.out.println("Servidor iniciado en puerto " + PUERTO);

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

``HiloCliente.java``
```java
import java.io.*;
import java.net.*;
import java.util.*;

public class HiloCliente extends Thread {

    private Socket socket;

    public HiloCliente(Socket socket) {
        this.socket = socket;
    }

    private boolean autenticar(String usuario, String contrasena) {
        try (BufferedReader br = new BufferedReader(new FileReader("Usuarios_autorizados.dat"))) {
            String linea;
            while ((linea = br.readLine()) != null) {
                String[] partes = linea.split(":");
                if (partes[0].equals(usuario) && partes[1].equals(contrasena)) {
                    return true;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return false;
    }

    public void run() {
        try (
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
        ) {
            // 1. Recibir usuario:contrasena
            String login = in.readLine();
            String[] partes = login.split(":");
            String usuario = partes[0];
            String contrasena = partes[1];

            // 2. Autenticar
            if (!autenticar(usuario, contrasena)) {
                out.println("ERROR");
                socket.close();
                return;
            }
            out.println("OK");

            // 3. Recibir número de líneas
            int numLineas = Integer.parseInt(in.readLine());
            out.println(numLineas); // devolver al cliente para mostrar

            // 4. Crear fichero log con timestamp
            String timestamp = new java.text.SimpleDateFormat("yyyyMMdd_HHmmss").format(new java.util.Date());
            String nombreFichero = timestamp + "_log_" + usuario + ".dat";
            BufferedWriter bw = new BufferedWriter(new FileWriter(nombreFichero));

            StringBuilder recibidoServidor = new StringBuilder();

            // 5. Recibir líneas
            for (int i = 0; i < numLineas; i++) {
                String linea = in.readLine();
                bw.write(linea);
                bw.newLine();
                recibidoServidor.append(linea);
                out.println(linea); // enviar de vuelta línea a línea
            }

            bw.close();

            // 6. Recibir string del cliente y comprobar
            String recibidoCliente = in.readLine(); // string del cliente que contiene todas las líneas
            if (recibidoCliente.equals(recibidoServidor.toString())) {
                out.println("OK");
            } else {
                out.println("ERROR");
            }

            socket.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 💻 Cliente
``Cliente.java``
```java
import java.io.*;
import java.net.*;
import java.util.*;

public class Cliente {

    public static void main(String[] args) {

        Scanner teclado = new Scanner(System.in);

        try {
            Socket socket = new Socket("localhost", 5001);

            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);

            // 1. Pedir usuario y contraseña
            System.out.print("Usuario: ");
            String usuario = teclado.nextLine();
            System.out.print("Contraseña: ");
            String contrasena = teclado.nextLine();

            String login = usuario + ":" + contrasena;
            out.println(login);

            // 2. Recibir OK/ERROR
            String respuesta = in.readLine();
            if ("ERROR".equals(respuesta)) {
                System.out.println("Usuario/contraseña incorrectos");
                socket.close();
                return;
            }

            // 3. Pedir número de líneas
            System.out.print("Número de líneas a enviar: ");
            int numLineas = Integer.parseInt(teclado.nextLine());
            out.println(numLineas);

            // Recibir número de líneas de vuelta
            System.out.println("Servidor recibirá " + in.readLine() + " líneas");

            // 4. Enviar líneas
            StringBuilder todoCliente = new StringBuilder();
            List<String> lineas = new ArrayList<>();
            for (int i = 0; i < numLineas; i++) {
                System.out.print("Línea " + (i+1) + ": ");
                String linea = teclado.nextLine();
                lineas.add(linea);
                todoCliente.append(linea);
                out.println(linea);

                // Recibir línea de vuelta
                System.out.println("Servidor envió de vuelta: " + in.readLine());
            }

            // 5. Enviar todas las líneas concatenadas para comprobación
            out.println(todoCliente.toString());

            // 6. Recibir OK/ERROR final
            String finalCheck = in.readLine();
            System.out.println(finalCheck);

            socket.close();

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Características de esta implementación

- Autenticación usuario:contraseña desde Usuarios_autorizados.dat

- Envío de varias líneas al servidor

- Cada línea se envía de vuelta y se guarda en un fichero log con timestamp y nombre de usuario

- El cliente y servidor comprueban que el string concatenado de líneas coincide

- Servidor multihilo para aceptar múltiples clientes simultáneamente

- Cumple exactamente el protocolo de comunicaciones