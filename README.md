# MHC-C — Core capture skills for Claude Code

**Filesystem-only. No server. No network. Plain markdown out. AGPL-3.0.**

MHC-C ships seven Claude Code skills that turn ad-hoc AI work into an auditable trail you can hand to a colleague six months from now and have them reconstruct *why the deliverable looks the way it does*.

```
/mhc-status     light orientation — where are we, what was the last session about
/mhc-note       quick capture — an insight, a decision, a reference, anything small
/mhc-trace      epistemic trace — crystallize an exploratory session into a structured artifact
/mhc-modlog     modification log — track the intellectual decisions behind a revision
/mhc-pdl        prompt development log — document what a prompt should generate, with alternatives
/mhc-output     primary output — save a deliverable (memo, brief, script, paper) with provenance
/mhc-onboard    project bootstrap — set up a project to use these skills (one-shot)
```

## Cosa fa, in una frase

Trasforma una sessione con un assistente AI in un archivio markdown navigabile, dove ogni artefatto cita le sue fonti a monte e il suo contesto di sessione. Nessun lock-in: plain markdown, layout filesystem standard, niente che esca dalla tua macchina.

## Per chi è

- Avvocati che vogliono tracciare il ragionamento dietro un parere
- Ricercatori che usano AI come parte del processo e devono poterlo rendere
- Sviluppatori che scrivono codice insieme a Claude e devono poter ricostruire le decisioni
- Chiunque produca lavoro intellettuale con AI e debba poter mostrare il sentiero che lo ha prodotto

Non è obbligatorio adottare la cornice intera — puoi usare anche solo `/mhc-note` se è tutto ciò che ti serve.

## Come si installa

In **Claude Code Desktop**, apri **Settings → Plugins** e aggiungi questo marketplace:

```text
MicheleLoi/MHC-C
```

Poi installa il plugin `mhc-c` dalla lista.

