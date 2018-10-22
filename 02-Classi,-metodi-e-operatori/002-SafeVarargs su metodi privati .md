La funzionalità **varargs** (abbreviazione di _variable arguments_) è stata introdotta in Java per facilitare la realizzazione di **metodi con un numero variabile di argomenti** senza ricorrere a parametri di tipo array o a versioni sovraccaricate (overloading di metodi) di uno stesso metodo. L’idea alla base del varargs è di utilizzare, per la dichiarazione di un metodo con un numero di parametri variabile, sintassi del tipo:

```
 void nomeMetodo(Java_Type...t){
 //do something
} 
```

Dove con `Java_Type` indichiamo un qualsiasi tipo Java. Ad esempio un metodo con un numero variabile di parametri di tipo `int`, potrebbe essere definito come:

```
 void mioMetodoInt(int...t){
 //do something
} 
```

L’aspetto importante da comprendere è come il compilatore elabora questo costrutto del linguaggio. Ogni volta che il compilatore incontra l’espressione `Java_Type...t` la trasforma in una dichiarazione di array di tipo Java_Type (`Java_Type[]`). Questo significa che il parametro `t` del codice di esempio deve essere trattato, all’interno del corpo del metodo, come fosse una variabile di tipo array, mentre a runtime avremo un automatico `new Java_Type[]` non appena s’incontra una richiesta di esecuzione del metodo con la specifica dei parametri:

```
 void mioMetodoInt(int...t){
 for(int value : t) {
  System.out.println(value);
 }
} 
```

Potendo invocare il metodo passando zero o più parametri:

```
 mioMetodoInt(); //Runtime : t=new int[]
 mioMetodoInt(1); //Runtime : t=new int[]{1}
 mioMetodoInt(1,2); //Runtime : t=new int[]{1,2}
 mioMetodoInt(1,2,3)); //Runtime : t=new int[]{1,2,3} 
```

L’utilizzo congiunto della funzionalità varargs con il tipo Generics può portare a problematiche di difficile interpretazione per il programmatore non esperto. Vediamo un esempio interessante: realizziamo la seguente classe che fa uso di due metodi varargs:

```
 public class NotSafeVarargs {
	private <E> E[] metodo1(E... e) {
		return e;
	}
	private <T> T[] metodo2(T t1, T t2) {
		return metodo1(t1, t2);
	}
	public static void main(String[] args) {
		String[] lettere = new NotSafeVarargs().metodo2("a", "b");
		for (String lettera : lettere) {
			System.out.println(lettera);
		}
	}
} 
```

Il codice non da errori di compilazione ma alcuni _warning_. Sul `metodo1()` abbiamo la segnalazione:

```
 Type safety: Potential heap pollution via varargs parameter e 
```

Il compilatore ci informa della possibilità che il tipo restituito possa non essere compatibile con il tipo dichiarato come valore di ritorno del metodo. Ci rendiamo conto della bontà di questo _warning_ attraverso il secondo _warning_ sull’istruzione `return` del `metodo2()`:

```
 Type safety: A generic array of T is created for a varargs parameter 
```

In questo secondo caso il compilatore ci avvisa che invocando il `metodo2()` è sicuramente definito il tipo effettivo per il parametro `T`, ma non vi è alcuna definizione specifica per il parametro `E` del `metodo1()`, ragion per cui a runtime verrà scelto il tipo `Object` per il parametro `E`, avendo quindi un `new Object[]` e non il `new String[]` atteso. Il risultato finale è un crash del programma a runtime per incompatibilità di tipo con l’array `lettere`.

Il primo `warning` può essere forzato con la definizione di utilizzo `Safe(sicuro)` attraverso l’uso dell’annotation `@SafeVarargs`. Questa possibilità (`@SafeVarargs` su metodi privati non `final`) è una nuova caratteristica di Java 9, infatti in precedenza era possibile applicare questa annotazione soltanto a metodi di cui non è possibile effettuare l’override (metodi statici, metodi dichiarati `final` e costruttori) escludendo senza una ragione ben precisa i metodi privati non `final`.

Inseriamo la classe all’interno di un semplice progetto NetBeans, usando la versione scaricata e configurata nel precedente capitolo, con l’accortezza di abilitare il _warning_ del compilatore Java:

Figura 1. Attivazione warning.

![Attivazione Warning](https://tbm-html.s3.amazonaws.com/app/uploads/2017/12/cap2_img1.png)

Modifichiamo quindi il codice utilizzando l’annotation `@SafeVarargs` in modo tale che sia compilato senza `warning` ed eseguito senza errori:

```
 public class NotSafeVarargs {
    @SafeVarargs
    private <E> E[] metodo1(E... e) {
        return e;
    }
    private <T> T[] metodo2(T t1, T t2) {
        return metodo1(t1, t2);
    }
    public static void main(String[] args) {
        Object[] lettere = new NotSafeVarargs().metodo2("a", "b");
        for (Object lettera : lettere) {
            System.out.println(lettera);
        }
    }
} 
```