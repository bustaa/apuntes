## 📁 Ficheros del servidor (se proporcionan)

`usuarios.data`
```
salva
pascual
raquel
```

``inbox_refs.data``

Formato:
```
usuario;ficheroInbox
```

Ejemplo:
```
salva;inbox1.data
pascual;inbox2.data
raquel;inbox3.data
```
``inbox1.data, inbox2.data, inbox3.data``

Formato de cada línea:
```
remitente:mensaje
```

Ejemplo (``inbox1.data``):

```
pascual:Hola Salva
raquel:¿Qué tal?
```

## 🖥️ Servidor multihilo
``Servidor.java``
```java
import java.net.*;
import java.io.*;

public class Servidor {

    public static final int PUERTO = 5001;

    public static void main(String[] args) {

        System.out.println("Servidor inbox iniciado en puerto " + PUERTO);

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

``Hilo.java``
```java
import java.io.*;
import java.net.*;

public class Hilo extends Thread {

    private Socket socket;

    public Hilo(Socket socket) {
        this.socket = socket;
    }

    private String obtenerInbox(String usuario) throws IOException {
        BufferedReader br = new BufferedReader(
                new FileReader("inbox_refs.data"));
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

            // 1. Recibir usuario
            String usuario = in.readLine();

            // 2. Enviar opciones
            out.println("1. Leer inbox");
            out.println("2. Enviar un mensaje");

            // 3. Recibir opción
            String opcion = in.readLine();

            // OPCIÓN 1: leer inbox
            if (opcion.equals("1")) {

                String inbox = obtenerInbox(usuario);
                BufferedReader br = new BufferedReader(
                        new FileReader(inbox));

                int contador = 0;
                while (br.readLine() != null) contador++;
                br.close();

                out.println(contador);

                br = new BufferedReader(new FileReader(inbox));
                String linea;
                while ((linea = br.readLine()) != null) {
                    out.println(linea);
                }
                br.close();

                socket.close();
                return;
            }

            // OPCIÓN 2: enviar mensaje
            if (opcion.equals("2")) {

                // Enviar usuarios disponibles
                BufferedReader br = new BufferedReader(
                        new FileReader("usuarios.data"));
                StringBuilder usuarios = new StringBuilder();
                String linea;

                while ((linea = br.readLine()) != null) {
                    usuarios.append(linea).append(" ");
                }
                br.close();

                out.println(usuarios.toString().trim());

                // Recibir mensaje
                String mensaje = in.readLine();
                String[] partes = mensaje.split(":", 2);
                String destino = partes[0];
                String cuerpo = partes[1];

                String inboxDestino = obtenerInbox(destino);

                BufferedWriter bw = new BufferedWriter(
                        new FileWriter(inboxDestino, true));
                bw.newLine();
                bw.write(usuario + ":" + cuerpo);
                bw.close();

                socket.close();
            }

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
import java.util.Scanner;

public class Cliente {

    public static void main(String[] args) {

        Scanner teclado = new Scanner(System.in);

        try {

            System.out.print("IP del servidor: ");
            String ip = teclado.nextLine();

            System.out.print("Puerto: ");
            int puerto = Integer.parseInt(teclado.nextLine());

            System.out.print("Usuario (salva/pascual/raquel): ");
            String usuario = teclado.nextLine();

            Socket socket = new Socket(ip, puerto);

            BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(
                    socket.getOutputStream(), true);

            // Enviar usuario
            out.println(usuario);

            // Recibir opciones
            System.out.println(in.readLine());
            System.out.println(in.readLine());

            // Elegir opción
            System.out.print("Opción: ");
            String opcion = teclado.nextLine();
            out.println(opcion);

            // OPCIÓN 1
            if (opcion.equals("1")) {

                int numMensajes = Integer.parseInt(in.readLine());
                System.out.println("Mensajes recibidos: " + numMensajes);

                for (int i = 0; i < numMensajes; i++) {
                    System.out.println(in.readLine());
                }

                socket.close();
                return;
            }

            // OPCIÓN 2
            if (opcion.equals("2")) {

                System.out.println("Usuarios disponibles:");
                System.out.println(in.readLine());

                System.out.print(
                        "Mensaje (destino:mensaje): ");
                out.println(teclado.nextLine());

                socket.close();
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Qué cumple esta solución

- ✔ Servidor multihilo real
- ✔ Gestión de inbox por usuario
- ✔ Lectura y escritura en ficheros .data
- ✔ Dos funcionalidades bien diferenciadas
- ✔ Protocolo seguido exactamente
- ✔ Código claro y nivel examen alto
