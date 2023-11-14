Vi sono diversi tipi di handler per i fault.
- Fault: classico stile try-catch;
- Termination: utile nel caso di processo parallelo, uccidendo in modo "smooth" per tornare, in qualche modo, in uno stato consistente;
- Compensation: torna indietro per un'attività già eseguita.

Si definisce uno scope nel codice in cui il corpo può triggerare un fault generico di una parte.

```jolie
scope(name) {

}
```

il `main` è uno scope implicito.

Si possono generare fault, chiamati logici, come
```jolie
throw(faultname, data)
```

per accede a tali dati si prende da `<scopename>.<faultname>`.
Gli handler possono essere installati dinamicamente come
```jolie
install(faultname1 => ..., faultname2 => ...)
```

Per lo scope corrente viene fatto con
```jolie
install(this => ...)
```

I recovery degli scope possono avvenire in back (torna indietro nelle attività) o forward (una specie di undo).

`install` ha priorità nell'esecuzione dei fault, risolvendo problemi di atomicità.

L'operatore `^` freeza il valore delle variabili.

In una `RequestResponse` potrebbero generare un fault e notificarlo a chi ne fa richiesta. Viene definita come
```jolie
RequestResponse: <op> throws <faultname>(<type>)
```


