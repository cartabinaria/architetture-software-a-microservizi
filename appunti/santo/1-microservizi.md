# 1. Microservizi

Fuoco sulla robustezza delle applicazioni. La disponibilità è più importante. Anche la velocità è centrale in questo paradigma.
Semplificando, questo pattern architetturale ha forze in modo tale che i servizi siano sviluppati, messi online e scalati in modo indipendenti.

Qui si fa forte uso della metodologia di sviluppo Agile: si cerca in tutti i modi di ridurre l'overhead di gestione del processo. Si usa molto il concetto della virtualizzazione usando molti container. Introdotta la gestione da database, anche non relazionali (NoSQL), ma anche l'uso di message broker. Ogni microservizio potrebbe avere gestori della persistenza diversi per ogni microservizio (e in caso integrare i dati attraverso gli altri).

Vi è un approccio bottom-up: si possono scomporre i microservizi senza riprogettare il sistema per intero. Vi è un uso di diversi stack tecnologici. Nasce il concetto di DevOPs, con continui *deploy* delle patch allo sviluppo.

Ogni componente è un servizio, come nelle SOA.
I servizi sono piccoli definiti in base alle responsabilità del business che deve essere supportato. L'azienda deve intendere i servizi come se serviti da un'entità esterna. I team si parlano come se fosse una relazione cliente <-> fornitore.
Non si fa affidamento su un'infrastruttura intelligente (diverso da Kubertenes).
Regole non centralizzate: ogni servizio è indipendente nel design o tempo di realizzazione. Si ha, chiaro, un minimo di regole comuni per garantire la operatività tra i servizi.
Le infrastrutture hanno un sistema di automazione per il *deploy*.
Non si parte col presupposto che il servizio con cui voglio interagire (anche se interno alla soluzione stessa) possa essere raggiungibile. 

Un'applicazione **monolitica** ha anche essa un'architettura flessibile ma col fine di un unico servizio. Vi è un insieme di sviluppatori che lavora alle funzionalità e, ad ogni patch, si rifà il build/test/deploy di tutto il sistema.
Nello sviluppo a microservizi si ha il team di sviluppo partizionato, dunque si rifarà il build/test/deploy del singolo microservizio a cui è stata inserita la patch.

## API Gateway
Utilizzano i servizi interni dialogando anche con l'esterno (servizio rivolto verso il consumatore esterno). Si usa il classico pattern Facade con l'autenticazione, potendo sia organizzare la struttura sia gestendo la sicurezza. Il "request shaping" è un insieme di politiche in cui un insieme di richieste non creano sovraccarico. Ogni servizio sulla rete deve avere meccanismi di questo genere guardando il rate di richieste che arriva da un client e, in tal caso, capire se sono anomale o meno. Chiedere di implementare un servizio di "request (traffic) shaping" su ogni microservizio sembra un pelo esagerato, quindi se ne mette uno nell'infrastruttura.
Si può fare monitoring, load balacing e caching.

## Serverless
Modello di ingegnerizzazione in cui le funzionalità di base girano in un sistema non visibile dallo sviluppatore. I componenti logici (funzionali) sono lasciati in mano all'infrastruttura che si occupa di eseguirlo.
Il business model dei servizi che offrono ciò (AWS Lambda, Google Cloud, Azure o IBM) è basato su pay-per-use. Il servizio si scala in base alla quantità di richieste: chiaramente in base a limiti settati in base anche al prezzo.