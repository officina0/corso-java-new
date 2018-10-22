La **reactive programming** definisce un modello basato sul processo asincrono di uno stream di dati. Esso prevede due attori: il **publisher** che pubblica un flusso di dati e il **subscriber** pronto a riceverli tramite una registrazione asincrona.

Si tratta di una tipologia di programmazione particolarmente efficiente dal punto di vista dell’utilizzo della memoria. Il modello prevede inoltre la presenza di un terzo attore, **Processor**, che agisce sia come Publisher che come Subscriber ponendosi tra il publisher ed il subscriber dell’app al fine di realizzare trasformazioni sullo stream di dati.

Le Flow APIs di JDK implementano la reactive programming attraverso le seguenti interfacce:

```
 @FunctionalInterface   
public static interface Flow.Publisher<T> {  
    public void    subscribe(Flow.Subscriber<? super T> subscriber);  
}   
public static interface Flow.Subscriber<T> {  
    public void    onSubscribe(Flow.Subscription subscription);  
    public void    onNext(T item) ;  
    public void    onError(Throwable throwable) ;  
    public void    onComplete() ;  
}   
public static interface Flow.Subscription {  
    public void    request(long n);  
    public void    cancel() ;  
}   
public static interface Flow.Processor<T,R>  extends Flow.Subscriber<T>, Flow.Publisher<R> {  
} 
```

Il metodo `subscribe()` dell’interfaccia Publisher consente di registrare un Subscriber. I metodi dell’interfaccia Subscriber sono invece metodi di callback invocati al verificarsi di particolari eventi sullo stream di dati.

Il metodo `onSubscribe()` viene invocato per primo la sua esecuzione si attiva nel momento della registrazione del Subscriber presso un Publisher generatore di uno stream. Tale metodo prevede un riferimento all’interfaccia Subscription che rappresenta il link di comunicazione tra Publisher e Subscriber.

Un Subscriber utilizza l’oggetto Subscription ottenuto dinamicamente, invocando su di esso il metodo `request()` con cui si attiva la richiesta di un determinato numero di dati.

La richiesta viene ricevuta dal Publisher che, in modo asincrono, attiva l’esecuzione del metodo di callback `onNext()`, ma ossono essere necessarie ulteriori request per il recupero di nuovi dati: si consideri una situazione in cui il Subscriber richiede un dato alla volta all’interno di un stream contenente N di dati, appare chiara la necessità di invocare `request()` in `onNext()` per richiedere il dato successivo a quello appena ricevuto.

La notifica per un Subscriber della conclusione di un flusso di dati avviene attraverso l’esecuzione del metodo `onComplete()`:

Figura 1. Reactive Streams.

![Reactive Streams](https://tbm-html.s3.amazonaws.com/app/uploads/2018/05/cap14_image1.png)

Vediamo adesso un esempio di implementazione della recative programming utilizzando le API Java. Immaginiamo di avere oggetti di una classe `Mail` e di voler generare un flusso di dati email, attraverso Publisher, che desideriamo venga processato da un Subscriber:

```
 public class Mail {
    private String address;
    private String message;
    public Mail(String address, String message) {
      this.address = address;
      this.message = message;
    }
    //metodi get/set
} 
```

Creiamo quindi una classe Subscriber che recuperi un oggetto alla volta dallo stream:

```
 import java.util.concurrent.Flow.Subscriber;
import java.util.concurrent.Flow.Subscription;
public class MailSubscriber implements Subscriber<Mail>{
    private Subscription subscription;
    @Override
    public void onSubscribe(Subscription subscription) {
        this.subscription = subscription;
        subscription.request(1);
    }
    @Override
    public void onNext(Mail mail) {
        System.out.println("Indirizzo:" + mail.getAddress() + " Testo:" + mail.getMessage());
        subscription.request(1); 
    }
    @Override
    public void onError(Throwable throwable) {
        throwable.printStackTrace();  
    }
    @Override
    public void onComplete() {
        System.out.println("Processo mail completato"); 
    }
} 
```

Per la realizzazione del Publisher facciamo uso dell’implementazione esistente `java.util.concurrent.SubmissionPublisher`, possiamo quindi realizzare il seguente programma dimostrativo:

```
 import java.util.Arrays;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Flow;
import java.util.concurrent.SubmissionPublisher;
public class PublishSubscribeDemo1 {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(1);
        try (
            //Creazione del publisher
            SubmissionPublisher<Mail> publisher = 
                    new SubmissionPublisher<>(executor, Flow.defaultBufferSize())) {
            //Creazione e registrazione del subscriber
            MailSubscriber subscriber = new MailSubscriber();
            publisher.subscribe(subscriber);
            Mail[] mailData = new Mail[]{
                new Mail("mail1@mail.it", "Testo 1"),
                new Mail("mail2@mail.it", "Testo 2")
            };
            //Pubblicazione lista mail
            Arrays.asList(mailData).stream().forEach(mail -> publisher.submit(mail));
            executor.shutdown();
        }
    }
} 
```

Per la notifica asincrona un Publisher utilizza uno o più Thread all’interno di un oggetto `ExecutorService`, definiamo quindi un Publisher che riceva un oggetto di questo tpo specificando la classe `Mail` nel riferimento generics. Successivamente istanziamo e registriamo un oggetto Subscriber utilizzando la classe relativa alla nostra implementazione.

In questa fase Subscription viene creato e passato al Subscriber. Realizziamo quindi uno stream di oggetti Mail che il Publisher provvede a pubblicare. Se eseguiamo il programma avremo un output del tipo:

```
 Indirizzo:mail1@mail.it Testo:Testo 1
Indirizzo:mail2@mail.it Testo:Testo 2
Processo mail completato 
```

L’esempio conclude l’introduzione al modello Publish-Subscribe della reactive programming in Java, in seguito approfondiremo l’uso di un Processor per la trasformazione di stream.