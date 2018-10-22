Come programmatori Java siamo probabilmente abituati a legare il termine **StackTrace** ad un contenuto informativo associato ad un oggetto di tipo `java.lang.Exception`. Ricordiamo infatti che la classe `Exception`, attraverso il metodo `StackTraceElement[] getStackTrace()`, fornisce un accesso, a livello di codice di programmazione, alle informazioni visualizzabili con una chiamata al classico `printStackTrace()`.

In realtà un oggetto `StackTraceElement` rappresenta un concetto più generale denominato **Stack Frame**. Nello specifico un oggetto `StackTraceElement`, e quindi un singolo Stack Frame, rappresenta un’invocazione di metodo. Le informazioni relative ad invocazioni successive di metodi che originano dall’invocazione di un singolo metodo, vengono collocate in una struttura a pila (Stack) gestita con politica _Lifo(Last In First Out)_.

Uno Stack è un’area di memoria limitata, ma accessibile velocemente, dove trovano spazio tutte quelle strutture dati che vengono create e cancellate automaticamente. All’interno dello Stack trovano posto, ad esempio, i parametri di un metodo e le variabili locali utilizzate all’interno del metodo stesso non allocate dinamicamente (memoria Heap). Evidenziamo che ogni Thread possiede la sua area di Stack privata separata da tutti gli altri Thread.

In ambito Java un oggetto `StackTraceElement` consente l’accesso alle informazioni della pila di chiamate utili per analizzare il comportamento di un programma. Vediamo un esempio di utilizzo che stampi la catena di chiamate che origina dall’invocazione di un singolo metodo. Realizziamo preliminarmente le seguenti classi:

```
 public class C {
    public void methodC() {..}
}
public class B {
    public void methodB(){
       C c = new C();
       c.methodC();
    }
}
public class A {
    public void methodA(){
       B b = new B();
       b.methodB();
    }
}
public class AppWalk {
    public static void main(String[] args){
     A a = new A();
     a.methodA();
    }
} 
```

Completiamo il metodo della classe `C` aggiungendo il codice che recupera ed utilizza uno StackTrace:

```
 for (StackTraceElement stackTraceElem : new Throwable().getStackTrace()){
           System.out.println("Class:"+stackTraceElem.getClassName());
           System.out.println("Method:"+stackTraceElem.getMethodName());
           System.out.println("File name:"+stackTraceElem.getFileName());
           System.out.println("Line number:"+stackTraceElem.getLineNumber());
   } 
```

se eseguiamo l’applicativo otterremo una stampa simile alla seguente:

```
 Class:it.html.stackwalk.C
Method:methodC
File name:C.java
Line number:18
Class:it.html.stackwalk.B
Method:methodB
File name:B.java
Line number:19
Class:it.html.stackwalk.A
Method:methodA
File name:A.java
Line number:19
Class:it.html.stackwalk.AppWalk
Method:main
File name:AppWalk.java
Line number:15 
```

Quali sono i problemi legati a questo approccio? Il problema principale è relativo al suo impatto in termini di performance. La Java Virtual Machine cattura sempre un’istantanea dell’intero StackTrace (ad eccezione di stack frames nascosti), un’operazione che risulta inefficiente in tutti quei casi in cui si è interessati ad una porzione ridotta dello StackTrace, si è infatti costretti a consumare del tempo computazionale per processare stack frames che non sono significativi.

Altro aspetto importante è l’impossibilità di accedere all’istanza `java.lang.Class` della classe che dichiara il metodo rappresentato dallo Stack Frame corrente. Per poter accedere all’oggetto `Class` siamo costretti ad estendere la classe `java.lang.SecurityManager` e invocare il metodo `getClassContext()`.

Java 9 supera queste problematiche con la classe thread-safe `java.lang.StackWalker` che rende efficiente processare lo Stack filtrando gli Stack Frames ed introducendo l’accesso all’istanza della classe `Class` associata all’invocazione di un metodo.

L’utilizzo di uno StackWalker è molto semplice, sostituiamo il codice precedentemente inserito con:

```
 ..
import static java.lang.StackWalker.Option.RETAIN_CLASS_REFERENCE;
import java.util.List;
import java.util.stream.Collectors;
..
..
   StackWalker sw = StackWalker.getInstance(RETAIN_CLASS_REFERENCE);
   List<StackWalker.StackFrame> stackFrames = sw.walk(frames-> frames.limit(2).collect(Collectors.toList()));
      stackFrames.forEach(frame-> {
       System.out.println(frame.getClassName());
       System.out.println(frame.getMethodName());
       System.out.println(frame.getDeclaringClass());
      }); 
```

l’esempio evidenzia l’utilizzo di un filtro che limita il processo dello Stack alle ultime due chiamate della pila mostrando inoltre il recupero un oggetto della classe `Class` attraverso il metodo `getDeclaringClass()`.

L’oggetto all’interno dell’espressione lambda è di tipo `StackWalker.StackFrame` e consente l’accesso al contenuto di uno Stack Frame che non si limita a quanto mostrato nell’esempio. Tra le Informazioni accessibili di uno Stack Frame troviamo:

*   **Bytecode index**: l’indice dell’istruzione bytecode relativa all’inizio del metodo.
*   **Class name**: il nome della classe che dichiara il metodo chiamato
*   **Declaring class**: il’oggetto Class della classe che dichiara il metodo chiamato.
*   **Method name**: il nome del metodo chiamato.
*   **Is native**: informazione sul metodo, nativo oppure no.

Concludiamo la trattazione evidenziando le opzioni possibili per il tipo di istanza `StackWalker` ottenibile con il metodo `StackWalker.getInstance()`:

*   Nessuna opzione specificata.
*   `StackWalker.Option.RETAIN_CLASS_REFERENCE`.
*   `StackWalker.Option.SHOW_HIDDEN_FRAMES`.
*   `StackWalker.Option.SHOW_REFLECT_FRAMES`.

La prima opzione salta tutti gli Stack Frames nascosti e non recupera riferimenti ad oggetti `Class`. La seconda consente il recupero di riferimenti `Class`. La terza permette la visualizzazione degli Stack Frames nascosti mentre la quarta i frames di riflessione evitando però la visualizzazione di quelli nascosti.

Le opzioni possono essere combinate come mostra il seguente esempio:

```
 ..
StackWalker.getInstance(Set.of(RETAIN_CLASS_REFERENCE, SHOW_HIDDEN_FRAMES));
.. 
```