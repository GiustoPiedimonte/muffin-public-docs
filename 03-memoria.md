# 03. Sistema di memoria

> Il modello a layer, l'entity graph come rappresentazione del mondo, la bi-temporalità che permette di sbagliare e correggersi senza perdere storia, la provenance che ancora ogni claim alla sua sorgente, e il dream cycle notturno che consolida e rigenera.

---

## Premessa: cosa significa "ricordare"

La maggior parte dei sistemi che chiamiamo "AI con memoria" sono in realtà sistemi che fanno *retrieval*: pesco dal passato i pezzi che assomigliano alla domanda di adesso, li metto nel contesto, e il modello fa il resto. Funziona discretamente per use-case di breve respiro, ma collassa su due dimensioni.

La prima è la *temporalità*: la realtà di una persona cambia. Sei mesi fa lavorava su un certo progetto; oggi non più. Un sistema di retrieval ingenuo continuerà a recuperare il fatto vecchio per anni, perché è memorizzato e non distingue *vero allora* da *vero adesso*.

La seconda è la *comprensione*. Recuperare cento episodi simili non equivale a capire un pattern. *"Ogni domenica sera scrive 'sono stanco'"* è un'osservazione. *"Le domeniche sera sono difficili per lui, probabilmente per anticipazione del lunedì"* è comprensione. La seconda non si recupera, si costruisce — e va distinta dalla prima per non confondere prove con interpretazioni.

Muffin organizza la memoria per affrontare entrambe le dimensioni.

---

## Il modello a layer

Sette strati, ognuno con governance e ruolo distinto. La gerarchia non è gerarchia di importanza — è gerarchia di *quanto un'informazione si avvicina a essere "comprensione" rispetto a "registrazione"*.

### Layer A — Sostrato conversazionale

I messaggi grezzi, gli episodi (eventi atomici di tipo generico — un messaggio dell'utente, un commit ricevuto, un evento di calendario, un dato da sensore quando arriverà). Questo strato è **immutabile**: una volta scritto, non si modifica e non si cancella mai. È il dataset accumulato letteralmente. Ogni altro layer è derivato da questo, e questa è la garanzia che derivati corrotti possono sempre essere rigenerati.

### Layer B — Modello del mondo (entity graph)

Persone, luoghi, progetti, strumenti, organizzazioni come **nodi di prima classe**. Non sono menzioni sepolte nel testo degli episodi — sono entità con identità persistente, attributi propri, relazioni pesate con altre entità.

Quando l'utente menziona una persona per la quinta volta, il sistema non riparte da zero — c'è un nodo che cresce in dettaglio: *quando è stata menzionata la prima volta, in che contesti, con che frequenza, in relazione a chi*. La granularità del grafo è la differenza tra un sistema che *ricorda di una persona* e un sistema che *modella una persona*.

Gli attributi e le relazioni delle entità non hanno un solo tempo: ognuno è registrato con quattro timestamp — quando la cosa è diventata vera nel mondo (e quando ha smesso di esserlo), e quando il sistema l'ha appresa (e quando l'ha eventualmente superata). La distinzione tra *event time* e *transaction time* è ciò che permette di rispondere a domande come *"da quando il sistema crede X?"* in modo separato da *"da quando X è vero?"*. Ogni entità porta inoltre uno *scope di consapevolezza* (privato del proprietario, condiviso, pubblico) che determina chi può leggerla — l'invariante di separazione tra contesto privato e gruppi è enforced a livello di schema, non lasciato all'applicazione.

L'entity resolution — il problema di capire se "quel collega", "Tizio" e "T." sono la stessa persona — segue una scala di costo crescente: match esatto sul nome canonico, fuzzy matching, similarità vettoriale, e solo come ultima risorsa una decisione del modello su un set ridotto di candidati. Il determinismo viene prima; il giudizio del modello entra solo quando le regole non bastano.

### Layer C — Strato riflessivo

Tre tipi di derivati emergono dal substrato:

