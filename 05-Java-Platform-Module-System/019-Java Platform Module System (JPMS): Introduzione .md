Il **JPMS** (progetto _Jigsaw_) è l’innovazione più importante di Java 9. Sua base è il concetto di **modulo** con cui dar vita ad applicazioni modulari, che offre benefici in termini di produttività e manutenibilità.

Ma cos’è un modulo? in Java le classi vengono organizzate in package seguendo il principio _Object Oriented_ di **alta coesione**. Essa porta a definire classi con metodi legati al raggiungimento di specifiche funzionalità, per cui la classe è progettata, e ad organizzare in un insieme classi che concorrono in modo diverso o complementare ad offrire il medesimo servizio.

In un package possiamo trovare le classi per manipolare stringhe e caratteri, in un altro classi per gestire numeri e calcoli. La **modularità** utilizza il concetto base di **package** aggiungendo un più alto livello di aggregazione basato sul loro uso.

Un modulo è quindi un’aggregazione di package relazionati nella quale è possibile inserire risorse di altra natura (immagini, XML..). Ogni modulo è dotato di un **descrittore** che specifica:

*   nome del modulo;
*   moduli dai quali dipende (dipendenze);
*   package resi esplicitamente disponibili o non disponibili ad altri moduli;
*   servizi che offre e usa;
*   a quali altri moduli rende possibile la reflection.

Il descrittore risiede nel file `module-info` che contiene la dichiarazione del modulo. Ogni dichiarazione inizia con la keyword `module` seguita dal nome del modulo e da parentesi graffe. Ciò che è inserito tra le parentesi rappresenta il corpo del modulo:

```
 module modulename {
      //body
     } 
```

Nel corpo del modulo possiamo inserire direttive per agire sugli aspetti prima elencati. Con `requires`, seguita dal nome di un modulo, specifichiamo le sue dipendenze. Essa consente di utilizzare la keyword `static` per indicare che la dipendenza è richiesta in compilazione e non a runtime. Un aspetto molto interessante legato a tale direttiva è la **implied readability**. Possiamo specificarla facendo seguire a `requires` la keyword `transitive` seguita dal nome di un modulo. Con essa intendiamo dire che un modulo A che richieda l’uso del modulo B diventa dipendente anche dal modulo transitive C dichiarato in B.

Consideriamo il modulo `java.desktop` che dichiara `requires transitive java.xml`: ogni modulo A che legge `java.desktop` deve leggere anche `java.xml`. Supponiamo che in A si usi un metodo di una classe all’interno del modulo `java.desktop` che ritorni un tipo creato con una classe del modulo `java.xml`, il codice di A diventa automaticamente dipendente di `java.xml`.

Senza `requires transitive` nella dichiarazione di `java.desktop` la compilazione di A non sarebbe possibile a meno di una dichiarazione esplicita della dipendenza `java.xml` in A.

In un modulo in genere non si vuole rendere pubblici tutti i package, ci possono essere infatti package destinati solo ad uso interno. La keyword `esports` permette di specificare quali package sono visibili ad altri moduli. Quando un package di A è visibile ad un altro modulo B il codice di B può accedere a quanto dichiarato `public` nel package di A.

Un modulo può utilizzare specifici servizi divenendo un **service consumer**, cioè un oggetto di classe che implementa l’interfaccia, o estende la classe astratta, specificata con la direttiva `uses`:

```
 module it.html.module {
        exports it.html.spi
        uses it.html.spi.IProblemProvider;
       } 
```

In questo caso il modulo esporta il `package it.html.spi` contenente l’interfaccia `IProblemProvider` e la sua classe `Problem`. Ciò abilita moduli che implementano `IProblemProvider` ad accedere all’implementazione fornita dal modulo.

`uses` indica che in `it.html.module` vengono usati oggetti di classi che implementano l’interfaccia `IProblemProvider`, tale aspetto fa del modulo un service consumer.

L’uso di un servizio da parte di un modulo avviene tramite reflection. Utilizzando la direttiva `provides...with` un modulo può diventare fornitore di servizi. Con essa specifichiamo un’interfaccia, o una classe astratta, nella sezione `provides` indicando la sua implementazione nella parte `with`:

```
 module it.mycustom.module {
        requires it.html.convertitore;
        provides it.html.convertitore.spi.IConvertitore 
            with com.mymodule.ConvertitoreImpl
       } 
```

`it.mycustom.module` dichiara la dipendenza da `it.html.convertitore` implementando la sua interfaccia (`IConvertitore`) presente nel package `it.html.convertitore.spi`. L’implementazione è la classe `ConvertitoreImpl` del package `com.mymodule` (modulo `it.mycustom.module`).

Le direttive `opens`, `opens .. to` ed `open` sono utili invece per l’accessibilità dei package verso altri moduli. Utilizzando `opens`, seguita dal nome di un package del modulo, stiamo dicendo che le classi `public` (ed i loro campi e metodi `public` e `protected`) all’interno del package sono accessibili in altri moduli soltanto a runtime, consentendo l’accesso con reflection a tutti i tipi all’interno delle classi del package.

`opens..to` consente di specificare i moduli verso i quali quanto detto precedentemente è possibile. In un modulo potremmo dichiarare qualcosa come `opens mypackage to module1,module2` per indicare che il package è accessibile con le modalità indicate da `open` ai soli `module1` e `module2`. Se invece tutti i package di un modulo devono essere accessibili a runtime e via reflection agli altri moduli, è possibile aprire totalmente un modulo con `open` a livello di dichiarazione di modulo:

```
 open module it.mycustom.module {
          ..
       } 
```