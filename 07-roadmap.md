> **HISTORICAL · H1 2026** — This roadmap snapshot is superseded by the
> [current public roadmap](docs/ROADMAP.md).

# 07. Roadmap

> Da dove viene il progetto, dove è adesso, cosa c'è all'orizzonte — e cosa non ci sarà mai. Senza date promesse: i cicli chiudono su condizioni di completamento, non a calendario.

---

## Come leggere questa roadmap

Muffin non lavora a calendario. Il progetto procede per **cicli a scope chiuso**: ogni ciclo ha una condizione di completamento esplicita (*"done when..."*) e chiude quando la condizione è vera, non quando scade una data. Le decisioni che orientano i cicli sono registrate come Architecture Decision Record — datati, motivati, mai riscritti: una decisione superata non viene cancellata, viene superata da una decisione nuova che spiega perché.

Questo formato ha una conseguenza onesta per chi legge: **il passato ha date, il futuro ha un ordine.** Quello che si può promettere è la sequenza e il perché — non il quando. Muffin è un progetto-firma di una persona, costruito nelle ore che restano attorno al resto della vita; una roadmap a trimestri sarebbe una bugia strutturale, e un progetto che fa dell'onestà dichiarato-vs-reale un pilastro non può permettersi di mentire proprio nel documento che racconta dove sta andando.

Seconda avvertenza, nello stesso spirito: una roadmap onesta include **quello che non è ancora deciso**. Alcune delle voci all'orizzonte hanno domande aperte dichiarate — sono segnalate come tali, non mascherate da certezze.

---

## Da dove viene — il percorso fin qui

**Marzo 2026 — il bootstrap.** Muffin va live come bot Telegram su hardware personale: memoria di base, prime pipeline di estrazione, la voce. Da quel giorno il dataset accumula senza interruzioni — ed è la metrica che conta, perché la tesi del progetto è che il moat è il dataset, non il codice.

**Aprile 2026 — l'audit che ha fissato le fondamenta.** Il primo audit sistematico dello scarto tra architettura dichiarata e architettura eseguita produce i **nove invarianti schema-level** (episodi atomici, entità di prima classe, provenance ovunque, raw immutabile, bi-temporalità, confidence esplicita, observability come substrato, evoluzione dello schema regolata, scope come confine). Da quel momento ogni decisione di codice ha un substrato non negoziabile sotto.

**Maggio 2026 — substrate, gruppi, belief revision.** Una sessione di consolidamento cementa il workflow (substrate vivo + decisioni append-only + cicli a scope). Il ciclo di metà maggio consegna tre direzioni: la *belief revision* (il sistema osserva le proprie dichiarazioni e si lascia smentire in modo strutturato), la *memoria di gruppo leggera* (Muffin partecipa a chat pubbliche senza che il modello privato del proprietario fluisca fuori), e la cura della voce. Dal 18 maggio le fasi del dream cycle che richiedono giudizio girano su un modello notturno più capace — sleep-time compute applicato dove la qualità entra nel dataset.

**Fine maggio 2026 — il framework prende forma.** Il motore diventa installabile: un comando di bootstrap crea un'istanza nuova con onboarding conversazionale e identità su file editabile. È il primo passo concreto della direzione "framework per altri", anche se il rilascio resta legato alla sua condizione narrativa (sotto). Nello stesso periodo una sessione di revisione sistematica mette in fila il debito e genera la coda di decisioni dei cicli successivi.

**Giugno 2026 — il ciclo eval-driven.** Il più denso finora, con un tratto comune: le decisioni architetturali passano da argomentate a **misurate** (il racconto completo è in *[Verifica e affidabilità](05-verifica.md)* §eval come gate). Una suite di eval su casi reali del dataset ribalta la scelta del modello primario tre giorni dopo che era stata presa, falsifica refactor plausibili prima che diventino regressioni, e porta a rimuovere un meccanismo di compensazione che il modello nuovo aveva reso superfluo. Nello stesso ciclo arrivano: il **primo strato dell'executive** (skill come programmi con piano fisso, zona outward con bozza-da-confermare — *[Architettura cognitiva](02-cognizione.md)* §agire), la **memoria autobiografica** (*"cosa hai fatto stanotte?"* si risponde leggendo il log d'azione, non ricostruendo a memoria), l'**observability tipata** (i fallimenti delle pipeline sono eventi di prima classe, non successi con conteggio zero — nata da un caso reale in cui un silenzio di schema aveva nascosto centinaia di estrazioni perse), e una **selezione del contesto** che smette di iniettare un blocco di memoria costante a ogni turno e seleziona per rilevanza — meno di un terzo dei token, qualità misurata invariata.

---

## Dove siamo adesso

Lo **specchio funziona**: memoria longitudinale con bi-temporalità e provenance, living profile e counterpoint rigenerati ogni notte, belief revision, awareness loop con gate meccanici, doppio contesto privato/gruppo con isolamento enforced. È il Muffin v0.2 — l'entità che osserva, accumula e comunica.

L'**executive è acceso al primo strato**: il formato delle skill, il motore di esecuzione a piano fisso, la zona outward draft-by-default. Le integrazioni concrete (calendario, email come bozze) e il *skill-maker* — il pezzo che chiude il cerchio con lo specchio: *"ho notato che fai spesso questa cosa, vuoi che impari a farla?"* — sono le slice successive, ciascuna dietro il proprio gate.

