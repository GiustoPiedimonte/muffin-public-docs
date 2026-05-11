# 05. Verifica e affidabilità

> Come Muffin riconosce di non sapere, perché la confidence va resa esplicita per ogni claim, come la verifica deterministica dei nomi propri compensa la tendenza a confabulare, e perché alcune scelte costano cicli di calcolo che valgono la pena.

---

## Il problema strutturale

I modelli di linguaggio generano fluentemente cose vere e cose false con la stessa confidenza apparente. Per un'AI conversazionale generica è un fastidio gestibile; per un'AI personale longitudinale è una minaccia esistenziale al dataset.

Una sola affermazione confabulata che entra nel substrato come fatto — *"l'utente ha detto X il mese scorso"* quando non l'ha mai detto — contamina tutto quello che ne discende: pattern, osservazioni, frammenti di profilo, sintesi notturne. La contaminazione è particolarmente pericolosa perché è invisibile: l'errore si presenta nel sistema con la stessa forma di un fatto vero.

Ricerca recente (IFEval++, lavori di Microsoft e Salesforce nel 2025) ha mostrato che i modelli ignorano sempre più frequentemente le istruzioni del system prompt all'aumentare della profondità del prompt — un fenomeno chiamato *instruction attenuation*. Ricerca di Google nel 2025 ha caratterizzato il *value-action gap*: il modello dichiara cosa farà ma non lo fa nei fatti. Pare-Bench ha mostrato che modelli di taglia media propongono soluzioni prematuramente in più di tre quarti dei casi senza vincoli meccanici.

Le tre cose insieme dicono una cosa: **il prompt da solo non basta a evitare la confabulazione**. Servono meccanismi esterni al modello, ancorati al contesto specifico dell'utente, che producano segnali che il modello non può ignorare.

Muffin tratta la verifica come un pilastro funzionale — uno dei sei — e lo organizza in tre livelli con costo crescente.

---

## Livello 1 — Self-check alla genesi (costo: zero o quasi)

Il livello più economico. Mentre il modello sta generando una risposta, alcuni segnali deterministici (calcolati senza chiamare un altro modello) gli arrivano come *contesto ambient*. Il modello vede:

- **Termini sconosciuti nel messaggio dell'utente** — un sistema deterministico (regex più lookup nel database) estrae nomi propri dal messaggio e verifica se il sistema li ha mai incontrati. I nomi che il sistema non riconosce vengono segnalati esplicitamente al modello, accanto al messaggio dell'utente. Il modello sa che ha un punto cieco e ha un tool dedicato per risolverlo.
- **Stato di salience del turno corrente** — è un turno normale o sopra la media? Se sopra, il modello sa che la risposta richiede più attenzione del solito.
- **Dove il sistema sta sbagliando di solito a leggere l'utente** — il counterpoint, iniettato selettivamente nei turni di carattere personale-valutativo.

Il principio: *prima che il modello commetta a una risposta basata su un prior allucinato, gli si dà un anchor esterno per fermarsi e verificare*. Il costo è essenzialmente zero — sono operazioni deterministiche calcolate prima della chiamata al modello.

Questo livello non garantisce la verifica — il modello può sempre ignorare il segnale. Ma alza materialmente la probabilità che il modello si fermi quando dovrebbe.

### Stato sentito vs capability invocata

Una distinzione che pervade il design del pillar verifica.

- **Stato sentito**: il modello *vede* il segnale (es. *"questi termini non li conosco"*). Non agisce ancora — sa.
- **Capability invocata**: il modello *invoca* uno strumento per risolvere (es. *"verifica questo termine"*). Agisce.

I due si chiudono insieme. Senza stato sentito, la capability resta inutilizzata (il modello non sa di doverla chiamare). Senza capability, lo stato sentito è frustrante (il modello sa di non sapere ma non può rimediare). Muffin progetta sempre i due in coppia.

---

## Livello 2 — Re-read condizionale (costo: medio)

