# 📡 Programación de comunicaciones en red (Java)

## 1. Conceptos básicos

### Cliente / Servidor

* **Servidor**: espera peticiones, procesa datos y responde.
* **Cliente**: inicia la comunicación, envía peticiones y espera respuestas.
* Un **servidor debe ser multifil** para atender varios clientes a la vez.

```java
while(true) {
  Socket s = servidor.accept();
  new Thread(new Peticion(s)).start();
}
```

### Arquitectura TCP/IP

* **IP** → identifica la máquina.
* **Puerto** → identifica la aplicación.
* Java usa **TCP** principalmente mediante *sockets*.

---

## 2. Clases Java para red (`java.net`)

### URL

Sirve para acceder a recursos web (HTTP).

Clases y métodos importantes:

* `URL(String url)`
* `openStream()` → devuelve `InputStream`
* `openConnection()` → devuelve `URLConnection`

⚠️ Limitación: solo recursos accesibles por URL.

---

## 3. Sockets

### Socket (cliente)

* Conecta con un servidor remoto.

```java
Socket s = new Socket(host, puerto);
```

### ServerSocket (servidor)

* Escucha peticiones entrantes.

```java
ServerSocket ss = new ServerSocket(puerto);
Socket cliente = ss.accept();
```

### Tipos

* **TCP (orientado a conexión)**: `Socket`, `ServerSocket`
* **UDP (no orientado a conexión)**: `DatagramSocket`

👉 En el examen: **TCP casi seguro**.

---

## 4. Establecimiento de conexión

### Flujo típico

**Servidor**:

```java
ServerSocket ss = new ServerSocket(5000);
Socket s = ss.accept();
```

**Cliente**:

```java
Socket s = new Socket("localhost", 5000);
```

Ambos usan:

* `InputStream` → recibir
* `OutputStream` → enviar

---

## 5. Comunicación cliente-servidor

### A) Comunicación con Strings (la más usada en exámenes)

#### Enviar

```java
PrintWriter pw = new PrintWriter(socket.getOutputStream(), true);
pw.println("mensaje");
```

#### Recibir

```java
BufferedReader br = new BufferedReader(
  new InputStreamReader(socket.getInputStream())
);
String msg = br.readLine();
```

⚠️ **Muy importante**: usar `\n` y `readLine()`.

---

### B) Comunicación con objetos

* Las clases deben implementar `Serializable`.

```java
ObjectOutputStream oos = new ObjectOutputStream(socket.getOutputStream());
oos.writeObject(obj);

ObjectInputStream ois = new ObjectInputStream(socket.getInputStream());
Objeto o = (Objeto) ois.readObject();
```

⚠️ El orden de creación de streams **debe coincidir** en cliente y servidor.

---

## 6. Ejemplo típico de examen: Servidor de cálculo

### Protocolo

1. Operación (`+`)
2. Número 1
3. Número 2
4. Servidor responde con el resultado

Todo terminado en `\n`.

### Validaciones habituales

* Si no es número → usar 0
* Máx 8 cifras por número

```java
try {
  num = Integer.parseInt(linea);
} catch (NumberFormatException e) {
  num = 0;
}
```

---

## 7. Servidor multifil (MUY IMPORTANTE)

### Idea clave

👉 **Un cliente = un hilo**

### Estructura

**Servidor principal**:

```java
while(true) {
  Socket s = server.accept();
  new Thread(new Peticion(s)).start();
}
```

**Clase Peticion**:

```java
public class Peticion implements Runnable {
  Socket socket;
  public void run() {
    // leer
    // procesar
    // responder
  }
}
```

⚠️ Nunca hacer todo dentro del `while` sin hilos.

---

## 8. Errores comunes en el examen ❌

* No hacer `flush()`
* Cliente y servidor no leen en el mismo orden
* Olvidar `\n`
* No cerrar sockets/streams
* Bloqueo por `readLine()` esperando datos

---

## 9. WebSocket vs Socket (teórico)

| Socket TCP              | WebSocket           |
| ----------------------- | ------------------- |
| Bajo nivel              | Alto nivel          |
| Streams                 | Mensajes            |
| Gestión manual de hilos | Hilos automáticos   |
| No nativo web           | Nativo en navegador |

