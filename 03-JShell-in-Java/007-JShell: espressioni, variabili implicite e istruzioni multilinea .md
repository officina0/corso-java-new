Quando inseriamo un’espressione all’interno della JShell, il risultato ad essa associato viene assegnato ad una variabile creata automaticamente dalla JShell stessa. Queste variabili prendono il nome di **variabili implicite** e il loro nome ha `$#` come formato, dove con il simbolo `#` si indica il generico ID assegnato all’istruzione.

Ad esempio l’espressione `10+8` viene valutata dalla JShell con il valore `18` assegnato alla variabile implicita `$4` creata per contenerlo:

Figura 1. Variabili implicite.

![Variabili implicite](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap7_img1.png)

Le variabili implicite sono un esempio di come la JShell reinterpreta il comportamento standard di Java per introdurre interattività. Nell’esempio precedente la JShell deduce il tipo `(int)` della variabile dal tipo degli operandi (Entrambi _int_) dell’espressione.

Per visualizzare il valore di una variabile è sufficiente digitarne il nome e premere Invio:

Figura 2. Visualizzare il valore di una variabile.

![Visualizzare il valore di una variabile](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap7_img2.png)

Se la lista di istruzioni digitate all’interno della JShell non è più di utilità, possiamo usare il comando `/reset` per cancellare tutte le istruzioni inserite fino al punto corrente:

Figura 3. Comando reset.

![Comando reset](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap7_img3.png)

Supponiamo adesso di voler testare uno blocco di istruzioni. Un esempio potrebbe essere l’istruzione `if` con più linee di codice. Realizziamo quindi una semplice sequenza di istruzioni che dichiari due variabili implicite intere, le confronti attraverso un’istruzione condizionale, è stampi una stringa. Iniziamo con il blocco:

Figura 4. Istruzioni su più linee.

![Istruzioni su più linee](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap7_img4.png)

L’esempio mostra come la JShell interpreti l’istruzione `if`, terminante con la parentesi graffa aperta sulla stessa linea, come **istruzione non terminata** aggiungendo, nel caso di uso con Netbeans, un’ulteriore parentesi di chiusura.

La JShell si predispone per accogliere eventuali istruzioni all’interno del blocco `if`, inseriamo quindi un’istruzione di stampa e premiamo Invio per ottenerne la valutazione:

Figura 5. Istruzioni su più linee.

![Istruzioni su più linee](https://tbm-html.s3.amazonaws.com/app/uploads/2018/01/cap7_img5.png)

L’uso di variabili e blocchi di codice all’interno della JShell, come visto nel presente capitolo, è abbastanza semplice e linerare, nel successivo capitolo vedremo come definire ed utilizzare altri costrutti.