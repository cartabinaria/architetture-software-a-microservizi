# 0. Introduzione

> Definiamo alcuni termini come:
> Servizio => comportamento degli elementi
> Processo => come collaborano fra loro
> BPMN => UML usato per astrazioni di processo
> 
> Buona lettura è quella di Sam Newman "Building Microservices: Designing Fine-Grained Systems"

---

Un'astrazione crea un concetto da un esempio specifico, eliminando i "dettagli", i quali possono essere considerati come aspetti meno significativi. 

Ad esempio: un generico pallone non ha un colore specifico, mentre, se prendiamo come riferimento "un pallone che abbiamo a casa", esso avrà dei dettagli. A livello d'astrazione, invece, ogni pallone si rompe se viene colpito da uno spillo.

Dunque, ci permette di fare ragionamenti semplificati. I modelli, che rappresentano le astrazioni, sono di una complessità inferiore rispetto all'esempio reale perché non hanno i dettagli dell'oggetto reale.
Potremmo paragonare un modello ad un *oracolo*: un oggetto che ritorna le risposte alle domande che gli poniamo. Voglio un'astrazione che mi dia la medesima risposta sia se la domanda è posta all'oracolo sia se all'entità reale. Non esiste un unico modello per ogni entità.
Il punto di vista è l'insieme di domande a cui siamo interessati.
Si vuole sapere il comportamento emergente, grazie all'astrazione di processo, che si ha col modello, comunicando con i vari servizi. Nei SOA, i processi comunicano fra di loro mediante la rete.

## Service-Oriented Architecture (SOA)

La granularità di "needs" e "capabilities" possono essere diverse in base alla SOA di riferimento. Può essere in cascata: chi offre una "capability" è in grado di farlo se altri servizi dialogano con esso offrendo altre "capabilities".

L'accoppiamento tra i servizi deve essere quanto più lasco possibile. Non ci deve essere un concetto di affinità di sessione: il servizio non deve capire da quale "cliente" avviene. I servizi devono essere ricercabili e componibili dagli altri. Usando gli standard si massimizza il dialogo tra di essi.

Nella prima generazione di SOA vi era, tra le varie, il problema che i servizi non fossero autonomi: se due servizi dipendono da un unico database, non sono autonomi.

Il teorema CAP ci permette di dimostrare che, dato che Internet prende il concetto di *partizionabilità*, non possiamo avere disponibilità e correttezza nel medesimo momento.

Usare SOA permette di rimpiazzare il codice presente nei servizi in modo più scalabile. Nessuno potrebbe rendersi conto che un servizio è stato sostituto.

Oramai si è spostati all'uso dei micro servizi. Questo spostamento è dato dal cambio di problemi da risolvere, più facilmente risolvibili con un'architettura a micro servizi. L'evoluzione data dagli obbiettivi, creando servizi che siano inerenti alla medesima organizzazione.