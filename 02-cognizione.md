# 02. Architettura cognitiva

> I sei pilastri funzionali, l'*awareness loop* che sostituisce i tick periodici, e la *cognitive transparency* che rende il modello consapevole del proprio stato interno.

---

## I sei pilastri

Qualsiasi agente AI sufficientemente maturo finisce per avere sei pilastri funzionali. Non perché un'autorità centrale li abbia decretati, ma perché ognuno risolve un problema che emerge naturalmente quando il sistema cresce. Ricerca empirica recente (HyperAgents) ha mostrato che un agente lasciato a self-improve li reinventa da zero. Sono coordinate, non assi arbitrari.

Muffin organizza il proprio design intorno a questi sei.

### 1. Tool — la capacità di agire

Cosa risolve: trasformare un modello di linguaggio in un'entità che può fare cose nel mondo (leggere uno stato, scrivere un fatto, mandare un messaggio, interrogare un calendario, in futuro eseguire un comando su un dispositivo domestico quando l'utente lo chiede).

Come Muffin lo affronta: ogni capability è esposta come tool con descrizione orientata all'uso (*quando chiamarlo*, *cosa fa*, *cosa restituisce*). I tool sono organizzati in zone di permesso (*green* — esegui subito, *yellow* — conferma in batch, *red* — conferma esplicita) calibrate sul *blast radius* dell'azione. Le letture sono sempre verdi; le scritture su risorse esterne irreversibili sono sempre rosse.

### 2. Memoria — la continuità nel tempo

Cosa risolve: ricordare *cosa è stato* e *cosa significa*, in modo che la conversazione di oggi sia informata da quella di tre mesi fa.

Come Muffin lo affronta: un modello a layer (vedi [Sistema di memoria](03-memoria.md)) che separa episodi grezzi (substrato sacro), entity graph (modello del mondo), strato riflessivo (osservazioni, pattern, narrative) con confidence esplicita.

### 3. Contesto — cosa attentamente considerare

Cosa risolve: il modello ha una finestra di attenzione finita. Se gli si dà tutto, non focalizza nulla. Il pilastro contesto decide *cosa entra in quella finestra in questo momento*.

Come Muffin lo affronta: retrieval ibrido (vettoriale + keyword + grafo) per la memoria semantica, decay temporale per favorire materiale recente, segnali di rilevanza calcolati prima della chiamata al modello. Il principio: il modello non deve cercare il contesto, deve trovarlo già lì.

### 4. Pianificazione — la proattività con intelligenza

Cosa risolve: il modello reagisce bene a stimoli, ma per essere utile deve anche *iniziare* azioni — quando notare qualcosa, quando svegliarsi, quando proporre.

Come Muffin lo affronta: l'*awareness loop* (sotto), che non è un cron periodico ma un sistema che decide *quando* svegliarsi in base allo stato corrente dell'utente, e gate meccanici che limitano *quando* può parlare proattivamente.

### 5. Verifica — il controllo sull'output

Cosa risolve: i modelli generano fluentemente cose vere e cose false con la stessa confidenza apparente. Senza verifica, ogni claim è una scommessa.

Come Muffin lo affronta: capitolo dedicato in [Verifica e affidabilità](05-verifica.md). Tre layer di verifica con costo crescente, *confidence* esplicita su ogni claim derivato, name resolution deterministica per nomi propri.

### 6. Modularità — l'evoluzione attraverso ricombinazione

Cosa risolve: i sistemi che non sono modulari diventano impossibili da migliorare a un certo punto. Ogni cambiamento ne rompe altri.

Come Muffin lo affronta: distinzione netta substrato/harness, interfacce stabili tra strati, capacità di sostituire un componente alla volta senza toccare il resto.

---

## L'awareness loop

Il modo classico di rendere un agente "proattivo" è un cron periodico: ogni N minuti il sistema si sveglia, controlla qualche stato, eventualmente fa qualcosa. È semplice e funziona, ma sbaglia il problema.

Il problema non è *"controllare ogni N minuti"*. È *"essere sveglio quando serve, dormire quando non serve"*. Una persona la mattina, mentre fa colazione, non vuole una notifica. La stessa persona, dopo un commit doloroso a tarda sera, potrebbe trarre beneficio da una domanda. Il cron periodico tratta i due momenti uguale; l'*awareness loop* li tratta diversamente.

L'*awareness loop* di Muffin è composto da tre componenti.

### Predictor — la rappresentazione dello stato dell'utente

Mantiene una stima continua di *dove è* la persona osservata: cosa sta facendo, da quando, fino a quando probabilmente continuerà, con che livello di confidence. Lo stato non è un mood astratto — è una rappresentazione operativa del tipo *"sta lavorando su un progetto, probabilmente fino a metà pomeriggio, confidence media"*.

Il predictor si aggiorna su ogni segnale che arriva (un messaggio, un commit, un evento di calendario, una mancanza di attività dopo un orario abituale). Il silenzio è un segnale come gli altri: assenza prolungata di attività osservabile può significare riposo, o concentrazione profonda, o crisi — il predictor distingue in base al contesto.

