# DESIGN.md — TechPath

Sistema visual do MVP: cores, tipografia, espaçamento, componentes e regras
de responsividade. Deve ser lido junto com `NAVEGACAO.md` (telas e estados)
e seguido na implementação do `assets/css/style.css`.

> **Objetivo do desenho:** transmitir acolhimento e clareza para quem está
> começando, sem parecer infantil. Interface calma, legível e previsível —
> a tecnologia é o assunto, não o obstáculo.

---

## 1. Princípios

1. **Clareza antes de beleza.** Uma ação principal por tela, texto direto.
2. **Acolhimento sem infantilizar.** Cores vivas em detalhes, muito espaço
   em branco e linguagem respeitosa.
3. **Previsível.** Mesmo botão, mesma cor, mesmo lugar em todas as telas.
4. **Mobile-first.** Desenhar para 360px e ampliar; nunca o contrário.
5. **Acessível por padrão.** Contraste, foco visível, rótulos e teclado não
   são etapa final.
6. **Leve.** Sem fontes externas, sem ícones decorativos pesados, sem
   animações que distraiam.

---

## 2. Identidade

- **Nome:** TechPath
- **Assinatura visual:** caminho/trilha. Usar a metáfora em detalhes
  discretos (linha de progresso, marcadores de etapa), não em ilustrações
  grandes.
- **Tom:** "você" direto, encorajador, sem jargão. Ex.: "Você ainda não tem
  trilhas" em vez de "Nenhum registro encontrado".
- **Logotipo:** apenas texto "TechPath" em peso 700. Sem imagem no MVP.

---

## 3. Cores

Paleta **pastel** construída em torno de `#db2777`, com contraste verificado
para WCAG 2.1 AA (texto normal ≥ 4,5:1).

### 3.1 Conceito

**Fundo pastel, acento rosa.** O rosa `#db2777` é a única cor saturada da
interface; todos os outros tons são pastéis suaves ou neutros com leve
undertone rosado.

- **70%** neutros (branco, off-white rosado, textos).
- **20%** pastéis (fundos de cartões, seções, badges).
- **10%** rosa de destaque (ações, links, progresso).

O pastel é sempre **fundo ou detalhe**, nunca cor de texto. Textos e ícones
informativos usam os neutros escuros ou as versões escuras das cores
semânticas.

### 3.2 Tokens

```css
:root {
  /* Acento — cor-assinatura */
  --cor-destaque:        #db2777;
  --cor-destaque-escura: #be185d;  /* hover/active */
  --cor-destaque-clara:  #fce7f3;  /* fundos e badges */
  --cor-destaque-suave:  #fdf2f8;  /* fundo de página/seção */
  --cor-foco:            rgba(219, 39, 119, 0.35);

  /* Primária = acento (mantém os componentes já especificados) */
  --cor-primaria:        var(--cor-destaque);
  --cor-primaria-escura: var(--cor-destaque-escura);
  --cor-primaria-clara:  var(--cor-destaque-clara);

  /* Pastéis de apoio (100 — fundos) */
  --cor-pessego-claro:   #ffedd5;
  --cor-lavanda-clara:   #ede9fe;
  --cor-menta-clara:     #d1fae5;
  --cor-ceu-clara:       #e0f2fe;
  --cor-manteiga-clara:  #fef3c7;

  /* Pastéis de apoio (200 — detalhes, bordas, marcadores) */
  --cor-rosa:            #fbcfe8;
  --cor-pessego:         #fed7aa;
  --cor-lavanda:         #ddd6fe;
  --cor-menta:           #a7f3d0;
  --cor-ceu:             #bae6fd;
  --cor-manteiga:        #fde68a;

  /* Semânticas — fundo pastel + texto escuro */
  --cor-sucesso:         #065f46;
  --cor-sucesso-clara:   #d1fae5;
  --cor-aviso:           #92400e;
  --cor-aviso-clara:     #fef3c7;
  --cor-erro:            #9f1239;
  --cor-erro-clara:      #ffe4e6;
  --cor-info:            #5b21b6;
  --cor-info-clara:      #ede9fe;

  /* Neutros com leve undertone rosado */
  --cor-texto:           #2e1a24;
  --cor-texto-suave:     #6b5563;
  --cor-borda:           #f2e3eb;
  --cor-borda-forte:     #e3c9d6;
  --cor-borda-campo:     #a97e93;  /* 3,45:1 — limite visível de campo */
  --cor-fundo:           #fdf8fb;
  --cor-superficie:      #ffffff;
  --cor-desabilitado:    #b9a3ae;
}
```

