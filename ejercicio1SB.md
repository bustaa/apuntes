## App:

```java
package tema5.springboot.cuadernoAPIs.ejercicio1;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class, args);
    }
}

```

## UserController:
```java
package tema5.springboot.cuadernoAPIs.ejercicio1.controllers;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import tema5.springboot.cuadernoAPIs.ejercicio1.models.User;

import java.io.File;
import java.io.IOException;

@RestController
public class UserController {
    private final ObjectMapper mapper = new ObjectMapper();

    @PostMapping("/verne/registro")
    public ResponseEntity<Void> guardarUsuario(@RequestBody User user) {
        user.setOnline("false");
        String path = "./src/main/resources/usuarios/";
        String fileName = user.getUser() + ".json";

        try {
            mapper.writerWithDefaultPrettyPrinter().writeValue(new File(path + fileName), user);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return ResponseEntity.status(204).build();
    }

   @PostMapping("/verne/login")
   public ResponseEntity<Void> loginUsuario(@RequestBody User user) throws IOException {
        File dirUsuarios = new File("./src/main/resources/usuarios");
        for (File file : dirUsuarios.listFiles()) {
            if (file.getName().substring(0, file.getName().length() - 5).equals(user.getUser())) {
                User posibleUser = mapper.readValue(file, User.class);
                if (posibleUser.getPassword().equals(user.getPassword())) {
                    posibleUser.setOnline("true");
                    mapper.writerWithDefaultPrettyPrinter().writeValue(file, posibleUser);
                    return ResponseEntity.status(204).build();
                }
            }
        }
        return ResponseEntity.status(401).build();
   }

   @GetMapping("/verne/logout")
    public ResponseEntity<Void> logoutUsuario(@RequestParam String usuario) throws IOException {
        File dirUsuarios = new File("./src/main/resources/usuarios");
        for (File file : dirUsuarios.listFiles()) {
            if (file.getName().substring(0, file.getName().length() - 5).equals(usuario)) {
                User userData = mapper.readValue(file, User.class);
                if (userData.getOnline().equals("true")) {
                    userData.setOnline("false");
                    System.out.println("ddfdd");
                    mapper.writerWithDefaultPrettyPrinter().writeValue(file, userData);
                    return ResponseEntity.status(204).build();
                }
            }
        }
        return ResponseEntity.status(401).build();
    }
}

```

## DestinoController:
```java
package tema5.springboot.cuadernoAPIs.ejercicio1.controllers;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import tema5.springboot.cuadernoAPIs.ejercicio1.models.Destino;
import tema5.springboot.cuadernoAPIs.ejercicio1.models.Filtro;
import tema5.springboot.cuadernoAPIs.ejercicio1.models.User;

import java.io.File;
import java.io.IOException;

@RestController
public class DestinoController {
    private final ObjectMapper mapper = new ObjectMapper();

    @GetMapping("/verne/filtroPrecio")
    public ResponseEntity<Object> filtroPrecio(
            @RequestParam String usuario,
            @RequestParam int precio,
            @RequestParam String tipo
    ) throws IOException {
        File dirUsuarios = new File("./src/main/resources/usuarios");
        for (File file : dirUsuarios.listFiles()) {
            if (file.getName().substring(0, file.getName().length() - 5).equals(usuario)) {
                User userData = mapper.readValue(file, User.class);
                if (userData.getOnline().equals("true")) {
                    String tipoLimpio = tipo.replaceAll("[{}]", "");
                    String[] valores = tipoLimpio.split(",");
                    boolean mayor_ig = Boolean.parseBoolean(valores[0].trim());
                    boolean menor_ig = Boolean.parseBoolean(valores[1].trim());
                    File dirDestions = new File("./src/main/resources/destinos");
                    String destinos = "";
                    for (File destino : dirDestions.listFiles()) {
                        Destino destObj = mapper.readValue(destino, Destino.class);
                        if (mayor_ig && destObj.getPrecio() >= precio) destinos += destObj.getDestino() + ", ";
                        if (menor_ig && destObj.getPrecio() <= precio) destinos += destObj.getDestino() + ", ";
                    }
                    if (destinos.isEmpty()) {
                        return ResponseEntity.status(404).build();
                    } else {
                        Filtro filtro = new Filtro();
                        filtro.setDestinos(destinos);
                        return ResponseEntity.status(200).body(filtro);
                    }
                }
            }
        }

        return ResponseEntity.status(401).build();
    }
}

```