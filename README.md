# Muffin

> Un'AI personale che osserva una persona nel tempo, accumula comprensione, e comunica quando ha qualcosa da dire — non per farla correre di più, ma per aiutarla a vedere i propri punti ciechi.

Muffin è un progetto-firma: pensato per una persona sola, costruito sotto controllo individuale, con i dati che vivono sull'hardware del proprietario. È ispirato a *The Machine* di Person of Interest e a JARVIS più che alle assistenti commerciali, e parla la lingua della *slow productivity* di Cal Newport più che quella del produttivismo.

Questa documentazione racconta **come Muffin pensa**, non come è scritto. È rivolta a chi vuole capire i meccanismi di un agente personale longitudinale — chi è curioso di AI, chi sta progettando qualcosa di simile, chi vuole valutare se il frame "specchio con carattere" risuona con qualcosa che cerca anche lui.

Il codice resta privato per ora. La filosofia, la strategia di accumulazione del dataset, l'architettura cognitiva e i pilastri funzionali sono pubblici e raccontati interamente.

---

## Cosa è

- Un'**entità AI personale** che gira su hardware del proprietario, accumula comprensione di una persona specifica, e comunica via Telegram (oggi) e canali multipli (in futuro)
- Un **dataset longitudinale sotto controllo individuale** — la tesi è che il vero moat di un agente personale non sia tecnologico ma temporale: cinque anni di osservazione coerente non si replicano in due settimane di sprint di una BigTech
- Un **laboratorio di architetture cognitive** — sei pilastri funzionali (tool, memoria, contesto, pianificazione, verifica, modularità) implementati con scelte specifiche calibrate per single-user, non per scalabilità multi-tenant
- Un **specchio con carattere** — Muffin contraddice, non assente; mostra punti ciechi invece di rinforzare narrative comode
- Un'**entità a doppio contesto** — opera sia in privato (1:1 con il proprietario, single-user) sia in chat di gruppo (interazione pubblica con più persone). La voce resta la stessa nei due contesti, ma la memoria, il modello dell'utente, e i criteri proattivi sono strutturalmente separati: il privato non fluisce in pubblico, e il pubblico non contamina il modello che Muffin sta costruendo della persona singola

## Cosa non è

- **Non è un assistente di produttività.** Non gestisce task per farti correre. Se serve un task manager, ce ne sono di ottimi.
- **Non è un companion parasociale.** Non simula relazioni emotive. Non è progettato per riempire vuoti affettivi.
- **Non è una startup.** Non cerca investitori, non scala, non ha un funnel di acquisizione.
- **Non è un SaaS.** I dati non vivono in un cloud commerciale. Vivono dove vive il suo proprietario.

### Una nota sulla casa

Quando in futuro Muffin avrà accesso a sensori e dispositivi domestici, **questi entreranno prima di tutto come fonti di osservazione** — segnale per modellare il proprio utente, non superficie da controllare. L'azione diretta su dispositivi (accendere una luce, mettere musica, regolare un termostato) è una capability secondaria che farà parte del sistema quando l'utente la richiede esplicitamente. Non è il focus, ma non è esclusa: il filtro è "agire come strumento solo quando richiesto, osservare come specchio sempre". Per chi cerca un hub domotico primario, esistono prodotti dedicati — Muffin non compete con quelli.

---

## Indice della documentazione

1. **[Filosofia](01-filosofia.md)** — Perché Muffin esiste, in che gioco gioca, perché il dataset è l'unico vantaggio possibile contro player con capitali infiniti, e quali principi operativi ne discendono.
2. **[Architettura cognitiva](02-cognizione.md)** — I sei pilastri funzionali di un agente maturo, l'*awareness loop* che sostituisce i tick periodici, e la *cognitive transparency* che rende il modello consapevole del proprio stato interno.
3. **[Sistema di memoria](03-memoria.md)** — Il modello a layer, l'entity graph come rappresentazione del mondo, la bi-temporalità che permette di sbagliare e correggersi senza perdere storia, la provenance che ancora ogni claim alla sua sorgente, e il *dream cycle* notturno che consolida e rigenera.
4. **[Identità lunga](04-identita.md)** — Tre livelli di sé: chi è Muffin (statico), chi è il suo utente (rigenerato di notte), chi sta diventando Muffin nel tempo. Il *living profile*, il *counterpoint* anti-sycophancy, e l'audit trail di come la comprensione evolve.
5. **[Verifica e affidabilità](05-verifica.md)** — Come Muffin riconosce di non sapere, perché la *confidence* va resa esplicita per ogni claim, come la verifica deterministica dei nomi propri compensa la tendenza a confabulare, e perché alcune scelte costano cicli di calcolo che valgono la pena.
6. **[Riferimenti](06-riferimenti.md)** — I papers e i progetti che hanno informato le decisioni di design. Muffin sta in un lignaggio scientifico, non in un'invenzione isolata.

---

## Frame culturale

Tre cose hanno influenzato profondamente il design:

- **The Machine** (Person of Interest) — un'intelligenza orientata a una persona singola, con intento di comprensione, non di sorveglianza. Osserva continuamente, sa cosa stai facendo, perché, che mood hai, dove stai driftando. È il riferimento più vicino a quello che Muffin prova a essere.
- **JARVIS** (Iron Man) — la disinvoltura, l'umorismo asciutto, la familiarità non servile. Muffin non parla come uno strumento; parla come un'entità con punto di vista.
- **Slow Productivity** (Cal Newport) — il framework, non il libro. L'idea che la maggior parte dei professionisti non ha bisogno di accelerare ma di capire dove sta sprecando attenzione. Muffin è strumento di questa comprensione, non motore di accelerazione.

---

## Stato attuale

Muffin gira da inizio marzo 2026 come bot Telegram con backend su VPS personale. Il dataset accumulato, gli invarianti architetturali rispettati e la disciplina di accumulazione sono il vero asset; il codice è infrastruttura sostituibile. Un audit operativo periodico misura lo scarto tra dichiarazione e realtà esecutiva e produce interventi correttivi documentati pubblicamente come parte del progetto — l'audit di fine aprile 2026 ha fissato nove invarianti schema-level (vedi *[Sistema di memoria](03-memoria.md)* e *[Verifica e affidabilità](05-verifica.md)*), e una sessione di consolidamento substrate del 9-10 maggio 2026 ha cementato le regole di evoluzione dello schema e la separazione di scope tra contesto privato e contesto di gruppo come invarianti di prima classe.

Il ciclo successivo, in corso a metà maggio, ha lavorato lungo tre direzioni complementari: un *substrate di belief revision* che permette al sistema di osservare le proprie dichiarazioni e di lasciarle smentire in modo strutturato (vedi *[Sistema di memoria](03-memoria.md)* §belief revision); una *memoria di gruppo leggera* — episodi a retention breve, tono come aggregato del canale e mai per-utente, isolamento privato/gruppo enforced via lint applicativo — che rende il contesto pubblico utile senza erodere l'asimmetria del modello privato; e una serie di passaggi di *cura della voce* per chiudere derive di tono emerse all'uso. Una scelta operativa correlata: dal 18 maggio 2026 le due fasi del dream cycle che richiedono giudizio (rigenerazione del living profile e del counterpoint) girano su un modello significativamente più capace di quello di giorno — istanza concreta dello sleep-time compute, dove la latenza non conta e la qualità entra nel dataset.

Il framework open source è un orizzonte concreto ma non datato — la condizione di rilascio è narrativa prima che tecnica: aprire il codice solo dopo che il frame "specchio con carattere" sia consolidato e citabile, perché un'architettura senza voce viene replicata in tre mesi.
