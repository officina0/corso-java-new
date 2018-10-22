Per creare un’applicazione Java standalone seguiamo i passi che portano alla creazione di una classe che contiene il metodo `main`, inseriamo il codice Java necessario, compiliamo il programma, correggiamo gli eventuali errori di compilazione e infine mandiamo in esecuzione il programma.

Attraverso la compilazione ed esecuzione automatica di porzioni di codice Java la **JShell** elimina l’overhead relativo alla creazione di classi container. Su Microsoft Windows è possibile utilizzare la JShell da riga di comando aprendo una console e digitando:

```
 $JAVA_HOME/bin/jshell 
```

con la variabile `$JAVA_HOME` che si riferisce al percorso relativo ad un’installazione delle JDK 9. In questo, e nei successivi capitoli, non utilizzeremo però la console di Windows ma l’integrazione della JShell nella versione di Netbeans che abbiamo configurato nel primo capitolo. Mandiamo in esecuzione l’ambiente Netbeans e, dal menù “Tools” dell’IDE, selezioniamo “Open Java Platform JShell”:

Figura 1. JShell

![JShell](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap6_img1.png)

La JShell verrà aperta e inizializzata. La console accetta due tipi di input: il codice Java e i comandi propri della console.

Vediamo subito un esempio. Supponiamo di voler provare alcuni metodi della classe `String` per comprenderne il funzionamento. Senza la JShell avremmo realizzato una classe del tipo:

```
 public class StringTest {
     public static void main(String[] args) {
       String str = "12345";
       System.out.println(str.substring(0,2));
     }
    } 
```

Con la JShell la classe `StringTest` non è necessaria, è sufficiente inserire lo statement di dichiarazione della variabile `String`, `str`, e successivamente l’istruzione `System.out.println()`:

Figura 2. JShell primo esempio

![JShell primo esempio](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap6_img2.png)

Figura 3. JShell, seconda parte primo esempio

![JShell, seconda parte primo esempio](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap6_img3.png)

L’esecuzione della prima istruzione evidenzia la sintassi JShell che indica l’avvenuta creazione dell’oggetto `String` inizializzato con il valore specificato. L’esecuzione della seconda istruzione produce invece la stampa del risultato dell’invocazione del metodo `substring()` sulla console di output.

Analizziamo ora un esempio di errore mostrato dalla JShell e supponiamo di voler sommare due variabili intere di cui una non dichiarata:

Figura 4. JShell esempio di errore

![JShell esempio di errore](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap6_img4.png)

La sintassi del punto di errore viene indicata con il simbolo `^--^`. L’errore di compilazione indica chiaramente che stiamo utilizzando una variabile non dichiarata.

Un aspetto interessante della JShell è la presenza di una **history** degli input inseriti. Infatti possiamo recuperare la situazione precedente dichiarando la variabile `num2`:

Figura 5. JShell, esempio di uso della history

![JShell, esempio di uso della history](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap6_img5.png)

Eseguendo nuovamente la somma recuperandola dalla history attraverso i tasti freccia della tastiera possiamo notare la presenza di un numero preceduto dal simbolo `sharp` accanto ad ogni istruzione. Questo numero è un’identificatore univoco dell’istruzione all’interno della JShell. Se eseguiamo il comando `/list` possiamo infatti visualizzare le istruzioni eseguite con successo e l’identificatore associato:

Figura 6. JShell, comando list

![JShell, comando list](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap6_img6.png)

Possiamo utilizzare l’identificatore di uno statement per riferirci ad esso nell’ambito di una particolare operazione. Ad esempio se abbiamo la necessità di eseguire nuovamente uno statement precedentemente inserito, possiamo digitare l’ID e premere invio:

Figura 7. JShell, uso ID

![JShell, uso ID](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap6_img7.png)

Facciamo notare che durante l’uso della JShell è possibile avere blocchi o errori, si tratta infatti di una versione in fase di sviluppo.