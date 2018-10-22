La classe `java.lang.Process` mette a disposizione metodi per la comunicazione tra processi consentendo il collegamento tra stream di input ed output di un processo. Altre caratteristiche sono la possibilità di attesa sulla terminazione di un processo, verifiche sullo stato e terminazione. La definizione generale della classe `Process` è la seguente:

```
 public abstract class Process extends Object {
    public Process();
    public abstract void destroy();
    public abstract int exitValue();
    public abstract InputStream getErrorStream();
    public abstract InputStream getInputStream();
    public abstract OutputStream getOutputStream();
    public abstract int waitFor();
} 
```

I metodi che meritano particolare attenzione sono quelli per il recupero di uno stream di input/output sul processo creato, al fine di ricevere ed inviare dati da e verso il processo stesso.

Nello specifico il metodo `getInputStream()` restituisce uno stream per il processo corrente collegato allo standard output del processo creato.Il metodo `getOutputStream()` restituisce uno stream, sempre per il processo corrente, collegato però allo standard input del processo creato.

Un esempio d’uso interessante è la creazione di un **processo per l’esecuzione del comando tasklistdi Windows**:

```
 ..
        import java.io.BufferedReader;
        import java.io.IOException;
        import java.io.InputStream;
        import java.io.InputStreamReader;
        ..
        ProcessBuilder processBuilder = new ProcessBuilder("tasklist");
        Process process = processBuilder.start();
        System.out.println("Tasklist Process ID	 : " + process.pid());
        InputStream stdin = process.getInputStream();
        InputStreamReader reader = new InputStreamReader(stdin);
        try (BufferedReader buffer = new BufferedReader(reader)) {
            String line;
            while ((line = buffer.readLine()) != null) {
                System.out.println(line);
            }
        } 
```

Il primo passo consiste nell’utilizzare la classe `ProcessBuilder` per la definizione di un processo. La definizione prevede almeno la specifica del nome del programma di cui il processo sarà un’istanza in esecuzione. Il processo viene quindi effettivamente creato e un riferimento alla classe `Process` ottenuto, attraverso l’esecuzione del metodo `start()`.

Successivamente si recupera l’InputStream collegato allo stream di output del processo `tasklist`, si realizza un chain per una lettura bufferizzata dei caratteri, ed infine si stampano i dati ottenuti:

Figura 1. Esecuzione processo Tasklist di Windows.

![Esecuzione processo Tasklist di Windows](https://tbm-html.s3.amazonaws.com/app/uploads/2018/03/cap10_img1.png)

Con Java 9 la classe `java.lang.Process` è stata arricchita di nuovi metodi attraverso l’introduzione della nuova interfaccia `ProcessHandle`:

```
 public abstract class Process extends Object{
 ..
 Stream<ProcessHandle> children()
 Stream<ProcessHandle> descendants()
 long pid()
 ProcessHandle.Info info()
 CompletableFuture<Process> onExit()
 boolean supportsNormalTermination()
 ProcessHandle toHandle()
 ..
 } 
```

Una volta creato un processo, possiamo utilizzare il metodo `toHandle()` per recuperare un riferimento di tipo `ProcessHandle` e ottenere un insieme di informazioni sul processo stesso:

```
 ..
 import java.io.IOException;
 import java.lang.ProcessHandle.Info;
 import java.time.Duration;
 import java.time.Instant;
 ..
 ProcessBuilder processBuilder = new ProcessBuilder("tasklist");
 Process process = processBuilder.start();
 ProcessHandle processHandle = process.toHandle();
 Info processInfo = processHandle.info();
 System.out.println(processInfo.command().orElse("Nessun path"));
 String[] arguments = processInfo.arguments().orElse(new String[]{});
 for(String value : arguments) {
            System.out.println(value);
 }
 System.out.println(processInfo.user().orElse("Nessun proprietario"));
 System.out.println(processInfo.startInstant().orElse(Instant.now()).toString());
 System.out.println(processInfo.totalCpuDuration().orElse(Duration.ofMillis(0)).toMillis() + " millisecondi");
 .. 
```

Il codice stamperà informazioni sul percorso di esecuzione del programma, eventuali argomenti passati, proprietario, istante di esecuzione e durata. Utili sono i metodi `children()` e `descendants()` per ottenere informazioni su processi figli, ed eventuali loro discendenti, creati dal processo Java corrente.Supponiamo di voler creare un processo per il programma `cmd` e uno per il `tasklist` recuperandoli successivamente da una lista:

```
 ..
import java.io.IOException;
import java.lang.ProcessHandle.Info;
import java.time.Duration;
import java.time.Instant;
..
ProcessBuilder processBuilder1 = new ProcessBuilder("tasklist");
ProcessBuilder processBuilder2 = new ProcessBuilder("cmd");
Process process1 = processBuilder1.start();
Process process2 = processBuilder2.start();
ProcessHandle.current().children().forEach((p) -> printInfo(p));
..
private static void printInfo(ProcessHandle processHandle){
        Info processInfo = processHandle.info();
        System.out.println(processInfo.command().orElse("No Path"));
        String[] arguments = processInfo.arguments().orElse(new String[]{});
        for(String value : arguments) {
            System.out.println(value);
        }
        System.out.println(processInfo.user().orElse("No Owner"));
        System.out.println(processInfo.startInstant().orElse(Instant.now()).toString());
        System.out.println(processInfo.totalCpuDuration().orElse(Duration.ofMillis(0)).toMillis() + " milliseconds");
} 
```

In questo caso utilizziamo il metodo statico `current()` per ottenere un ProcessHandle associato al processo corrente. Invochiamo su questo riferimento il metodo `children()` per ottenere la lista di oggetti ProcessHandle relativi ai processi precedentemente creati.

Eseguiamo quindi su ciascun ProcessHandle il metodo di stampa `printInfo()`. All’interno di esso recuperiamo l’oggetto `Info` associato al ProcessHandle corrente, estraiamo da esso una serie di informazioni associate al processo e stampiamo infine le informazioni recuperate sulla console di output.

Figura 2. Utilizzo ProcessHandle.

![Utilizzo ProcessHandle](https://tbm-html.s3.amazonaws.com/app/uploads/2018/03/cap10_img2.png)