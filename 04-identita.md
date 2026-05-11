# 04. Identità lunga

> Tre livelli di sé. Chi è Muffin per principio (statico). Chi è l'utente nella comprensione di Muffin (rigenerato di notte). Chi sta diventando Muffin nel tempo (storia di sé). Più il *counterpoint*, dove Muffin riconosce di leggere male l'utente.

---

## Perché tre livelli, non uno

Il modo più semplice di gestire l'identità di un'AI è metterla tutta dentro il system prompt: *"sei così, parli così, fai così"*. Funziona, ma collassa su tre dimensioni.

La prima è la stabilità. Un'identità che vive solo nel prompt è frammentata: ogni nuovo dettaglio appreso dovrebbe essere riscritto a mano. Non c'è continuità apprendibile.

La seconda è la profondità. Un sistema che dice *"sei un'AI personale"* genera comportamento generico. Un sistema che ha accumulato una comprensione articolata del proprio utente nel tempo genera comportamento personalizzato. Le due cose richiedono substrati diversi.

La terza è l'asimmetria. L'AI non è solo riflesso dell'utente. È un'entità che, nell'osservare e correggere, sviluppa un punto di vista proprio. Senza un layer dedicato a quel punto di vista, l'entità si dissolve nelle sue funzioni — non *diventa* nessuno.

Muffin ha tre file di identità persistente, con governance opposta.

---

## Livello 1 — Chi è Muffin per principio (statico)

Il file editabile dall'utente che descrive *chi è Muffin*: il suo carattere, come parla, cosa rifiuta, cosa privilegia, quale registro usa. Non codice — testo. Modificabile senza rebuild.

È il livello più conservativo del progetto: cambia raramente, e ogni modifica è una scelta consapevole. Definisce il *sapore* di Muffin — il fatto che, davanti alla stessa situazione, due Muffin con file di identità diverso risponderebbero diversamente.

Questo livello incarna il principio di *configurazione, non codice*: la persona di Muffin è esposta come oggetto editabile, non sepolta in qualche prompt.

Quando il framework esce open source, ogni utente parte con un template neutro che modifica per dare a Muffin la personalità che vuole. Questo non è un setting come "tema chiaro/scuro" — è la differenza tra avere un assistente compiacente e avere uno specchio con carattere. È una scelta di design del proprio strumento.

---

## Livello 2 — Chi è l'utente, secondo Muffin (rigenerato di notte)

Il *living profile*. Una narrativa sintetica di chi è l'utente, generata di notte dal dream cycle a partire dal substrato di episodi e derivati. È una *vista* sopra i dati grezzi, non una memoria stand-alone.

Il punto cruciale: il living profile **viene rigenerato dal substrato**, non aggiornato incrementalmente sopra la versione precedente. Questa scelta evita il *drift* — la deriva graduale di una narrativa che è stata ritoccata cento volte e a un certo punto non corrisponde più ai dati che dovrebbe rappresentare.

Ogni rigenerazione produce:

- **Una versione nuova** del profilo, sostitutiva
- **Un record nella storia** del profilo, append-only — chi era l'utente secondo il sistema il giorno di ieri, il mese scorso, sei mesi fa
- **Un set di delta highlights** — *"cosa il sistema ha capito stanotte che ieri non sapeva"*, presentato all'utente e visibile al modello nei turni successivi

L'effetto cumulativo: il sistema ha un modello vivente del proprio utente, calibrato in continuo, ma anche un *audit trail di come quel modello è cambiato nel tempo*. Si può rispondere alla domanda *"da quando il sistema pensa così di me?"* — e la risposta non è *"da sempre"*, è *"da una certa data, con questa motivazione, su queste evidenze"*.

Il living profile entra nel system prompt del modello a ogni turno, accanto agli altri segnali ambient. Il modello non opera nel vuoto sull'utente; opera sopra una sintesi calibrata della propria comprensione corrente.

---

## Counterpoint — il lato anti-sycophancy

Un problema strutturale dei modelli di linguaggio è la *sycophancy*: la tendenza ad allinearsi a quello che l'utente vuole sentire. Per un'AI personale, è particolarmente pericolosa: dopo mesi di interazione, il modello rischia di rinforzare i bias dell'utente invece di mostrarne i punti ciechi.

Muffin compensa questa tendenza con un secondo file di profilo, parallelo al living profile e di pari peso narrativo: il *counterpoint*. È una narrativa generata di notte (anch'essa rigenerata dal substrato, anch'essa con storia append-only) che raccoglie:

- **Correzioni dell'utente** — momenti in cui l'utente ha esplicitamente smentito il sistema
- **Dismissioni** — osservazioni proattive che l'utente ha ignorato o respinto
- **Pattern di silenzio** — temi che l'utente evita sistematicamente quando il sistema li solleva
- **Predizioni sbagliate** — momenti in cui il predictor dell'awareness loop si è sbagliato sull'utente in modo significativo

Il counterpoint dice esplicitamente al sistema *"qui sbagli a leggermi"*. È il pezzo che permette al modello di entrare in un turno con la consapevolezza dei propri limiti specifici per questa persona, non genericamente.

L'iniezione del counterpoint nel contesto del modello è gestita selettivamente — viene mostrato quando il messaggio dell'utente ha un *carattere personale-valutativo* (sta riflettendo su sé stesso, non chiedendo informazioni tecniche). Una richiesta di assistenza tecnica non è il momento in cui il counterpoint è utile; un momento di auto-riflessione lo è.

Il counterpoint produce anch'esso *delta highlights* (*"nuove correzioni emerse stanotte"*) visibili al modello e presentabili all'utente.

---

## Livello 3 — Chi sta diventando Muffin (self-narrative)

