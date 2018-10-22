Dopo aver analizzato la struttura generale di un modulo e le direttive utilizzabili al suo interno, andiamo a creare un’ applicazione modulare contenente un singolo modulo. Utilizzeremo Netbeans per la creazione e l’esecuzione del progetto, ricordando che ogni operazione è eseguibile anche da riga di comando tramite i comandi Java.

Il modulo d’esempio ha come obiettivo quello di fornire la funzionalità di esecuzione di un video recuperato dal Web. Il primo passaggio è definire un’applicazione modulare nella quale andremo ad inserire il modulo. Utilizziamo il menù “file” di Netbeans scegliendo “New Project”.

Figura 1. Module Application.

![Module Application](https://tbm-html.s3.amazonaws.com/app/uploads/2018/07/cap19_img1.png)

Nella successiva schermata scegliamo “Java Modular Project”:

Figura 2. Module Applicatio.n

![Module Application](https://tbm-html.s3.amazonaws.com/app/uploads/2018/07/cap19_img2.png)

Inseriamo quindi un nome per l’applicazione e clicchiamo su “Finish”:

Figura 3. Module Application.

![Module Application](https://tbm-html.s3.amazonaws.com/app/uploads/2018/07/cap19_img3.png)

L’applicazione creata è priva di moduli, possiamo crearne uno facendo click con il tasto destro del mouse sul nome dell’applicazione e scegliendo “New->Module”:

Figura 4. Module Application.

![Module Application](https://tbm-html.s3.amazonaws.com/app/uploads/2018/07/cap19_img4.png)

La successiva schermata ci invita a definire un nome per il modulo. Possiamo inserire `it.video.company` che rispetta la naming convention per la definizione del nome di un modulo (identica a quella per i nomi di package):

Figura 5. Module Application.

![Module Application](https://tbm-html.s3.amazonaws.com/app/uploads/2018/07/cap19_img5.png)

Clicchiamo su “Finish” e andiamo ad espandere la struttura del modulo nel progetto:

Figura 6. Struttura di un modulo.

![Struttura di un modulo](https://tbm-html.s3.amazonaws.com/app/uploads/2018/07/cap19_img6.png)

La cartella `classes` conterrà tutti i package realizzati con il modulo, mentre nel package di default trova posto il file `module-info.java` di descrizione del modulo.

Per poter realizzare un lettore di video presi dal Web, ad esempio video di Youtube, possiamo far uso dei moduli nei quali risiedono le API JavaFX. Andiamo quindi a dichiarare nel file `module-info.java` le dipendenze di cui abbiamo bisogno utilizzando la direttiva `requires`:

```
 module it.video.company {
    	requires javafx.graphics;
    	requires javafx.media;
    	requires javafx.web;
     } 
```

Vogliamo sottolineare l’inserimento automatico e silenzioso della dipendenza dal modulo `java.base`, dal quale dipende ogni altro modulo creato e che contiene i package delle API fondamentali di Java.

Avendo dichiarato tutte le dipendenze possiamo passare alla realizzazione del package e della classe che esportaremo per il suo uso da parte di altri moduli. Nel definire i package in un modulo è convezione utilizzare il nome del modulo stesso come punto di partenza. Nel nostro caso definiremo un solo package, ragion per cui utilizzeremo il nome dato al modulo in fase di creazione. Creiamo quindi il package `it.video.company` e al suo interno inseriamo la classe `VCPlayer`:

```
 package it.video.company;
import javafx.application.Application;
import javafx.scene.Scene;
import javafx.scene.web.WebView;
import javafx.stage.Stage;
public class VCPlayer extends Application {
    private static final String 
            MEDIA_URL = "https://www.youtube.com/watch?v=G58XWF6B3AA";
    @Override
    public void start(Stage stage) {
        stage.setTitle("Video Company Media Player");
        WebView webview = new WebView();
        webview.getEngine().load(
                MEDIA_URL
        );
        webview.setPrefSize(640, 390);
        stage.setScene(new Scene(webview));
        stage.show();
    }
    public static void main(String[] args) {
        Application.launch(args);
    }
} 
```

Il package e la classe non sono automaticamente visibili all’esterno, abbiamo la necessità di dichiarare la sua visibilità ad altri moduli con la direttiva `exports`:

```
 module it.video.company {
    	requires javafx.graphics;
    	requires javafx.media;
    	requires javafx.web;
    	exports it.video.company;
     } 
```

Possiamo eseguire il modulo con un click con il tasto destro sul nome `VideoModularApplication` e scegliere la voce di menu “Run”:

Figura 7. Esecuzione del modulo.

![Esecuzione del modulo](https://tbm-html.s3.amazonaws.com/app/uploads/2018/07/cap19_img7.png)

Netbeans consente la visualizzazione del grafo delle dipendenze di un modulo, per visualizzarlo è sufficiente aprire il file di descrizione del modulo `module-info.java` e cliccare su “Graph”:

Figura 8. Grafo delle dipendenze.

![Grafo delle dipendenze](https://tbm-html.s3.amazonaws.com/app/uploads/2018/07/cap19_img8.png)

Nel successivo capitolo vedremo come esportare in formato jar il modulo realizzato per l’utilizzo con altri moduli.