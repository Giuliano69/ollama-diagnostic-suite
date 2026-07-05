# ADR-005 — Formato dei dati salvati (CSV SQLite-friendly + log)
**Status:**                       | **Data:**            | **Autore:**    |
|---------------------------------|----------------------|----------------|
|Proposed/Accepted/Superseded     |   2026-07-04         | PM             |

## Contesto
L'utilizzo di motori database relazionali (es. PostgreSQL, SQLite) o a serie temporali (es. InfluxDB) per archiviare la telemetria della versione v0.1 aggiungerebbe layer di dipendenze software, configurazioni di rete e overhead computazionale non giustificati, allontanando il focus operativo dal debug immediato.

I dati raccolti da `recorder.py` vanno salvati in un formato ispezionabile a occhio oggi, con un percorso di migrazione economico verso analisi comparative tra più test in futuro (SQLite), senza introdurre una dipendenza da database in V0.1.

## Decisione

    - Una directory per ogni test, dentro `logs/`, nominata secondo lo schema   `[timestamp]_[nome_modello]`.    
    - CSV separati per categoria all'interno della directory: `mem.csv`,   `gpu.csv`, `ollama.csv`, `pressure.csv`.
    - prompt_performance.csv: Generato dal Runner (!), traccia i metadati prestazionali di ogni singola chiamata effettuata: prompt_id, prompt_length, response_length, ttft_ms, tps, status_code.
    - Libreria standard `csv` di Python per la scrittura, niente `pandas`.
    - Colonne fisse per ciascun CSV, timestamp in formato ISO-8601, per   rendere l'importazione futura in SQLite semplice senza doverlo progettare ora.
    - Log testuali (output di `journalctl`, `dmesg`, eventuale JSON di   `intel_gpu_top`) tenuti come file separati nella stessa directory del   test, non mescolati ai CSV.

## Alternative considerate

- **SQLite (singolo DB per test, o per l'intero progetto)**: scartata per
  V0.1 — aggiunge una dipendenza e un costo di progettazione dello schema
  prima che sia dimostrata una reale necessità di query trasversali tra
  test. Rimane un'opzione futura, documentata in `project_state.md`, da
  attivare se il progetto si dimostrasse utile e usato di frequente (query
  tipo "mostrami tutti i test dove RSS è cresciuto oltre 2GB").
- **Un'unica cartella `reports/` separata da `logs/`**: scartata per V0.1
  — utile solo quando esisterà un `html-report.py` (V1.0+); fino ad allora
  duplicherebbe la logica di path senza un consumatore reale.

## Conseguenze

- Nessuna dipendenza da database in V0.1-V0.3.
- Migrazione a SQLite, se necessaria in futuro, richiede solo uno script di
  importazione, grazie alla convenzione di colonne fisse e timestamp ISO
  adottata fin da ora.
- Zero dipendenze esterne; facilità di compressione, archiviazione e trasferimento dei log; immunità da corruzioni di database in caso di spegnimento improvviso del container.