### 3.3 Combinações recomendadas

O rosa pastel (`#fce7f3`) é a base de quase todo bloco pastel. Os demais
tons entram como apoio, **um por vez**:

| Combinação | Onde usar |
|------------|-----------|
| Rosa + lavanda | padrão da interface (assinatura) |
| Rosa + pêssego | cartões de destaque, boas-vindas |
| Rosa + menta | confirmações, progresso concluído |
| Rosa + céu | dicas e textos informativos |
| Rosa + manteiga | avisos suaves |

**Regra:** no máximo **dois** pastéis diferentes no mesmo bloco. Nunca
rosa, lavanda e menta juntos na mesma tela.

### 3.4 Uso

| Cor | Onde usar | Onde **não** usar |
|-----|-----------|-------------------|
| Destaque `#db2777` | botão principal, links, foco, barra de progresso, item ativo | fundos grandes, textos longos |
| Destaque claro `#fce7f3` | badge de área, fundo de destaque, cabeçalho de seção | texto |
| Pastéis 100 | fundos de cartões, seções, estados vazios, ícones de apoio | texto, botões |
| Pastéis 200 | bordas de destaque, marcadores, detalhes gráficos | texto |
| Sucesso | status "concluída", confirmações | decoração geral |
| Aviso | sessão expirada, seleção vazia na IA | erros de validação |
| Erro | mensagens de erro, ação "Excluir" | textos explicativos |
| Neutros | 70% da interface | — |

**Contraste (medido):**

| Combinação | Razão |
|------------|-------|
| Texto `#2e1a24` sobre branco | 16,3:1 |
| Texto `#2e1a24` sobre fundo `#fdf8fb` | 15,5:1 |
| Texto suave `#6b5563` sobre branco | 6,8:1 |
| Texto `#2e1a24` sobre rosa `#fce7f3` | 13,9:1 |
| Texto `#2e1a24` sobre lavanda `#ede9fe` | 13,7:1 |
| Branco sobre destaque `#db2777` | 4,6:1 (AA) |
| Branco sobre destaque escuro `#be185d` | 6,0:1 |
| Sucesso `#065f46` sobre `#d1fae5` | 6,8:1 |
| Erro `#9f1239` sobre `#ffe4e6` | 6,7:1 |
| Aviso `#92400e` sobre `#fef3c7` | 6,4:1 |
| Info `#5b21b6` sobre `#ede9fe` | 7,6:1 |
| Borda de campo `#a97e93` sobre branco | 3,45:1 |

**Regras de cor:**

1. Texto branco **somente** sobre `#db2777` ou `#be185d`. Nunca sobre
   pastel.
2. Pastel nunca é cor de texto nem de botão; é fundo, borda ou forma.
3. Cor nunca é o único sinal. Status sempre tem texto ("Pendente",
   "Concluída") além da cor; erro sempre tem ícone/texto além da borda.
4. Em uma mesma tela, o rosa de destaque aparece em no máximo 3 elementos
   (ação principal, link ativo e progresso), para não competir consigo.
5. Badges de área usam os pastéis 100 em rotação, sempre com texto escuro.

---

## 4. Tipografia

```css
--fonte-base: system-ui, -apple-system, "Segoe UI", Roboto,
              "Helvetica Neue", Arial, sans-serif;
```

Sem Google Fonts (privacidade, desempenho e regra de não usar CDN).

| Papel | Tamanho (celular → desktop) | Peso | Altura de linha |
|-------|------------------------------|------|-----------------|
| Título de página (h1) | 1,75rem → 2,25rem | 700 | 1,2 |
| Título de seção (h2) | 1,375rem → 1,5rem | 600 | 1,3 |
| Título de cartão (h3) | 1,125rem | 600 | 1,4 |
| Corpo | 1rem | 400 | 1,6 |
| Apoio / legenda | 0,875rem | 400 | 1,5 |
| Rótulo de campo | 0,9375rem | 600 | 1,4 |

- Largura máxima de leitura: **68ch** (textos de área e descrições).
- Nunca texto em caixa alta para frases inteiras; apenas em badges curtos.
- Números de progresso em `font-variant-numeric: tabular-nums`.

---

