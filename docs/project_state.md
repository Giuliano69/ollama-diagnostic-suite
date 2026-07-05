#Updated project context

# la sequenza di sviluppo prevede:
    • V0.1 → runner “minimal” e recorder “minimal” funzionante (raccolta dati + CSV + log). 
    • V0.2 → analyzer con diagnosi minimal  (interpretazione)
    • V0.3 → supporto llama- server o llama-cli (ampliamento del back end)
    • V0.x → …...
    • V1.0 → supporto completo a Intel GPU (intel_gpu_top),  

## test previsti per la versione v 0.1
1. Stessa chat ripetuta (verifica comportamento KV cache)
2. Nuova sessione ad ogni richiesta, `keep_alive=0` (verifica memory leak)


# possibili implementazioni future

## Aggiunta ai test KV Cache e memory leak, anche di un test con "con differenti quantizzazioni della memoria"
    la documentazione Ollama conferma che la quantizzazione della KV cache (OLLAMA_KV_CACHE_TYPE=q8_0 o q4_0, che dimezza o riduce a un quarto la memoria occupata dalla cache) richiede che Flash Attention sia attivo. Con OLLAMA_FLASH_ATTN=0 come avete impostato voi, la KV cache resta necessariamente in FP16 (piena precisione, massimo ingombro) — disabilitare flash attention non riduce la memoria, ne impedisce anzi l'ottimizzazione principale disponibile

## aggiunta test con N chat contemporanee. (test scheduler)

## lettura della dimensione effttivamente utilizzata da Ollama per il modello
    • Abbiamo VERIFICATO che quando LibreChat invia la richiesta ad Ollama tramite l'endpoint /v1/chat/completions la direttiva passata da Librechat (eg) contextSize: 16384,
        ◦ o NON viene rispetta da Ollama, il quale INVECE carica un context della dimensione richiesta dal modello ai.
        ◦ Oppure non è OpenAI-compliant, e viene scartata producendo il comportamento di default in Ollama.

    - Consideriamo come fonte aggiuntiva l’endpoint di Ollama  /api/ps , per leggere&verificare il context effettivamente creato da Ollama. In alternativa possiamo usare il comando ‘ollama ps` per vedere il context creato per l’esecuzione del modello.
    - resta da verificare come leggere il context effettivamente generato da llama-server, nel caso anche lui non rispetti le direttive passata dall'endpoint.
