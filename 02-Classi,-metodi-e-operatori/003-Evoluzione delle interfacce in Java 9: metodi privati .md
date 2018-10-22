Con Java 8 le interfacce hanno subito notevoli cambiamenti rispetto alla loro definizione originaria. Si è passati infatti dalla consolidata concezione di interfaccia come costrutto nel quale è possibile inserire esclusivamente definizioni di metodi, alla possibilità di aggiungere implementazioni attraverso le parole chiave `static` e `default`.

Si è inoltre esplicitato il concetto di **interfaccia funzionale come interfaccia che prevede la definizione di un solo metodo**. Un’interfaccia funzionale può essere marcata con l’annotation `@FunctionalInterface` da applicare sul nome dell’interfaccia stessa. Il beneficio dell’uso dell’annotation risiede nell’errore di compilazione che si ottiene se si tenta di aggiungere ulteriori definizioni di metodi.

A partire dalla versione 8 di Java un’interfaccia con un solo metodo è automaticamente un’interfaccia funzionale. Le implementazioni dei metodi `static` o `default` sono caratterizzate dall’obbligo dell’uso del modificatore di accesso `public`. Si tratta di un vincolo che può creare disagi a livello di programmazione come vedremo tra qualche istante.

Java 9 supera la problematica attraverso la possibilità di poter **applicare il modificatore di accesso `private` ad implementazioni di metodi**.

Supponiamo di voler realizzare un’interfaccia per l’ordinamento di array di interi. Vogliamo dotarla di vari metodi di ordinamento in base al particolare algoritmo che il client dell’interfaccia desidera utilizzare ma, allo stesso tempo, riteniamo utile inserire come default l’implementazione del più semplice algoritmo di ordinamento: _Bubble Sort_.

Vediamo come avremmo realizzato l’interfaccia con Java 8, per semplicità abbiamo omesso controlli sui dati di input per la rilevazione di errori:

```
 public interface ISortInt {
      public void quickSort(int[] integers);
      public void mergeSort(int[] integers);
      public default void bubbleSort(int[] integers){
        int j;
        int i=0;
        int n = integers.length;
        boolean finished;  
        do {
          i++;
          finished=true;
          for(j=n-1;j>=i;j--) {
           if(integers[j] < integers[j-1]){
             swap(integers, j, j-1);
             finished=false;
           }
          }
        } while (!finished || i<n-1);
      }
      public default void swap(int[] integers, int i, int j){
        int temp;
        temp = integers[i];
        integers[i]=integers[j];
        integers[j]=temp;
      }
} 
```

Il problema di questa implementazione è relativo al metodo `swap()` che scambia gli elementi dell’array. Si tratta di un metodo sicuramente necessario per avere un codice più leggibile e modulare, ma che non desideriamo sia visibile al client dell’interfaccia perchè destinato ad un solo uso interno. La possibilità di utilizzare la parola chiave `private`, introdotta da Java 9, ci consente di risolvere questa situazione problematica definendo il metodo `swap()` come:

```
 private void swap(int[] integers, int i, int j){
        int temp;
        temp = integers[i];
        integers[i]=integers[j];
        integers[j]=temp;
   } 
```

Notiamo la scomparsa del modificatore di accesso **default** e concludiamo con un semplice esempio d’uso:

```
 public class InterfaceDemo {
  public static void main(String[] args) {
     ISortInt sortIntImpl = new ISortInt(){
       @Override
       public void quickSort(int[] integers){}
       @Override
       public void mergeSort(int[] integers){}
      };
      int[] integers = {5,2,1,3,4};
      sortIntImpl.bubbleSort(integers);
      for(int value: integers) {
       System.out.print(value);
      }
  }
} 
```