Le API Java consentono di **ridirigere i log interni delle JDK verso una specifica piattaforma di logging** (Log4j2, SLF4J…). Java fornisce servizi per il logging che consentono alle applicazioni di configurare le JDK in modo tale che esse, e l’applicazione di riferimento, utilizzino il framework di logging desiderato.

Immaginando di avere un’applicazione che faccia uso della libreria di logging SLF4J, vedremo come collegare il log interno della classe `java.net.URLConnection` alla libreria SLF4J. Il primo passo è considerare la classe statica definita all’interno della classe `java.lang.System`:

```
 public static abstract class LoggerFinder {
    public abstract Logger getLogger(String name, Module module);
} 
```

L’implementazione fornisce una classe per la selezione del sistema di logging che si intende utilizzare e che verrà registrata come servizio utilizzabile dal sistema interno di logging delle JDK. L’implementazione di `LoggerFinder` deve restituire un riferimento all’interfaccia `Logger`, anch’essa definita all’interno della classe `java.lang.System` che sarà utilizzata per il routing dei log interni delle JDK verso il framework di logging.

Iniziamo quindi con l’implementare l’interfaccia `Logger` inserendo le istruzioni riferite a SLF4J:

```
 package it.html.logging;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.ResourceBundle;
public class Slf4jRouterLogging implements System.Logger {
  private final Logger slf4jLogger;
  private final String name;
  public Slf4jRouterLogging (String name, Module module) {
      this.name = name;
      slf4jLogger = LoggerFactory.getLogger(module.getName() + "-" + name);
  }
  @Override
  public void log(Level level, ResourceBundle bundle, String msg, Throwable thrown) {
      slf4jLogger.trace(msg);
  }
  @Override
  public void log(Level level, ResourceBundle bundle, String format, Object... params) {
      slf4jLogger.trace(format, params);
  }
  @Override
  public String getName() {
      return name;
  }
  @Override
  public boolean isLoggable(Level level) {
      return true;
  }
} 
```

Utilizziamo quindi un oggetto della classe per l’implementazione di un `LoggerFinder`:

```
 package it.html.logging;
public class Slf4jLoggerFinder extends System.LoggerFinder {
  @Override
  public System.Logger getLogger(String name, Module module) {
      return new Slf4jRouterLogging (name, module);
  }
} 
```

Per registrare il `LogFinder` come servizio abbiamo bisogno di inserire un file `java.lang.System$LoggerFinder` nel `META-INF/services` di un progetto NetBeans. A tal fine creiamo una cartella `META-INF` nella cartella sorgenti `src` del progetto e al suo interno la cartella `services`.

Proseguiamo con la creazione del file `java.lang.System$LoggerFinder` nella cartella `services`. Editiamo `java.lang.System$LoggerFinder` con il full qualified name della classe LoggerFinder: `it.html.logging.Slf4jLoggerFinder`.

Inoltre, poichè intendiamo utilizzare la classe `java.net.URLConnection`, abbiamo bisogno di una configurazione del logging tramite il file `logging.properties`. Essp va inserito nel package contenente le classi di riferiemento dell’applicativo, nel nostro caso `it.html.logging`. Il contenuto da inserire all’interno del file di `properties` è:

```
 handlers= java.util.logging.ConsoleHandler
java.util.logging.ConsoleHandler.level = FINEST
sun.net.www.protocol.http.HttpURLConnection.level=ALL 
```

La configurazione termina con la definizione del formato dei messaggi di logging. A tal fine proseguiamo creando il file `logback.xml` in `src`, inserendo al suo interno:

```
 <?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss:SSS} %5p %t %c{2}:%L - %m%n</pattern>
        </encoder>
    </appender>
    <root level="TRACE">
        <appender-ref ref="stdout"/>
    </root>
</configuration> 
```

Affinchè tutto funzioni dobbiamo recuperare il file jar della libreria [SLF4J](https://www.slf4j.org/download.html) e il jar di [Logback](https://logback.qos.ch/download.html).

Concludiamo la configurazione inserendo nel classpath del progetto i file `logback-classic-1.2.3.jar`, `logback-core-1.2.3.jar` e `slf4j-api-1.7.25.jar`, estratti dagli Zip scaricati precedentemente. Possiamo ora realizzare una classe demo che mostri il routing del log nelle JDK:

```
 import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.URL;
import java.net.URLConnection;
public class LoggingDemo {
    public static void main(String[] args) throws Exception {
        String path = LoggingDemo.class.getClassLoader()
                .getResource("it/html/logging/logging.properties")
                .getFile();
        System.setProperty("java.util.logging.config.file", path);
        URL url = new URL("http://www.google.com/");
        URLConnection urlConnection = url.openConnection();
        try (BufferedReader bufferIn = new BufferedReader(new InputStreamReader(
                urlConnection.getInputStream()))) {
        }
    }
} 
```

L’output sarà:

Figura 1. Platform Logging API.

![Platform Logging API](https://tbm-html.s3.amazonaws.com/app/uploads/2018/04/cap13_img1.png)

In allegato il progetto Netbeans completo del codice mostrato fino al presente capitolo.