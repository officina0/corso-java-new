Try-With-Resources
------------------

Lo statement `try-catch` con Java 8 è stato notevolmente potenziato per garantire la chiusura automatica delle risorse aperte eliminando quindi la probabilità di errori e rendendo il codice più compatto.

Supponiamo di dover aprire e processare un file `txt` utilizzando la classe `FileReader`. Prima della versione 8 di Java avremmo scritto:

```
 public class TryCatchPreJava8 {
    public static void main(String[] args){
       FileReader reader = null;
       try {
           reader = new FileReader("c:\\myfile.txt");
           //do something
       }catch(FileNotFoundException e){
            //do something
       }catch (IOException ex) {
           //do something
       }finally{
          if(reader!=null){
              try {
                  reader.close();
              } catch (IOException ex) {
                  //do something
              }
          }
       }
    }
} 
```

Evidenziamo la necessità di un blocco `finally` per assicurarci la chiusura della risorsa precedentemente aperta. Con Java 8 scompare l’esigenza del blocco `finally` potendo quindi scrivere il codice precedente nella forma:

```
 public static void main(String[] args) {
       try(FileReader reader=new FileReader("c:\\myfile.txt")) {
           //do something
       } catch (FileNotFoundException ex) {
           //do something
       } catch (IOException ex) {
           //do something
       }
    } 
```

Ciò che Java 8 introduce è **l’auto-chiusura** di risorse aperte a livello dello statement `try-catch`. Che cosa si è dunque fatto con Java 9 per migliore ulteriormente il `try-catch`? Consideriamo il seguente codice ottenuto dal precedente aggiungendo un nuovo metodo e cambiando il contenuto del `main()`:

```
 public class TryCatchJava8 {
    public static void main(String[] args) {
       try { 
             processFile(new FileReader("c:\\myfile.txt"));
       } catch (FileNotFoundException ex) {
             //do something
       }
    }
    private static void processFile(FileReader reader){
       try(FileReader r=reader) {
           //do something
       } catch (FileNotFoundException ex) {
           //do something
       } catch (IOException ex) {
           //do something
       }
    }
} 
```

Vorremmo poter passare direttamente il parametro `reader`, effettivamente `final`, al blocco `try` senza eseguire una dichiarazione con inizializzazione. In Java 8 questo purtroppo non è possibile perchè la sintassi non lo prevede. Java 9 ha quindi modificato la sintassi di `try-catch` rendendo possibile il passaggio diretto di parametri `final`:

```
 public class TryCatchJava9 {
     public static void main(String[] args) {
         try { 
             processFile(new FileReader("c:\\myfile.txt"));
         } catch (FileNotFoundException ex) {
             //do something
         }
    }
    private static void processFile(FileReader reader){
       try(reader) {
           //do something
       } catch (IOException ex) {
           //do something
       }
    }
} 
```

Rimozione import warning deprecation
------------------------------------

Un’altra caratteristica introdotta dalla versione 9 di Java è l’eliminazione di _warning_ relativi all’importazione di classi o interfacce deprecate. Consideriamo il seguente esempio:

```
 import java.io.StringBufferInputStream;
@Deprecated
public class DeprecatedImport {
	StringBufferInputStream myStream;
	..
} 
```

La classe `StringBufferInputStream` è deprecata e il compilatore lo segnala con un _warning_. Ciò che possiamo fare in Java 8 è sopprimere soltanto _warning_ situati all’interno della dichiarazione di classe mediante annotation `@Deprecated`.

La classe utilizza correttamente il tipo deprecato, possiamo ad esempio immaginare che faccia parte di un vecchio progetto. Essendo quindi essa stessa deprecata non c’è bisogno di avere un _warning_ segnalato a livello di `import` per eventuali altre classi deprecate. Java 9 rimuove quindi la notifica di warning per `import` deprecati.