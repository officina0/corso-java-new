Con la versione 7 di Java è stato introdotto l’**operatore Diamond** `<>` al fine di rendere il codice Generics più compatto e leggibile. Per comprendere l’utilità dell’operatore Diamond si consideri la semplice istanza di una mappa generica:

```
 Map<String, List<String>> values = new HashMap<String, List<String>>(); 
```

Questo esempio fa notare come la sintassi Generics tenda ad aumentare la lunghezza delle istruzioni con l’aumentare dell’intensità del suo utilizzo. Al problema dell’eccessiva lunghezza delle istruzioni si somma quello della leggibilità. L’esempio appena visto può assumere forme anche più complesse:

```
 Map<String, List<Map<String,String>>> values = new HashMap<String, List<Map<String,String>>>(); 
```

L’operatore Diamond migliora la stesura del codice grazie al meccanismo di **Type Inference** del compilatore Java. Con il termine “Type Inference” si indica la capacità del compilatore di eseguire un deduzione logica del tipo. Con riferimento agli esempi precedenti possiamo infatti scrivere le istruzioni nella seguente forma più compatta e leggibile:

```
 Map<String, List<String>> values = new HashMap<>();
       Map<String, List<Map<String,String>>> values = new HashMap<>(); 
```

Il compilatore deduce, sulla base della struttura Generics di dichiarazione, che le mappe istanziate sono del tipo di quelle dichiarate. Una limitazione dell’operatore Diamond con le versioni 7 ed 8 di Java, risiede nell’impossibilità di utilizzarlo nell’istanza di **classi anonime**. Riprendiamo l’esempio introdotto nel capitolo 3 per gli algoritmi di ordinazione numerica proponendo una nuova versione attraverso la seguente classe astratta:

```
 public abstract class Numeric<T extends Number> {
     public void bubbleSort(T[] numbers){
        int j;
        int i=0;
        int n = numbers.length;
        boolean finished;  
        do {
          i++;
          finished=true;
          for(j=n-1;j>=i;j--) {
           if(min(numbers[j],numbers[j-1])){
             swap(numbers, j, j-1);
             finished=false;
           }
          }
        } while (!finished || i<n-1);
     }
     private void swap(T[] numbers, int i, int j){
	  T temp;
      temp = numbers[i];
      numbers[i]=numbers[j];
      numbers[j]=temp;
     }
     private boolean min(T t1, T t2){
      return t1.doubleValue()<t2.doubleValue();
     }
     public abstract void quickSort(T[] numbers);
     public abstract void mergeSort(T[] numbers); 
```

Lo scopo della classe è quello di fornire diversi algoritmi di ordinazione per array parametrici in cui il parametro stesso è limitato superiormente dal tipo `Number`. La classe fornisce l’implementazione per l’algoritmo _bubble sort_ lasciando alle sottoclassi l’implementazione dei restanti algoritmi.

Una classe client per la quale sia più che sufficente l’algoritmo _bubble sort_, potrebbe istanziare una classe anonima lasciando vuote le implementazioni dei metodi astratti:

```
 Numeric<Number> numeric = new Numeric<Number>(){
            @Override
            public void quickSort(Number[] numbers) {}
            @Override
            public void mergeSort(Number[] numbers) {}
          }; 
```

Se a questo punto volessimo rendere l’istruzione più compatta con l’operatore Diamond:

```
 Numeric<Number> numeric = new Numeric<>(){....} 
```

avremmo un errore di compilazione: (_<\> cannot be used with anonymous classes_). Java 9 interviene “rilassando” l’utilizzo dell’operatore Diamond e consentendo istruzioni come la precedente. Concludiamo il capitolo mostrando il codice completo da inserire all’interno di un metodo main() di dimostrazione:

```
 Numeric<Number> numeric = new Numeric<>(){
        @Override
        public void quickSort(Number[] numbers) {}
        @Override
        public void mergeSort(Number[] numbers) {}
       };
       Number[] values = new Integer[]{5,2,1,4,3};
       numeric.bubbleSort(values);
       for(Number n: values) {
        System.out.println(n);
       } 
```