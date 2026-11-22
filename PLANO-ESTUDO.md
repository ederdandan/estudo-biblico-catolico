# Plano: Estudo por Perícopes (Bíblia Matos Soares)

> Documento de continuidade. Se a conversa com o Claude for limpa (`/clear`), comece por:
> **"Leia PLANO-ESTUDO.md e continue da próxima fase pendente."**

Última atualização: 2026-11-22 (revisão por episódio dos Lotes 7–9 concluída: 69 + 71 + 91; total 1804)

---

## 1. Objetivo

Acrescentar ao PWA (`docs/`) uma camada de **estudo por perícope**:

- divisão do texto em perícopes, com título;
- usos litúrgicos (Lecionário Romano);
- os **quatro sentidos** da Escritura (CIC 115–118);
- roteiro de **Lectio Divina**;
- âncoras por versículo: parágrafos do **CIC**, síntese dos **Padres** (Catena Aurea) e nota exegética.

Amostra visual aprovada como modelo: https://claude.ai/artifact/1VKTJ8dVdazJJ1ygJY1T5q
(Mt 5,1-12 · Mc 10,46-52 · Lc 15,11-32)

Uso pessoal, não comercial.

---

## 2. Decisões já tomadas (não rediscutir)

| Decisão | Motivo |
|---|---|
| **O texto bíblico nunca passa pela IA.** O JSON de estudo guarda só referências; o app junta com `docs/data/books/*.json`. | Fidelidade de 100% ao Matos Soares |
| **Fatos vêm de scripts; a IA só redige ou resume.** CIC = índice de citações; Padres = texto da Catena; liturgia = tabela do Lecionário. | Nunca inventar parágrafo do CIC nem frase de Padre |
| Perícope usa `ini: {c, v}` e `fim: {c, v}` (pode cruzar capítulos). Âncoras indexadas por `"c:v"`. | Ex.: Mc 8,39 = Mc 9,1 moderno |
| Cada bloco tem **origem**: `fonte` · `tabela` · `indice` · `ia+fonte` · `ia`. Notas geradas pela IA levam `origem: "ia"`. | Separar fato de reflexão |
| Cada perícope tem `revisado: false` até a revisão teológica humana. | Controle de qualidade |
| Um arquivo por perícope ou por livro, carregado sob demanda. | PWA leve |
| Vizinhança (perícope anterior/próxima) é calculada pelo app, não gravada. | Evita inconsistência |
| **Geração no Claude Pro** (dentro do Claude Code), um livro por vez. API Batch só se o usuário decidir depois (~US$ 100–120 para a Bíblia inteira com Opus 5.5). | O usuário tem Claude Pro, sem créditos de API |
| NotebookLM: não serve para gerar o conteúdo; pode ajudar a conferir fontes. | Sem API para gerar em massa, sem JSON garantido |
| Catena local usada **sem licença**, por decisão do usuário (uso pessoal). | Ver §6 |
| **CIC e usos litúrgicos ficam para o FINAL** (decisão de 2026-11-20). Até lá: `cic: []` e `liturgia.usos: []`. O validador recusa `cic` preenchido. | Priorizar os 4 sentidos, a Lectio e os Padres |
| Cada síntese patrística aponta o trecho exato da Catena (`"catena": "1:4–8#15"`). O validador confere se o autor confere com o trecho. | Atribuição verificável |

---

## 3. Fases

Legenda: ✅ feito · 🔄 em andamento · ⏳ pendente

### Fase 0: Análise e amostra ✅
- Viabilidade, schema, estimativa de tokens e protótipo visual (link acima).

### Fase 1: Fontes determinísticas (scripts, sem IA)