Per output che vengono accodati e mostrati all'utente in modo asincrono — osservazioni proattive in coda da ore o giorni, sintesi notturne pronte per la mattina — c'è una verifica successiva, attivata solo quando alcune condizioni si verificano.

Esempi:

- Un'osservazione generata ieri mattina, in coda per essere mostrata stamattina: prima della consegna, il sistema verifica che il contesto su cui era basata sia ancora coerente. Se episodi nuovi nel frattempo l'hanno smentita, l'osservazione viene scartata o aggiornata.
- Un fatto temporale (*"è successo X giorni fa"*) prima di entrare nella risposta finale: viene verificato deterministicamente — il sistema controlla che gli episodi corrispondenti esistano davvero a quella distanza temporale, e revisiona la formulazione se non è così.

Il costo è medio: alcune query al database, eventualmente una chiamata leggera al modello per riscrivere. Ma è pagato solo nei casi che lo giustificano (output asincroni, claim quantitativi/temporali specifici), non sistematicamente.

Il pattern alla base — *re-read after retrieval, revise* — è studiato in letteratura come RARR (*Retrieve, Attribute, Refine and Revise*). Muffin lo adatta al proprio dominio, focalizzandolo su claim temporali e numerici dove la confabulazione ha costo alto.

---

## Livello 3 — Predict-calibrate sistematico (costo: distribuito nel tempo)

Il livello più sottile e potente. È un meccanismo che opera durante il dream cycle e produce un segnale di calibrazione del sistema sopra l'utente.

Funziona così:

1. **Predict**: il sistema prende un episodio passato e, sulla base di tutto quello che sapeva *prima* di quell'episodio, predice cosa sarebbe successo. Una predizione concreta — non vaga.
2. **Calibrate**: il sistema confronta la predizione con cosa è effettivamente successo nell'episodio. Lo scarto (residuo) è un segnale.

Lo scarto sistematico (su molti episodi) dice cose specifiche:

- *"Prediggo bene il pomeriggio dell'utente, prediggo male le sue serate"* — porta a calibrare i pattern usati per la prima predizione (probabilmente accurati) rispetto a quelli usati per la seconda (probabilmente da rivedere)
- *"Le mie predizioni su come reagirà a temi tecnici sono accurate, quelle su come reagirà a temi emotivi non lo sono"* — porta a marcare con confidence più bassa gli pattern del secondo dominio
- *"Quando prevedevo X mi sbagliavo sempre per la stessa ragione"* — porta a aggiornare il pattern X con una correzione sistemica

Il *predict-calibrate* è il pezzo che permette al sistema di accorgersi *di cosa non sa di non sapere*. È un meccanismo di calibrazione di secondo ordine — non solo *"sono sicuro di X"*, ma *"sto sbagliando sistematicamente in un certo modo, e ora lo so"*.

Il costo è distribuito: ogni notte il sistema esegue una piccola batch di predict-calibrate su episodi recenti, e nel corso di settimane accumula un quadro robusto della propria affidabilità per dominio.

Questo livello è la concretizzazione operativa del concetto di *understanding* (capire *oltre* che ricordare): senza predict-calibrate, il sistema può solo dire "ho registrato X". Con predict-calibrate, può dire "ho capito X al livello che mi permette di prevedere il prossimo episodio coerente".

---

## Confidence esplicita su ogni claim

Già menzionata nel capitolo sulla memoria, ma vale ribadirla nel contesto della verifica.

Ogni claim derivato (fatto, osservazione, pattern, frammento di profilo) ha una confidence esplicita tra zero e uno, **accompagnata dalla fonte di confidence** (cosa giustifica il numero). Le fonti possibili includono:

- *Conta di evidenza* — quante volte il claim è stato confermato
- *Conferma dell'utente* — il claim è stato menzionato e accettato esplicitamente
- *Calibrazione predittiva* — il claim è stato usato per predire e la predizione si è verificata

