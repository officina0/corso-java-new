Con l’introduzione alla Reactive programming abbiamo visto come il modello ad esso associato preveda due attori principali: il _publisher_ che pubblica un flusso di dati ed il _subscriber_ pronto a riceverli attraverso una registrazione asincrona.

Proseguiamo la trattazione introducendo un terzo attore, denominato **Processor**, particolarmente importante per processare il flusso dati. Un _Processor_ può agire sia come _Publisher_ che come _Subscriber_ inserendosi esattamente tra le due entità principali. In questo scenario un _Processor_ si registra su un _Publisher_, emettitore di un flusso dati, divenendo cosi un _Subscriber_ nei confronti del _Publisher_ di origine.

Il _Subscriber_ finale non si registra più direttamente verso il _Publisher_ di origine, ma effettua la sua registrazione direttamente sul _Processor_ che diventa cosi, nei suoi confronti, un _Publisher_ emettitore di un flusso dati differente dall’originale.

Quanto appena detto è evidenziato dal seguente schema a blocchi:

Figura 1. Reactive Stream Processors.

![Reactive Stream Processors](https://tbm-html.s3.amazonaws.com/app/uploads/2018/05/cap14_image2.png)

Riprendiamo l’esempio della gestione di un flusso di email introducendo un semplice _Processor_ che operi sul flusso dati inserendo in ogni email una firma. Presentiamo il codice descrivendo le parti essenziali:

```
 import java.util.concurrent.ExecutorService;
import java.util.concurrent.Flow.Processor;
import java.util.concurrent.Flow.Subscription;
import java.util.concurrent.SubmissionPublisher;
import java.util.function.Function; 
public class MailProcessor extends SubmissionPublisher<Mail> implements Processor<Mail,Mail> {  
  private final Function<Mail,Mail> function;  
  private Subscription subscription;  
  private final ExecutorService executor;
  public MailProcessor(Function<Mail,Mail> function, ExecutorService executor, int maxBufferCapacity) {  
    super(executor,maxBufferCapacity);  
    this.function = function;  
    this.executor = executor;
  }  
  @Override  
  public void onSubscribe(Subscription subscription) {  
    this.subscription = subscription;  
    subscription.request(1);  
  }  
  @Override  
  public void onNext(Mail mail) {  
    submit(function.apply(mail));
    subscription.request(1);  
  }  
  @Override  
  public void onError(Throwable t) {  
    t.printStackTrace();  
  }  
  @Override  
  public void onComplete() {  
    System.out.println("Processo flusso mail completato"); 
    close();  
    executor.shutdown();
  }  
} 
```

Per realizzare un _Processor_ dobbiamo implementare l’interfaccia `java.util.concurrent.Flow.Processor` considerando il tipo generics per i flussi di dati _Publisher-Processor_ e _Processor-Subscriber_. Nel nostro caso il tipo di dati contenuto all’interno dei due flussi non cambia la sua natura, avremo sempre oggetti `Mail` ma con stato differente.

All’interno della classe facciamo notare la presenza di un riferimento `java.util.function.Function`. Esso rappresenta la funzione per il processo del flusso di dati proveniente dal _Publisher_ di origine.

I restanti metodi svolgono le azioni viste in precedenza per il _Subscriber_ classico. Abbiamo in particolare il metodo di registrazione presso il _Publisher_ di origine, ottenendo un oggetto `Subscription` che la rappresenta, ed il metodo `onNext()` per la ricezione dei dati. In quest’ultimo evidenziamo l’utilizzo della funzione per il processo con relativo submit verso il _Subscriber_ finale.

Andiamo avanti realizzando un semplice programma che illustri il funzionamento:

```
 import java.util.Arrays;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Flow;
import java.util.concurrent.SubmissionPublisher;
public class PublishSubscribeDemo2 {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor1 = Executors.newFixedThreadPool(1);
        ExecutorService executor2 = Executors.newFixedThreadPool(1);
        //Creazione e registrazione di un processore;
        try ( //Creazione di un publisher
                SubmissionPublisher<Mail> publisher = new SubmissionPublisher<>(executor1, Flow.defaultBufferSize())) {
            //Creazione e registrazione di un processore;
            MailProcessor mailProcessor = new MailProcessor(
                    mail -> {
                        mail.setMessage(mail.getMessage() + "[Firma per indirizzo:" + mail.getAddress()+"]");
                        return mail;
                    }, executor2, Flow.defaultBufferSize());
            publisher.subscribe(mailProcessor);
            //Creazione e registrazione di un subscriber
            MailSubscriber mailSubscriber = new MailSubscriber();
            mailProcessor.subscribe(mailSubscriber);
            Mail[] mailData = new Mail[]{
                new Mail("mail1@mail.it", "Testo mail 1"),
                new Mail("mail2@mail.it", "Testo mail 2")
            };
            //Pubblicazione lista mail
            Arrays.asList(mailData).stream().forEach(mail -> publisher.submit(mail));
        }
        executor1.shutdown();
    }
} 
```

Instanziamo due thread executor: uno per il _Publisher_ originario ed un altro per il _Processor_ che agirà anche come Publisher. Attraverso la classe `SubmissionPublisher` utilizziamo un’implementazione esistente nelle JDK 9 di un Publisher.

Creiamo quindi un’istanza del nostro _Processor_ definendo, attraverso lambda expression, la funzione di processo del flusso originario. Registriamo il _Processor_ sul _Publisher_ di origine attraverso l’istruzione `publisher.subscribe(mailProcessor)`.

Il _Processor_ è quindi registrato presso il _Publisher_ per la ricezione asincrona dei dati. Proseguiamo creando un oggetto della classe `MailSubscriber` rappresentativa del _Subscriber_ finale. Registriamo `MailSubscriber` con il _Processor_ utilizzando sempre il metodo `subscribe()`.

Il _subscriber_ finale è adesso registrato per la ricezione asincrona del flusso dati elaborato uscente dal _Processor_. Concludiamo l’implementazione definendo come flusso di dati un array di oggetti `Mail` pubblicati come stream dal _Publisher_ di origine. Eseguendo il programma all’interno di Netbeans avremo, per il _Subscriber_ finale, un output del tipo:

```
 Processo flusso mail completato
	Indirizzo:mail1@mail.it Testo:Testo mail 1[Firma per indirizzo:mail1@mail.it]
	Indirizzo:mail2@mail.it Testo:Testo mail 2[Firma per indirizzo:mail2@mail.it]
	Processo mail completato 
```

Che mostra il flusso processato ottenuto dal Subscriber.