| # | Tarefa | Status | Saída |
|---|---|---|---|
| 1.1 | **Catena Aurea** → JSON na numeração do Matos Soares | ✅ | `scripts/build-catena.mjs` → `docs/data/catena/{MAT,MRK,LUK,JHN}.json` |
| 1.2 | **Índice de citações bíblicas do CIC** → `{ "MRK 10:47": [2616, ...] }` | ⏸ adiado para o fim | `scripts/build-cic.mjs` → `docs/data/cic-index.json` |
| 1.3 | **Lecionário Romano** (domingos A/B/C, solenidades, dias de semana) → tabela | ⏸ adiado para o fim | `docs/data/lecionario.json` |
| 1.4 | **Mapa de versificação** Vulgata ↔ moderna (para cruzar CIC e Lecionário) | 🔄 | Evangelhos prontos dentro de `build-catena.mjs`; falta extrair para um módulo comum e cobrir Salmos e o AT |
| 1.5 | **Validador**: schema, cobertura contínua, atribuição à Catena, ausência de texto bíblico (falta: CIC no índice) | ✅ | `node scripts/estudo.mjs check` |

### Fase 2: Piloto de Marcos (IA no Claude Pro) 🔄

| # | Tarefa | Status |
|---|---|---|
| 2.1 | Divisão de Marcos: **75 perícopes em 8 seções** | ✅ `scripts/pericopes/MRK.txt` → `docs/data/estudo/MRK/index.json` |
| 2.2 | Gerar por perícope: quatro sentidos, Lectio, tema central, sínteses dos Padres e notas | ✅ **75/75** — Marcos completo, todas validadas (`check`). Sínteses dos Padres conferidas com o texto integral da Catena |
| 2.3 | Rodar o validador | ✅ a cada lote |
| 2.4 | Revisão teológica pelo usuário (`revisado: true`) | ⏳ |
| 2.5 | Medir o consumo do limite do Pro por perícope e decidir o ritmo | ⏳ |

Estimativa: 2–4 dias de calendário na geração; 5–8 h de revisão.

#### Fluxo para gerar cada perícope (seguir à risca)
1. `node scripts/estudo.mjs status MRK` → mostra a próxima perícope pendente.
2. `node scripts/estudo.mjs contexto MRK <c>.<v>` → imprime o texto Matos Soares e todos os trechos da Catena com o identificador `[faixa#n]`.
3. Escrever `docs/data/estudo/MRK/<c>.<v>.json` no schema da §4, usando **só** trechos impressos no passo 2 para os Padres (com `"catena": "faixa#n"`). Sentidos: ~100–160 palavras cada. Meditatio: 3 perguntas. Notas: `{texto, origem: "ia"}` ou `null`.
4. `node scripts/estudo.mjs check MRK` precisa terminar em "tudo certo".
5. Atualizar o contador da tarefa 2.2 neste arquivo.

Se mudar a divisão em `scripts/pericopes/MRK.txt`, rode `node scripts/estudo.mjs index MRK`.

### Fase 3: Integração no PWA ✅ (feita em 2026-11-20, testada no navegador)
- `docs/js/study.js` (dados), `studyView` e `periHeadHtml` em `docs/js/app.js`, estilos "Estudo por perícopes" no fim de `docs/css/app.css`.
- `docs/sw.js`: VERSION 1.3.0, cache `estudo-1` (stale-while-revalidate para `data/estudo/`).
- `docs/data/estudo/livros.json` lista os livros com estudo (gerado por `estudo.mjs index`).
- Perícope ainda sem arquivo mostra "em preparação".
- Pendente para depois: Evangelho do domingo na tela inicial (depende do Lecionário, fase 1.3).

Itens originais desta fase:
- Rota `#/estudo/{LIVRO}.{c}.{v}` com abas: Texto · 4 Sentidos · Lectio Divina · Padres & CIC.
- Título da perícope e botão "Estudar" dentro do leitor.
- Tela inicial: Evangelho do domingo → perícope com a Lectio.
- Service worker: guardar em cache os arquivos de estudo só quando forem abertos.
- Corrigir o bug do protótipo: âncoras em perícopes que cruzam capítulos.