- **Fatti** — assertions strutturate (*"vive in X"*, *"lavora a Y"*, *"il suo orario di sonno tipico è Z"*). Hanno un'aspettativa di verità nel tempo e una *confidence* esplicita.
- **Osservazioni** — interpretazioni preliminari del comportamento (*"sembra essere più produttivo la mattina presto"*, *"fa fatica a dire di no"*). Hanno una *depth* che cresce quando vengono confermate da nuova evidenza, e una *valence* che ne segna il tono.
- **Pattern** — regolarità statistiche (*"quando dorme meno di sei ore, il giorno dopo è meno paziente"*, *"i progetti che annuncia il martedì hanno tasso di completamento più alto"*). Sono esplicitamente probabilistici, non leggi.

Tutti questi sono **interpretazioni** — distinguibili da fatti osservati direttamente per via di un campo che ne traccia la natura. Quando un'osservazione viene mostrata all'utente, viene marcata come tale (*"forse..."*, *"sembra che..."*) per non confonderla con osservazione diretta.

### Layer D — Stato di consapevolezza

Lo stato corrente del sistema rispetto all'utente: dove pensa che sia, cosa sta facendo, da quando, fino a quando. Aggiornato in continuo dal *Predictor* dell'awareness loop. È il pezzo che permette di rispondere *"come stai?"* senza ripartire da zero ogni volta.

### Layer E — Identità lunga

