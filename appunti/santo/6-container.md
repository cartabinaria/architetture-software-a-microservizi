Leggeri e con tempi di installazioni più brevi, i container sono molto usati nei microservizi. Un container è una partizione isolata dell'OS: un'astrazione per accedere alle risorse fisiche della macchina (CPU, RAM, ...). Inoltre gestisce l'isolamento delle risorse per ogni processo. I container dipendono da questo supporto.

Negli ambienti virtualizzati vi è un **hypervisor** che si occupa della gestione e di far comunicare bene l'OS con la VM, anche se, a volte, la VM stessa riesce a collaborare bene con l'OS. Dentro ogni VM si replica l'intero stack dell'ambiente, quindi nuovo OS, librerie di sistema e app.
Nel sistema a container si ha un **runtime** al posto dell'hypervisor. Per "OS", in questo caso, possiamo intendere la qualunque cosa: Linux, un kernel, gestione delle finestre o altro. L'OS non viene replicato ma condiviso con tutti i container. Il kernel non cambia! Far girare nuove versioni dell'OS potrebbe richiede funzionalità del kernel non presenti nella versione usata localmente. Su Windows vi è un sistema ibrido che gira su un hypervisor: questo rende il sistema molto più pesante rispetto a Linux.

## Container su Linux
Vi sono i
- `namespace`: piccole partizioni per la gestione delle astrazioni;
- `cgroups`: limita le risorse dei processi. Spesso si associano ai namespace.

Un altro modo è quello di usare il filesystem sfruttando le `chroot`. Le chroot permettono di spostare la visione dell'albero del fs a partire da un nodo e da quel sotto albero. Si può montare un fs virtuale *layered*.
L'OS fa partire un container inizializzando una visione dell'OS creata a partire dall'immagine stessa. A corredo delle informazioni si hanno i metadati tra cui il processo iniziale che ha instanza all'interno di esso. Il minimo che si offre al fs sono librerie di sistema perché sennò il processo dovrebbe continuare a fare `trap`.
Si costruiscono sovrapposizione di immagini di filesystem: ad esempio si ha, in strati, Alpine -> Apache -> JVM -> l'app. La ricerca di un file vien fatta in ordine inverso, a partire dalla cima dello stack dei livelli. In realtà questi livelli creati sono solo in read-only; vi è un ulteriore livello in cima che crea un runtime fs in read/write. Se vien fatta una modifica ad un file presente in qualcosa di read-only, questo file viene preso, copiato in runtime fs e modificato lì; la prossima volta sarà presente in runtime fs e non più *solamente* nello stack read-only.

## Docker
In Docker vi è un passaggio dall'immagine al container in cui crea una nuova immagine chiamata **stopped container** in cui vi è un fs iniziale in read-only ma dentro di esso vi è un container congelato con risorse runtime, runtime fs e fs iniziale. In cima mette un ulteriore fs in scrittura. Un processo che è fatto ripartire ha visione di com'era dopo le ultime scritture.
Un container creato da Docker si occupa di creare namespace e cgroup per le risorse, ma deve prima esser avviato.

I comandi fanno una richiesta REST al daemon. Se il comando ha a che fare con i repository delle immagini, allora tale gestione viene fatta dal daemon. Quando si fa qualcosa a livello di container (creazione, reti, gestione, etc.) chiama il livello gestito da **containerd** usando richieste gRPC. Quest'ultimo può richiedere la gestione più a basso livello grazie a **runc** usando shim.

Containerd si occupa anche della gestione delle reti virtuali che permettono di accedere al container dall'esterno usando le varie porte. Si può anche creare una rete virtuale che viene condivisa da 2+ container.

### Dockerfile

L'immagine del container può essere scritta mediante Dockerfile, il quale usa un linguaggio interno (DSL) che viene interpretato da Docker. Un esempio di file è il seguente.

```Dockerfile
#syntax=docker/dockerfile:1    # <-- questa è una direttiva
FROM golang:1.21-alpine	       # <-- prima cerca in locale, se non la trova va sul Dockerhub
WORKDIR /src
COPY . .                       # <-- da fs locale a quello delll'immagine
RUN go mod download
RUN go build -o /bin/client ./cmd/client
RUN go build -o /bin/server ./cmd/server
ENTRYPOINT ["/bin/server"]
```

dopo aver fatto la lettura del file si sta nel layer di r/w. Una volta finito tale layer diviene persistente e se ne mette uno di sopra. Si ha un primo strato di `alpine` (preso da `golang:1.21-alpine`), poi si ha uno strato con tutto il resto che diviene in sola scrittura e alla fine si ha quello con l'`ENTRYPOINT`.

Si fa la build nel container stesso perché avere un ambiente controllato a livello di versioni permette di controllare la gestione delle dipendenze uguale a quella del sistema locale. 

Si può dividere il processo in modo tale da dividere la fase di build con quella di esecuzione.

```Dockerfile
#syntax=docker/dockerfile:1
FROM golang:1.21-alpine as build
WORKDIR /src
COPY . .
RUN go mod download
RUN go build -o /bin/client ./cmd/client
RUN go build -o /bin/server ./cmd/server

FROM scratch        # <-- è un'immagine minimale
COPY --from=build /bin/client /bin/server /bin/
ENTRYPOINT ["/bin/server"]
```

In questo modo si evitano tutti gli artefatti generati dalla build di `go`.

### Compose

Compose permette di definire un insieme di container che sfruttano risorse in comune.