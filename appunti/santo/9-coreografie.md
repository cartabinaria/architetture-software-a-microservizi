Serve per far comunicare diverse parti. Una coreografia descrive come avviene lo scambio di messaggi dal punto di vista globale.
Esse sono astrazioni: si concentrano su quali messaggi vengono scambiati. In un qualche momento un partecipante $A$ manda ad un partecipante $B$ qualche messaggio $m$. $B$ si aspetta tale messaggio!
Le coreografie non definisco le computazioni locali o quali vengono inviati.

La differenza con l'orchestrazione è che quest'ultima descrive la conversazione da un punto di vista centralizzato.

Un monitor è un componente che lancia una "red flag" se il partecipante che monitora non si comporta come dovrebbe. E questo lo fa a runtime.

Prendendo ad esempio la scrittura di un logger: proiettando la coreografia chi fa il logger interessa solo la descrizione locale di esso. Quindi vi è questa distinzione tra visione globale e locale.

La sintassi è formata da un algebra
- interazione
  $$C ::= o: a \to b$$
Il partecipante $a$ manda un messaggio a $b$ usando l'operazione $o$.

- composizione sequenziale
$$C ::= C;C$$
- composizione parallela
$$C ::= C|C$$
- scelta non deterministica
$$C ::= C+C$$
succede una cosa o l'altra.

- iterazione
$$C ::= C*$$
la cosa avviene 0 o più volte.

- salto
$$C::= 1$$


Prendiamo l'esempio
$$(\text{ask}: \text{cl} \to \text{se} ; (\text{answer}: \text{se} \to \text{cl} | \text{log}: \text{se} \to \text{logger}))*; \text{end}: \text{cl} \to \text{se}; \text{endl}: \text{se} \to \text{logger}$$
significa che $0+$ volte avviene un azione $\text{ask}$ da client al server. In modo sequenziale il server risponde al client e logga al logger. Infinite, sequenzialmente, il client esegue $\text{end}$ al server e il server invia $\text{endl}$ al logger.

Un sistema definisce i ruoli, che nel caso sopra sarebbero client, server e logger.
$$S ::= (P)_a$$
- una composizione parallela è
$$S::= S|S$$
qui $P$ è un partecipante.

- un output è dato come
$$P ::= \overline{o}@a$$
manda un messaggio ad $a$ sull'operation $o$.

- un input è
$$P::= o@a$$
e poi, in ordine, composizione sequenziale, parallela, non deterministica, iterazione e salto.
$$P;P$$
$$P|P$$
$$P+P$$
$$P*$$
$$1$$
Per far sì che interagiscono, l'iterazione $o:a\to b$ è lo stesso di fare un output $\overline{o}@b$ e un input $o@a$.
Se vien fatto in modo sincrono va bene, ma in modo asincrono vuol dire che l'input gira nella rete e dunque il dato dev'essere atteso e quindi si ha una $\text{send}$ e poi dopo una $\text{receive}$.

$$proj(o:a\to b,a)=\overline{o}@b$$
$$proj(o:a\to b,b)=o@a$$
$$proj(o:a\to b,c)=1$$

un omomorfismo negli altri operatori è
$$proj(C; C^\prime, a)= proj(C, a); proj(C^\prime, a)$$

Prendiamo un esempio del tipo
$$o: a \to b; o^\prime : c \to d$$
nel caso sincrono (e atomico) la prima interazione deve avvenire prima; nel caso asincrono vi sono modi diversi di esecuzione.

$$(\overline{o}@b; 1) a\ |\ (o@a; 1) b\ |\ (1; \overline{o^\prime}@d) c\ |\ (1; o^\prime@c) d$$
si può anche semplificare togliendo gli $1$ perché non hanno influenza.
$$(\overline{o}@b) a\ |\ (o@a) b\ |\ (\overline{o^\prime}@d) c\ |\ (o^\prime@c) d$$

non c'è modo di sapere che l'altro abbia finito! Verosimilmente una soluzione potrebbe essere
$$o: a \to b; o^{\prime\prime} : b \to c; o^\prime : c \to d$$
$$(\overline{o}@b) a\ |\ (o@a; \overline{o^{\prime\prime}}@c) b\ |\ (o^{\prime\prime}@b; \overline{o^\prime}@d) c\ |\ (o^\prime@c) d$$
dunque, la soluzione corretta per questo sistema dovrebbe essere una in parallelo.
$$o: a \to b| o^\prime : c \to d$$

Se invece fosse
$$o: a \to b; o^\prime : c \to b$$
si avrebbe correttamente in modo sincrono
$$(\overline{o}@b) a\ |\ (o@a; o^\prime@c) b\ |\ (\overline{o^\prime}@b) c$$

ma non in modo asincrono non si saprebbe quale dei due arriverà prima.

La correctness per la sequenza verrebbe data da:
$$o: a\to b; o^\prime: c \to d$$
- sincro: $a=c$ o $a=d$ o $b=c$ o $b=d$
- sender: $a=c$ o $b=c$
- receiver: $b=c$ o $b=d$
- disjoint: $b=c$

*Esempio del compratore e dell'helper*
$b$ = buyer
$s$ = seller
$h$ = helper
$$((\text{buy}:b \to s+(\text{askHelp}: b \to h ; (\text{buyHalf}: b \to s | \text{buyHalf}^\prime: h \to s ))); \text{sell}:s \to b)^\star$$
oppure
$$(\text{askPrice}: b \to s; \text{sendPrice}: s \to b; ((\text{pay}:b\to s) + ((\text{askHelp}:b \to h; \text{payHalf}: h \to s)|\text{payHalf}^\prime: b \to s)); \text{sendItem}:s \to b)^\star;(\text{end}:b \to s | \text{end}^\prime:b \to h)$$
in realtà ci starebbe anche in parallelo nel $\text{pay}: b \to s$ con la richiesta di non aiuto e di non pagamento dell'helper.

---
$$((\overline{\text{askPrice}}@s)b|(\text{askPrice}@b)s;$$
$$(\overline{\text{sendPrice}}@b)s|(\text{sendPrice}@s)b;$$
$$(((\overline{\text{pay}}@s)b|(\text{pay@b})s)+$$
$$(((\overline{\text{askHelp}}@h)b|(\text{askHelp}@b;\overline{\text{payHalf}}@s)h|(\text{payHalf}@h)s))|(\overline{\text{payHalf}^\prime}@s)b|(\text{payHalf}^\prime@b)s)|$$
$$((\overline{\text{sendItem}}@b)s|(\text{sendItem}@s)b))^\star;$$
$$(\overline{\text{end}}@s)b|(\text{end}@b)s;(\overline{\text{end}^\prime}@h)b|(\text{end}^\prime@b)h$$
---

$$proj(C, b)=(\overline{\text{askPrice}}@s;\text{sendPrice}@s;\overline{\text{pay}}@s+(\overline{\text{askHelp}}@h|\overline{\text{payHalf}^\prime}@s);\text{sendItem}@s)^\star;(\overline{\text{end}}@s|\overline{\text{end}^\prime}@h)$$
$$proj(C, s)=(\text{askPrice}@b;\overline{\text{askPrice}}@b;\text{pay}@b+(\text{payHalf}@h|\text{payHalf}^\prime@s);\overline{\text{sendItem}}@b)^\star;(\text{end}@b)$$
$$proj(C, h)=(\text{askHelp}@b;\text{payHalf@b});\text{end}^\prime@b$$
