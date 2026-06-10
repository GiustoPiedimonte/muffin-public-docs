# 05. Verifica e affidabilità

> Come Muffin riconosce di non sapere, perché la confidence va resa esplicita per ogni claim, come la verifica deterministica dei nomi propri compensa la tendenza a confabulare, perché "cosa hai fatto stanotte?" si risponde leggendo un log e non ricostruendo a memoria — e come la stessa disciplina di verifica si applica, via eval su casi reali, alle decisioni architetturali del progetto stesso.

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

Una variante specifica di re-read condizionale: nelle risposte di gruppo in cui Muffin ha appena fatto una ricerca web, gli URL nella risposta finale vengono estratti deterministicamente (regex) e confrontati uno a uno con la concatenazione dei testi tornati dalle ricerche del turno. Se un URL nella risposta non compare verbatim nei risultati di ricerca, è quasi certamente *fabbricato* — un pattern classico in cui il modello inventa uno slug plausibile a partire dal titolo. Il match è una funzione di una riga; il valore è alto perché un URL inventato che entra in un canale pubblico è uno dei modi più visibili in cui un'AI personale può perdere credibilità.

Il pattern alla base — *re-read after retrieval, revise* — è studiato in letteratura come RARR (*Retrieve, Attribute, Refine and Revise*). Muffin lo adatta al proprio dominio, focalizzandolo su claim temporali, numerici e link dove la confabulazione ha costo alto.

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

## Contradizione come segnale di prim'ordine

Una proprietà collegata alla confidence esplicita, che merita di essere trattata come pillar verifica a sé: il sistema osserva ed estrae anche le proprie *dichiarazioni* — non solo i fatti del substrato, ma le cose che Muffin ha asserito di sé, dell'utente o del mondo nelle sue risposte recenti. Queste dichiarazioni diventano interrogabili come storia.

Quando un turno successivo introduce informazione che potrebbe contraddire una dichiarazione precedente, un classifier valuta la relazione in due passaggi: prima un filtro deterministico di similarità (per evitare di passare al modello coppie semanticamente lontane), poi un classifier di *Natural Language Inference* leggero che distingue tra *contraddizione logica*, *implicazione* e *neutralità*. È una distinzione che il prompt da solo non sa fare bene: un utente che riformula la stessa idea con parole diverse non sta contraddicendo, e trattarlo come tale produce inversioni di posizione gratuite (sycophancy).

Quando emerge una contraddizione genuina, il claim del sistema non sparisce — viene marcato come *contraddetto*, con riferimento al turno che lo ha smentito. Il pool di claim contraddetti recenti viene iniettato come segnale ambient nei turni successivi: il modello sa di avere posizioni in tensione e ha tre vie esplicite (cambiare idea motivando, mantenere la posizione spiegando perché l'evidenza non basta, ignorare il pivot se è già stato risolto). La forma a tre vie è importante per non degenerare nei due estremi (flippare al primo segnale, oppure ignorare la smentita finché l'utente la urla).

Il dettaglio operativo è trattato nel capitolo memoria sotto la voce *belief revision* — qui basta dire che è parte integrante del pillar verifica: la verifica non è solo *"controllo che il claim corrisponda alla realtà"*, è anche *"controllo che il claim corrisponda al resto di quello che ho già detto, e quando non lo fa, lo dichiaro invece di nasconderlo"*.

---

## Memoria autobiografica — narrare leggendo, non ricostruendo

C'è una classe di confabulazione che i meccanismi visti finora non coprono: quella del sistema **sulle proprie azioni**. *"Stanotte ho consolidato il grafo"*, *"te l'ho già salvato ieri"* — detti con piena confidenza, perfettamente coerenti, e inventati. È una classe insidiosa per una ragione precisa: i metodi che rilevano l'allucinazione misurando l'*incertezza* del modello (campionare più risposte e confrontarle, stimare l'entropia semantica) qui sono ciechi, perché una confabulazione d'azione non è incerta — è confidente e internamente coerente. Il modello non sta esitando tra versioni: sta raccontando con convinzione una giornata plausibile che non è avvenuta.

Il progetto ha una storia istruttiva su questo punto. Una prima risposta al problema era stata un *verificatore post-hoc* di claim temporali: regex che riconoscevano formule come "stamattina" e "stanotte" nelle risposte, e un controllo a valle. In tutta la sua vita operativa ha flaggato **una volta**. È stato rimosso in una consolidation, a ragione. Quando il problema è riemerso mesi dopo, la tentazione era reintrodurlo — e la ricerca fatta prima di cedere alla tentazione (due survey indipendenti, una sui paper e una sui sistemi in produzione) ha mostrato che nessun sistema maturo costruisce un verificatore dedicato per questo: l'ancoraggio emerge dal **log d'azione strutturato**. *"Cosa ho fatto"* non è una generazione da verificare — è una *lettura* da eseguire.