## 5. Espaçamento, forma e sombra

### 5.1 Escala (base 4px)

```css
--esp-1: 0.25rem;  /*  4px */
--esp-2: 0.5rem;   /*  8px */
--esp-3: 0.75rem;  /* 12px */
--esp-4: 1rem;     /* 16px */
--esp-5: 1.5rem;   /* 24px */
--esp-6: 2rem;     /* 32px */
--esp-7: 3rem;     /* 48px */
--esp-8: 4rem;     /* 64px */
```

- Entre elementos relacionados: `--esp-2`.
- Entre blocos de um formulário: `--esp-4`.
- Entre seções: `--esp-6`.
- Respiro lateral no celular: `--esp-4`; no desktop: `--esp-5`.

### 5.2 Raios e sombras

```css
--raio-sm: 6px;    /* badges, inputs pequenos */
--raio-md: 10px;   /* inputs, botões */
--raio-lg: 16px;   /* cartões, painéis */
--raio-pill: 999px;

--sombra-1: 0 1px 2px rgba(17, 24, 39, 0.06);
--sombra-2: 0 4px 12px rgba(17, 24, 39, 0.08);
```

Usar sombra só em cartões elevados e no menu aberto. Bordas fazem o resto.

---

## 6. Layout e responsividade

### 6.1 Estrutura de página

```text
┌────────────────────────────────────────────┐
│ Header (sticky, 56px / 64px)               │
├────────────────────────────────────────────┤
│ Conteúdo (container 1080px, padding 16/24) │
│                                            │
│                                            │
├────────────────────────────────────────────┤
│ Rodapé simples (TechPath · ano)            │
└────────────────────────────────────────────┘
```

### 6.2 Breakpoints (mobile-first)

```css
/* Base: 0–639px — celular */
@media (min-width: 640px)  { /* tablet pequeno */ }
@media (min-width: 768px)  { /* tablet */ }
@media (min-width: 1024px) { /* computador */ }
```

### 6.3 Comportamento por tela

| Tela | Celular | Computador |
|------|---------|------------|
| Login/Cadastro | cartão único, 100% da largura menos 32px | cartão central de 400px |
| Painel | cartões empilhados | resumo em 3 colunas + lista |
| Áreas | 1 coluna | 2–3 colunas |
| Trilhas | 1 coluna | 2–3 colunas |
| Detalhe da trilha | atividades empilhadas | conteúdo 2/3 + ações 1/3 |
| Formulários | 1 coluna, botão full-width | 1 coluna, máximo 640px |
| Revisão de IA | cartões com checkbox | lista em coluna única de 720px |

### 6.4 Menu (sem JS)

No celular, usar `<details>` nativo:

```text
┌──────────────────────────┐
│ TechPath          Menu ▾ │
├──────────────────────────┤  ← aberto
│ Painel                   │
│ Áreas                    │
│ Minhas trilhas           │
│ Sair                     │
└──────────────────────────┘
```

No desktop, links em linha no header. Item ativo recebe
`aria-current="page"` e uma borda inferior de 2px na cor primária.

---

## 7. Componentes

### 7.1 Botões

| Variante | Aparência | Uso |
|----------|-----------|-----|
| Primário | fundo primário, texto branco | ação principal |
| Secundário | fundo branco, borda, texto primário | ações alternativas |
| Discreto (ghost) | sem fundo, texto suave | "Cancelar", "Voltar" |
| Perigo | fundo erro, texto branco | excluir (só após confirmação) |
| Link | texto primário sublinhado no hover | navegação inline |

- Altura mínima: **44px** (toque). No desktop, 40px é aceitável para ações
  secundárias.
- Padding horizontal: `--esp-4` (16px); primário no celular: largura total.
- Estados: `hover` (escurece 8%), `active` (escurece 12%), `focus-visible`
  (anel de 3px), `disabled` (opacidade 55%, cursor `not-allowed`),
  `carregando` (texto "Salvando…" + `aria-busy`).
- Nunca botão só com ícone sem `aria-label` e texto acessível.

```text
[   Criar trilha   ]   ← primário
[   Cancelar       ]   ← discreto
[   Excluir        ]   ← perigo
```

### 7.2 Campos de formulário

```text
Rótulo *                      ← 0,9375rem, peso 600
┌──────────────────────────┐
│ Placeholder / valor      │  ← 44px, borda 1px
└──────────────────────────┘
Texto de apoio (opcional)     ← 0,875rem, cinza
Mensagem de erro              ← 0,875rem, vermelho, com ícone
```

