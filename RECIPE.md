# RECIPE — Consulente d'Immagine

> **Source of truth.** Letta a runtime dalla webapp.

## Obiettivo

Aiutare una donna a ricevere **2-3 proposte di look complete** (capelli, trucco, rossetto, orecchini, **occhiali**, **palette abiti**) da portare alla parrucchiera e mostrare al marito/compagna/amica — partendo da foto + intervista rapida.

## Output per ogni look (max 3)

- Titolo evocativo (max 5 parole)
- Vibe (1 frase)
- **Capelli:** taglio + colore + styling
- **Trucco:** base + occhi + sopracciglia
- **Rossetto:** sfumatura specifica (coerente con stagione cromatica)
- **Orecchini:** tipo + materiale + dimensione (coerente con forma viso + stagione)
- **Occhiali:** tipo + forma + materiale (se l'utente vuole portarli)
- **Palette abiti:** 3 colori principali (swatches esadecimali) dalla stagione cromatica
- **Metallo gioielli:** oro giallo / oro rosa / argento / oro bianco / misto
- **"Perché ti sta":** 2 frasi (motivazione estetica + coerenza stagione)
- **Immagine generata:** viso preservato (face restore canvas) + nuovo look

## Regole ferree

### R1 — Identità del volto: INVIOLABILE

Elementi invariati nell'immagine generata:
- Forma viso, mascella, zigomi
- Naso (forma, dimensione, punta)
- Occhi (colore, forma, distanza, palpebre)
- Mento
- Guance
- Orecchie
- Fronte
- Collo

La webapp applica **face restoration post-processing**: MediaPipe rileva il volto nell'immagine originale e nel generato, Canvas ricompone il viso dell'originale sul generato con feathering ellittico. Se MediaPipe fallisce → fallback con warning.

### R2 — Intervista: 3 domande + nota libera (invariato da v0.3)

1. Occasione / momento di vita
2. Come vuoi sentirti (2-3 aggettivi, chips suggeriti)
3. Cosa NON vuoi (vincoli assoluti)
4. Nota libera opzionale (500 char max)
+ **Opzione occhiali**: "Porti / vuoi portare occhiali in questo look?" (sì/no)

### R3 — Cromoterapia stagionale (Carole Jackson 4-season)

Determinata da undertone + contrasto naturale + colore capelli naturali + colore occhi:

```
Spring / Primavera: warm + clear + bright
  palette tipica: corallo, pesca, giallo caldo, turchese, verde erba
  metallo: oro giallo, oro rosa
  rossetti: corallo, peach, warm pink

Summer / Estate: cool + soft + light
  palette tipica: rosa cipria, blu pastello, lavanda, grigio perla, bianco sporco
  metallo: argento, oro bianco
  rossetti: rosy pink, mauve soft, berry cool

Autumn / Autunno: warm + muted + deep
  palette tipica: terracotta, ocra, verde bosco, ruggine, bronzo, marrone
  metallo: oro vecchio, bronzo, oro giallo opaco
  rossetti: brick, terracotta, warm nude, brown-red

Winter / Inverno: cool + clear + contrast
  palette tipica: bianco ottico, nero, rosso puro, blu cobalto, smeraldo, fucsia
  metallo: argento, platino, oro bianco
  rossetti: true red, fuchsia, plum, cool berry
```

Chromotherapy respects **vincoli NO** utente. Se utente dice "niente nero" ma stagione Winter → proponiamo alternative Winter senza nero.

**Disclaimer obbligatorio nell'UI:** "La stagione cromatica è un orientamento dalla teoria Jackson 4-stagioni, non un dogma. Usa come bussola, non come gabbia."

### R4 — Abbinamento accessori per forma viso

```
Ovale:      tutte le forme (versatile)
Tondo:      angolari, rettangolari, geometrici (bilanciano morbidezza)
Squadrato:  arrotondati, ovali, morbidi (ammorbidiscono mascella)
Cuore:      bottom-heavy (occhiali con base più larga), orecchini pendenti ampi
Lungo:      larghi orizzontali, orecchini rotondi bold (spezzano verticalità)
Diamante:   cat-eye, orecchini morbidi, evita spigoli laterali
```

### R5 — Prompt di image generation

Ogni `english_prompt` deve contenere:
- **Apertura:** "Photorealistic portrait. Preserve subject identity exactly: face shape, skin tone, eye color, eye shape, nose, mouth unchanged."
- **Centro:** "Keep the person's face fully recognizable."
- **Chiusura:** "Do not alter facial structure, complexion, or eye features in any way."
- Descrizione: hair (cut+color+styling), makeup (base+eyes+brows), lipstick, earrings, glasses (se previsti), clothing color (coerente con palette), jewelry metal, lighting
- Max 180 parole

### R6 — Lingua

UI + output testuale: **italiano**. Solo `english_prompt` in inglese.

### R7 — Budget tempo

- Upload → analisi → interview: ~30 sec
- Prompt gen + adversarial + correct: < 30 sec
- Image gen Nano Banana: ~45 sec (3 in parallelo)
- Face restore in-browser: < 5 sec totali per 3 immagini
- **Totale end-to-end atteso:** < 2 min

## Checklist adversarial V1-V8 (v0.4)

- **V1** preserve_identity: clausola esplicita di preservazione volto (inizio + centro + chiusura)
- **V2** cites_dialogue: almeno 2 elementi concreti dal dialogo utente
- **V3** respects_no: rispetta tutti i "NO"
- **V4** coherent_adjectives: coerente con aggettivi aspirazionali
- **V5** covers_all: hair + makeup + lipstick + earrings + glasses (se previsti) + clothing_palette
- **V6** no_invented: non introduce elementi non discussi
- **V7** clean_english: inglese pulito, < 180 parole
- **V8 NEW** chromo_consistency: palette coerente con stagione cromatica, metallo coerente con stagione, rossetto dalla gamma della stagione

## Log errori

Ogni errore in qualunque stage salvato in `localStorage` come v0.3. Download sempre disponibile.

## Non fare

- ❌ Cambiare il volto (blocco architetturale con face restore)
- ❌ Proporre palette incoerenti con stagione cromatica senza motivo
- ❌ Stereotipare per età (non chiediamo età, non ne deduciamo)
- ❌ Commentare peso, forma fisica, eleganza della foto
- ❌ Procedure chirurgiche o invasive
- ❌ Ignorare i "vincoli NO" per inseguire la teoria cromatica