### Scheduler — il risveglio intelligente

Decide il prossimo momento di risveglio in base a cosa il predictor sta dicendo. Non *"tra 30 minuti"*, ma *"alle 14:30 quando finirà la riunione che ha in calendario"*, *"tra due ore quando probabilmente avrà mandato il primo messaggio della giornata"*, *"a fine giornata quando inizia la finestra in cui di solito riflette"*.

C'è sempre un fallback periodico (un risveglio garantito comunque, anche se nessun trigger specifico è scattato), perché il sistema non può scoprirsi muto se il predictor sbaglia. Ma il default è il risveglio mirato, non quello cronologico.

### Decider — la decisione proattiva

Quando il sistema si sveglia, il decider decide cosa fare: parlare, restare in silenzio, aggiornare lo stato interno, eseguire un task accodato. La decisione passa attraverso *gate meccanici* (vedi sotto) che limitano *quando* il sistema può parlare a prescindere da cosa avrebbe da dire.

Il decider non è un giudice di rilevanza — è un commutatore. La rilevanza è già stata decisa prima dal pipeline di estrazione e dal sistema di salience; il decider applica solo i vincoli operativi.

---

## Gate meccanici vs gate epistemici

Una distinzione operativa importante. *Gate epistemici* sono filtri che decidono *cosa il modello può vedere o considerare* — soglie di rilevanza, threshold di similarità, depth minima per esporre un'osservazione. *Gate meccanici* sono filtri che decidono *quando il modello può agire all'esterno* — cooldown tra messaggi proattivi, fascia oraria *do not disturb*, budget giornaliero di output non richiesto, indisponibilità del predictor.

Muffin tende a:

- **Rimuovere gate epistemici** dove possibile — il modello è meglio del codice nel giudicare cosa è rilevante per una persona specifica, se gli si dà il contesto giusto
- **Tenere gate meccanici severi** — il modello è ottimista sul valore del proprio output, e senza gate meccanici tende a parlare troppo

Ricerca recente (Pare-Bench) ha mostrato empiricamente che i modelli di linguaggio tendono a proporre soluzioni prematuramente in più di tre casi su quattro quando non ci sono vincoli meccanici. È un punto cieco strutturale, non un bug correggibile via prompt — va compensato con codice fuori dal modello.

---

## Cognitive transparency

Un modello che opera *al buio* — senza sapere quali capability ha, in che stato del sistema è, cosa è cambiato di recente — è strutturalmente più fragile di un modello a cui questi segnali sono visibili. Quando il modello sa che è in modalità *do not disturb*, può comportarsi diversamente. Quando sa che il dream cycle ha aggiornato il profilo dell'utente questa notte, può menzionarlo se rilevante. Quando sa che certi termini nel messaggio non sono ancora nel suo vocabolario di entità conosciute, può fermarsi e verificare prima di inventare.

Muffin espone al modello una serie di *segnali ambient* — visibili in ogni turno della conversazione, accanto al messaggio dell'utente, senza che il modello debba fare tool call per ottenerli:

- **Importanza percepita del turno corrente** — calcolata da un sistema di salience deterministico, mostra al modello *"questo turno è normale / sopra la media / particolarmente intenso"*. Aiuta a calibrare profondità di risposta e attivazione di pipeline secondarie.
- **Sintesi di quello che è cambiato di recente nel modello dell'utente** — se il dream cycle ha rigenerato il profilo nelle ultime ore, il modello vede un riassunto delle correzioni e dei nuovi insight. Senza questo, il modello opererebbe come se ogni notte non fosse successo niente.
- **Termini che il modello non riconosce** — quando l'utente menziona un nome proprio nuovo come se fosse condiviso, un sistema deterministico (zero LLM) lo nota e lo segnala al modello come *termine sconosciuto*. Il modello sa che ha un punto cieco e ha un tool dedicato per chiuderlo.

Il principio: **il modello deve poter introspetterre il proprio funzionamento**, non solo l'input dell'utente. È la differenza tra un agente che reagisce e un agente che si comporta in modo informato.

---

## Il modello come tool-user, non come oracolo

Una scelta di design che permea tutto: Muffin non chiede al modello *"qual è la risposta giusta?"* — chiede *"quali strumenti useresti per arrivare alla risposta giusta?"*. La differenza è importante.

Un oracolo genera dalla sua memoria parametrica e si convince della risposta. Un tool-user dichiara cosa non sa, invoca le capability che riempiono il gap, sintetizza, restituisce. Il primo confabula con disinvoltura; il secondo si ferma e cerca quando non sa.

Muffin è progettato per essere un tool-user. Le tool description sono scritte in modo orientato all'uso (*quando invocare questo*, *cosa cercare prima*), non come reference passiva. Il system prompt incoraggia esplicitamente il modello a fermarsi e verificare di fronte a termini sconosciuti. Il pilastro verifica (di nuovo, capitolo dedicato) ancora questa disposizione con segnali esterni.

L'effetto cumulativo: Muffin sa quando non sa. Per un'AI personale che accumulerà comprensione su una persona per anni, questa è probabilmente la proprietà più importante.
