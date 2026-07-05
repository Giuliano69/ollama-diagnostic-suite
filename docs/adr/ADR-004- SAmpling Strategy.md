> ADR Template V 0.1 - Date : 1/07/2026 - first edition 

# ADR-00X: [Titolo Breve e Chiaro]
**Status:**                       | **Data:**            | **Autore:**    |
|---------------------------------|----------------------|----------------|
|Accepted                         |   2026-07-04         | PM             |

## 1. Il Contesto / Problema
L’obiettivo principale di ODS è individuare le cause di rallentamenti, blocchi e anomalie durante l’esecuzione di modelli LLM, raccogliendo informazioni provenienti sia dal processo di inferenza sia dal sistema operativo.

Occorre quindi definire una strategia di campionamento che privilegi semplicità, riproducibilità e basso impatto sulle prestazioni.

`recorder.py` deve raccogliere dati di sistema durante l'esecuzione del test, senza introdurre overhead o variabilità che comprometta la riproducibilità delle misure, ed evitando l'over-engineering segnalato come rischio trasversale del progetto.

L'acquisizione continua di metriche prestazionali rischia di indurre un pesante sovraccarico di CPU e continue operazioni di fork di sistema, alterando i risultati dei test (Effetto Heisenberg). 

L'esecuzione iterativa a intervalli brevi di comandi shell esterni tramite subprocess.run() (es. invocare costantemente journalctl o tool di GPU) può generare picchi di context switch nel kernel Linux.

## 2. Decisione Adottata
**Campionamento**: intervallo fisso, impostato da `config.yaml`(`SAMPLE_INTERVAL_SEC`) stabilito inizialmente in 2 secondi. Nessuna logica adattiva in produzione.

**Calibrazione (autometer)**: Inserimento di un controllo temporale elementare interno al loop del Recorder per misurare il tempo effettivo impiegato dall'iterazione di campionamento; se supera i 100ms, viene registrato un warning nel log.
Usiamo un flag booleano `AUTOMETER` in config.yaml. Se attivo, aggiunge una colonna `self_overhead_ms` al CSV, calcolata con `time.perf_counter()` prima/dopo ogni ciclo di raccolta. Serve solo a
scegliere empiricamente un valore sensato per `SAMPLE_INTERVAL_SEC` nei run di calibrazione iniziali — non introduce comportamento adattivo a runtime.
Se il peso risultasse alto, si prevede (non ancora implementato) di strumentare con `perf_counter()` ogni singolo collector per capire quale pesa di più, prima di considerare un'eventuale logica adattiva.

**Struttura del modulo**: un solo file `recorder.py`, funzioni separate (`get_cpu()`, `get_gpu()`, `get_mem()`, ...), nessun pacchetto `collectors/` in V0.1. Ogni funzione restituisce un dizionario con struttura uniforme, per rendere economico un eventuale refactor futuro verso moduli separati (pattern `collect() -> dict`).

**Processi tracciati**: processo Ollama, processo figlio, processo ODS stesso (solo se `AUTOMETER` attivo, con set ridotto di parametri).

**Parametri, in ordine di introduzione progressiva**:
    - 1. `psutil` per estrarre la percentuale CPU e l'utilizzo di RAM globale del sistema,
    - 2. Letture dirette da `/proc/meminfo`, `/proc/pressure/memory`,    `/proc/<pid>/smaps_rollup` (dati non coperti da psutil, quali RSS (Resident Set Size) e PSS del processo del server LLM.
    - 3. `dmesg --follow`, `journalctl -u ollama -f`, `journalctl -k -f` — tutti    lasciati **aperti in streaming** (subprocess.Popen con pipe, letti in un thread separato non bloccante), non rilanciati ad ogni ciclo
    - 4. `intel_gpu_top -s <millisecondi>`, con frequenza di lettura allineata    alla frequenza nativa di campionamento del tool (mai più frequente)

    -  `vulkaninfo` è stato rimosso dal set: in ambiente headless/LXC genera errori legati all'assenza di un display server (`XDG_RUNTIME_DIR`); il dato di device/memoria che fornirebbe è già ricavabile da altre fonti nel set sopra, quindi non  giustifica l'investimento di tempo per risolverlo ora.


## 3. Motivazione e Alternative Scartate
    - **Campionamento adattivo con auto-tuning a runtime**: scartata —   reintrodurrebbe la variabilità che il progetto vuole eliminare, e allontanerebbe il primo run funzionante.
  
    - **Pacchetto `collectors/` con un file per fonte dati fin da subito**:   scartata per V0.1 — costo di progettazione anticipato prima di avere   dati reali su quali collector servano davvero.    
   
    - **Rilancio di `intel_gpu_top`/`journalctl` ad ogni ciclo via   `subprocess.run()`**: scartata — overhead di fork/exec/kill ripetuto, in favore di processi tenuti aperti in streaming.
  
    - **`vulkaninfo`**: rimosso dal set per V0.1-V1.0 (vedi sopra); da   rivalutare solo se emerge un bisogno diagnostico specifico non coperto   da `intel_gpu_top`.

## 4. Conseguenze

    - `recorder.py` resta un solo file fino a quando la necessità di modularizzare sarà dimostrata da dati reali, non da previsione.
    
    - L'overhead degli strumenti esterni resta basso grazie alla modalità streaming invece dello spawn ripetuto.
    
    - Il valore di `SAMPLE_INTERVAL_SEC` per l'hardware specifico va   determinato empiricamente con un run di calibrazione (`AUTOMETER=true`)   prima di fissarlo in `config.yaml` per l'uso normale.

## 5. Stato di Revisione
Questo ADR dovrà essere rivalutato solamente se:

    - gli obiettivi del progetto cambieranno significativamente;
    - ODS evolverà da semplice suite diagnostica a piattaforma di monitoraggio permanente;
    - emergeranno casi d’uso incompatibili con lo scopo attuale.

## 6. Collegamenti