- Rótulo **acima** do campo, sempre visível (nunca só placeholder).
- Campo obrigatório marcado com `*` e explicação no topo: "* obrigatório".
- Erro: borda erro + mensagem com `id` ligado por `aria-describedby`.
- Foco: borda primária + anel suave de 3px.
- Textarea: mínimo 4 linhas, redimensionável na vertical.
- Select nativo estilizado apenas em cores/borda (sem recriar o componente).
- Checkbox/radio: alvo de 44px, caixa de 20px, rótulo clicável.

### 7.3 Cartões

- Fundo branco, borda `--cor-borda`, raio `--raio-lg`, padding `--esp-4/5`.
- Sombra `--sombra-1` apenas quando clicável (hover eleva para `--sombra-2`).
- Estrutura: título → metadados (área, contagem) → progresso → ações.
- Cartão inteiro clicável só quando houver **uma** ação; caso contrário,
  apenas o título é link.

### 7.4 Barra de progresso

```text
3 de 8 concluídas — 38%
▓▓▓▓▓▓░░░░░░░░░░░░
```

- Altura 8px, fundo `--cor-primaria-clara`, preenchimento primário.
- 100%: preenchimento em `--cor-sucesso`.
- Sempre acompanhada de texto com números (não depender da barra).
- Acessível: `role="progressbar"`, `aria-valuemin="0"`,
  `aria-valuemax="100"`, `aria-valuenow`, `aria-label="Progresso da trilha"`.
- Animação de largura de 200ms (respeitar `prefers-reduced-motion`).

### 7.5 Badges de status

| Status | Texto | Estilo |
|--------|-------|--------|
| Pendente | "Pendente" | fundo `--cor-aviso-clara`, texto `--cor-aviso` |
| Concluída | "Concluída" | fundo `--cor-sucesso-clara`, texto `--cor-sucesso` |
| Área | nome da área | fundo `--cor-primaria-clara`, texto primário |

Badge: `--raio-pill`, padding 4px 10px, 0,8125rem, peso 600. Sempre com
texto; nunca só um ponto colorido.

### 7.6 Mensagens (flash e alertas)

```text
┌────────────────────────────────────────┐
│ ✓  Trilha criada! Agora adicione…      │
└────────────────────────────────────────┘
```

- Barra lateral de 4px na cor semântica, fundo claro, texto escuro.
- Ícone simples (✓, !, i) com `aria-hidden` + texto.
- Sucesso: `role="status"`; erro: `role="alert"`.
- Fechável apenas quando não for erro de formulário.
- Erros de formulário também aparecem em resumo no topo, com links para os
  campos.

### 7.7 Estados vazios

```text
        ( )        ← forma simples em SVG, aria-hidden
   Nenhuma trilha ainda
   Crie sua primeira trilha ou
   comece por um modelo.
   [ Criar trilha ]  [ Ver modelos ]
```

- Título curto + uma frase + até duas ações.
- Centralizado, largura máxima 360px, padding `--esp-7`.
- Nunca usar apenas "Nenhum resultado".

### 7.8 Carregamento

- **Botão:** mantém a largura, troca texto para gerar/salvar + `disabled`.
- **IA:** 3 cartões esqueleto com animação suave de opacidade (1,2s),
  desativada em `prefers-reduced-motion`.
- **Concluir atividade:** o item mostra "Atualizando…" e fica inativo.
- **Sem overlay de página inteira.**

### 7.9 Confirmação de exclusão

Diálogo curto, foco preso nele, `Esc` cancela:

```text
Excluir a trilha "Fundamentos de Web"?
As atividades dela também serão excluídas.
Esta ação não pode ser desfeita.

[ Cancelar ]  [ Excluir ]
```

Implementação simples: `<dialog>` nativo com fallback de página de
confirmação. O POST só acontece após confirmar.

### 7.10 Navegação

- Header fixo no topo, fundo branco, borda inferior.
- Logo à esquerda; links à direita (desktop) ou `<details>` (celular).
- Link ativo: primário + `aria-current="page"`.
- "Sair" em estilo discreto, sempre por último.
- Skip link "Pular para o conteúdo" como primeiro elemento focável.

---

## 8. Movimento

