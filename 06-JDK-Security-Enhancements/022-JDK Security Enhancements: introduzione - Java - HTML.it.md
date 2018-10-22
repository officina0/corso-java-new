**Java Secure Socket Extension** (JSSE) è il framework che consente alle applicazioni Java di instaurare comunicazione sicure sulla rete Internet. L’obiettivo della presente lezione e delle successive è quello di partire dalle basi del protocollo SSL/TLS fino ad arrivare alla costruzione di un semplice programma Java Client/Server che illustri l’utilizzo del protocollo DTLS (_Datagram over SSL_).

Per il principiante rispetto a tematiche di sicurezza alcuni termini utilizzati risulteranno di difficile comprensione, chiediamo uno sforzo in termini di pazienza poiché nelle successive lezioni spiegheremo il significato di molti di essi fornendo le basi per l’approfondimento delle restanti.

Invitiamo quindi a considerare l’elenco introduttivo che segue come riferimento di sintesi da leggere nuovamente al termine dell’implementazione del programma Java dimostrativo.

Le novità introdotte sono dunque le seguenti:

*   supporto al protocollo DTLS versioni 1.0 e 1.2 per il framework JSSE e il SunJSSE security provider.
*   Il client ed il server di una comunicazione TLS possono negoziare il protocollo da utilizzare. Attraverso l’utilizzo del protocollo **ALPN** (_Application-Layer Protocol Negotiation_), il client invia una lista di protocolli applicativi supportati come parte del messaggio TLS **ClientHello**. Il server sceglie un protocollo e restituisce la scelta come parte del messaggio TLS **ServerHello**.
*   Il server, all’interno di una connessione TLS, può verificare se un certificato X.509 è stato revocato. Questa operazione è realizzata durante l’handshaking TLS contattando un responder **OCSP** (_Online Certificate Protocol Status_) per il certificato.
*   Implementazione del meccanismo _DRBG-Based SecureRandom_.
*   Sfruttare le istruzioni della CPU per GHASH ed RSA.
*   Miglioramento della configurazione della sicurezza delle JDK fornendo un più flessibile meccanismo di disabilitazione di una catena di certificati X.509 con signature SHA-1.
*   Modifica del tipo base JKS, per un keystore, nel tipo PKCS12. PKCS12 è un formato estensibile, standard e ampiamente supportato per la memorizzazione di chiavi crittografiche. Apre nuove opportunità all’interoperabilità con altri sistemi come OpenSSL che supportano PKCS12.
*   Supporto a funzioni hash crittografiche SHA-3. I seguenti algoritmi standard sono supportati dalle API `java.security.MessageDigest`: SHA3-224, SHA3-256, SHA3-384, and SHA3-512.

Comprendere il significato del protocollo **SSL** (_Secure Socket Layer_)/TLS è di particolare importanza poiché è alla base dell’interazione tra due attori che desiderano comunicare con caratteristiche di segretezza ed integrità dei dati. Utilizziamo SSL per i seguenti motivi:

*   vogliamo essere sicuri che l’entità con la quale comunichiamo sia esattamente chi dice di essere.
*   I dati possono essere intercettati da una terza parte non autorizzata.
*   Un attaccante può utilizzare i dati intercettati e modificarli prima di propagarli al destinatario.

SSL è un protocollo che va a collocarsi esattamente tra lo strato applicativo e quello di trasporto della pila protocollare di Internet e, generalmente, è utilizzato insieme al protocollo HTTP fornendo cosi un accesso sicuro HTTPS.

Va in ogni caso precisato che il suo utilizzo è possibile con qualsiasi protocollo dello strato applicativo. SSL utilizza la **crittografia dei dati** per garantire i propri servizi, una conoscenza di base dei meccanismi di cifratura dei dati è quindi essenziale.

Proseguiamo introducendo brevemente il funzionamento della crittografia a **chiave segreta** e della crittografia a **chiave pubblica**. Consideriamo due attori, che identifichiamo con i nomi di Bob e Alice, che desiderano comunicare in modo sicuro evitando che un terzo attore, che chiamiamo Trudy, possa intercettare e manipolare i dati che essi si scambiano.

