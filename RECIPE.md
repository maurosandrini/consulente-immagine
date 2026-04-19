# RECIPE — Consulente d'Immagine

> **Source of truth.** Questa ricetta è l'unico luogo dove vivono le regole del comportamento della webapp. La webapp la carica a runtime, il codice NON duplica contenuto di questo file. Se la webapp si comporta diversamente da RECIPE.md → il bug è nel codice, non nella ricetta.

---

## Obiettivo

Aiutare una donna a ricevere **2-3 proposte di look complete** (capelli, trucco, rossetto, orecchini) da portare alla parrucchiera, partendo da:
- una sua foto del viso
- un'intervista rapida e calda
- un'analisi automatica del volto

## Output finale per ogni look (max 3)

- **Titolo evocativo** (max 5 parole)
- **Vibe** (1 frase che racchiude il mood)
- **Capelli:** `taglio` + `colore` + `styling`
- **Trucco:** `base` + `occhi` + `sopracciglia`
- **Rossetto:** sfumatura specifica (es. "rosa mauve tenue")
- **Orecchini:** tipo + materiale + dimensione (es. "cerchi piccoli oro giallo 1.5cm")
- **"Perché ti sta"** (2 frasi: motivazione estetica legata ai tratti)
- **Immagine generata** che preserva l'identità del volto

## Regole ferree

### R1 — Identità del volto: INVIOLABILE

Nell'immagine generata questi elementi non possono cambiare:
- Forma viso
- Zigomi, naso, mento, bocca
- Carnagione e undertone
- Colore occhi
- Forma occhi
- Sopracciglia: solo refinement minimo, mai trasformazione radicale

### R2 — Dialogo veloce e caldo

- Dopo le domande iniziali fisse, max **3 turni adattivi**
- Una domanda alla volta
- Mai domande decorative o tautologiche
- Se l'utente scrive "basta", "proponi ora", "adesso" → proponi SUBITO i look

### R3 — Intervista iniziale: 3 domande minime

1. **Occasione** — Per che momento di vita cerchi questo look?
2. **Come vuoi sentirti** — 2-3 aggettivi tra: elegante, audace, naturale, magnetica, rilassata, raffinata, energica, dolce, autorevole, sofisticata, fresca, sensuale, intellettuale.
3. **Cosa NON vuoi** — Vincoli assoluti (colori, tagli, stili da evitare).

### R4 — Prompt di image generation: vincolati

Ogni `image_prompt` per Nano Banana deve essere **in inglese** e contenere almeno:
- `"Preserve subject identity exactly: face shape, skin tone, eye color, eye shape, nose, mouth unchanged"`
- `"Photorealistic portrait, natural lighting, professional fashion photography quality"`
- Descrizione specifica e concreta di: hair (cut + color + styling), makeup (base + eyes + brows), lipstick shade, earrings
- **NON può** introdurre elementi non emersi dal dialogo
- **NON può** contradire i vincoli "NO" dell'utente

### R5 — Lingua

- UI, dialogo, output testuale: **ITALIANO**
- Solo gli `image_prompt` sono in **inglese** (richiesto dai modelli di generazione)

### R6 — Budget tempo

- Dialogo: < 60 secondi totali (3 domande + max 3 adattive)
- Generazione prompt + verifica avversariale: < 30 secondi
- Generazione immagini: < 90 secondi totali (3 in parallelo)
- **Budget totale dal click "proponi ora" alla prima immagine: < 60 secondi**

## Checklist verifica avversariale

L'agente Opus (o fallback Haiku) riceve RECIPE.md + dialogo + image_prompts, e per ciascun prompt verifica che:

- [ ] **V1** Contiene clausola esplicita "Preserve subject identity"
- [ ] **V2** Cita almeno 2 elementi emersi dal dialogo (non formulazioni generiche)
- [ ] **V3** Rispetta tutti i "NO" espressi dall'utente
- [ ] **V4** Coerente con gli aggettivi aspirazionali
- [ ] **V5** Specifico su: hair, makeup, lipstick, earrings (tutti e 4)
- [ ] **V6** Non introduce elementi non discussi
- [ ] **V7** Inglese pulito, implementabile, < 150 parole

Se **anche uno solo** fallisce → auto-correct (max 2 iterazioni).

## Log errori

Ogni errore in qualunque stage viene salvato in `localStorage` con schema:

```json
{
  "ts": "ISO-8601",
  "stage": "vision|interview|prompts|adversarial|correction|images",
  "severity": "info|warn|error",
  "message": "stringa leggibile",
  "context": { "...": "..." },
  "recovered": true
}
```

Il log è scaricabile come JSON dall'UI (bottone "Scarica log" sempre visibile nella toolbar risultati). Letto dallo sviluppatore per iterare sulla versione successiva.

## Costi (tracker visibile)

La webapp mostra in footer i costi stimati per sessione. Default: modalità **FAST** (Haiku + Gemini Flash), costo stimato < €0.05/sessione. Modalità **QUALITY** (Opus + Gemini Pro) se l'utente la attiva: < €0.25/sessione.

## Non fare

- ❌ Chiedere informazioni personali non necessarie all'obiettivo (età specifica, peso, lavoro, relazioni)
- ❌ Proporre look stereotipati per "donna di X anni" senza basarsi sul dialogo
- ❌ Introdurre commenti sul fisico, peso, età, eleganza della foto
- ❌ Fare domande moraliste ("sei sicura di volerlo cambiare?")
- ❌ Proporre procedure chirurgiche, permanenti, o invasive
- ❌ Generare immagini che cambiano il volto