Il meccanismo attuale segue quella linea. Quando il messaggio dell'utente chiede conto dell'attività recente del sistema (*"cosa hai fatto stanotte?"*, *"hai poi sistemato quella cosa?"*), un trigger semantico lo riconosce — per similarità vettoriale con il concetto di "domanda sull'attività del sistema", non con una word-list di formule da matchare — e il sistema inietta nel contesto un blocco costruito da una **lettura deterministica del proprio log d'azione** delle ultime 24 ore, con voci timestampate. Muffin narra *leggendo*, non ricostruendo a memoria.

È lo stesso pattern della working memory per i deittici (capitolo memoria): invece di correggere a valle quello che il modello ha generato male, gli si mette sotto gli occhi la fonte giusta *prima* che generi. Rimuovere la confabulazione alla fonte batte rincorrerla a valle — e costa meno: una query, zero chiamate aggiuntive al modello, nessun verificatore da mantenere.

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

## Eval come gate — verificare il sistema, non solo l'output

Tutto quanto sopra verifica *l'output* del sistema. Da fine maggio 2026 la stessa disciplina si applica un livello sopra: alle **decisioni architetturali** del progetto. Lo strumento è una suite di eval costruita su casi reali del dataset — fixture curate a mano da conversazioni effettive, inclusa una collezione di *turni difficili* (i casi in cui il sistema storicamente sbaglia: salti di tool, riferimenti temporali, ambiguità deittiche) — che misura come un modello si comporta sui *nostri* dati, non su benchmark generici.

La regola che ne è discesa: **un cambio di modello non è un flip di configurazione — è un eval gate.** Un prompt e una superficie di tool ottimizzati per un modello perdono decine di punti su un altro; chi cambia il motore ri-misura tutto, prima. (È anche la storia del futuro open source: *porta il tuo modello, ri-baselina con la suite*.)

Tre episodi del giugno 2026 mostrano cosa significa in pratica prendere questa regola sul serio.

**Il reversal.** Il progetto aveva scelto un nuovo modello primario sulla base delle sue caratteristiche dichiarate. Le misure, fatte dopo, hanno detto l'opposto: sulla qualità di estrazione (il moat) e sulla tenuta della voce (l'identità) il modello scelto era *peggiore*, e su un asse critico — l'affidabilità agentica — fingeva azioni quasi metà delle volte. La scelta è stata ribaltata in tre giorni. Non è un incidente imbarazzante da nascondere: è il sistema di decisione che funziona. Una decisione sbagliata sopravvissuta tre giorni costa poco; la stessa decisione difesa per orgoglio per sei mesi avvelena il dataset.

**Il refactor falsificato.** La letteratura suggeriva di sostituire la superficie a meta-tool del sistema di memoria con tool atomici piatti. Misurato sul modello in produzione: meno trenta punti di accuratezza nella selezione. Il refactor non è stato shippato. La barra era deliberatamente asimmetrica — il cambiamento doveva *migliorare* le misure per passare, non limitarsi a non peggiorare — e ha falsificato un'idea plausibile prima che diventasse una regressione.

**Il meccanismo rimosso.** Un eval aveva mostrato che il modello allora in produzione fingeva certe azioni la metà delle volte (*"te lo ricordo"* senza chiamare davvero il tool del promemoria). La risposta era stata un meccanismo deterministico che eseguiva l'azione da codice. Mesi dopo, la *stessa misura ripetuta sul modello effettivamente deployato* ha dato 30 su 30: il modello nuovo non finge. Il meccanismo è stato rimosso — non per pulizia estetica, ma perché la misura che lo giustificava non valeva più. Resta il monitor passivo, e un criterio esplicito di ritorno: se le azioni finte riappaiono in produzione, il meccanismo si riaccende mirato su quella famiglia di azioni. *Osserva prima, imponi solo su evidenza.*

La lezione trasversale dei tre episodi: **un eval sintetico valida il meccanismo, non la magnitudine sul modello reale** — e una misura fatta su un modello non si trasferisce al successivo. È la stessa onestà dichiarato-vs-reale degli audit periodici (capitolo riferimenti), applicata alle decisioni invece che alle pipeline: lo scarto tra ciò che il progetto crede e ciò che è misurabilmente vero produce interventi, in entrambe le direzioni — anche quando l'intervento è disfare qualcosa di costruito da poco.

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
