## 📁 Ficheros necesarios en el servidor

En el mismo directorio donde se ejecute el servidor:

Usuarios_autorizados.txt

juan
ana
pedro


Contrasenyas_autorizadas.txt

1234
abcd
qwerty


Las líneas deben ser correlativas (usuario línea N ↔ contraseña línea N).

## 📦 Clase para el objeto Usuario (Serializable)

👉 Obligatoria, ya que el cliente envía un objeto al servidor.

Usuario.java
```java
import java.io.Serializable;

public class Usuario implements Serializable {

    private String nombre;
    private String contrasena;

    public Usuario(String nombre, String contrasena) {
        this.nombre = nombre;
        this.contrasena = contrasena;
    }

    public String getNombre() {
        return nombre;
    }

    public String getContrasena() {
        return contrasena;
    }
}
```

## 🖥️ Servidor multihilo

Servidor.java
```java
import java.net.*;
import java.io.*;

public class Servidor {

    public static final int PUERTO = 6000;

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
        cargarDatos();
    }

    private void cargarDatos() {
        usuarios = new ArrayList<>();
        contrasenas = new ArrayList<>();

        try (BufferedReader brU = new BufferedReader(new FileReader("Usuarios_autorizados.txt"));
             BufferedReader brC = new BufferedReader(new FileReader("Contrasenyas_autorizadas.txt"))) {

            String linea;
            while ((linea = brU.readLine()) != null) {
                usuarios.add(linea);
            }

            while ((linea = brC.readLine()) != null) {
                contrasenas.add(linea);
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void run() {

        try (
            ObjectInputStream ois = new ObjectInputStream(socket.getInputStream());
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()))
        ) {

            // 1. Recibir objeto Usuario
            Usuario usuario = (Usuario) ois.readObject();

            int indice = usuarios.indexOf(usuario.getNombre());

            if (indice == -1 ||
                !contrasenas.get(indice).equals(usuario.getContrasena())) {

                out.println("ERROR");
                socket.close();
                return;
            }

            out.println("200 OK");

            // 2. Recibir número de líneas
            int numLineas = Integer.parseInt(in.readLine());

            // 3. Crear fichero y avisar preparado
            BufferedWriter bw = new BufferedWriter(new FileWriter("contenido.txt"));
            out.println("PREPARADO");

            // 4. Recibir líneas
            for (int i = 0; i < numLineas; i++) {
                String linea = in.readLine();
                System.out.println("Recibido: " + linea);
                bw.write(linea);
                bw.newLine();
            }

            bw.close();

            // 5. FIN CLIENTE / FIN SERVIDOR
            String finCliente = in.readLine();
            if ("FIN CLIENTE".equals(finCliente)) {
                out.println("FIN SERVIDOR");
            }

            socket.close();

        } catch (IOException | ClassNotFoundException e) {
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
    public static final int PUERTO = 6000;

    public static void main(String[] args) {

        try (
            Socket socket = new Socket(HOST, PUERTO);
            ObjectOutputStream oos = new ObjectOutputStream(socket.getOutputStream());
            BufferedReader in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            PrintWriter out = new PrintWriter(socket.getOutputStream(), true);
            Scanner teclado = new Scanner(System.in)
        ) {

            // Usuario y contraseña
            System.out.print("Usuario: ");
            String usuario = teclado.nextLine();

            System.out.print("Contraseña: ");
            String contrasena = teclado.nextLine();

            Usuario u = new Usuario(usuario, contrasena);
            oos.writeObject(u);

            // Respuesta autenticación
            String respuesta = in.readLine();
            if (!"200 OK".equals(respuesta)) {
                System.out.println("Autenticación incorrecta");
                return;
            }

            // Número de líneas
            System.out.print("Número de líneas a enviar: ");
            int numLineas = Integer.parseInt(teclado.nextLine());
            out.println(numLineas);

            // Esperar PREPARADO
            respuesta = in.readLine();
            if (!"PREPARADO".equals(respuesta)) {
                return;
            }

            // Envío de líneas
            for (int i = 0; i < numLineas; i++) {
                System.out.print("Línea " + (i + 1) + ": ");
                out.println(teclado.nextLine());
            }

            // Fin de comunicación
            out.println("FIN CLIENTE");
            System.out.println(in.readLine());

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Qué cumple esta solución

* ✔ Servidor multihilo
* ✔ Autenticación mediante objeto serializado
* ✔ Uso de ObjectInputStream / ObjectOutputStream
* ✔ Protocolo seguido paso a paso
* ✔ Escritura en fichero contenido.txt
* ✔ Comunicación clara FIN CLIENTE / FIN SERVIDOR