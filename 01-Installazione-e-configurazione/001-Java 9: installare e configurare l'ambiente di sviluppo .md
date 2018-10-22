Questa guida segue il rilascio dell’attesa versione 9 del linguaggio Java con l’obiettivo di illustrare le caratteristiche più importanti e innovative della piattaforma, senza trascurare le modifiche minori introdotte per una migliore programmazione. Per chi volesse avere una panoramica sugli argomenti in oggetto è disponibile il nostro [approfondimento](http://www.html.it/articoli/java-9-programmazione-modulare-e-jshell/) preliminare.

Iniziamo con lo scaricare le [JDK 9](http://www.oracle.com/technetwork/java/javase/downloads/jdk9-downloads-3848520.html). Dalla pagina che verrà visualizzata scarichiamo la versione per Microsoft Windows:

Figura 1. Url di Download delle JDK 9

![Url per il download delle JDK 9](https://tbm-html.s3.amazonaws.com/app/uploads/2017/12/cap1_img1.png)

Il processo di installazione è abbastanza lineare, abbiamo la necessità di specificare, in due fasi distinte, il percorso sul file system dove risiederanno le JDK ed il _Java Runtime Environment_ (**JRE**). La schermata di selezione del percorso JDK è la seguente:

Figura 2. Percorso di installazione per le JDK 9

![Percorso di installazione per le JDK 9](https://tbm-html.s3.amazonaws.com/app/uploads/2017/12/cap1_img2.png)

Mentre per il _Java Runtime Environment_:

Figura 3. Percorso di installazione per il JRE

![Percorso di installazione per il JRE](https://tbm-html.s3.amazonaws.com/app/uploads/2017/12/cap1_img3.png)

Proseguiamo attendendo il completamento della fase d’installazione. Per poter operare agevolmente con il linguaggio abbiamo bisogno di un IDE che supporti la nuova versione. Per la nostra guida riteniamo più che sufficiente utilizzare l’ultima build di NetBeans per il supporto di Java 9 scaricabile al seguente url: [NetBeans IDE Build 201709220002](http://bits.netbeans.org/download/trunk/nightly/latest/):

Figura 5. NetBeans IDE Build 201709220002

![NetBeans IDE Build 201709220002](https://tbm-html.s3.amazonaws.com/app/uploads/2017/12/cap1_img4.png)

Durante la fase di istallazione prestiamo attenzione alla specifica del percorso relativo all’IDE e alle JDK 9, in particolare annotiamo quello specificato per l’IDE:

Figura 6. Installazione NetBeans IDE Build 201709220002

![Installazione NetBeans IDE Build 201709220002](https://tbm-html.s3.amazonaws.com/app/uploads/2017/12/cap1_img5.png)

Al termine dell’installazione, apriamo la cartella nel percorso relativo all’IDE, entriamo nella sottocartella `bin`, ed eseguiamo il programma `netbeans64.exe` per lanciare NetBeans. L’ambiente dovrebbe aprirsi correttamente.

Nel successivo capitolo proseguiremo introducendo le modifiche apportate al linguaggio di programmazione.