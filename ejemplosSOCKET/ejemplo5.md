## 🧠 Idea general (con objetos)

Usaremos estos objetos:

- PalabrasClave → lista de palabras clave

- Pregunta → número + texto

- Respuesta → texto de la respuesta

La comunicación será así:

- Servidor → Cliente: PalabrasClave

- Cliente → Servidor: String (palabra elegida)

- Servidor → Cliente: ArrayList<Pregunta>

- Cliente → Servidor: int (nº de pregunta)

- Servidor → Cliente: Respuesta

Todo con ObjectInputStream / ObjectOutputStream.

## 📦 Clases de objetos (Serializable)
PalabrasClave.java
```java
import java.io.Serializable;
import java.util.List;

public class PalabrasClave implements Serializable {
    private List<String> palabras;

    public PalabrasClave(List<String> palabras) {
        this.palabras = palabras;
    }

    public List<String> getPalabras() {
        return palabras;
    }
}
```

Pregunta.java
```java
import java.io.Serializable;

public class Pregunta implements Serializable {
    private int numero;
    private String texto;

    public Pregunta(int numero, String texto) {
        this.numero = numero;
        this.texto = texto;
    }

    public int getNumero() {
        return numero;
    }

    public String getTexto() {
        return texto;
    }
}
```

Respuesta.java
```java
import java.io.Serializable;

public class Respuesta implements Serializable {
    private String texto;

    public Respuesta(String texto) {
        this.texto = texto;
    }

    public String getTexto() {
        return texto;
    }
}
```

## 🖥️ Servidor multihilo
Servidor.java
```java
import java.net.*;
import java.io.*;

public class Servidor {

    public static final int PUERTO = 9000;

    public static void main(String[] args) {

        System.out.println("Servidor BOT con objetos iniciado...");

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
import java.util.*;

public class Hilo extends Thread {

    private Socket socket;

    public Hilo(Socket socket) {
        this.socket = socket;
    }

    private List<String> leerFichero(String nombre) throws IOException {
        List<String> lista = new ArrayList<>();
        BufferedReader br = new BufferedReader(new FileReader(nombre));
        String linea;
        while ((linea = br.readLine()) != null) {
            lista.add(linea);
        }
        br.close();
        return lista;
    }

    public void run() {

        try (
            ObjectOutputStream oos = new ObjectOutputStream(socket.getOutputStream());
            ObjectInputStream ois = new ObjectInputStream(socket.getInputStream())
        ) {

            while (true) {

                // PASO 2: enviar palabras clave
                List<String> palabras = leerFichero("palabras_clave.txt");
                oos.writeObject(new PalabrasClave(palabras));

                // PASO 3: recibir palabra clave
                String palabra = (String) ois.readObject();
                if ("fin".equalsIgnoreCase(palabra)) {
                    socket.close();
                    return;
                }

                // PASO 4: enviar preguntas
                List<String> lineasPreguntas = leerFichero("preguntas.txt");
                List<Pregunta> preguntas = new ArrayList<>();
                int contador = 0;

                for (String l : lineasPreguntas) {
                    String[] partes = l.split(";");
                    if (partes[0].equalsIgnoreCase(palabra)) {
                        preguntas.add(new Pregunta(contador++, partes[1]));
                    }
                }

                oos.writeObject(preguntas);

                // PASO 5: recibir número de pregunta
                int num = (int) ois.readObject();
                String textoPregunta = preguntas.get(num).getTexto();

                // PASO 6: buscar y enviar respuesta
                List<String> respuestas = leerFichero("respuestas.txt");
                for (String r : respuestas) {
                    String[] partes = r.split(";");
                    if (partes[0].equals(textoPregunta)) {
                        oos.writeObject(new Respuesta(partes[1]));
                        break;
                    }
                }
            }

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
import java.util.*;

public class Cliente {

    public static final String HOST = "localhost";
    public static final int PUERTO = 9000;

    public static void main(String[] args) {

        try (
            Socket socket = new Socket(HOST, PUERTO);
            ObjectOutputStream oos = new ObjectOutputStream(socket.getOutputStream());
            ObjectInputStream ois = new ObjectInputStream(socket.getInputStream());
            Scanner teclado = new Scanner(System.in)
        ) {

            while (true) {

                // PASO 2: recibir palabras clave
                PalabrasClave pc = (PalabrasClave) ois.readObject();
                System.out.println("\nPalabras clave:");
                for (String p : pc.getPalabras()) {
                    System.out.println("- " + p);
                }

                // PASO 3: elegir palabra
                System.out.print("Elige palabra (fin para salir): ");
                String palabra = teclado.nextLine();
                oos.writeObject(palabra);

                if ("fin".equalsIgnoreCase(palabra)) {
                    socket.close();
                    return;
                }

                // PASO 4: recibir preguntas
                @SuppressWarnings("unchecked")
                List<Pregunta> preguntas = (List<Pregunta>) ois.readObject();

                System.out.println("\nPreguntas:");
                for (Pregunta p : preguntas) {
                    System.out.println(p.getNumero() + " - " + p.getTexto());
                }

                // PASO 5: elegir pregunta
                System.out.print("Número de pregunta: ");
                oos.writeObject(Integer.parseInt(teclado.nextLine()));

                // PASO 6: recibir respuesta
                Respuesta r = (Respuesta) ois.readObject();
                System.out.println("\nRespuesta:");
                System.out.println(r.getTexto());
            }

        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

## ✅ Por qué esta versión es mejor para examen

- ✔ Uso correcto de objetos serializados
- ✔ Clases claras y bien separadas
- ✔ Servidor multihilo real
- ✔ Comunicación estructurada, no solo texto
- ✔ Fácil de justificar si te preguntan “¿por qué usar objetos?”

### 👉 Frase típica de examen:

“Uso objetos serializados para encapsular la información y facilitar la comunicación cliente-servidor.”