### Fase 4: Expansão 🔄
Ordem: Evangelhos ✅ → resto do Novo Testamento em lotes (curtos e mais lidos primeiro) → Salmos → demais livros do AT.
Lotes do NT (contagens estimadas; o número real sai ao criar cada `scripts/pericopes/{ID}.txt`):
- **Lote 1** (curtas e muito lidas, ~60): Filipenses PHP (~8) · Efésios EPH (~10) · Colossenses COL (~8) · 1 Tessalonicenses 1TH (~7) · Tiago JAS (~8) · 1 Pedro 1PE (~9) · 1 João 1JN (~10) — ✅ 66 perícopes
- **Lote 2** (curtíssimas, ~15): Filémon PHM · 2 João 2JN · 3 João 3JN · Judas JUD · 2 Tessalonicenses 2TH · Tito TIT · 2 Pedro 2PE — ✅ 18 perícopes
- **Lote 3** (médias, ~20): Gálatas GAL · 1 Timóteo 1TI · 2 Timóteo 2TI — ✅ 27 perícopes
- **Lote 4** (grandes, ~150): Romanos ROM · 1 Coríntios 1CO · Hebreus HEB · 2 Coríntios 2CO · Apocalipse REV · Atos ACT — ✅ 142 perícopes
Regra: commit + push ao fechar cada livro do lote.
Progresso: Marcos ✅ 75 · Mateus ✅ 107 · Lucas ✅ 78 · João ✅ 48. **Os quatro Evangelhos estão completos: 308 perícopes.**
Lote 1: Filipenses ✅ 10 · Efésios ✅ 12 · Colossenses ✅ 8 · 1 Tessalonicenses ✅ 8 · Tiago ✅ 9 · 1 Pedro ✅ 9 · 1 João ✅ 10. **Lote 1 concluído: 66 perícopes.**
Lote 2: Filémon ✅ 2 · 2 João ✅ 1 · 3 João ✅ 1 · Judas ✅ 2 · 2 Tessalonicenses ✅ 4 · Tito ✅ 3 · 2 Pedro ✅ 5. **Lote 2 concluído: 18 perícopes.**
Lote 3: Gálatas ✅ 11 · 1 Timóteo ✅ 10 · 2 Timóteo ✅ 6. **Lote 3 concluído: 27 perícopes.**
Lote 4: Romanos ✅ 30 · 1 Coríntios ✅ 26 · Hebreus ✅ 17 · 2 Coríntios ✅ 16 · Apocalipse ✅ 22 · Atos ✅ 31. **Lote 4 concluído: 142 perícopes.**
**Novo Testamento completo: 27 livros, 561 perícopes** (Evangelhos 308 + Lote 1 66 + Lote 2 18 + Lote 3 27 + Lote 4 142). AT em andamento: Lotes 5 a 10 ✅ (521 perícopes).

