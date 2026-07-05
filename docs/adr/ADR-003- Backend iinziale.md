# ADR-003: Backend iniziale
**Status:**                       | **Data:**            | **Autore:**    |
|---------------------------------|----------------------|----------------|
|Proposed/Accepted/Superseded     |   2026-07-04         | PM             |

## 1. Il Contesto / Problema
Il sistema deve poter validare se i blocchi riscontrati siano specifici della gestione di memoria interna di Ollama o se siano strutturali al codice di inferenza a basso livello di llama.cpp. 

È  necessario definire fin dall’inizio una strategia che favorisca l’estendibilità senza aumentare inutilmente la complessità della V0.1.


## 2. Decisione Adottata
Sviluppare client HTTP nativi differenziati per gli endpoint proprietari di Ollama (/api/chat) e di llama-server (/completion) duplicherebbe l'onere di implementazione del parsing dei payload e la manipolazione dei token.

Si decide di standardizzare il modulo runner.py implementando una classe interna denominata OpenAIClient puntata sull'endpoint unificato /v1/chat/completions, poiché funziona senza modifiche sia con Ollama sia con llama-server. 

Per V0.1, la commutazione tra l'istanza di Ollama e il server nativo standalone di llama-cpp (llama-server) verrà gestita esclusivamente modificando il parametro stringa backend_url all'interno del file di configurazione centrale config.yaml, senza alcuna modifica logica o strutturale al codice Python.
TTFT e TPS vengono calcolati lato client con `time.perf_counter()` attorno alla richiesta streaming, non letti da campi nativi (non disponibili su questo endpoint).

## 3. Motivazione e Alternative Scartate
    - **Due classi backend separate (`OllamaBackend`/`LlamaBackend`)**: scartata
per V0.1 — costo di progettazione anticipato su un'ipotesi di divergenza tra le due API non ancora verificata.

    - **Endpoint nativo `/api/chat` fin da subito**: scartata per V0.1 in favore   della semplicità  dell'endpoint standard OpenAI-compatible; da rivalutare se i limiti sopra elencati si rivelano bloccanti nei primi test.

## 4. Conseguenze
- Pro: Massimo riutilizzo del codice; astrazione agnostica del backend; implementazione immediata e a
rischio zero per la versione v0.1.

- Contro: Temporanea rinuncia all'estrazione di metadati diagnostici finissimi forniti solo dalle API native (es.conteggio dei layer VRAM allocati da /api/ps di Ollama). Tali letture sono differite alla v0.3 qualora i dati di telemetria sistemistica non risultassero esaustivi.

## 5. Stato di Revisione
Questo ADR dovrà essere rivalutato se:

    - emergeranno backend non compatibili con il modello REST adottato;
    - sarà necessario utilizzare funzionalità disponibili esclusivamente tramite librerie dedicate;
    - le API OpenAI-like perderanno rilevanza come standard di fatto.

## 6. Collegamenti