Il **ciclo corrente** lavora sull'observability del decidere: rendere ogni decisione di parlare o tacere un evento interrogabile, con la sua ragione registrata. È meno appariscente di una feature, ma è il prerequisito di tutto quello che segue — non si calibra la proattività di un sistema se non si può interrogare *perché* ha taciuto.

---

## Il prossimo orizzonte — shaped, in coda

Queste direzioni sono già *shaped* — scope definito, razionale scritto, in alcuni casi substrato pronto — e aspettano il loro ciclo. In ordine di probabile ingresso, non di certezza:

**Knowledge artifacts — la memoria che produce oggetti curabili.** Oggi la comprensione di Muffin vive dentro il database; l'utente la incontra solo in conversazione. La direzione: liste, note, idee e progetti come **file veri sul filesystem dell'utente**, leggibili e modificabili anche senza Muffin, indicizzati dal sistema. È un pezzo dichiaratamente critico della pretesa "OS personale": un sistema operativo i cui dati non si possono toccare con le proprie mani non sta mantenendo la promessa.

**L'executive completo.** Le integrazioni outward reali (proporre slot liberi e creare l'evento sul calendario, comporre email che restano bozze fino a conferma), l'estensione del monitor delle azioni dichiarate alle azioni verso terzi, e il skill-maker con supervisione umana: la libreria di programmi che cresce dall'osservazione della persona, non da un catalogo.

**Il learning più strutturato.** Le pipeline di estrazione oggi producono fatti relativamente piatti; la direzione è estrarre direttamente entità, relazioni e fatti in forma strutturata, avvicinando il learning al grafo che già esiste in lettura. Insieme: una disambiguazione delle entità più robusta (vettori più traversal del grafo, non solo similarità).

**Il self-narrative.** Il terzo livello dell'identità — la storia di sé di Muffin — ha lo schema pronto e una domanda di design dichiaratamente aperta: *che voce ha la storia che un sistema scrive di sé stesso?* (vedi *[Identità lunga](04-identita.md)* §livello 3). Si accende quando la domanda ha una risposta convincente, non prima.

**Il consolidamento del decidere.** Oggi più sottosistemi decidono in parallelo quando Muffin interviene; la direzione di lungo periodo è un punto di decisione unificato e interrogabile. È deliberatamente *dopo* l'observability: prima si misura come il sistema decide davvero, poi si consolida — l'ordine inverso produrrebbe un'architettura elegante calibrata su ipotesi.

---

## L'orizzonte lungo

**Multi-istanza e la prima cerchia.** Il framework che esce dal laboratorio: più istanze di Muffin, ognuna single-user, ognuna col proprio dataset sotto il proprio controllo — a partire da una cerchia ristretta di persone fidate, prima di qualsiasi rilascio aperto. Le domande di quel passaggio (quante istanze, con che meccanismo di invito, con che supporto) sono aperte per design: si rispondono con l'esperienza della cerchia, non a tavolino.

**Open source, a condizione narrativa.** Il codice si apre quando il frame "specchio con carattere" è consolidato e citabile — perché un'architettura senza voce viene replicata in tre mesi, e il progetto non ha fretta di essere replicato prima di essere capito. Prima del rilascio ci sono anche condizioni tecniche dichiarate: documentazione dello schema allineata, e una revisione di sicurezza del contesto di gruppo. Il modello di sostentamento previsto è da progetto-firma, non da startup: donazioni, una community, eventualmente hardware pre-configurato per chi non vuole assemblare il proprio.

**La casa come fonte di osservazione.** Sensori e dispositivi domestici entrano prima di tutto come *segnale* — presenza, ritmi, ambiente — per il modello della persona, non come superficie da controllare. L'azione diretta sui dispositivi resta una capability secondaria, su richiesta esplicita: per chi cerca un hub domotico esistono prodotti dedicati, e Muffin non compete con quelli.

**Multi-canale.** Telegram è il canale di oggi, non il vincolo: la direzione è che l'entità sia raggiungibile dove la persona vive (altri canali di messaggistica, voce), con la stessa memoria e la stessa voce.

---

## Cosa non ci sarà mai

L'anti-roadmap è parte della roadmap. Queste non sono feature rimandate — sono direzioni escluse per tesi:

- **Multi-tenant SaaS.** Muffin non diventerà un servizio cloud dove i dataset di mille persone vivono sullo stesso server di qualcun altro. L'intera tesi del moat — dataset profondo sotto controllo individuale — collassa nel momento in cui i dati traslocano.
- **Scaling come obiettivo.** Nessun funnel, nessuna metrica di crescita, nessun "engagement". La metrica del progetto è la qualità dell'accumulazione su una persona, moltiplicata — un giorno — per il numero di persone che possiedono la *propria* istanza.
- **Companion parasociale.** Muffin non simulerà relazioni emotive e non sarà progettato per riempire vuoti affettivi. Lo specchio contraddice; il companion asseconda. Sono prodotti opposti.
- **Configurazione esplicita del comportamento.** Niente pannelli di settings sul carattere, niente form "come vuoi che ti parli?". Muffin inferisce, osserva, adatta — chiedere all'utente di configurare l'entità la trasforma in un tool.
- **Azione autonoma irreversibile.** Per quanto l'executive cresca, un'azione che esce verso terzi e non si può disfare non partirà mai senza che l'ultima parola sia stata umana. Non è una limitazione temporanea da rimuovere quando i modelli saranno più bravi: è un principio di design.
