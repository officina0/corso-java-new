Diverse applicazioni Desktop sono scritte utilizzando il linguaggio Java. Dalla versione 9 in poi il supporto all’interazione con il sistema operativo sottostante è stato sensibilmente migliorato.

Le migliorie riguardano essenzialmente la classe `java.awt.Desktop` alla quale sono state aggiunte nuove funzionalità. Purtroppo non si ha la garanzia che una determinata funzionalità sia supportata dal sistema operativo corrente e l’esecuzione della stessa potrebbe produrre un risultato nullo.

Tutte le funzionalità hanno una valore all’interno dell’enumerazione `java.awt.Desktop.Action`. Per capire se una funzionalità è supportata o meno è possibile utilizzare il metodo `isSupportedAction(Desktop.Action action)` della classe `Desktop`. Quest’ultima permette di interagire con il sistema effettuando operazioni come:

*   Lanciare il browser di default per visualizzare uno specifico URL.
*   Lanciare il mail client di default con un URI `mailto` opzionale.
*   Lanciare un’applicazione registrata per l’apertura di un particolare file.

Il metodo `addAppEventListener(SystemEventListener listener)` di `java.awt.Desktop` consente di definire dei listener per diversi tipi di eventi di sistema. Il suo parametro è un’interfaccia dalla quale discendono altre interfacce che la estendono rappresentando un particolare evento:

Interfaccia

Descrizione

**AppForegroundListener**

Notifica entrata e uscita dallo stato di appllicazione in foreground.

**AppHiddenListener**

Notifica ricevuta quando l’utente mostra o nasconde l’applicazione.

**AppReopenedListener**

Notifica ricevuta alla richiesta di una nuova apertura dell’applicazione.

**ScreenSleepListener**

Notifica ricevuta quando lo schermo del sistema entra nella modalità risparmio energetico.

**SystemSleepListener**

Notifica ricevuta quando il sistema entra o esce dallo stato di riposo energetico.

**UserSessionListener**

Notifica ricevuta per la fine di una sessione utente o l’inizio di una nuova.

Se il sistema non supporta un particolare evento possiamo rimuoverlo con il metodo `removeAppEventListener(SystemEventListener listener)`. Spostare in foreground un’applicazione è invece possibile utilizzando il metodo `requestForeground(boolean allWindows)`.

Analizziamo ora un semplice esempio di applicazione **Swing** che sperimenta alcuni eventi di sistema:

```
 public class NativeDskIntDemo {
    public static void main(String[] args) {
        NativeDskInt nativeDskInt = new NativeDskInt();
        nativeDskInt.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
    private static class NativeDskInt extends JFrame {
        public NativeDskInt() {
            final JTextArea textArea = new JTextArea();
            Container container = getContentPane();
            BorderLayout border = new BorderLayout();
            container.setLayout(border);
            container.add(textArea, BorderLayout.CENTER);
            final Desktop desktop = Desktop.getDesktop();
            desktop.addAppEventListener(new AppForegroundListener() {
                @Override
                public void appRaisedToForeground(AppForegroundEvent e) {
                    textArea.append("\n Foreground");
                }
                @Override
                public void appMovedToBackground(AppForegroundEvent e) {
                    textArea.append("\n Background");
                }
            });
            setSize(300, 400);
            setVisible(true);
        }
    }
} 
```

Invitiamo se possibile a provare il seguente codice su diverse piattaforme, ad esempio quello presentato in questo capitolo è stato realizzato su piattaforma Windows 8 dove purtroppo gli eventi presenti all’interno del codice non sono supportati. Mostriamo invece esempi funzionanti per l’apertura di un URL o di un file PDF anche su Windows 8 e versioni successive del sistema Microsoft:

**URL**:

```
 Desktop desktop = Desktop.getDesktop();
       URI uri;
       try {
            uri = new URI("https://github.com/MoonShade/sgl-Editor/wiki");
            desktop.browse(uri);
       } catch (IOException e) {
            e.printStackTrace();
       } catch (URISyntaxException e1) {
            e1.printStackTrace();
       } 
```

**File PDF**:

```
 if (Desktop.isDesktopSupported()) {
          try {
           File myFile = new File("c:\\test.pdf");
           Desktop.getDesktop().open(myFile);
          } catch (IOException ex) {
           ex.printStackTrace();
          }
       } 
```