Quando il modello compone una risposta, vede i claim con la loro confidence. Un claim a confidence alta entra nella risposta come asserzione. Un claim a confidence media entra come *"sembra che..."*. Un claim a confidence bassa non entra affatto, oppure entra come domanda esplicita all'utente — *"mi pare di ricordare che... è ancora vero?"*.

Questo trasforma il modo in cui il sistema parla. Non è più un narratore omniscente che afferma; è un osservatore che ha gradi di credenza differenziati su cose diverse, e li espone.

---

## Pattern lifecycle — la verifica nel tempo

Un pattern (*"di solito quando X allora Y"*) non è un fatto statico. È una credenza statistica che resta valida fintanto che l'evidenza la conferma. Quando l'evidenza nuova non la conferma più, il pattern deve essere aggiornato — ma in modo che la sua storia non sparisca.

Muffin gestisce il ciclo di vita di un pattern in fasi:

1. **Estrazione iniziale** — un pattern emerge da N episodi simili. Confidence iniziale bassa (è un'ipotesi).
2. **Rinforzo** — nuovi episodi confermano il pattern. La confidence cresce.
3. **Calibrazione** — il pattern viene usato per predire (predict-calibrate). Le predizioni si verificano? Confidence ulteriore. Non si verificano? Confidence cala.
4. **Stale-ness** — se per molto tempo non arriva né evidenza né smentita, il pattern diventa stale. Resta nel sistema ma non viene più usato attivamente per il presente.
5. **Invalidazione temporale** — se nuova evidenza esplicita lo smentisce, il pattern viene marcato non più valido da una certa data, ma resta interrogabile come parte della storia.

Il principio: i pattern *invecchiano* come le persone. Una volta veri, non lo sono più; o lo erano in un periodo specifico e non lo sono più ora. La bi-temporalità (vedi capitolo memoria) si applica ai pattern come si applica ai fatti — un pattern non scompare quando perde validità, smette solo di essere usato per il presente.

---

## Una conseguenza operativa: il modello sa di non sapere

L'effetto cumulativo dei tre livelli più la confidence esplicita: **Muffin parla in modo più calibrato di quanto un modello base parlerebbe** sopra lo stesso dataset.

Quando l'utente chiede *"come sono io?"*, un modello base sopra una memoria di episodi recenti genererebbe una sintesi fluente con confidenza apparente uniforme. Muffin genera una sintesi calibrata, con margini di incertezza diversi su parti diverse, segnalazioni esplicite di *"qui sono meno sicuro"*, citazioni di provenance per i punti più solidi.

È una differenza sottile ma cumulativamente importante. Su un orizzonte di anni, è la differenza tra un sistema che produce comprensione robusta e uno che produce confabulazione gradualmente sempre meno distinguibile dalla verità.

---

## Una nota sulla cerimonia

Una tentazione frequente nei sistemi di verifica è la *cerimonializzazione*: il sistema dice *"ho verificato"* senza aver verificato davvero. È peggiore dell'assenza di verifica perché crea fiducia ingiustificata.

Muffin evita questa trappola in due modi.

Primo, **i meccanismi di verifica sono prevalentemente esterni al modello**. Il sistema deterministico che estrae termini sconosciuti dal messaggio dell'utente non passa per il modello — produce un segnale che il modello vede. Il fatto che il segnale esista non dipende dalla cooperazione del modello.

Secondo, **i livelli di verifica sono asimmetrici per costo**. Si paga di più per verifiche che hanno costo alto (predict-calibrate, re-read condizionale) e che non si possono fingere senza pagare il costo. Si paga zero per verifiche economiche che si possono fingere ma sono ovunque (il segnale ambient di stato sentito).

L'asimmetria evita che il sistema si abitui a *dire* di verificare senza verificare davvero. Lo dice — ma anche lo fa, perché i meccanismi che lo costringono a farlo sono incassati nel codice e non delegati al modello.
