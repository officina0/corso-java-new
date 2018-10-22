Non di rado sono state mosse critiche al linguaggio Java accusandolo di **eccessiva verbosità**. In effetti in alcune situazioni, come ad esempio quelle che richiedono la costruzione di collezioni non modificabili, il codice richiede l’implementazione di blocchi con diverse istruzioni che possono degenerare in qualcosa di complesso.

Vediamo alcuni esempi di implementazione di collezioni non modificabili con versioni Java precedenti alla Java 9. La creazione di una _List_ non modificabile in Java 8 può essere realizzata come mostra il seguente frammento di codice:

```
 ArrayList<String> modifiableList = new ArrayList<String>();
        modifiableList.add("1");
        modifiableList.add("2");
        List<String> unModifiableList = Collections.unmodifiableList(modifiableList); 
```

Il codice conserva una sufficiente leggibilità ma richiede quattro istruzioni per ottenere il risultato voluto. Un altro esempio interessante riguarda la creazione di un tipo `Set`:

```
 Set<String> immutableSet = Collections.unmodifiableSet(new HashSet<String>(Arrays.asList("1", "2"))); 
```

In questo caso le istruzioni si riducono a una ma la complessità aumenta sensibilmente. Con Java 8 possiamo utilizzare gli `Streams` per realizzare collezioni immutabili:

```
 Set<String> unmodifiableSet = Collections.unmodifiableSet(Stream.of("1","2").collect(Collectors.toSet())); 
```

Anche in questo caso abbiamo una sola istruzione la cui complessità aumenta rispetto alla soluzione precedente e gli `Streams` non sembrano aiutarci molto in questo contesto.

Vediamo altri interessanti esempi che coinvolgono la creazione di liste e insiemi immutabili:

```
 List<String> listA = Collections.unmodifiableList(new ArrayList<String>() {{ add("1"); add("b"); add("c"); }});
	List<String> listB = Collections.unmodifiableList(Stream.of("1", "2").collect(Collectors.toList())); 
```

Con la prima lista utilizziamo un’**anonymous inner class** per ridurre la verbosità. Si tratta di una tecnica che può risultare poco chiara, richiede una classe extra e può portare a problemi di memoria e serializzazione.

La seconda lista mostra un altro esempio di uso di Streams per ridurre la verbosità. Il problema che abbiamo in questo caso è la creazione di oggetti non necessari e un maggiore complessità computazionale non effettivamente necessaria.

Java integra ora dei metodi statici del tipo `of()` utilizzabili sulle interfacce `List`, `Set` e `Map` che ritornano in un modo molto semplice collezioni non modificabili. Ecco alcuni esempi d’uso:

```
 List<String> immutableList = List.of("1", "2", "3");
    Set<String> immutableSet = Set.of("one", "two", "three");
    Map<String , Integer> immutableMap = Map.of("one", 1, "two", 2); 
```

Facciamo inoltre notare un nuovo aspetto particolarmente importante: i valori `null` non sono più aggiungibili ad una `List` o `Set`.

Un tipo `Map` non consente il valore `null` né come chiave né come valore. Se proviamo ad eseguire il seguente frammento di codice:

```
 public class DemoCollectionFactory {
       public static void main(String[] args) {
         List<String> immutableList = List.of("one", null);
      }
     } 
```

Otterremo un errore del tipo `NullPointerException`:

```
 Exception in thread "main" java.lang.NullPointerException
	at java.base/java.util.Objects.requireNonNull(Objects.java:221)
	at java.base/java.util.ImmutableCollections$List2.<init>(ImmutableCollections.java:185)
	at java.base/java.util.List.of(List.java:822)
	at it.collectionfactory.DemoCollectionFactory.main(DemoCollectionFactory.java:48) 
```