- Duração: 150ms para hover/foco; 200ms para barras e abertura de menu.
- Curva: `ease-out`.
- Animar apenas `opacity`, `transform` e `width` (progresso).
- Respeitar `prefers-reduced-motion: reduce` desativando transições.

---

## 9. Acessibilidade (mínimos obrigatórios)

1. Contraste AA em texto e ícones informativos.
2. Foco visível com anel de 3px e offset de 2px.
3. Hierarquia de títulos correta (um `h1` por página).
4. Landmarks: `header`, `nav`, `main`, `footer`.
5. Todo campo com `<label for>`; erros ligados por `aria-describedby`.
6. Mensagens dinâmicas com `role="status"`/`role="alert"`.
7. Alvos de toque ≥ 44×44px.
8. Zoom de 200% sem perda de conteúdo.
9. Nada de texto dentro de imagem.
10. Tabelas (se surgirem) com `<th scope>`; no MVP preferir cartões.

---

## 10. Conteúdo e microcopy

| Situação | Escrever assim | Evitar |
|----------|----------------|--------|
| Lista vazia | "Você ainda não criou trilhas." | "Nenhum registro encontrado." |
| Erro de campo | "Informe um e-mail válido." | "Erro 422." |
| Erro de sistema | "Não foi possível salvar. Tente novamente." | "SQLSTATE[HY000]…" |
| Confirmação | "Trilha criada! Agora adicione atividades." | "Operação realizada com sucesso." |
| IA aguardando | "Gerando sugestões… Isso pode levar alguns segundos." | "Loading…" |
| IA vazia | "Não encontramos sugestões. Ajuste o objetivo e tente novamente." | "Nenhum resultado." |
| Sem permissão | "Você não tem acesso a esta página." | "403 Forbidden." |

Regras: frases curtas, voz ativa, "você", sem culpar a usuária, sem
maiúsculas gritadas, sem emoji na interface.

---

## 11. Organização do CSS

Um único arquivo `assets/css/style.css`, com seções comentadas nesta ordem:

```text
1. Tokens (:root)
2. Reset leve e base (box-sizing, body, tipografia)
3. Layout (container, header, footer, grid)
4. Componentes (botões, campos, cartões, alertas, badges, progresso)
5. Estados vazios e utilitários
6. Responsivo (media queries no fim, mobile-first)
```

Nomenclatura simples, tipo BEM leve e em `kebab-case`:

```text
.btn  .btn--primario  .btn--perigo  .btn--carregando
.campo  .campo__rotulo  .campo__erro
.cartao  .cartao__titulo  .cartao__meta
.alerta  .alerta--sucesso  .alerta--erro
.badge  .badge--pendente  .badge--concluida
.progresso  .progresso__barra  .progresso__texto
.estado-vazio
```

- Sem `!important`, sem estilo inline, sem ID para estilo.
- Classes utilitárias apenas para espaçamento e visibilidade
  (`.mt-4`, `.oculto`); não criar mini-framework.

---

## 12. O que evitar

- Gradientes fortes, sombras pesadas e excesso de rosa saturado na tela
  inteira.
- Três ou mais pastéis competindo no mesmo bloco.
- Texto branco ou claro sobre fundo pastel.
- Fontes externas, ícones de bibliotecas e CDNs.
- Carrossel, autoplay, parallax e animações decorativas.
- Tabelas em telas pequenas.
- Texto cinza claro sobre branco.
- Botão "Excluir" ao lado do botão principal sem separação visual.
- Modais para navegação ou para formulários longos.
- Placeholder como único rótulo.
- Mensagens técnicas de erro na interface.

---

## 13. Referência rápida de telas

Cada tela de `NAVEGACAO.md` usa apenas estes blocos:

| Tela | Blocos principais |
|------|-------------------|
| Login / Cadastro | cartão, campos, botão primário, alerta |
| Áreas | grade de cartões, badge, botão secundário |
| Detalhe da área | seções de texto, badge, CTA |
| Painel | cartões de resumo, lista com progresso, estado vazio |
| Trilhas | abas, grade de cartões, badge, progresso, confirmação |
| Detalhe da trilha | cabeçalho com progresso, lista de atividades, badge |
| Formulários | campos, resumo de erros, botões primário/discreto |
| IA | campos, esqueleto, lista com checkbox, alerta |
| Erro | estado vazio grande, botão de saída |

Qualquer componente novo deve ser adicionado a este documento antes de ir
para o CSS.
