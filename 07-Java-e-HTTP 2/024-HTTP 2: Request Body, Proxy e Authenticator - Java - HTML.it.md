Nella costruzione di una **POST Request** per impostare il `BODY` si utilizza l’interfaccia `HttpResponse.BodyProcessor<T>`. Il suo impiego è abbastanza semplice e intuitivo. Vediamo alcuni esempi per l’invio di una stringa, uno stream di input o un file all’interno del BODY HTTP/2. L’invio di una stringa è realizzabile attraverso poche istruzioni:

```
 import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import jdk.incubator.http.HttpClient;
import jdk.incubator.http.HttpClient.Version;
import jdk.incubator.http.HttpRequest;
import jdk.incubator.http.HttpResponse;
import jdk.incubator.http.HttpResponse.BodyHandler; 
..
      HttpClient client = HttpClient
               .newBuilder()
               .version(Version.HTTP_2)
               .build();
       HttpRequest httpRequest = HttpRequest.newBuilder()
               .uri(new URI("https://postman-echo.com/post"))
               .headers("Content-Type", "text/plain;")
               .POST(HttpRequest.BodyProcessor.fromString("Hello HTTP/2"))
               .build();
       HttpResponse<String> httpResponse = client.send(httpRequest, BodyHandler.asString());
       String body = httpResponse.body();
       System.out.println(body); 
```

Postman è un eco service molto interessante per testare servizi REST inviando richieste `GET`, `POST` e `PUT`, quindi particolarmente utile per gli esempi proposti. Considerando il codice d’esempio precedente, cambiamo la tipologia di costruzione del `BODY` attraverso l’inserimento di uno stream di byte recuperato da una stringa:

```
 import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import jdk.incubator.http.HttpClient;
import jdk.incubator.http.HttpClient.Version;
import jdk.incubator.http.HttpRequest;
import jdk.incubator.http.HttpResponse;
import jdk.incubator.http.HttpResponse.BodyHandler;
..
byte[] bytes = "Hello HTTP/2".getBytes();
HttpRequest httpRequest = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/post"))
  .headers("Content-Type", "text/plain;")
  .POST(HttpRequest.BodyProcessor
  .fromInputStream(() -> new ByteArrayInputStream(bytes)))
  .build(); 
```

Si può anche inserire il flusso di byte all’interno del `BODY` variando il codice precedente con l’utilizzo di un `ByteArrayProcessor`:

```
 byte[] bytes = "Hello HTTP/2".getBytes();
HttpRequest httpRequest = HttpRequest.newBuilder()
  .uri(new URI("https://postman-echo.com/post"))
  .headers("Content-Type", "text/plain;")
  .POST(HttpRequest.BodyProcessor.fromByteArray(bytes))
  .build(); 
```

Supponiamo ora di voler inserire nel `BODY` della request il contenuto di un particolare file. L’operazione è abbastanza lineare e richiede di specificare il path del file come input del `BodyProcessor`:

```
 import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import java.nio.file.Paths;
import jdk.incubator.http.HttpClient;
import jdk.incubator.http.HttpClient.Version;
import jdk.incubator.http.HttpRequest;
import jdk.incubator.http.HttpResponse;
import jdk.incubator.http.HttpResponse.BodyHandler;
 ..
 HttpRequest httpRequest = HttpRequest.newBuilder()
               .uri(new URI("https://postman-echo.com/post"))
               .headers("Content-Type", "text/plain;")
               .POST(HttpRequest.BodyProcessor.fromFile(
                       Paths.get("/body.txt")))
               .build(); 
```

Nei casi in cui si ha la necessità di contattare un Proxy Server, possiamo costruire il client HTTP specificando un Proxy Server di default:

```
 import java.io.IOException;
import java.net.ProxySelector;
import java.net.URI;
import java.net.URISyntaxException;
import java.nio.file.Paths;
import jdk.incubator.http.HttpClient;
import jdk.incubator.http.HttpClient.Version;
import jdk.incubator.http.HttpRequest;
import jdk.incubator.http.HttpResponse;
import jdk.incubator.http.HttpResponse.BodyHandler;
..
HttpClient client = HttpClient
               .newBuilder()
               .proxy(ProxySelector.getDefault())
               .version(Version.HTTP_2)
               .build(); 
```

Possiamo cambiare il Server Proxy di default utilizzando il metodo `setDefault(ProxySelector)` della classe `ProxySelector`. Nel caso in cui il Server da contattare richieda della credenziali di autenticazione, possiamo facilmente costruire una client che fornisca uno username ed una password attraverso gli oggetti `Authenticator` e `PasswordAuthentication`:

```
 import java.io.IOException;
import java.net.Authenticator;
import java.net.PasswordAuthentication;
import java.net.ProxySelector;
import java.net.URI;
import java.net.URISyntaxException;
import java.nio.file.Paths;
import jdk.incubator.http.HttpClient;
import jdk.incubator.http.HttpClient.Version;
import jdk.incubator.http.HttpRequest;
import jdk.incubator.http.HttpResponse;
import jdk.incubator.http.HttpResponse.BodyHandler;
..
HttpClient client = HttpClient.newBuilder()
    .authenticator(new Authenticator() {
        @Override
        protected PasswordAuthentication getPasswordAuthentication() {
            return new PasswordAuthentication("myuser", "mypassword".toCharArray());
        }
    })
    .build(); 
```

Nel prossimo capitolo concluderemo la trattazione delle API HTTP/2 con lo studio delle **richieste asincrone**.