**Antigo Testamento em lotes** (aprovado em 2026-11-21; contagens estimadas). Regra: ao fechar cada lote, atualizar este .md + commit + push.
**Granularidade (decidido em 2026-11-21):** uma perícope por episódio (em geral um capítulo ou menos), com o conteúdo desenvolvido por inteiro, sem agrupar capítulos para render mais. Só leis, listas, genealogias e descrições técnicas (ex.: santuário) podem ficar em blocos de cerca de um capítulo. Os Lotes 7 a 9 foram feitos em blocos grossos e foram revistos por episódio em 2026-11-22. Em Provérbios, as coleções de sentenças ficam uma perícope por capítulo.
- **Lote 5** Salmos PSA: um salmo por perícope; Sl 118 (Vulgata) dividido nas 22 estrofes — ✅ **171 perícopes** em 5 seções (Livros I–V do Saltério)
- **Lote 6** Narrativas curtas — ✅ Rute RUT 4 · Tobias TOB 13 · Judite JDT 10 · Ester EST 13 (acréscimos gregos da Vulgata, 10,4–16,24, em seção própria). **40 perícopes**
- **Lote 7** Profetas menores — ✅ revisto por episódio: Oseias HOS 15 · Joel JOE 4 · Amós AMO 9 · Abdias OBA 1 · Jonas JON 4 · Miqueias MIC 7 · Naum NAH 3 · Habacuc HAB 3 · Sofonias ZEP 3 · Ageu HAG 2 · Zacarias ZEC 14 · Malaquias MAL 4. **69 perícopes** (Joel e Malaquias seguem a Vulgata: 3 e 4 capítulos)
- **Lote 8** Sapienciais curtos — ✅ revisto por episódio: Sabedoria WIS 19 · Eclesiastes ECC 12 · Cântico SNG 9 · Provérbios PRO 31 (um capítulo por perícope). **71 perícopes**
- **Lote 9** Profetas maiores I — ✅ revisto por episódio: Isaías ISA 64 · Lamentações LAM 5 · Baruc BAR 5 (cap. 6 = Carta de Jeremias; os blocos são unidades literárias únicas: oração, poema da Sabedoria, consolação, carta) · Daniel DAN 17 (Dn 3,24-90, 13 e 14 da Vulgata incluídos; 10 e 11 separados; 14 dividido em Bel e o dragão). **91 perícopes**
- **Lote 10** Origens: Gênesis GEN · Êxodo EXO — ✅ refeito por episódio (a primeira versão, de 61 perícopes em blocos grossos, foi substituída): GEN 74 · EXO 56 = **130 perícopes**
- **Lote 11** Profetas maiores II: Jeremias JER · Ezequiel EZE — ✅ por episódio: JER 64 · EZE 51 = **115 perícopes**
- **Lote 12** Sapienciais longos: Jó JOB · Eclesiástico SIR — ✅ por episódio: JOB 37 · SIR 102 = **139 perícopes**
- **Lote 13** História I: Josué JOS · Juízes JDG · 1 Samuel 1SA · 2 Samuel 2SA — ✅ por episódio: JOS 27 · JDG 30 · 1SA 36 · 2SA 30 = **123 perícopes**
- **Lote 14** História II: 1 Reis 1KI · 2 Reis 2KI · 1 Macabeus 1MA · 2 Macabeus 2MA — ✅ por episódio: 1KI 29 · 2KI 30 · 1MA 29 · 2MA 22 = **110 perícopes**
- **Lote 15** Lei: Levítico LEV · Números NUM · Deuteronômio DEU — ✅ por episódio (leis e listas em blocos de cerca de um capítulo): LEV 29 · NUM 36 · DEU 35 = **100 perícopes**
- **Lote 16** Crônicas e retorno — ✅ por episódio (genealogias e listas em blocos): 1CH 28 · 2CH 33 · EZR 10 · NEH 13 = **84 perícopes**. **AT completo.**
No AT também não há Catena: `padres: []`. Salmos seguem a numeração da Vulgata (o título do salmo é o v. 1).

Fluxo fora dos Evangelhos: criar `scripts/pericopes/{ID}.txt` → `index {ID}` → escrever o conteúdo num módulo `{ID}.mjs` (array de `{k, tema, literal, alegorico, moral, anagogico, lectio, perguntas, oratio, foco:[texto, ref], notas:{"c:v": texto}}`) → `node scripts/gen-estudo.mjs {ID} <caminho>/{ID}.mjs` (o módulo fica no scratchpad; o gerador preenche id/título/seção/ref/ini/fim do índice, `padres: []`, `cic: []`, `origem: "ia"`) → `check {ID}`. Nas notas, não citar números do CIC (fica para a fase 1.2). `estudo.mjs` agora tira as abreviaturas de `docs/data/index.json`.
Fora dos Evangelhos não há Catena. **Decisão:** nesses livros, `padres` fica `[]`; as âncoras levam só `nota` (IA) quando houver algo útil.

---

## 4. Schema do arquivo de estudo (por perícope)