👉 Para el práctico: **Socket TCP clásico**.

---

## 🧠 Checklist mental para el examen

✔ ServerSocket → accept()
✔ Socket cliente → connect()
✔ BufferedReader + PrintWriter
✔ `\n` + `flush()`
✔ Hilos
✔ Validar entradas

---

# 🧩 Ejemplos completos de aplicaciones Socket en Java

> Ejemplos **realistas de examen**, listos para copiar, entender y adaptar.

---

## 🟢 EJEMPLO 1: Cliente / Servidor básico (Strings)

### 📌 Servidor (eco: devuelve lo que recibe)

```java
import java.io.*;
import java.net.*;

public class ServidorEco {

  public static void main(String[] args) throws IOException {
    ServerSocket servidor = new ServerSocket(5000);
    System.out.println("SERVIDOR >> Esperando cliente...");

    Socket cliente = servidor.accept();
    System.out.println("SERVIDOR >> Cliente conectado");

    BufferedReader br = new BufferedReader(
        new InputStreamReader(cliente.getInputStream()));
    PrintWriter pw = new PrintWriter(cliente.getOutputStream(), true);

    String mensaje = br.readLine();
    System.out.println("SERVIDOR >> Recibido: " + mensaje);

    pw.println("Eco: " + mensaje);

    cliente.close();
    servidor.close();
  }
}
```

---

### 📌 Cliente

```java
import java.io.*;
import java.net.*;

public class ClienteEco {

  public static void main(String[] args) throws IOException {
    Socket socket = new Socket("localhost", 5000);

    PrintWriter pw = new PrintWriter(socket.getOutputStream(), true);
    BufferedReader br = new BufferedReader(
        new InputStreamReader(socket.getInputStream()));

    pw.println("Hola servidor");

    String respuesta = br.readLine();
    System.out.println("CLIENTE >> Respuesta: " + respuesta);

    socket.close();
  }
}
```

---

## 🟡 EJEMPLO 2: Servidor de cálculo (tipo examen)

### 📌 Servidor

```java
import java.io.*;
import java.net.*;

public class ServidorCalcul {

  static int parseNumero(String s) {
    try {
      int n = Integer.parseInt(s);
      if (n >= 100000000) return 0;
      return n;
    } catch (NumberFormatException e) {
      return 0;
    }
  }

  public static void main(String[] args) throws IOException {
    ServerSocket servidor = new ServerSocket(6000);
    System.out.println("SERVIDOR >> Escuchando...");

    Socket cliente = servidor.accept();

    BufferedReader br = new BufferedReader(
        new InputStreamReader(cliente.getInputStream()));
    PrintWriter pw = new PrintWriter(cliente.getOutputStream(), true);

    String op = br.readLine();
    String n1 = br.readLine();
    String n2 = br.readLine();

    int num1 = parseNumero(n1);
    int num2 = parseNumero(n2);
    int resultado = 0;

    if (op.charAt(0) == '+') {
      resultado = num1 + num2;
    }

    pw.println(resultado);

    cliente.close();
    servidor.close();
  }
}
```

---

### 📌 Cliente

```java
import java.io.*;
import java.net.*;

public class ClienteCalcul {

  public static void main(String[] args) throws IOException {
    Socket socket = new Socket("localhost", 6000);

    PrintWriter pw = new PrintWriter(socket.getOutputStream(), true);
    BufferedReader br = new BufferedReader(
        new InputStreamReader(socket.getInputStream()));

    pw.println("+");
    pw.println("100");
    pw.println("50");

    String resultado = br.readLine();
    System.out.println("Resultado: " + resultado);

    socket.close();
  }
}
```

---

## 🔵 EJEMPLO 3: Servidor MULTIHILO (imprescindible)

### 📌 Servidor principal

```java
import java.net.*;

public class ServidorMultiFil {

  public static void main(String[] args) throws Exception {
    ServerSocket servidor = new ServerSocket(7000);
    System.out.println("SERVIDOR >> Multihilo activo");

    while (true) {
      Socket cliente = servidor.accept();
      new Thread(new Peticion(cliente)).start();
    }
  }
}
```

