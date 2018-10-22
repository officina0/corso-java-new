Essendo Java un linguaggio orientato agli oggetti è molto importante poter **dichiarare ed utilizzare classi all’interno della JShell** al fine di testarne il funzionamento. Riprendiamo il codice relativo all’algoritmo di ordinamento Bubble Sort introdotto nei capitoli precedenti, ed utilizziamolo all’interno della JShell attraverso la realizzazione di una semplice classe. Iniziamo con la definizione della classe `BubbleAlgorithm` come mostra la seguente immagine della JShell all’interno di NetBeans:

Figura 1. Dichiarazione di classe.

![Dichiarazione di classe](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap8_img1.png)

La classe `BubbleAlgorithm` rappresenta un nuovo tipo. All’interno della JShell possiamo visualizzare nuovi tipi dichiarati attraverso il comando `/types`:

Figura 2. Visualizzazione tipi.

![Visualizzazione tipi](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap8_img2.png)

La dichiarazione di un oggetto della classe segue le regole sintattiche del linguaggio ed è interessante notare come la JShell evidenzi l’inizializzazione a `null` del riferimento:

Figura 3. Dichiarazione di un oggetto.

![Dichiarazione di un oggetto](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap8_img3.png)

Proseguiamo valorizzando il riferimento con un’istanza della classe seguendo le regole grammaticali del linguaggio Java:

Figura 4. Istanziare un oggetto.

![Istanziare un oggetto](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap8_img4.png)

In questo caso la JShell evidenzia la valorizzazione del riferimento con una istanza specifica. Vogliamo adesso utilizzare l’oggetto creato invocando il metodo `bubbleSort()`. Definiamo preliminarmente un array di interi da passare al metodo:

Figura 5. Array di interi.

![Array di interi](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap8_img5.png)

Facciamo notare la disponibilità della funzionalità di **autocompletamento del codice**:

Figura 6. Autocompletamento del codice.

![Autocompletamento](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap8_img6.png)

Invochiamo il metodo `bubbleSort()` sull’istanza creata ed inseriamo un ciclo `for` per la visualizzazione dell’array ordinato:

Figura 7. Invocazione del metodo.

![Invocazione del metodo](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap8_img7.png)

Concludiamo il capitolo illustrando il comando `/vars` della JShell utile per la visualizzazione di tutte le variabili dichiarate:

Figura 8. Variabili dichiarate.

![Variabili dichiarate](https://tbm-html.s3.amazonaws.com/app/uploads/2018/02/cap8_img8.png)