```json
{
  "id": "MRK.10.46",
  "livro": "MRK",
  "referencia": "Mc 10,46-52",
  "titulo": "A cura do cego Bartimeu",
  "secao": "A caminho de Jerusalém",
  "ini": { "c": 10, "v": 46 },
  "fim": { "c": 10, "v": 52 },
  "liturgia": {
    "tema_central": "string",
    "usos": [{ "celebracao": "30º Domingo do Tempo Comum – Ano B", "leitura": "Mc 10,46-52", "cor": "verde" }]
  },
  "quatro_sentidos": { "literal": "", "alegorico": "", "moral": "", "anagogico": "" },
  "lectio_divina": {
    "lectio_resumo": "",
    "meditatio_perguntas": ["", "", ""],
    "oratio_guia": "",
    "contemplatio_foco": { "texto": "", "ref": "Mc 10,47" }
  },
  "ancoras": {
    "10:46": {
      "cic": [],
      "padres": [{ "autor": "São Jerônimo", "obra": "string|null", "catena": "10:46–52#0", "sintese": "" }],
      "nota": { "texto": "", "origem": "ia" }
    }
  },
  "revisado": false
}
```

Regras:
- `cic` fica `[]` até a fase 1.2. Depois, só números presentes no índice para aquele versículo.
- `padres` só a partir de trechos reais de `docs/data/catena/`, com `catena` apontando o trecho; o `autor` tem de ser o mesmo do trecho.
- O texto dos versículos **não** entra no arquivo.
- O arquivo se chama `docs/data/estudo/<LIVRO>/<c>.<v>.json`; o `index.json` do livro lista a divisão.

Exemplo completo: `docs/data/estudo/MRK/1.1.json`.

---

## 5. Dados gerados até agora

### `docs/data/catena/{ID}.json` (fase 1.1)
```json
{
  "obra": "Catena Aurea — Santo Tomás de Aquino",
  "versificacao": "Matos Soares (Vulgata)",
  "blocos": [
    { "ini": {"c":10,"v":46}, "fim": {"c":10,"v":52}, "catena": "10:46–52",
      "trechos": [{ "autor": "São Jerônimo", "obra": null, "texto": "..." }] }
  ]
}
```
- 820 blocos e 12.736 trechos (Mt 281 · Mc 105 · Lc 246 · Jo 188).
- Fonte: `C:\Users\ederd\estudo\catena-aurea\data\*.json` (repositório de G. Evangelista).
- Regerar: `node scripts/build-catena.mjs` (o caminho da Catena pode ser passado como argumento).
- O texto evangélico desse repositório é de **outra tradução** (estilo Bíblia Pastoral) e é **descartado**.
- O campo `catena` guarda a faixa original, na numeração moderna.

### Versificação: Catena (moderna) → Matos Soares (Vulgata)
| Catena | Matos Soares |
|---|---|
| Mt 17,14-15 | Mt 17,14 · depois, v−1 |
| Mc 4,40-41 | Mc 4,40 |
| Mc 9,1 | **Mc 8,39** |
| Mc 9,2-50 | Mc 9,1-49 (v−1) |
| Jo 6,51 | Jo 6,51-52 · depois, v+1 |

Aviso esperado do script: `JHN 6:47–51 sobrepõe 6:51`. A sobreposição já existe na própria Catena e é inofensiva.

---

## 6. Pontos de atenção

- **O repositório `bibliacatolica` publica `docs/` no GitHub Pages (público).** Os arquivos `docs/data/catena/` ficam acessíveis a qualquer pessoa quando forem enviados (push). Se quiser manter só local, mova para fora de `docs/` ou ponha em `.gitignore`.
- A tradução portuguesa da Catena não tem licença declarada. O usuário decidiu usar mesmo assim (uso pessoal).
- Os parágrafos do CIC e as atribuições aos Padres da **amostra** vieram da memória da IA. Ali, "Jericó = lua" foi atribuído a Beda, mas na Catena é de São Jerônimo. Por isso o piloto precisa das fases 1.1 e 1.2 prontas.
- Notas do próprio Pe. Matos Soares: não temos a fonte digital. Talvez já estejam em domínio público (ele morreu em 1955; 70 anos após a morte), mas seria preciso fazer OCR de uma edição impressa.

---

## 7. Instruções do usuário para o trabalho autônomo (2026-11-20)

