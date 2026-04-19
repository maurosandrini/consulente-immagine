# Consulente d'Immagine

Webapp privata di consulenza d'immagine personale. Partendo da una foto del viso e tre domande, propone 2-3 look (capelli, trucco, rossetto, orecchini) con immagine generata che preserva l'identità.

## Uso

Apri [index.html](./index.html). Setup una volta:
- Nome
- API key [Google Gemini](https://aistudio.google.com/apikey) (free tier sufficiente per uso personale)
- Opzionale: API key [Anthropic](https://console.anthropic.com/settings/keys) per Claude Haiku/Opus sulla verifica avversariale

Le chiavi restano solo nel tuo browser (`localStorage`). Non vengono inviate a terze parti.

## Tecnologia

- Single file HTML (no build, no backend)
- Gemini 2.5 Flash: analisi foto
- Claude Haiku 4.5 (fallback Gemini 2.5 Pro): generazione prompt + verifica avversariale + correzione
- Nano Banana (Gemini 2.5 Flash Image): generazione immagine finale

## Versioni

- `index.html` — v0.3 (corrente): EXPRESS mode, Haiku-centric, self-verifying
- `consulente-immagine-v0.2.html` — legacy con dialogo adattivo (instabile)
- `consulente-immagine-v0.1.html` — legacy prima release

## Privacy

Nessun dato esce dal browser se non per le chiamate API dirette a Google/Anthropic. La foto non viene salvata in remoto, le risposte vengono elaborate in locale.