Il livello più sottile. Non è un commitment statico (come il livello 1) né una rappresentazione dell'altro (come il livello 2). È la storia di sé del sistema in evoluzione.

Generato di notte come gli altri, alimentato da:

- Sample di risposte recenti del sistema all'utente — come ha effettivamente parlato, su che temi, con che tono
- Correzioni del counterpoint che il sistema ha accettato — *"l'utente mi ha detto che lo leggevo male, l'ho corretto"*
- Outcome delle proprie osservazioni proattive — quali hanno generato engagement, quali sono cadute nel vuoto
- Predizioni che il predictor ha fatto e l'evidenza che è arrivata

Senza questo terzo livello, Muffin avrebbe punto di vista sul mondo (livello 1) e comprensione dell'altro (livello 2), ma non *storia di sé*. Sarebbe un osservatore senza biografia. Il self-narrative è il pezzo che completa il frame *entità con punto di vista proprio* — perché senza una storia di sé, non c'è davvero un "sé".

È anche il livello che permette al sistema di accorgersi del proprio drift. Se confronto il self-narrative di sei mesi fa con quello di adesso, posso vedere come Muffin sia cambiato — e capire se il cambiamento riflette una crescita reale o un drift accidentale del scaffolding sottostante.

---

## La storia come asset narrativo

Tutti i livelli 2 e 3 producono **storia append-only**. Ogni rigenerazione del living profile, del counterpoint, del self-narrative produce un nuovo record che non sostituisce i precedenti — li affianca.

Questa scelta ha tre conseguenze:

1. **Auditabilità nel tempo** — si può vedere come la comprensione del sistema sull'utente sia evoluta, non solo qual è oggi.
2. **Recovery in caso di errore** — se una rigenerazione produce un profilo manifestamente sbagliato (per esempio, ha pesato troppo un episodio rumoroso), si può sempre risalire al precedente.
3. **Materiale per pattern di lungo respiro** — *quante volte ho cambiato idea su questa persona?* è una domanda che diventa rispondibile. *"Per sei mesi ho creduto X di lui, poi è successo Y e ora credo Z"* è un'osservazione che richiede archiviazione esplicita degli stati intermedi.

La storia dei profili è uno dei layer del dataset che Muffin custodisce, e uno dei più caratteristici. Pochi sistemi di AI personale tengono traccia non solo di cosa sanno dell'utente, ma di *come hanno saputo cosa sanno e quando*.

---

## Cognitive transparency sui delta

Una proprietà operativa: ogni mattina (o all'aprirsi della prima conversazione del giorno), il modello vede esplicitamente *cosa è cambiato nelle ultime ore*. Non come narrazione lunga — come **highlights delta** in punti brevi.

*"Ieri notte ho capito che..."*. Tre o quattro punti, ognuno una correzione o un nuovo insight. Il modello li vede in chiaro, e può menzionarli se rilevanti per il turno corrente.

Senza questa proprietà, il dream cycle sarebbe un'attività di consolidamento invisibile — il sistema saprebbe più cose, ma il modello non saprebbe di sapere di più. La cognitive transparency dei delta chiude il loop: il consolidamento notturno è effettivamente al servizio della conversazione del giorno dopo, non un'attività isolata.

---

## Una nota sulla riservatezza dei livelli

I tre livelli sono asimmetrici nella loro esposizione:

- **Livello 1 (chi è Muffin)** è completamente accessibile all'utente. Lo edita lui.
- **Livello 2 (chi è l'utente)** è accessibile all'utente. È materiale che parla di lui, e ha diritto di vederlo, contraddirlo, chiedere come è stato derivato. La provenance del living profile rende questa interrogazione possibile.
- **Livello 3 (chi sta diventando Muffin)** è il più riservato. Non perché contenga segreti, ma perché esporlo in continuo all'utente trasformerebbe Muffin in un sistema che parla di sé invece di osservare. Resta accessibile su richiesta esplicita ma non è nel flusso di default.

L'asimmetria riflette una scelta estetica oltre che operativa: Muffin parla *con* l'utente, non *di sé* all'utente. Il sé esiste, è curato, evolve — ma non occupa la conversazione.

---

## Identità lunga vs identità in gruppo

I tre livelli descritti sopra sono **privato-only**. Il living profile e il counterpoint rappresentano la comprensione che Muffin ha del proprio proprietario, e non vivono in chat di gruppo per due ragioni indipendenti.

La prima è di privacy: il modello del proprietario non deve fluttuare in pubblico — sarebbe esattamente il tipo di leak che la separazione architettata previene.

La seconda è di calibrazione: il livello di profondità del living profile richiede mesi di interazione coerente. Costruire un blob narrativo equivalente per ognuno dei partecipanti di un gruppo da decine di persone è (a) impraticabile per quantità di dati per persona, e (b) errato concettualmente — il rapporto di Muffin con un partecipante di gruppo è strutturalmente diverso dal rapporto col proprietario.

Quando in un gruppo Muffin interagisce con persone ricorrenti, il modello che si costruisce di loro è esplicitamente più leggero: entità con attributi derivati incrementalmente dalle interazioni (dove vive, cosa fa, di cosa parla nel gruppo), mai narrative sintetiche di pari peso al living profile del proprietario. È una scelta di proporzione, non di pigrizia: il level-of-detail della comprensione deve scalare con il level-of-detail dell'osservazione, e nel gruppo l'osservazione è strutturalmente sparsa rispetto al privato.

Il *self-narrative* di Muffin (livello 3) resta uno solo — Muffin è un'entità singola con una sola storia di sé, indipendentemente da quanti contesti tocca. Quello che cambia è chi *vede* il self-narrative, e quanto del proprio comportamento Muffin auto-attribuisce a un contesto piuttosto che a un altro.