Documentazione canonica Anthropic in italiano: [Trova e installa plugin](https://code.claude.com/docs/it/discover-plugins) (passi visuali aggiornati con la UI corrente). Prima volta con Claude Code? Vedi [code.claude.com/docs/it/desktop](https://code.claude.com/docs/it/desktop).

Una volta installato, dentro qualsiasi progetto su cui vuoi adottare MHC-C:

```text
/mhc-onboard
```

`/mhc-onboard` ti chiede lingua, nome progetto, natura (design / governance / research / code-embedded / operational / hybrid), crea le cartelle base, scrive `.mhc-config.json` + `adapt.md` + `CLAUDE.md` + `methodology.md`, e l'ambiente è pronto. Il setup è one-shot; le sessioni successive sono silenziose.

> **Nota Claude Code CLI**: se usi il CLI da terminale (`claude` via npm) invece dell'app Desktop, l'installazione dei plugin segue un meccanismo diverso. Riferimento canonico Anthropic per la modalità CLI: documentazione plugin nella stessa sezione [code.claude.com/docs/it/discover-plugins](https://code.claude.com/docs/it/discover-plugins).

## Cosa genera

Su un nuovo progetto, dopo `/mhc-onboard`:

```
<your-project>/
├── .mhc-config.json          # project state — SID, session_history, folder mappings
├── adapt.md                  # project adaptation — schema, content_schemas, behavioral rules
├── CLAUDE.md                 # Claude's auto-loaded context — Decalogo + skills list
├── methodology.md            # artifact-model reference — how the pieces relate
├── session_topology.yaml     # session topology — goal + inputs + artifacts per SID
├── notes/                    # /mhc-note writes here (or note/ for Italian projects)
├── traces/                   # /mhc-trace writes here
├── modlogs/                  # /mhc-modlog writes here
├── pdl/                      # /mhc-pdl writes here
├── drafts/                   # /mhc-output writes here (or bozze/ for Italian)
└── conversations/            # exported session conversations
```

Tutto markdown. Niente database. Niente cloud. Niente che lasci la tua macchina.

## Il Decalogo

MHC-C ha un'identità: il **Decalogo** — tre principi epistemici (foundations) + sette regole operative.

**Three Principles:**
1. **Apollo** — *Assumpta patefiant*. Surface your assumptions before acting.
2. **Themis** — *Auctoritas ante actum*. Map the authority before recommending the action.
3. **Fides** — *Nihil sine teste*. Ground every claim in something citable.

**Seven Rules:**
1. **Gabriele** — Announce before consequential actions, wait for confirmation.
2. **Salomone** — Surface meaningful choices; distinguish alternatives before asking.
3. **Thot** — Offer to write up what was decided when substantial ground has been covered.
4. **Esdra** — Validate artifacts with the user before saving.
5. **Dioscuri** — Design processes that use both human and AI strengths.
6. **Ockham** — Default: reuse what exists; addition requires justification.
7. **Minerva** — Surface the trade-offs (especially the cons) before executing.

Il Decalogo non è una cornice decorativa — è la grammatica che ogni skill assume quando lavora con te. Apollo, Themis, Fides sono **fondazioni epistemiche**: violate quelle, nessuna regola operativa downstream ti salva. Le sette regole sono **disciplina operativa**: una volta che le fondazioni sono onorate, le regole governano come l'azione si svolge.

I nomi (Apollo, Themis, Fides, Gabriele, Salomone, Thot, Esdra, Dioscuri, Ockham, Minerva) sono **identità del prodotto** — sono il volto di MHC-C e servono anche a Claude come ancore di memoria. Non sono un'intrusione da rimuovere.

## Content schemas — universalità + configurabilità

Le skill di cattura (`mhc-note`, `mhc-trace`, `mhc-modlog`, `mhc-pdl`, `mhc-output`) hanno un pattern comune:

1. **Step 0**: leggono `content_schemas.<tipo>.fields` dal frontmatter di `adapt.md`.
2. Se il blocco è dichiarato, usano quei campi.
3. Se assente, usano il **fallback hardcoded** nello skill (preserva backward compatibility).

Esempio: il `trace` default ha campi `[insights, conceptual_map, formulations, open_questions, context_forward, warnings]` — è il *modo dialettico* di pensare. Un dominio clinico vuole `[presenting_problem, history, examination, assessment, plan]`. Modifichi il blocco `content_schemas:` in `adapt.md` di quel progetto, e il trace della clinica nasce con i campi del medico. Lo skill è universale, lo schema è del dominio.

```yaml
# adapt.md frontmatter
content_schemas:
  trace:
    fields: [presenting_problem, history, examination, assessment, plan]
  note:
    fields: [topic, content, references]
  # etc.
```

## AGPL — implications

MHC-C è rilasciato sotto **AGPL-3.0-only**. Significa:

- **Puoi usarlo gratis**, per qualsiasi scopo, inclusi quelli commerciali.
- **Puoi modificarlo** liberamente per i tuoi bisogni.
- **Se distribuisci una versione modificata** (o offri un servizio basato su di esso accessibile via rete), devi rilasciare anche la tua versione modificata sotto AGPL e renderla disponibile agli utenti.
- **L'uso interno in un'organizzazione** non è "distribuzione" e non innesca l'obbligo di rilascio.

L'AGPL è la licenza scelta perché MHC-C è un **asset asset-based consulting**: libero per la profession, condivisibile, riusabile. Vuole essere usato, esteso, ramificato. Non vuole essere chiuso in un fork proprietario.

Vedi `LICENSE` per il testo completo. Quando non sei sicuro: chiedi.

## Ecosistema

MHC-C è una delle componenti del bundle **MHC** (Methodology for Hybrid Coordination):

- **MicheleLoi/MHC-C** (questo repo) — Core capture skills, AGPL, marketplace pubblico, gratuito.
- **MicheleLoi/legal-tech-cowork** — Skill legal-specifiche (verifica fonti, catalogo, NDA, contract review, ecc.) complementari a MHC-C. Stesso framework, dominio legale italiano.
- **MicheleLoi/MHC-H** (in sviluppo) — Harness: server custode cieco + audit chain + cross-session memory + decision_log surfacing + open-tension tracking. Sblocca il valore epistemologico rigoroso che MHC-C standalone non promette. AGPL + SaaS pagata.

MHC-C funziona **standalone**: nessuna delle skill ha dipendenza obbligatoria su MHC-H. Quando MHC-H esce dal parking attuale e diventa attivo, le skill MHC-C aggiungeranno un hook opt-in `harness.register()` che registra ogni artefatto alla server custode cieco se configurata — ma il default standalone resta invariato.

## Boundary — cosa MHC-C NON fa

MHC-C è **light orientation + capture**. Non:

- Mantiene memoria cross-sessione oltre quello che `.mhc-config.json` registra localmente.
- Surfacizza decisioni passate da `decision_log.md` (anche se il progetto ne ha uno).
- Surfacizza tensioni aperte da `synthesis/` (anche se esistono).
- Verifica audit chain o integrità di catene hash.
- Chiama server, MCP tools, API esterne. Tutto filesystem locale, niente network.

Quei valori-prop richiedono **MHC-H** configurato (il harness con chiave di autorità). MHC-C è il punto d'ingresso gratuito; MHC-H è la profondità epistemologica sbloccata dalla chiave.

## Stato

**v0.1.0** — first release. 7 skill, AGPL, marketplace-installabile. Phase 1 di un piano roadmap che vede Phase 2 (integrazione opt-in con MHC-H quando disponibile) e Phase 3 (skill aggiuntive di dominio plug-and-play).

Bug + feature requests: aprire issue su GitHub.

---

*MHC-C — Core capture skills for Claude Code. Filesystem-only. AGPL-3.0. Maintained by Michele Loi.*
