# 3. Sistemi distribuiti

È una rete di endpoint che calcolano comunicando fra di loro mediante scambio di messaggi. È un sistema distribuito in grandi distanze geografiche, ma la distanza è relativa: due processi nel proprio PC che comunicano a scambio di messaggi potrebbero avere i medesimi errori se fatti in larga scala.
L'approccio a servizi è nato per l'interazione omogenea fra aziende.

Internet è un esempio di sistema distribuito. Anche Google Chrome lo fa in modo suo.

Le principali difficoltà sono: comunicazione, eterogeneità (tutti gli endpoint non possono avere medesime caratteristiche, quindi bisogna trovare standard per comunicare all'esterno), gestione dei *fault* (circostanze che avvengono, ad esempio l'utente che fornisce un input non previsto), evoluzione dei sistemi (servizi possono nascere, evolvere da soli o essere deprecati; l'ambiente si dovrebbe adattare alle esigenze).

Le complessità vengono semplificante mediante astrazioni. Ci sono linguaggi (vedi Erlang o Go) che supportano il sistema a messaggi (ma non forniscono astrazioni per l'eterogeneità).

## Service Oriented Computing
Un paradigma che distribuisce i servizi (ha un'interfaccia verso l'esterno che definisce, mediante operazioni, cosa può inviare e ricevere). Nato con l'idea di offrire dinamicità, data dalla "scoperta" dei servizi locati a *runtime*.
Ci si basa su standard, come XML, SOAP o WSD.

Un suo discendente è REST, che però si basa su servizi usando HTTP.