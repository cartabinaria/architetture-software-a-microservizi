# 4. Jolie

La standard library è fatta da servizi. Le interfacce hanno un'estensione `.iol`, mentre i servizi `.ol`. I *request response* attendono la risposta del messaggio inviato. Le *one-way* no.
Il `()` in
```jolie
println@Console("hi")()
```

attende il response.  Questo rende la stampa sincrona.
Per inviare messaggi si ha il nome dell'operazione `@` nome del servizio.

```jolie
sendNumber@B(5)
```

Ma come si fa a comunicare da servizio `A` a `B`? Si definisce la comunicazione tra le varie porte delle interfacce. Le input e output port devono essere d'accordo, decidendole in modo dichiarativo.

Il protocollo usato deve essere il medesimo.

Sia il receiver che il sender usano la medesima interfaccia per avere più controllo nell'uso dei metodi.

La gestione è data dal modello di *separation concern* usando lo stesso *behaviour* con diverse porte. 

Usare protocolli standard, come SOAP o HTTP, permette di definire dei servizi (web ad esempio) in cui non bisogna definire un servizio per comunicare col receiver.

Ogni dato è trattato come un albero, ed ogni albero può essere un array. Ogni dato è dinamicamente tipato: si definisce il tipo dell'interfaccia, non del singolo dato. Controlla il tipo solo quando il dato è inviato attraverso una richiesta.

Si può creare un blocco `init` per inizializzare le variabili globali.

Ogni sessione ha un ID, dunque si vede se il messaggio che arriva ha o meno un ID e da lì si capisce se è una nuova sessione. Una sessione ha uno stato interno per cui un messaggio potrebbe arrivare in un momento non consono.
Gli insiemi di correlazione sono una generalizzazione delle sessioni. In questo modo si gestisce un insieme di sessioni, ad esempio una chat in cui tutti possono leggere cosa è stato scritto. Sono più flessibili ma più complicati.
Ogni variabile è locale per le sessioni. Quelle condivise, creata appena si crea il servizio e non muore se non vi sono sessioni, definita come `global.nome_variabile`.
Si può condividere lo stato come
```jolie
synchronized(id) {
...
}
```

`new` è una primitiva che crea valori univoci.
Non vi è possibilità di fare intersezioni fra due `csets`.
