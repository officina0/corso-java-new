Riprendiamo il modulo creato precedentemente per la riproduzione di un video di YouTube per utilizzarlo in un ulteriore modulo che andremo ora a creare. L’insieme dei due moduli costituisce un semplice esempio di **applicazione modulare**. Modifichiamo quindi la classe `VCPlayer` del modulo `it.video.company` in modo tale che esponga un metodo (`startPlayer()`) per l’apertura della finestra di esecuzione del video:

```
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
    public void startPlayer() {
         Application.launch();
    }
} 
```

Per utilizzare il modulo `it.video.company` con altri moduli dobbiamo esportarlo in un file `jar` che verrà inserito nel `Module Path` del modulo client in modo che sia visibile ad esso. Per recuperare il `jar` di `it.video.company` dobbiamo infatti eseguire il build del progetto modulare `VideoModularApplication`, realizzato in precedenza, e recuperare il path che punta alla directory `dist` del progetto stesso.

All’interno di questa directory troveremo il jar di `it.video.company`. Seguendo gli stessi step per la creazione del progetto `VideoModularApplication` andiamo a creare un nuovo progetto modulare, `VideoClient`, dotato del descrittore `module-info.java`:

```
 module it.video.client {
    requires it.video.company;
} 
```

Il descrittore evidenzia come il nuovo modulo dichiari una dipendenza da `it.video.company`. Allo stato attuale l’IDE mostrerà una sottolineatura in rosso poichè non è ancora in grado di trovare il modulo indicato. Questa segnalazione è del tutto normale, non abbiamo ancora inserito all’interno del Module Path del progetto il `jar` contenente il modulo richiesto.

Prima di risolvere la dipendenza a livello di Module Path, definiamo il package `it.video.client` con al suo interno la seguente classe:

```
 package it.video.client;
import it.video.company.VCPlayer;
public class VideoClient {
    public static void main(String[] args) {
      VCPlayer vcPlayer = new VCPlayer();
      vcPlayer.startPlayer();
    }
} 
```

In questo modo abbiamo completato l’implementazione di `it.video.client`. Risolviamo ora il problema della dipendenza entrando nelle _Project Properties_ del progetto `VideoClient`. Possiamo aprire la finestre delle Project Properties attraverso un click con il tasto destro sul nome del progetto, seguito dalla scelta della voce “Properties” sul menù a tendina aperto come risultato dell’operazione.

Nella finestra “Project Properties” selezioniamo il `jar` del modulo richiesto come mostrato dall’immagine:

Figura 1. Module Application.

![Module Application](https://tbm-html.s3.amazonaws.com/app/uploads/2018/08/cap20_img1.png)

Una volta aggiunto `it.video.company` nel Module Path del progetto gli errori evidenziati in precedenza scompaiono in quanto il nuovo modulo è ora in grado d’individuare la dipendenza indicata nel suo descrittore.

Possiamo quindi effettuare il `Run` di `it.video.client` ottenedo l’apertura della finestra di esecuzione del video mostrata nel capitolo precedente.

Concludiamo evidenziando l’importante differenza tra il concetto di `CLASSPATH` pre Java 9 e il nuovo concetto di **MODULE PATH** introdotto con Java 9, e quanto quest’ultimo sia estremamente più robusto rispetto ad errori dovuti, ad esempio, all’inserimento nel `CLASSPATH` di `jar` contenenti stesse librerie ma con versioni differenti.

Sappiamo che per poter utilizzare una determinata classe dobbiamo inserire nel `CLASSPATH` il percorso al file `jar` contenente la classe desiderata, ma cosa succede se inseriamo nel `CLASSPATH` due `jar` di una determinata libreria corrispondenti a due sue versioni differenti?

La risposta è che andremo ad utilizzare quella trovata per prima dal Class Loader Java. In una situazione di questo tipo la versione utilizzata potrebbe non essere quella desiderata dall’applicativo ottenendo, come risultato, errori del tipo `NoClassDefFoundErrors` o `NoSuchMethodErrors` con conseguente fallimento a Runtime.

Da questo punto di vista la robustezza della progettazione modulare risiede proprio nel fatto che un modulo dichiara esplicitamente la dipendenza da un altro modulo e che non è possibile avere due moduli omonimi o che esportino lo stesso package. Questa caratteristica impedisce di avere situazioni pericolose come quella descritta in precedenza.