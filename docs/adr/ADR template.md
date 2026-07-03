> ADR Template V 0.1 - Date : 1/07/2026 - first edition 
> 
> Un ADR deve  rispondere alla domanda :"perché abbiamo scelto X invece di Y, e cosa ci 
> costerebbe tornare indietro" — non descrivere cosa fa un modulo (quello è compito del codice 
> stesso e del README, non di un documento decisionale).
> Una decisione = un ADR. Se cambiare decisione "costa" 5 minuti, non serve un ADR.


# ADR-00X: [Titolo Breve e Chiaro]
**Stato:** Proposed/Accepted/Superseded | **Data:** 2026-06-04 | **Autore:** PM

## 1. Il Contesto / Problema
[Descrizione della necessità del negozio in 2 righe. Es: Come gestire le caparre per la prenotazione di penne in edizione limitata?]

## 2. Decisione Adottata
[La scelta finale del PM. Es: Utilizzo del modulo OCA 'sale_deposit' con flusso TO-BE X.]

## 3. Motivazione e Alternative Scartate
* Scartata opzione GPT (Modulo custom) perché aumenta il debito tecnico.
* Scartata opzione Gemini (Soli acconti e-commerce) perché non copre il POS fisico.
* Scelta questa opzione perché copre sia l'omnichannel che i requisiti fiscali italiani.

## 4. Conseguenze
* Positivo: Processo standard, nessun codice da mantenere.
* Negativo: Richiede una leggera modifica al modo in cui il commesso eme

## 5. Stato di Revisione
- Riesaminare quando...
- Condizioni che potrebbero invalidare la decisione.

## 6. Collegamenti
- Issue
- Commit
- ADR correlate
