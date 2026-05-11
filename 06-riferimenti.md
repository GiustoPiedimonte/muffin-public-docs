# 06. Riferimenti

> I lavori di ricerca, i progetti open source e le ispirazioni culturali che hanno informato il design di Muffin. Sta in un lignaggio scientifico, non in un'invenzione isolata.

---

## Lignaggio scientifico

Muffin non sta costruendo un'architettura *nuova*. Sta combinando elementi che la ricerca sui modelli di linguaggio e sulla memoria a lungo termine ha già articolato individualmente, calibrandoli per un caso d'uso specifico (single-user, longitudinale, local-first). I riferimenti seguenti non sono citazioni di omaggio — sono i pezzi da cui Muffin ha preso scelte concrete.

### Memoria temporale e cicli di vita

**Graphiti** (Zep, 2024-2025) ha sistematizzato la *bi-temporalità* per knowledge graph in agenti AI: distinguere il tempo in cui qualcosa è diventato vero nel mondo dal tempo in cui il sistema l'ha registrato. La distinzione permette di sbagliare e correggersi senza perdere storia, e di rispondere a domande su stati di credenza passati. Muffin adotta la stessa distinzione (*valid time* / *recorded time*) come uno dei suoi invarianti architetturali.

**Memento** (2025) — *Memory-Augmented Agents Through Procedural Continuity* — ha proposto un'architettura in cui le entità del mondo (persone, luoghi, organizzazioni) sono nodi di prima classe nel grafo di memoria, e la entity resolution è un problema esplicito da gestire come parte del pipeline di learning. Muffin adotta entità di prima classe come invariante e tratta la entity resolution come operazione strutturale, non come dettaglio di estrazione.

### Memoria atomica e revisione

**A-MEM** (2025) — *Atomic Memory for LLMs* — ha studiato il trade-off tra memoria *atomica* (frammenti piccoli e indipendenti, recuperabili individualmente) e memoria *narrativa* (sintesi lunghe, semanticamente ricche ma difficili da revisionare). La conclusione è che servono entrambe: l'atomica è il substrato, la narrativa è la vista. Muffin riflette questa scelta nella separazione tra il substrato di episodi atomici e i layer riflessivi rigenerabili sopra.

**Mem0** (2024-2025) ha articolato il pattern di *retrieval ibrido* (vettoriale più keyword più ranking) e mostrato che la combinazione batte sistematicamente i singoli componenti. Muffin adotta retrieval ibrido come pratica standard, con un'aggiunta — il segnale di vicinanza nel grafo — che amplifica il contesto di entità correlate.

### Comprensione vs registrazione

**Nemori** (2025) — *Nemori: Episodic Memory and the Predictive Brain in LLM Agents* — ha proposto il *predict-calibrate* come meccanismo per distinguere registrazione (ricordare cosa è successo) da comprensione (essere in grado di predire cosa succederà coerentemente). Muffin implementa predict-calibrate come uno dei tre livelli del pilastro verifica, e lo considera il meccanismo operativo che traduce dataset in comprensione.

### Sleep-time compute e agenti dormienti

**Letta** (ex-MemGPT, 2024-2025) ha sviluppato l'idea di *sleep-time agents*: lavoro cognitivo eseguito quando l'utente non sta interagendo, in modo che il sistema possa eseguire pipeline costose senza che l'utente paghi la latenza. Muffin adotta lo sleep-time compute come paradigma centrale del dream cycle — gran parte della comprensione del sistema avviene di notte, e l'utente, di giorno, raccoglie il risultato.

### Gate meccanici e proattività

**Pare-Bench** (2025) ha empiricamente caratterizzato la tendenza di modelli di taglia media a proporre soluzioni prematuramente in più di tre quarti dei casi quando non ci sono vincoli meccanici. Il risultato ha informato direttamente la scelta di Muffin di mantenere gate meccanici severi sull'output proattivo, rimuovendo invece gate epistemici sul *cosa* il modello può considerare.

### Convergenza dei componenti agentici

**HyperAgents** (Meta AI, 2025) ha mostrato empiricamente che un agente LLM lasciato libero di self-improve — senza imporre un'architettura a priori — converge spontaneamente sugli stessi sei componenti: tool integration, memoria/stato, context engineering, planning, verifica, modularità. Non sono convenzioni ingegneristiche ma *vincoli funzionali*. Per Muffin questo risultato è centrale strategicamente: se l'architettura agentica è inevitabile, il vantaggio competitivo non sta nell'architettura (che diventa commodity come i modelli) ma nel dataset accumulato sotto quell'architettura. Muffin riconosce consapevolmente i sei pilastri come categorie esplicite di design, anziché nasconderle dietro nomi propri — la struttura è riconoscibile, ciò che la rende Muffin è l'accumulazione.

### Opinioni con confidence e correzione

**Hindsight** (Vectorize, 2025) — *Persistent Memory for AI Agents with Opinion Tracking* — propone un'architettura di memoria a quattro reti separate (world facts, experience facts, observations, opinions con confidence), con meccanismo esplicito di *change of mind*: una contraddizione forte riduce la confidence di un'opinione, e oltre soglia ne aggiorna il testo. Reporta 91.4% su LongMemEval con modello aperto di classe simile a quella usata da Muffin. Il pattern "opinions con confidence + meccanismo di revisione esplicito" è ciò che la confidence esplicita su ogni claim derivato (capitolo memoria + capitolo verifica) traduce in Muffin. Il counterpoint, a sua volta, è il pezzo che fa sì che ogni dismissione, correzione, predizione sbagliata diventi materiale da cui il sistema apprende cosa non sa fare bene — una versione del principio più generale, articolato originariamente in *Hindsight Experience Replay* (Andrychowicz et al., 2017), per cui imparare dai fallimenti richiede di rivisitare situazioni passate sapendo come sono finite, non solo accumulare i successi.