Tre file di identità persistente: *chi è Muffin* (statico, modificabile dall'utente), *chi è l'utente* (rigenerato di notte dal dream cycle), *chi sta diventando Muffin* (storia di sé in evoluzione). Capitolo dedicato in [Identità lunga](04-identita.md).

### Layer F — Gate meccanici

Lo stato delle limitazioni operative: *do not disturb*, finestre di silenzio, budget di output proattivo residuo, cooldown attivi. Layer separato perché governa *quando il sistema può agire*, distinto da *cosa il sistema può considerare*.

### Layer G — Scaffolding operativo (operational layer)

Code di lavoro, scheduling interno, log di sistema, heartbeat delle pipeline. Il *come funziona* il sistema, distinto dai dati che il sistema custodisce. È il layer che fa girare gli altri sei, ma è anche il più liberamente sostituibile.

---

## Bi-temporalità — sbagliare e correggersi senza perdere storia

Un fatto in Muffin non ha un solo tempo, ne ha due:

- **Quando è diventato vero nel mondo** (*valid from*)
- **Quando è stato registrato dal sistema** (*recorded at*)

E un fatto può essere **superato** senza essere cancellato: quando un nuovo fatto contraddice un fatto precedente, il fatto precedente non sparisce — viene marcato come *non più valido da una certa data*. Da quel momento in poi non è più recuperato per query sul presente, ma resta interrogabile per query sul passato.

Questo permette domande sottili come *"cosa sapevi di me a gennaio?"* — non sopra il dataset attuale, ma sopra una proiezione del dataset come era a gennaio. Permette anche di accorgersi che un fatto creduto vero per mesi era sbagliato, e di **correggerlo senza distorcere la storia di come la comprensione del sistema è evoluta**.

Il principio dietro: la verità non è un valore corrente che si sovrascrive — è una serie temporale di stati di credenza con motivazioni. Cancellare i vecchi stati significa cancellare la traccia del proprio errore, e con essa la possibilità di imparare a essere meno sbagliato.

---

## Provenance — ogni claim ancorato alla sua sorgente

Ogni derivato (fatto, osservazione, pattern, frammento di profilo) è collegato agli **episodi** che l'hanno prodotto. Non in modo vago — con un legame esplicito, interrogabile.

Conseguenze concrete:

1. **Se un episodio si rivela problematico** (per esempio, un messaggio inserito per errore o da un contesto che non doveva influenzare la pipeline di learning), tutti i derivati che ne discendono possono essere invalidati a cascata.
2. **Se il modello di estrazione cambia** (una versione nuova, un prompt rivisto), i derivati possono essere rigenerati a partire dal substrato senza perdita di dati. Il substrato è sacro; i derivati sono ricalcolabili.
3. **Quando il sistema risponde a una domanda**, può citare le fonti. Non *"ho letto questo da qualche parte"*, ma *"questa osservazione discende da questi tre messaggi specifici, in quelle date, in quel contesto"*.

La provenance è il meccanismo che rende possibile *l'auditabilità della propria comprensione*. Senza, ogni claim del sistema è una scommessa.

---

## Confidence esplicita

Ogni claim derivato (fatto, osservazione, pattern, frammento di profilo, frammento di counterpoint) ha un valore di confidence esplicito tra zero e uno, accompagnato da una **fonte di confidence**: cosa giustifica il numero.

Tre fonti tipiche:

- **Conta di evidenza** — quanto spesso il claim è stato confermato da episodi
- **Conferma diretta dell'utente** — il claim è stato menzionato esplicitamente e accettato
- **Calibrazione predittiva** — il claim è stato usato per fare una predizione, e la predizione si è verificata

I claim con confidence bassa non sono cancellati — sono *attenuati*. Quando entrano nel contesto del modello, sono presentati con margini di incertezza esplicita. Il modello sa che sa imperfettamente, e parla di conseguenza.

Quando un claim non riceve evidenza per un periodo prolungato, la sua confidence decade gradualmente. Non viene cancellato — diventa stale. Se nuova evidenza arriva, la stale-ness si resetta. Se non arriva mai, il claim alla fine smette di essere recuperato per query sul presente, ma resta interrogabile come parte della storia.

---

## Retrieval ibrido

Quando il modello deve rispondere all'utente, il sistema costruisce un *contesto dinamico* — un insieme di pezzi del passato rilevanti per il momento corrente. Tre segnali si combinano:

1. **Similarità semantica** (vettoriale) — pesca pezzi che assomigliano semanticamente al messaggio corrente, anche se non condividono parole specifiche
2. **Match di parole chiave** (full-text) — pesca pezzi che condividono parole specifiche, anche se sono semanticamente distanti
3. **Vicinanza nel grafo** — i pezzi connessi nel grafo a entità menzionate nel messaggio ricevono un boost

I tre segnali sono fusi via una combinazione che non penalizza nessuno dei tre quando uno è forte. Sopra a tutto, un *recency decay* favorisce materiale più recente, calibrato in modo da non far sparire materiale vecchio importante ma da non lasciare che pezzi datati dominino il contesto presente.

Il principio: nessun singolo segnale è abbastanza. Solo la similarità semantica perde nomi propri rari. Solo il keyword match perde parafrasi. Solo il grafo perde scambi nuovi che non hanno ancora prodotto entità cristallizzate. La combinazione produce contesto più robusto di qualsiasi componente singolo.

---

## Dream cycle — il consolidamento notturno

Una volta al giorno (di solito di notte), il sistema esegue una pipeline di consolidamento. È il momento in cui Muffin *capisce*, distinto dai momenti in cui *registra*.

Le fasi includono (a livelli di astrazione diversi):

- **Ri-estrazione di fatti**, osservazioni e pattern dagli episodi delle ultime ore o giorni che non erano stati ancora processati
- **Validazione di pattern** esistenti — se un pattern aveva fatto una predizione e l'evidenza arrivata la conferma o la smentisce, la confidence si aggiorna
- **Decadimento di osservazioni stale** — osservazioni non rinforzate per molto tempo perdono peso
- **Pruning di narrative dormienti** — narrative che non sono state riattivate per periodi lunghi vengono archiviate (con soft-delete, mai cancellate fisicamente)
- **Rigenerazione del profilo dell'utente** dal substrato (vedi capitolo dedicato), per evitare drift accumulato
- **Rigenerazione del counterpoint** — l'altro lato del profilo, dove Muffin riconosce di leggere male l'utente
- **Aggiornamento del self-narrative di Muffin** — chi sta diventando il sistema, alimentato dalla traccia delle proprie azioni recenti
- **Generazione di highlights di delta** — un sommario di *cosa è cambiato stanotte* che il modello vede in chiaro nei turni successivi (cognitive transparency)

Il dream cycle è il pezzo che traduce *registrazione* in *comprensione*. Senza, il sistema continuerebbe ad accumulare episodi senza mai sintetizzarli; il modello opererebbe sopra dati che diventano sempre più voluminosi e sempre meno comprensibili.

---

## Sleep-time compute — un nuovo paradigma di costo

Una proprietà importante del dream cycle: gran parte del lavoro cognitivo del sistema avviene quando l'utente non sta interagendo. È *sleep-time compute* — il sistema lavora sui propri dati, sintetizza, ricalcola, di notte; l'utente, di giorno, raccoglie il risultato senza pagare la latenza del consolidamento.

Questo cambia il calcolo costo/beneficio in modo non banale. Un'operazione cognitiva che costerebbe troppo eseguire al volo (parecchi secondi di pensiero, decine di chiamate al modello) diventa accettabile se eseguita una volta di notte. Permette al sistema di *capire* l'utente in modi che sarebbero impraticabili in tempo reale, e di rendere disponibile la comprensione in tempo reale al momento successivo.

È la stessa logica per cui un cervello consolida i ricordi durante il sonno. Non è un'analogia neuroscientifica forzata — è il modo in cui il vincolo *"il sistema deve rispondere veloce all'utente"* libera spazio per *"il sistema può pensare lentamente quando l'utente non c'è"*.

---

## Gli invarianti come spina dorsale

Tutto quanto sopra dipende dal fatto che alcune proprietà non cambino mai, indipendentemente da cosa si aggiunga al sistema. Sono gli **invarianti architetturali** — un piccolo numero di commitments di livello schema che, se rispettati, fanno sì che il dataset effettivamente si accumuli senza perdersi, contaminarsi, o ammalarsi in silenzio.

Gli invarianti sono nove al momento attuale:

- **Episodi sono atomici e generici** — un messaggio, un commit, un dato da sensore sono lo stesso tipo di oggetto. Non c'è una pipeline custom per ogni tipo di input.
- **Le entità del mondo sono di prima classe** — non sono sepolte nel testo degli episodi.
- **La provenance è ovunque sui derivati** — non c'è un singolo claim derivato senza ancoraggio a un episodio sorgente.
- **Il raw è immutabile, i derivati sono regenerabili** — la rigenerazione non perde dati.
- **Il ciclo di vita è bi-temporale** — il valid-time e il record-time sono distinti, e si applicano a fatti, pattern, osservazioni e narrative, non solo ai fatti.
- **La confidence è esplicita** — non c'è un claim senza una stima di quanto è solido, accompagnata da una fonte di confidence (cosa giustifica il numero).
- **L'observability è substrato** — il sistema produce abbastanza tracce per accorgersi del proprio cattivo funzionamento.
- **L'evoluzione dello schema segue regole esplicite** — aggiungere una colonna con default è gratis e ammesso senza cerimonia, cambiare il tipo di una colonna richiede una migration con finestra di dual-read documentata e un Architecture Decision Record, distruggere una tabella è quasi sempre vietato (si archivia, non si droppa). La policy è codificata perché senza policy esplicita prima o poi qualcuno fa una mutazione distruttiva "tanto è veloce" e rompe il moat.
- **Il contesto è confine di scope** — privato e gruppo sono separati per costruzione, non per applicazione. Il meccanismo schema-level che lo rende enforceable è lo *scope di consapevolezza* sulle entità e sui loro attributi: un attributo marcato come privato resta privato anche se l'entità che lo ospita è condivisa.

Questi invarianti non sono ottimizzazioni — sono la differenza tra un dataset che vale qualcosa tra cinque anni e un dataset che si è ammalato in silenzio durante la sua crescita. Sono inoltre formulati come *commitments di permanenza*: cambiarne uno è raro per design — se ci si trovasse a cambiarne uno ogni sei mesi, sarebbe il segnale che non erano davvero invarianti.