Bob e Alice possono decidere di utilizzare un algoritmo di cifratura dei dati che preveda una sola chiave condivisa da entrambi. Un algoritmo a chiave segreta è quindi esattamente un algoritmo in cui sia Bob che Alice cifrano e decifrano i dati utilizzando una sola chiave che, idealmente, dovrebbe essere solo di loro conoscenza. Il meccanismo di scelta della chiave condivisa è quindi un aspetto critico poiché una sua falla consentirebbe a Trudy di individuare la chiave e, non solo decifrare i dati che essi si scambiano, ma anche inviare dati ad uno dei due attori spacciandosi per l’altro.

Un algoritmo a chiave pubblica ( che vedremo tra poco) consente di risolvere questa problematica garantendo uno scambio sicuro della chiave condivisa e dell’algoritmo di cifratura scelto. Esempi di algoritmi a chiave simmetrica sono: _Advanced Encryption Standard_ (AES), _Triple Data Encryption Standard_ (3DES) e _Rivest Cipher 4_ (RC4).

Nella crittografica a chiave pubblica ciascun attore è dotato di una chiave pubblica, che può rendere disponibile sulla Rete, e di una chiave privata custodita segretamente. Ogni messaggio cifrato con la chiave pubblica può essere decifrato con la chiave privata e, viceversa, ogni messaggio cifrato con la chiave privata può essere decifrato con la chiave pubblica.

In questo scenario se Bob vuole inviare un messaggio segreto ad Alice utilizzando la crittografia a chiave pubblica, utilizzerà la chiave pubblica di Alice per cifrare il messaggio, ed invierà il messaggio cifrato ad Alice che sarà l’unica in grado di decifrarlo attraverso la sua chiave segreta.

Un altro scenario interessante è il seguente: se Alice cifra un messaggio utilizzando la sua chiave privata e lo invia a Bob, Bob potrà essere sicuro che i dati che riceve provengono effettivamente da Alice. Egli infatti potrà utilizzare la chiave pubblica di Alice per decifrare il messaggio sapendo che solo Alice può averlo cifrato con la chiave privata da lei custodita. Sebbene il problema evidente di questo scenario è rappresentato dal fatto che chiunque può decifrare il messaggio di Alice, esso pone le basi per il meccanismo della **firma digitale**.

Una firma digitale è parte di un certificato a chiave pubblica ed è utilizzato in SSL per l’autenticazione Client/Server. Esempi di algoritmi a chiave pubblica, detti anche a chiave asimmetrica, sono RSA (_Rivest Shamir Adleman_) e DH (_Diffie-Hellman_), quest’ultimo utilizzato particolarmente per lo scambio di chiavi segrete. Un algoritmo a chiave pubblica richiede una computazione intensiva ed è quindi molto lento, per queste ragioni è tipicamente utilizzato per cifrare piccole quantità di dati come le chiavi segrete di sessione tra due entità comunicanti.

Un certificato fornisce un modo sicuro di fornire la propria chiave pubblica. I certificati a chiave pubblica risolvono infatti il seguente problema: se Trudy (l’attaccante) crea la sua chiave pubblica e privata, può spacciarsi per Alice nei confronti di Bob inviandogli la chiave pubblica da lei generata. Bob sarà in grado di comunicare con Trudy ma in realtà egli crederà di comunicare con Alice.

Un certificato a chiave pubblica può essere pensato come l’equivalente digitale di un passaporto. Come un passaporto è infatti rilasciato da una organizzazione fidata che fornisce garanzia sull’identità del proprietario. Una organizzazione fidata che rilascia certificati a chiave pubblica è conosciuta come **Certificate Authority** (CA). Una CA può essere collegata ad un notaio pubblico. Per ottenere un certificato da una CA, un utente deve fornire prova della sua identità, una volta che la CA ha verificato l’identità dichiarata accertando la veridicità delle informazioni fornite, il certificato viene rilasciato.

All’interno del certificato troviamo diverse informazioni tra cui la chiave pubblica e, aspetto estremamente importante, il certificato è firmato digitalmente dalla CA che utilizzerà, per questa operazione, la propria chiave privata.