### Allineamento e silicon mirror

**Silicon Mirror** (2024) — un lavoro divulgativo che ha articolato il rischio di un sistema AI personale come *amplificatore di bias* invece che come *specchio di punti ciechi*. La distinzione informa direttamente la scelta di Muffin di tenere un counterpoint anti-sycophancy come narrativa di pari peso al living profile.

### Value-action gap

**ValueActionLens** (Google Research, 2025) ha caratterizzato il fenomeno per cui i modelli di linguaggio dichiarano comportamenti che poi non eseguono — il *value-action gap*. La consapevolezza di questo fenomeno ha portato Muffin a non fidarsi dei prompt come unico meccanismo di vincolo: i pillar verifica e i gate meccanici sono compensazioni esterne al modello.

### Pitfalls of reasoning

**Pitfalls of Reasoning** (Microsoft, 2025) — *IFEval++* — ha mostrato che modelli istruiti tendono a ignorare crescentemente le istruzioni del system prompt all'aumentare della profondità del prompt (*instruction attenuation*). Muffin adotta una conseguenza pratica: i segnali critici (termini sconosciuti, counterpoint, salience) non vivono nel system prompt ma vengono iniettati come *contesto ambient* accanto al messaggio dell'utente, dove l'attenzione del modello è più alta.

---

## Lignaggio culturale

Tre riferimenti non scientifici hanno influenzato profondamente il frame del progetto.

### The Machine — Person of Interest

Una serie televisiva americana del 2011-2016. Il pezzo rilevante è il personaggio della *Macchina*: un'AI di sorveglianza riprogrammata da un singolo uomo per essere orientata a *salvare* persone specifiche, non a *monitorare* la popolazione. La Macchina sa cosa la persona sta facendo, perché, che mood ha, cosa probabilmente farà, dove sta driftando — e comunica selettivamente quando è il momento.

Il frame *AI orientata a una persona singola, con intento di comprensione, non di controllo* è il riferimento più vicino a quello che Muffin prova a essere. La differenza più importante è la scala — la Macchina osserva chiunque per salvare alcuni; Muffin osserva uno per capirlo.

### JARVIS — Iron Man

Il riferimento per il *tono*. JARVIS non è servile, non è freddo, non è un assistente di tipo "yes sir". Ha umorismo asciutto, familiarità non servile, capacità di contraddire il proprio utente quando serve. Muffin punta a un registro simile — un'entità con punto di vista, non un esecutore.

### Slow Productivity — Cal Newport

Un framework di lavoro (non il libro come tale, che parla principalmente di knowledge work nel mondo accademico) basato sul principio che la maggior parte dei professionisti non ha bisogno di accelerare ma di capire dove sta sprecando attenzione. Tre componenti: *fare meno cose*, *lavorare a un ritmo naturale*, *ossessionarsi sulla qualità*.

Muffin si presenta come *strumento* di slow productivity, non come motore di accelerazione. La differenza: un'AI per produttività *fa più cose al posto tuo*. Un'AI per slow productivity *ti aiuta a vedere cosa stai facendo e perché*. Il primo modello è affollato di prodotti commerciali; il secondo è strutturalmente più scarso — è lo spazio in cui Muffin si posiziona.

---

## Lavori adiacenti che vale la pena conoscere

Non sono fonti dirette, ma stanno nello stesso quartiere intellettuale e potrebbero essere di interesse per chi vuole approfondire.

- **MemGPT** (precursore di Letta) — primo lavoro che ha trattato seriamente la memoria di un agente LLM come sistema con tier diversi (working memory, archivio, recall on-demand)
- **Charlie Mnemonic** — progetto open source di memoria persistente per LLM, con focus sulla revisione esplicita
- **Supermemory** — servizio commerciale di memoria longitudinale, con framing più orientato a productivity ma architetturalmente affine
- **Letta** — successore di MemGPT, ora una piattaforma per agenti con memoria di lungo respiro
- **Inflection AI / Pi** — un companion AI con memoria, framing diverso (più verso connessione emotiva)
- **Kin / Replika / Airi** — companion AI con memoria, framing parasociale

Muffin si distingue da tutti questi lungo due assi: il *frame* (specchio per punti ciechi vs assistant produttivo o companion emotivo) e il *modello di sostentamento* (single-user, local-first, dataset sotto controllo individuale vs SaaS multi-tenant).

---

## Una nota sulla forma di apprendimento del progetto

Muffin non ha un piano teorico fisso da cui discende l'implementazione. Procede per *audit periodici*: ogni paio di mesi si confronta cosa il sistema *sta dichiarando di fare* (CLAUDE.md, foundations, design) con cosa il sistema *sta facendo davvero* (query al database, log, tracce di pipeline reali). Lo scarto produce una lista di interventi.

Questo metodo è più importante di qualsiasi singola scelta di design. È quello che permette al progetto di essere onesto con sé stesso nel tempo — un'AI personale costruita per uso decennale ha bisogno di un meccanismo di auto-correzione, e l'audit periodico è l'unico che si è dimostrato funzionare nel medio periodo. La trasparenza pubblica del processo di audit (i suoi risultati vengono documentati e gli interventi correttivi sono documentati altrettanto) fa parte del posizionamento del progetto: *come si tiene onesto un sistema longitudinale* è una domanda interessante in sé, e Muffin la affronta in pubblico.
