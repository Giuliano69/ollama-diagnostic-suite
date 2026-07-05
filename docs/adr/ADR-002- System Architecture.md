> ADR Template V 0.1 - Date : 1/07/2026 - first edition 
> 
> Un ADR deve  rispondere alla domanda :"perché abbiamo scelto X invece di Y, e cosa ci 
> costerebbe tornare indietro" — non descrivere cosa fa un modulo (quello è compito del codice 
> stesso e del README, non di un documento decisionale).
> Una decisione = un ADR. Se cambiare decisione "costa" 5 minuti, non serve un ADR.


# ADR-002 Architettura del sistema
**Status:** Proposed/Accepted/Superseded | **Data:**            | **Autore:**    |
|----------------------------------------|----------------------|----------------|
|Accepted                                |   2026-07-04         | PM             |                      


## 1. Il Contesto / Problema
Il sistema deve inviare prompt a un backend LLM (Ollama, llama-server e futuri motori) e contemporaneamente campionare e registrare metriche hardware profonde (CPU, RAM, allocazioni kernel). 

Serve una pipeline riproducibile: inviare richieste di test a Ollama, misurare tempi di risposta, raccogliere dati di sistema durante il test, memorizzarli e produrre un report.
Il tutto restando in grado di terminare in modo controllato anche se uno dei tre componenti si blocca (lo scopo stesso del progetto è diagnosticare un sistema che si blocca, quindi ODS deve essere
resiliente ai propri stessi blocchi).

Due alternative sono state valutate per la sincronizzazione tra i moduli:
- un'architettura sequenziale processo padre/processi figli, oppure
- un'architettura a eventi (proposta da GPT: runner emette solo eventi
START/FIRST_TOKEN/TIMEOUT/STOP, analyzer li interpreta).

## 2. Decisione Adottata
Si opta per un'architettura a **disaccoppiamento rigido basata su processi nativi del sistema operativo**, implementata tramite il modulo Python multiprocessing.Process, strutturata su tre componenti sequenziali e lineari:

```
runner.py (processo padre)
  │
  ├─ lancia recorder.py come PROCESSO figlio (multiprocessing.Process,
  │  non thread)
  ├─ invia richieste HTTP a Ollama / llama-server
  ├─ cronometra le risposte
  ├─ rileva timeout
  ├─ segnala a recorder.py di fermarsi, poi:
  │     recorder_proc.terminate() / .join(timeout=N)
  │     se ancora vivo dopo il timeout → .kill() (SIGKILL)
  ├─ lancia analyzer.py, attende con lo stesso schema (join con timeout,
  │  poi terminate/kill se necessario)
  └─ chiude
```

Nel dettaglio;

- 1- Runner (Processo Principale): 
    - Agisce da orchestratore deterministico. 
    - Legge la configurazione, crea la directory dei log temporali, 
    - istanzia il Recorder come processo separato, 
    - esegue le chiamate sincrone HTTP/REST per i prompt di test calcolando i timing prestazionali interni (TTFT/TPS), 
    - notifica la terminazione a Recorder.

- 2- Recorder (Processo Separato): 
    - Eseguito in background dal Runner. 
    - Campiona in modo continuo e passivo le metriche hardware, scrivendole direttamente su file CSV piatti. 
    - Possiede un ciclo di vita  subordinato ad un oggetto multiprocessing.Event (stop_event). In caso di blocco o timeout del Runner, quest'ultimo invoca esplicitamente il metodo .terminate() seguito da .kill() sul processo Recorder, garantendo la pulizia del sistema operativo.

- 3- Analyzer (Esecuzione Post-Mortem): 
    - Lavora esclusivamente a freddo ("post-mortem") al termine del test. 
    - Analizza i file CSV e i log storici aggregati nella directory di sessione, applicando un motore a regole minimalista (sequenza di controlli if/else) per rilevare leak, frammentazione o deadlock.



## 3. Motivazione e Alternative Scartate

### Motivazione
Scelta adottata: Isolamento totale della memoria; invulnerabilità al GIL; garanzia di persistenza dei dati parziali scritti su CSV anche in caso di crash catastrofico dell'LLM.


###  Alternative Scartate
- Recorder come thread: scartata (vedi sopra — impossibilità di terminazione
  forzata).
- Architettura a eventi (event bus): scartata per V0.1-V1.0, premature
  abstraction rispetto allo scope attuale.


## 4. Conseguenze
-  Pro: 
    - Debug semplice: tre processi, flusso lineare, nessuna coda da ispezionare.
    - Capacità reale di forzare la chiusura di un recorder bloccato.
    - Refactor verso lo scenario "N chat concorrenti" resta contenuto in futuro,   perché runner e recorder sono già processi disaccoppiati.

-  Contro: 
    - Necessità di gestire i segnali del kernel e la terminazione dei sottoprocessi per prevenire processi zombie.

## 5. Stato di Revisione
Questo ADR verrà rivisto se:
    - saranno introdotti backend con modalità di funzionamento incompatibili;
    - emergerà la necessità di distribuire i componenti su processi o macchine differenti;
    - la struttura Runner / Recorder / Analyzer non sarà più sufficiente.

## 6. Collegamenti

