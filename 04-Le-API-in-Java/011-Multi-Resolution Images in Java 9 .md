Con Java 9 è stata introdotta l’interfaccia `java.awt.image.MultiResolutionImage` che consente di unire logicamente un insieme di immagini di diversa risoluzione in un’unica immagine a cui riferirci con il termine di **immagine a risoluzione multipla**.

Le funzionalità che riguardano la multi-risoluzione sono disponibili all’interno del package `java.awt.image` e permettono di ottenere non solo tutte le varianti di un’immagine, ma anche la risoluzione desiderata in base ad informazioni legate a metriche DPI (_Dots Per Inch_, punti per pollice).

Il DPI è una misura utilizzata nella stampa per indicare la densità di punti stampati su una linea di lunghezza pari ad un pollice (1 pollice = 2,54 cm). Maggiore è il valore **DPI** più è alta la risoluzione e la nitidezza dell’immagine. I DPI si riferiscono sempre a una densità fisica dei punti, rappresentano infatti la quantità di gocce d’inchiostro che la stampante può inserire in un pollice.

Per queste ragioni per un’immagine digitale sarebbe più corretto parlare di **PPI** (_Pixels Per Inch_, pixel per pollice). Finchè si rimane sull’immagine digitale e, non si ha bisogno di considerare aspetti legati all’alta qualità di stampa, possiamo considerare sostanzialmente 1 punto = 1 pixel e quindi DPI e PPI indicanti le stesse informazioni.

L’interfaccia `MultiResolutionImage` fornisce i due metodi:

```
 java.awt.Image getResolutionVariant(double destImageWidth,
			double destImageHeight) 
```

```
 List<java.awt.Image> getResolutionVariants() 
```

Il primo metodo restituisce l’immagine la cui risoluzione, tra le varianti disponibili, è in match con i valori specificati attraverso i parametri di risoluzione. Il secondo metodo restituisce la lista delle immagini corrispondenti a tutte le varianti disponibili.

Una concreta implementazione dell’interfaccia è fornita dalla classe `java.awt.image.BaseMultiResolutionImage` che andiamo ad utilizzare per la realizzazione di un semplice programma dimostrativo.

Prima di addentrarci nel codice, recuperiamo un set di immagini di diversa risoluzione. A tal fine possiamo visitare l’url [immagini a diversa risoluzione](https://commons.wikimedia.org/wiki/File:Suricata.suricatta.6861.jpg) e scaricare le diverse risoluzioni per l’immagine di un suricato:

Figura 1. Immagini di diversa risoluzione

![Immagini di diversa risoluzione](https://tbm-html.s3.amazonaws.com/app/uploads/2018/03/cap11_img1.png)

Realizziamo un progetto NetBean e inseriamo all’interno di una cartella (`images`) le immagini scaricate:

Figura 2. Immagini di diversa risoluzione.

![Immagini di diversa risoluzione](https://tbm-html.s3.amazonaws.com/app/uploads/2018/03/cap11_img2.png)

Possiamo ora creare una classe dotata di metodo `main` e inserire al suo interno le seguenti righe di codice:

```
 .
String projectDir = System.getProperty("user.dir");
List<File> filesPath= new ArrayList<>();
filesPath.add(new File(projectDir+File.separator+"images"+File.separator+"160px-Suricata.jpg"));
filesPath.add(new File(projectDir+File.separator+"images"+File.separator+"320px-Suricata.jpg"));
filesPath.add(new File(projectDir+File.separator+"images"+File.separator+"399px-Suricata.jpg"));
filesPath.add(new File(projectDir+File.separator+"images"+File.separator+"400px-Suricata.jpg"));
filesPath.add(new File(projectDir+File.separator+"images"+File.separator+"933px-Suricata.jpg"));
List<Image> images  = new ArrayList<>();
filesPath.forEach( (File file) -> {
    try {
          Image img = ImageIO.read(file);
          images.add(img);
    } catch (IOException ex) {
    }		
});
Image[] arrayImages = new Image[images.size()];
.. 
```

In questa prima parte andiamo a recuperare il percorso del progetto e costruiamo i percorsi dei file a differente risoluzione. Iteriamo quindi la lista di file per costruire una lista di immagini dalla quale ricaviamo un tipo array necessario nel passo successivo.

Arrivati a questo punto abbiamo tutto l’occorrente per testare le funzionalità dell’interfaccia. Recuperiamo un riferimento ad essa e invochiamo il metodo _getResolutionVariants()_:

```
 ..
MultiResolutionImage multiResImages = new BaseMultiResolutionImage(images.toArray(arrayImages));
List<Image> variants = multiResImages.getResolutionVariants();
System.out.println("Numero delle varianti disponibili:"+variants.size());
variants.forEach((Image image)->{
  System.out.println("Width:"+image.getWidth(null)+" "+"Height:"+image.getHeight(null));
});
.. 
```

Una volta ottenute le informazioni sulle varianti disponibili, possiamo recuperare la variante di nostro interesse utilizzando il metodo getResolutionVariant(double destImageWidth, double destImageHeight):

```
 Image image1 = multiResImages.getResolutionVariant(399, 599);
        System.out.println("Immagine 1:"+image1.getWidth(null)+"x"+image1.getHeight(null));
        Image image2 = multiResImages.getResolutionVariant(160, 240);
        System.out.println("Immagine 2:"+image2.getWidth(null)+"x"+image2.getHeight(null));
        Image image3 = multiResImages.getResolutionVariant(320, 480);
        System.out.println("Immagine 3:"+image3.getWidth(null)+"x"+image3.getHeight(null));
        Image image4 = multiResImages.getResolutionVariant(400, 600);
        System.out.println("Immagine 4:"+image4.getWidth(null)+"x"+image4.getHeight(null));
        Image image5 = multiResImages.getResolutionVariant(933, 1400);
        System.out.println("Immagine 5:"+image5.getWidth(null)+"x"+image5.getHeight(null)); 
```

Concludiamo mostrando l’output completo del programma:

Figura 3. Output del programma

![Output del programma](https://tbm-html.s3.amazonaws.com/app/uploads/2018/03/cap11_img3.png)