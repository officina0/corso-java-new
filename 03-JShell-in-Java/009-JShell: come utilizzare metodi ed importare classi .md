In questo capitolo concludiamo l’introduzione all’uso della JShell discutendo l’**utilizzo di metodi e l’importazione di package**. Alcune volte infatti si ha l’esigenza di provare rapidamente l’implementazione di un metodo, all’interno di una classe che si sta realizzando, per verificare da subito la correttezza di funzionamento su alcuni semplici casi di test.

Normalmente saremmo costretti ad inserirlo all’interno di una classe, parzialmente implementata, istanziare oggetti di questa classe oppure utilizzare il riferimento di classe se il metodo è statico, per un primo test di sviluppo. Si tratta di un’operazione che appesantisce il lavoro del programmatore che, probabilmente, per non ritardare il proprio lavoro può scegliere di effettuare il test globale della classe più in là nello sviluppo, abbandonando quindi l’idea di un primo rapido controllo.

La JShell è di aiuto consentendo di testare metodi singoli senza ricorrere a classi ed oggetti. Supponiamo di dover sviluppare metodi per una o più classi di un sistema per il calcolo numerico e di essere impegnati nella realizzazione di un metodo per la determinazione della potenza intera di un numero reale. Chiamiamo `pot()` il metodo in questione e realizziamo preliminarmente un metodo per il suo test:

Figura 1. Metodo per il calcolo di una potenza.

![Metodo per il calcolo di una potenza](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap9_img1.png)

Possiamo notare un aspetto interessante della JShell, il metodo viene creato con il commento di poter essere invocato solo dopo la creazione del metodo `pot()` referenziato al suo interno. La JShell, inoltre, sottolinea in rosso il metodo non definito. Proseguiamo con la definizione di `pot()`:

Figura 2. Metodo per il calcolo di una potenza.

![Metodo per il calcolo di una potenza](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap9_img2.png)

Come si può osservare dall’immagine, dopo la creazione del metodo per il calcolo della potenza facciamo uso del comando `/methods` che consente di visualizzare tutti i metodi disponibili nella JShell.

Possiamo ora testare il funzionamento del metodo:

Figura 3. Test metodo per il calcolo di una potenza.

![Test metodo per il calcolo di una potenza](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap9_img3.png)

All’interno della JShell possiamo importare classi di altri package utilizzando il comando `/classpath`. Supponiamo di voler testare una connessione al database noSQL **MongoDB** avendo quindi la necessità di inserire nel classpath il file jar del driver, e di istanziare quindi la classe client di collegamento:

Figura 4. Importazione package.

![Importazione package](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap9_img4.png)

Occorrono alcune considerazioni su quanto evidenziato in rosso nell’immagine. La JShell all’interno della versione di sviluppo di NetBeans che stiamo utilizzando consente sì la corretta aggiunta del file nel classpath, ma sembra non trovare classi e package in esso contenuti quando digitiamo un riferimento ad essi.

Possiamo infatti notare la sottolineatura ondulata di errore sull’inizio dell’importazione del package relativo a MongoDB client. In realtà precisiamo ancora che il file jar è correttamente inserito nel classpath ed il package correttamente importato come mostra il messaggio successivo della JShell, ciò che sembra non essere disponibile ancora in questa versione di sviluppo è il rilevamento ed il completamento automatico durante la digitazione dei package importati.