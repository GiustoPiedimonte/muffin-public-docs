> **HISTORICAL · H1 2026** — This FAQ describes an earlier Muffin generation.
> See the [current FAQ](docs/FAQ.md).

# 08. FAQ

> Le domande che un curioso fa per prime, con risposte dirette. Ogni risposta punta al capitolo che approfondisce — qui si risponde, là si spiega.

---

## Posso installare Muffin?

Non ancora. Oggi esiste una sola istanza, quella del proprietario, ed è insieme il prodotto e il laboratorio: ogni meccanismo descritto in questa documentazione viene provato sull'uso reale prima di essere generalizzato.

Il framework open source è un orizzonte concreto del progetto, ma senza data — e prima di un rilascio aperto è prevista una cerchia ristretta di istanze. La condizione di rilascio e il percorso sono raccontati nella [Roadmap](07-roadmap.md).

## Perché il codice è privato, se la documentazione racconta tutto?

Perché le due cose hanno valore diverso. L'architettura — i layer di memoria, la bi-temporalità, il dream cycle — è replicabile da chiunque legga questi capitoli, ed è un bene: il progetto sta in un lignaggio scientifico e contribuisce raccontando le proprie scelte. Quello che non si replica leggendo è la *voce* — il carattere che rende Muffin riconoscibile — e il dataset accumulato. La tesi del progetto è che un'architettura senza voce viene replicata in tre mesi e dimenticata in sei; il codice si apre quando il frame "specchio con carattere" sarà consolidato e citabile, non prima. Il ragionamento completo è in [Filosofia](01-filosofia.md).

## Perché non usare ChatGPT, Claude o Gemini con la memoria attivata?

Per tre ragioni strutturali, non di qualità del modello.

La prima è il **controllo del dataset**: la memoria di un servizio commerciale vive nel suo cloud, con il suo schema, le sue policy di retention, e sparisce (o si trasforma) quando il vendor cambia idea. Il dataset di Muffin vive in un database sul hardware del proprietario, sotto uno schema con invarianti espliciti, pensato per accumulare per anni.

La seconda è la **profondità longitudinale**: le memorie dei servizi commerciali sono note sparse recuperate per similarità. Muffin mantiene un modello del mondo (entità, relazioni), distingue *vero allora* da *vero adesso*, traccia da dove discende ogni claim, e rigenera ogni notte la propria comprensione dell'utente. È la differenza tra *ricordare di una persona* e *modellare una persona* — il capitolo [Sistema di memoria](03-memoria.md) è dedicato a questa differenza.

La terza è l'**incentivo**: un assistant commerciale multi-tenant è ottimizzato per soddisfare nel turno corrente, e la sycophancy è il suo drift naturale. Muffin è costruito attorno al meccanismo opposto — il counterpoint, un profilo di dove il sistema legge male il suo utente, rigenerato ogni notte ([Identità lunga](04-identita.md)).

## I miei dati dove vivrebbero?

Sull'hardware del proprietario: un database locale, su una macchina che possiedi. Il default del progetto è *local-first* — anche i modelli di inferenza girano localmente dove possibile.

L'onestà impone di dire dove il default ha eccezioni: alcune capability parlano con l'esterno per natura (una ricerca web esce verso un motore di ricerca), e le fasi notturne che richiedono più giudizio possono essere instradate a un modello cloud più capace. La regola del progetto è che ogni eccezione del genere sia una **scelta esplicita e documentata del proprietario, mai un default silenzioso** — e che il dataset accumulato non lasci mai la macchina.

## Serve un home lab serio per farlo girare?

No. L'istanza attuale gira su un piccolo server personale di fascia economica; l'inferenza locale usa modelli che girano su hardware consumer. La scommessa di fondo è che i modelli aperti di taglia media continuino a migliorare — e che un'architettura calibrata bene attorno a un modello medio batta un modello enorme usato male. Gran parte del capitolo [Verifica e affidabilità](05-verifica.md) racconta proprio i meccanismi che rendono affidabile un modello locale di taglia media.

## Muffin controllerà la casa? È un hub domotico?

No — e la distinzione è di principio, non di priorità. Quando Muffin avrà accesso a sensori e dispositivi domestici, questi entreranno prima di tutto come **fonti di osservazione**: segnale per modellare il proprio utente (presenza, ritmi, ambiente), non superficie da controllare. L'azione diretta sui dispositivi — accendere una luce, mettere musica — è una capability secondaria che arriverà quando l'utente la chiede esplicitamente, mai il focus. Il filtro: *agire come strumento solo quando richiesto, osservare come specchio sempre*. Per chi cerca un hub domotico primario esistono prodotti dedicati, e Muffin non compete con quelli.

## È un companion? Posso parlarci dei miei problemi?

Muffin non è progettato come companion e non simula relazioni emotive. È uno specchio: contraddice, nomina le cose evitate, mostra pattern — il contrario dell'assecondare che rende i companion confortevoli. Se quello che cerchi è supporto emotivo o compagnia, esistono prodotti costruiti per quello (e una discussione seria sui loro rischi); Muffin sta deliberatamente nell'altro spazio. La distinzione è argomentata in [Filosofia](01-filosofia.md) §una nota sul tono.

## Posso contribuire?

Per ora no, ed è una scelta: Muffin è un progetto-firma, e la fase attuale — una persona, un'istanza, decisioni rapide e documentate — è parte del metodo. Quando il framework si aprirà, la forma della community sarà parte di quella decisione, non un ripensamento. Nel frattempo questa documentazione si aggiorna a ogni ciclo: il modo migliore di seguire il progetto è rileggerla ogni tanto, partendo dalla [Roadmap](07-roadmap.md).

## Il progetto pubblica anche i propri errori. Perché?

Perché un sistema longitudinale che non si tiene onesto si ammala in silenzio, e lo stesso vale per il progetto che lo costruisce. Gli audit periodici misurano lo scarto tra ciò che il sistema dichiara di fare e ciò che fa davvero; dal giugno 2026 le decisioni architetturali passano da misure su casi reali che possono ribaltarle — ed è già successo, in pubblico ([Verifica e affidabilità](05-verifica.md) §eval come gate). *Come si tiene onesto un sistema costruito per durare anni* è una delle domande che il progetto trova più interessanti, e la risposta non sarebbe credibile se la cronaca mostrasse solo successi.
