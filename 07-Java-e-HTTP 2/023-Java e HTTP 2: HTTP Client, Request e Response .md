Andiamo ora ad esplorare un altro aspetto introdotto dalle API Java 9: il supporto all’uso del protocollo **HTTP/2**. Per poter comprendere le funzionalità offerte da questa versione del linguaggio è opportuno spendere qualche parola sul funzionamento sia della release HTTP/1.1 che di HTTP/2.

Consideriamo il funzionamento di un browser con protocollo HTTP/1.1, al ricevimento di una pagina tutto ciò che al suo interno si riferisce ad una risorsa (CSS, JS, immagini..) viene recuperato tramite connessioni aperte in parallelo con l’obiettivo di ridurre la latenza sul caricamento della pagina.

Dobbiamo tener presente che una connessione non può essere riutilizzata prima che la risorsa richiesta attraverso essa sia stata ottenuta. Ma utilizzare connessione persistenti non è abbastanza per migliorare la latenza, le connessioni dedicate consumano risorse significative sia lato client che lato server, perciò i browser non consentono in genere di avere più di 8 connessioni aperte sullo stesso dominio.

HTTP/1.1 tenta di risolvere queste problematiche attraverso un funzionamento di tipo **Pipeline**: una successiva richiesta può essere inviata prima di aver ricevuto la risposta a quella precedente. Anche questa tecnica presenta però delle lacune, il server risponde infatti nello stesso ordine in cui le richieste sono state ricevute e, risposte computazionalmente onerose, richiederanno un tempo tale da bloccare le altre, magari molto più leggere, associate a richieste successive.

HTTP/2 offre una soluzione alle problematiche precedenti attraverso un meccanismo ottimizzato del trasporto della semantica di HTTP. L’obiettivo è stato mantenere un alto livello di compatibilità con HTTP/1.1 con una struttura di basso livello del protocollo di rete che risulta però essere differente.

HTTP/2 definisce infatti un protocollo di tipo binario basato sul **multiplexing**. Grazie ad esso è possibile inviare contemporaneamente molteplici richieste indipendenti, ottenendo le risposte, utilizzando anche una sola connessione verso il dominio.

L’aspetto estremamente è la possibilità di utilizzare una sola connessione per caricare un’intera pagina. Le unità base di HTTP/2 sono i frame che vengono scambiati su una connessione TCP ed un messaggio HTTP, prima di essere trasmesso, viene suddiviso in più frame.

Per utilizzare le API HTTP/2 creiamo un’applicazione modulare avendo cura di inserire nel file `module-info.java` la dipendenza necessaria:

```
 module it.html.java9.httpclient {   
  requires jdk.incubator.httpclient;
} 
```

A differenza del funzionamento basato su `HttpURLConnection`, le nuove API consentono richieste di tipo **sincrono e asincrono**. Le classi principali dal punto di vista del client sono le seguenti:

Classe

Descrizione

**jdk.incubator.http.HttpClient**

Funziona come un container che contiene informazioni comuni a diverse richieste.

**jdk.incubator.http.HttpRequest**

Una richiesta inviata attraverso l’HTTPClient.

**jdk.incubator.http.HttpResponse**

Il risultato di una richiesta inviata attraverso l’HTTPClient.

Vediamo come utilizzarle per inviare una semplice richiesta GET sincrona verso un URL. Definiamo preliminarmente un client che utilizzi HTTP/2:

```
 import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;
import jdk.incubator.http.HttpClient;
import jdk.incubator.http.HttpClient.Version;
import jdk.incubator.http.HttpRequest;
import jdk.incubator.http.HttpRequest.Builder;
import jdk.incubator.http.HttpResponse;
import jdk.incubator.http.HttpResponse.BodyHandler;
..
  HttpClient client = HttpClient
    .newBuilder()
    .version(Version.HTTP_2)
    .build();
.. 
```

Costruiamo quindi la richiesta utilizzando un oggetto `Builder` al quale passiamo l’URL da contattare:

```
 Builder builder = HttpRequest.newBuilder(new URI("http://www.google.com"));
    HttpRequest httpRequest = builder.GET().build(); 
```

Inviamo la richiesta e recuperiamo il body della risposta come oggetto `String`:

```
 HttpResponse httpResponse = client.send(httpRequest, BodyHandler.asString());
 String body = httpResponse.body();
 System.out.println(body); 
```

Possiamo eseguire le istruzioni precedenti all’interno del metodo `main` di una classe da noi definita all’interno del modulo creato. Otteniamo quindi la stampa dell’HTML ottenuto dalla risposta. Esiste la possibilità di aggiungere parametri alla richiesta inviata:

```
 Builder builder = HttpRequest.newBuilder(new URI("http://www.google.com"));
HttpRequest httpRequest = builder.headers("Accept", "text/plain", "Accept-Charset", "utf-8").GET().build(); 
```

oppure aggiungere un timeout per l’attesa di una risposta:

```
 import java.time.Duration;
import static java.time.temporal.ChronoUnit.SECONDS;
..
   Builder builder = HttpRequest.newBuilder(new URI("http://www.google.com"));
   HttpRequest httpRequest = builder.timeout(Duration.of(10, SECONDS)).headers("Accept", "text/plain", "Accept-Charset", "utf-8").GET().build();
.. 
```

Concludiamo con questo primo esempio, Continueremo analizzando altre tipologie di richieste HTTP e con la loro gestione asincrona.