- Formato aprovado. **Gerar todas as perícopes de Marcos** sem parar para revisão.
- **Atualizar este .md a cada lote**, para retomar se a sessão travar.
- Se travar por limite de uso: ao liberar, recomeçar de onde parou (`node scripts/estudo.mjs status`).
- **Economizar tokens sem perder qualidade.** `contexto` corta cada trecho da Catena em 700 caracteres (`MAX=...` muda isso).
- Ao terminar Marcos: atualizar o .md, **fazer commit e push**, e só então começar o próximo Evangelho (Mateus): criar `scripts/pericopes/MAT.txt`, rodar `index MAT` e seguir o mesmo fluxo.

## 8. Próximo passo

1. **Mateus** (107/107, concluído). Fluxo usado: `PER=6 MAX=170 node scripts/estudo.mjs contexto MAT c.v` → JSON → `check MAT`. O check valida secao e exige que `contemplatio_foco` seja citação literal de Matos Soares.
   **Lucas** ✅ (concluído): `scripts/pericopes/LUK.txt` com 78 perícopes, índice gerado; a Catena de LUK já usa a numeração de Matos Soares. Fluxo: `PER=6 MAX=170 node scripts/estudo.mjs contexto LUK c.v` → JSON → `check LUK`. Feitas: 78/78.
   **João** ✅ (concluído): `scripts/pericopes/JHN.txt` com 48 perícopes em 6 seções; a Catena de JHN já usa a numeração de Matos Soares (Jo 6 tem 72 versículos: MS 6,52 = 6,51b moderno, e daí em diante MS = moderno + 1). Feitas: 48/48.
   **NT em lotes (ver Fase 4): Lotes 1, 2, 3 e 4 ✅: NT completo (561 perícopes).** **AT: Lote 5 (Salmos) ✅ 171 · Lote 6 (Rute, Tobias, Judite, Ester) ✅ 40 · Lote 7 (profetas menores) ✅ 69 · Lote 8 (sapienciais curtos) ✅ 71 · Lote 9 (Isaías, Lamentações, Baruc, Daniel) ✅ 91 · Lote 10 (Gênesis, Êxodo) ✅ 130 · Lote 11 (Jeremias, Ezequiel) ✅ 115 · Lote 12 (Jó, Eclesiástico) ✅ 139 · Lote 13 (Josué, Juízes, 1–2 Samuel) ✅ 123 · Lote 14 (1–2 Reis, 1–2 Macabeus) ✅ 110 · Lote 15 (Levítico, Números, Deuteronômio) ✅ 100 · Lote 16 (1–2 Crônicas, Esdras, Neemias) ✅ 84, por episódio. Bíblia inteira coberta por episódio (1804); revisão de granularidade dos Lotes 7 a 9 concluída.** Fora dos Evangelhos não há Catena: `padres` fica `[]` (decidido). Para cada livro: criar `scripts/pericopes/{ID}.txt`, rodar `index {ID}`, escrever os JSON e `check {ID}`.
2. Dica de fidelidade: os excertos de `contexto` com MAX vêm truncados; para sintetizar um excerto além do trecho visível, conferir o texto integral em `docs/data/catena/{ID}.json` (blocos[].trechos[i].texto).
   - **Atenção (Mateus)**: a Catena de MAT usa a numeração moderna, sem mapa. Divergências com Matos Soares: 5,4↔5,5 (mansos/choram invertidos) e o capítulo 17 a partir de ~17,15 (Vulgata une versículos). Ancorar cada Padre pelo conteúdo, não pelo número do bloco.
3. No fim (depois do conteúdo): fase 1.2 (CIC) e 1.3 (Lecionário, campo `usos`).

Fase 1.2, quando chegar a hora: encontrar uma fonte confiável do "Index of Citations" do Catecismo (vatican.va ou equivalente), extrair por script, aplicar o mapa de versificação e validar com casos conhecidos: Lc 15,11-32 → 1439; Mt 5,8 → 2518–2519; Mt 5,9 → 2305.
