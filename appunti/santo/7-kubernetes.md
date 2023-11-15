Usato per gestire delle applicazione fatte a container che sono dentro un cluster composto da multipli nodi. Provvede all'automazione del deploy, scaling, rete e gestione.

K8s ha un'API, accessibile anche mediante REST, che gestisce tutto. Gestisce lo stato del sistema sotto forma di insieme di oggetti che devono avere uno stato. Gli oggetti che sono gestisti da K8s attualmente sono presenti dentro un persisten-storage di chiave-valore chiamato `etcd`. Un componente ha uno `scheduler` che si occupa di decidere sotto quale nodo un servizio va messo, e nel nodo vi sono svariati `pod`: questa decisione è basata in base a quale servizi servono o a sistemi di affinità/non affinità (come decidere chi ha una GPU). Il `controller-manager` monitora lo stato del sistema e verifica che lo stato attuale sia allineato con lo stato specificato dentro `etcd`. Il `cloud-controller-manager` permette di dialogare con un'infrastruttura di virtualizzazione perché i nodi potrebbero essere sul cloud come delle locali VM. Dal punto di vista di replicazione funziona per maggioranza e dunque si tende ad avere un numero di nodi *dispari*, ma ogni replica deve avere visione dello stesso stato: `etcd` ha un protocollo di consenso che garantisce tale visione replicata.
Su ogni nodo si ha un componente di gestione del traffico di rete `k-proxy` e un componente `kube` che si occupa di gestire il resto.

La gestione degli oggetti (pod, secrets, etc.) può essere fatta in modo imperativo (comandi dirtetti) o con configurazione dichiarative (più diffusa, e tutto funziona col fine di raggiungere tale stato).
Il meccanismo di associazione avviene mediante delle label (coppie chiave-valore) e selettori usati per associare tali oggetti.

## Pod
K8s istanzia pod, non container. Esso è un elemento composto da 1+ container. Son pensanti, di default, per condividere storage e rete. Essendo l'unità di deployment, sono sempre fatti localizzati e schedulati nel medesimo nodo.

## Namespace
È una partizione isolata nel cluster. Posson essere fatti creati in modo gerarchico.

## Oggetti workload
I pod sono effimeri e dunque non si lavora direttamente con essi. Gli oggetti di lavoro sono

- ReplicaSet: replica i pod.
- Deployment: dichiara gli aggiornamenti per i pod, usati per fault-tolerance e load-balancing.
- StatefulSet: gestisce le repliche.
- DaemonSet: gestisce come i pod siano allocati nei nodi. Se così non fosse, K8s potrebbe allocare tutti i pod nel medesimo nodo.
- Job: gruppi di pod che funzionano offline.

## Oggetti storage
- PersistentVolume: volumi persistenti che fanno richiestono sono collocati i dati.

## Oggetti di configurazione
- ConfigMap: coppie chiavi-valore lette dai pod usati per fare cose parametriche.
- Secret: coppie chiavi-valore usati per essere confidenziali, come segreti TLS.

## Oggetti d'accesso
- Service: espone l'applicazione in una rete, sia interna e/o esterna. Costruito sopra deployment (o StatefulSet). Un servizio avrà un nome dentro il DNS interno: dunque comunicano richiamando tale nome.
- Ingress: gestisce l'accesso dall'esterno più facilmente identificabile con meccanismi più sofisticati. È essenzialmente un gateway elementare.

### Tipi di accesso
- ClusterIP: alloca un indirizzo IP virtuale al pod, riservato per tutti i nodi in cui gira K8s. `k-proxy` si occupa che il traffico dal nodo $\to$ ip, vengano rediretti a tale pod.
- NodePort: simile a quello di ClusterIP ma in più, in ogni nodo, istanzia un proxy (sempre `k-proxy` ma con port-number specifico). Funziona come NAT. Si accede ad una porta che poi viene rendirizzata al nodo specifico.
- LoadBalancer: usa un meccanismo che permette di accedere dall'esterno al NodePort ma mettendo davanti un load balance che funge da piccolo gateway.


## Test con Minikube
Si scrive il file `app.yaml` e poi si mette *up* con `kubectl apply -f app.yaml`.
Da lì si vede con `kubectl get nodes` i nodi che ci sono.
Minikube crea un nodo virtuale (che può essere una VM ma di default usa un altro container). Di solito, però, si fanno girare i nodi all'interno di VM o macchine fisiche.
Il pod, preso da `kubectl get pods`, si può vedere con il nome preso dalle specifiche + un ID. 
L'immagine deve girare nella repo locale del nodo, non della macchina host: o si usa una repo remota o si trasferisce tale immagine, grazie a `minikube image load quickstart-mp`.
Lo stato `ImagePullBackOff` del pod controlla con tempo periodico crescente che l'immagine che cerca esiste.
I deployment si vedono con `kubectl get deploy` e tale struttura è gestita dal `ReplicaSet`.
I servizi si vedono con `kubectl get svc`: si vedrà per primo quello di Kubernetes (ma non ci interessa). In tutti i nodi del cluster si potrà deviare la comunicazione negli IP selezionati (questo chiaramente dipende dal tipo di servizio, se NodePort o altro).
Il deployment sa sempre qual è lo stato di salute dei pod che sta gestendo.
L'indirizzo IP è quello virtuale (salvato automaticamente nel DNS locale), mentre l'endpoint è quello fisico a cui vien mappato. Dall'esterno invece si usa la specifica del NodePort (eg: 30191/TCP) a cui arriva all'indirizzo IP virtuale che poi va nell'endpoint: basta usare *qualunque* nodo del cluster, l'importante è la porta. Con `kubectl describe service/<nome>` si hanno tutte le info. Minikube espone automaticamente le porte all'esterno; di default non si può accedere ai container perché isolati. Si vede come `minikube service <nome> --url`.
Si possono fare più repliche da `kubectl scale --replicas=<n> deployment/<name>`.
In genere gli IP dei nodi locali non vogliono essere esposti, dunque si espone un servizio di load balancer (che sia di AWS se specificato o locale, ma deve esserci!); però il servizio dell'esempio deve essere `LoadBalancer` e non `NodePort` (vedi come fare Traefik).