**La serializzazione consiste nel convertire oggetti Java nella loro rappresentazione in byte**. In questo capitolo siamo interessati agli aspetti di sicurezza legati all’operazione inversa, la **deserializzazione**, cioè la ricostruzione di un oggetto dallo stream di byte frutto della sua serializzazione.

Le classi Java che implementano l’interfaccia `java.io.Serializable` sono serializzabili, le loro istanze sono convertibili in uno stream di byte. Iniziamo con l’introdurre del codice che mostri il processo di Serializzazione/Deserializzazione in Java:

```
 FileOutputStream fos = new FileOutputStream("myclass.ser");
      ObjectOutputStream oos = new ObjectOutputStream(fos);
      oos.writeObject(new MyClass());
      oos.close();
      fos.close();
      FileInputStream fis = new FileInputStream("myclass.ser");
      ObjectInputStream ois = new ObjectInputStream(fis);
      Object value = ois.readObject();
      fis.close();
      ois.close(); 
```

Le prime cinque righe serializzano un oggetto della classe `MyClass` sul file `myclass.ser`. Durante il processo di deserializzazione il metodo `readObject()` legge lo stream di byte e ricostruisce l’oggetto originario.

I **problemi di sicurezza** nascono nel momento in cui lo stream di byte viene intercettato da un attaccante in grado di modificarne il contenuto con l’intento di dare origine ad una specifica tipologia di attacco.

Immaginiamo di avere un metodo che utilizzi una costante definita a livello di classe come limite per la chiusura di un ciclo. Se un attaccante riuscisse a modificare i byte dello stream in modo da inserire un valore estremamente grande per tale costante, il ciclo potrebbe durare un tempo considerevole e portare anche ad una saturazione della memoria.

Si consideri la seguente classe:

```
 public class MyClass implements Serializable {
     private final int MAX = 5;
     private Integer[] values = new Integer[MAX];
     public MyClass(){
      for(int i=0;i<MAX ;i++)
          values[i] =1;}
    } 
```

Un attacco malevolo in fase di deserializzazione di un oggetto di questa classe che modificasse il valore della costante `MAX`, potrebbe portare ad un overflow di memoria e **mandare in crash l’applicativo**.

Come cercare di fronteggiare questo problema? L’approccio pre-Java 9, comunemente utilizzato, consiste nel definire una lista di classi per le quali è consentita la deserializzazione (_whitelist_) o una lista di classi per le quali la deserializzazione deve essere bloccata (_blacklist_), perchè note come classi con problemi di sicurezza.

L’utilizzo congiunto di entrambe le soluzioni è possibile. L’implementazione di questa tecnica richiede di estendere la classe `ObjectInputStream`, effettuare l’override del metodo `resolveClass()`, ed inserire in esso la logica di controllo relativa alla sicurezza. Nel nostro caso scegliamo di inserire in _blacklist_ `MyClas`s perchè siamo a conoscenza di problemi di sicurezza nella sua implementazione:

```
 public class LookAheadObjectInputStream extends ObjectInputStream {
    	public LookAheadObjectInputStream(InputStream inputStream) throws IOException {
        	super(inputStream);
    	}
    	@Override
    	protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException,
            ClassNotFoundException {
       		String name = desc.getName();
        	if (isInBlacklist(name)) {
            throw new SecurityException("Deserialization is blocked for security reasons");
        	}
        	return super.resolveClass(desc);
    	}
    	private boolean isInBlacklist(String name) {
        	return name.equals("it.html.serialization.MyClass");
    	}
    } 
```

Modifichiamo quindi l’istruzione di creazione dell’`ObjectInputStream` del codice iniziale, in modo tale che utilizzi la nuova classe di gestione della sicurezza:

```
 ..
   ObjectInputStream ois = new LookAheadObjectInputStream(fis);
   .. 
```

Se eseguiamo il codice di inizio capitolo non avremo alcun errore, mentre la sua esecuzione con l’istruzione precedente produrrà l’eccezione:

```
 Exception in thread "main" java.lang.SecurityException: Deserialization is blocked for security reasons 
```

Java include ora un nuovo meccanismo per la gestione dei problemi di sicurezza legati alla deserializzazione denominato **Serialization Filtering**. Si può utilizzare questa nuova caratteristica implementando l’intefaccia `java.io.ObjectInputFilter`, definendo cosi un filtro per lo stream ottenuto dalla deserializzazione.

L’interfaccia `ObjectInputFilter` richiede l’implementazione del metodo `Status checkInput(FilterInfo filterInfo)`. Il tipo di ritorno `Status` rappresenta un’enumerazione, definita all’interno dell’interfaccia, con i seguenti possibili valori: `Status.ALLOWED`, `Status.REJECTED` o `Status.UNDECIDED`.

Vediamo come utilizzarli per implementare il codice di un filtro di sicurezza per la classe `MyClass`. Sapendo che il numero di byte dello stream di deserializzazione della classe `MyClass` con valore `MAX=5` è pari a 230 byte, realizziamo un filtro di sicurezza per questo valore.

Decidiamo di sfruttare il metodo `streamBytes()` del parametro `FilterInfo`, che restituisce il numero di byte dello stream, per effettuare un test che impedisca la deserializzazione di oggetti della classe `MyClass` che superino questo valore:

```
 public class MyClassFilter implements ObjectInputFilter {
    private final int MAX_BYTES = 230;
    public Status checkInput(FilterInfo filterInfo) {
        if (filterInfo.streamBytes() > MAX_BYTES) {
            return Status.REJECTED;
        }
        if (filterInfo.serialClass() == null) {
            return Status.UNDECIDED;
        }
        return Status.ALLOWED;
    }
} 
```

Utilizzare il filtro è molto semplice, è sufficiente infatti modificare il codice di inizio capitolo in modo tale da includere l’istruzione:

```
 ois.setObjectInputFilter(new MyClassFilter()); 
```

Prima che venga eseguito il metodo `readObject()`.