---

### 📌 Clase Peticion

```java
import java.io.*;
import java.net.*;

public class Peticion implements Runnable {

  Socket socket;

  public Peticion(Socket socket) {
    this.socket = socket;
  }

  public void run() {
    try {
      BufferedReader br = new BufferedReader(
          new InputStreamReader(socket.getInputStream()));
      PrintWriter pw = new PrintWriter(socket.getOutputStream(), true);

      String msg = br.readLine();
      pw.println("Servidor responde a: " + msg);

      socket.close();
    } catch (IOException e) {
      e.printStackTrace();
    }
  }
}
```

---

## 🟣 EJEMPLO 4: Envío de OBJETOS

### 📌 Clase Serializable

```java
import java.io.Serializable;

public class Persona implements Serializable {
  String nombre;
  int edad;

  public Persona(String n, int e) {
    nombre = n;
    edad = e;
  }
}
```

---

### 📌 Servidor

```java
import java.io.*;
import java.net.*;

public class ServidorObjeto {

  public static void main(String[] args) throws Exception {
    ServerSocket servidor = new ServerSocket(8000);
    Socket cliente = servidor.accept();

    ObjectOutputStream oos = new ObjectOutputStream(cliente.getOutputStream());
    ObjectInputStream ois = new ObjectInputStream(cliente.getInputStream());

    Persona p = new Persona("Anonimo", 0);
    oos.writeObject(p);

    Persona pMod = (Persona) ois.readObject();
    System.out.println(pMod.nombre + " " + pMod.edad);

    cliente.close();
    servidor.close();
  }
}
```

---

### 📌 Cliente

```java
import java.io.*;
import java.net.*;

public class ClienteObjeto {

  public static void main(String[] args) throws Exception {
    Socket socket = new Socket("localhost", 8000);

    ObjectInputStream ois = new ObjectInputStream(socket.getInputStream());
    ObjectOutputStream oos = new ObjectOutputStream(socket.getOutputStream());

    Persona p = (Persona) ois.readObject();
    p.nombre = "Jose";
    p.edad = 30;

    oos.writeObject(p);

    socket.close();
  }
}
```

---

## 🧠 Plantilla mental rápida (para el examen)

1️⃣ ServerSocket
2️⃣ accept()
3️⃣ Streams
4️⃣ Protocolo claro (orden)
5️⃣ flush()
6️⃣ (Opcional) hilos

---

## 🔁 `flush()` y `autoFlush` en sockets (Java)

Cuando enviamos datos por un socket usando `PrintWriter`, los datos **no se envían inmediatamente**.  
Primero se guardan en un **buffer**, y solo se envían cuando ese buffer se vacía mediante un `flush()`.

---

### ✅ Uso de `autoFlush` (opción recomendada)
```java
    PrintWriter pw = new PrintWriter(socket.getOutputStream(), true);
    pw.println("mensaje");
```

- El segundo parámetro (`true`) activa el **autoFlush**
- Cada vez que se usa:
  - `println()`
  - `printf()`
- Se hace un `flush()` automático
- No es necesario llamar a `flush()` manualmente

👉 Es la opción más segura en exámenes, sobre todo cuando el receptor usa `readLine()`.

---

### ⚠️ Uso sin `autoFlush` (flush manual)
```java
    PrintWriter pw = new PrintWriter(socket.getOutputStream());
    pw.println("mensaje");
    pw.flush();
```

- El `PrintWriter` **NO envía los datos automáticamente**
- Es obligatorio llamar a `flush()`
- Si no se hace, el mensaje no llega al receptor

---

### ❌ Error típico de examen
```java
    PrintWriter pw = new PrintWriter(socket.getOutputStream());
    pw.println("mensaje");   // ❌ No se envía todavía
    // falta pw.flush();
```
Este error provoca que:
- El otro lado se quede bloqueado en `readLine()`
- El programa parezca colgado

---

### 🧠 Regla de oro para el examen

O usas `autoFlush = true`,  
o llamas siempre a `flush()` manualmente.

Nunca envíes datos sin asegurar el vaciado del buffer.
