# AGENTS.md — TechPath

Guia para qualquer pessoa (ou agente de IA) que for alterar este projeto.
Leia junto com `PRODUCT.md` (escopo) e `PLANO.md` (ordem das tarefas).

## 1. Contexto do projeto

- **Produto:** TechPath — plataforma web para mulheres iniciantes em
  tecnologia explorarem áreas, organizarem trilhas de estudo e acompanharem
  progresso.
- **Escopo atual:** apenas o MVP descrito em `PRODUCT.md`. Nada de rede
  social, chat livre, cursos próprios ou app nativo.
- **Idioma:** interface, mensagens, comentários e commits em português.
  Nomes de código (variáveis, funções, tabelas) em português, sem acentos.

## 2. Tecnologias permitidas

| Camada    | Permitido                                              |
|-----------|--------------------------------------------------------|
| Servidor  | PHP **7.3** (versão do ambiente atual)                 |
| Banco     | **MariaDB 10.3** via **PDO** com prepared statements   |
| Frontend  | HTML5, CSS3, JavaScript puro (sem framework)           |
| Estilo    | CSS com variáveis, mobile-first, um único `style.css`  |
| JS        | `fetch`, DOM API; ES5/ES6 compatível com navegadores atuais |
| IA        | Somente o provedor aprovado na Fase 6, via PHP         |

**Não permitido sem aprovação explícita:**

- Frameworks PHP (Laravel, Symfony, CodeIgniter etc.) ou micro-frameworks.
- Composer, npm, bundlers, SASS, Tailwind ou qualquer build step.
- ORMs e bibliotecas de terceiros (jQuery, Bootstrap, CDNs externos).
- MySQL “cru” fora do MariaDB instalado, ou outro banco.
- Serviços de IA não definidos na Fase 6.

> **Atenção à versão do PHP:** o ambiente tem PHP 7.3. Não use recursos de
> PHP 8: `match`, `str_contains()`, `str_starts_with()`, propriedades
> tipadas, `?->`, argumentos nomeados, `??=`. Use `password_hash()`,
> `password_verify()`, `random_bytes()` e `filter_var()`, que existem na 7.3.

## 3. Estrutura e organização

Siga a estrutura do `PLANO.md`. Regras práticas:

- Páginas acessadas pela usuária ficam em `areas/`, `trilhas/`,
  `atividades/`, `ia/` e na raiz.
- Código compartilhado fica em `includes/` e **nunca** é acessado direto.
- Toda página começa com `require` do `config.php`, do `db.php` e, quando
  exigir login, do `auth.php`.
- Toda página usa `header.php` e `footer.php` para manter o layout único.
- Nada de duplicar conexão, sessão ou HTML de topo/rodapé.
- Arquivos novos devem ter função clara; evite “arquivo utilitário” genérico
  sem necessidade.

## 4. Convenções de código

### PHP

- 4 espaços, sem tabs; arquivos em UTF-8 sem BOM, fim de linha LF.
- `declare(strict_types=1);` no topo de arquivos novos (compatível com 7.3).
- Tags completas `<?php`; nunca `<?` nem `<?=`.
- Variáveis e funções em `snake_case` (`calcular_progresso`).
- Constantes em `MAIUSCULAS_COM_UNDERSCORE`.
- Uma responsabilidade por arquivo/página.
- Processar POST **antes** de gerar HTML e redirecionar após sucesso
  (padrão POST → Redirect → GET).
- Não usar `extract()`, `eval()`, `@` para esconder erro, nem `global`.
- Erros: em desenvolvimento mostrar detalhes; em produção, mensagem
  genérica e registro no log (nunca senha, token ou dado pessoal).
- Consultas **somente** com prepared statements e parâmetros nomeados.
- Toda saída dinâmica passa por `e()` (`htmlspecialchars($v, ENT_QUOTES,
  'UTF-8')`).
- Toda ação que altera dados é POST, com token CSRF válido.
- Toda consulta de dados da usuária filtra por `usuaria_id` da sessão. Nunca
  confie em `id` vindo da URL sem verificar a dona.

### SQL / MariaDB

- Tabelas e colunas em `snake_case`, plural para tabelas (`users`,
  `trails`, `activities`), português quando fizer sentido no domínio.
- Engine `InnoDB`, charset `utf8mb4`, collation
  `utf8mb4_unicode_ci`.
- Toda tabela tem `id` (`INT UNSIGNED AUTO_INCREMENT PRIMARY KEY`) e
  `created_at`/`updated_at` quando aplicável.
- Chaves estrangeiras explícitas; `ON DELETE CASCADE` para atividades de uma
  trilha e para trilhas de uma usuária.
- Senhas nunca em texto puro: apenas hash do `password_hash()`.
- Alterações de estrutura entram em `database/schema.sql` **e** em um arquivo
  numerado em `database/migrations/` (ex.: `002_add_status.sql`), para que
  outra pessoa consiga reproduzir o banco.

### HTML / CSS / JS

- HTML semântico (`header`, `nav`, `main`, `section`, `form`, `label`).
- Todo campo de formulário tem `<label for>`; botões têm texto claro.
- CSS: variáveis em `:root`, mobile-first, classes em `kebab-case`. Evitar
  `!important` e estilos inline.
- JS: `let`/`const`, aspas simples, sem globais; código em `app.js` ou no
  fim da página, sempre com `defer` quando externo.
- JS é complemento: validação e regras de negócio **sempre** repetidas no
  PHP. Nada de confiar só no navegador.
- Sem CDNs externos (privacidade e funcionamento offline).

## 5. Segredos e dados sensíveis

- Credenciais ficam em `.env`, que **nunca** é versionado.
- `.env.example` é versionado e contém apenas nomes e valores fictícios.
- `.gitignore` deve cobrir: `.env`, `*.log`, dumps (`*.sql.gz`, `dump*`),
  `.DS_Store`, `.idea/`, `.vscode/`.
- Nunca colocar senha, chave de API, token ou string de conexão em: código
  PHP, HTML, JS, CSS, commits, issues, prints ou mensagens de chat.
- A chave de IA é lida do `.env` **somente no servidor**. O frontend nunca
  recebe a chave; ele chama um endpoint PHP.
- Não versionar banco com dados reais. `seed.sql` só com dados fictícios.
- Se um segredo vazar: revogar/rotacionar imediatamente, remover do arquivo,
  e avaliar limpeza do histórico do Git (não basta apagar no commit seguinte).
- Logs não devem conter e-mail, nome, senha, token nem conteúdo de prompts
  com dados pessoais.

## 6. Procedimento de testes

Não há suíte automatizada nesta fase. O teste é manual, guiado por checklist,
e deve ser repetido antes de cada commit que fecha uma tarefa.

### 6.1 Antes de commitar

```bash
# 1. Sintaxe de todos os arquivos alterados
php -l caminho/do/arquivo.php

# 2. Sintaxe de todos os PHP do projeto (rápido)
find . -name '*.php' -not -path './vendor/*' -print0 | xargs -0 -n1 php -l

# 3. Conferir que nenhum segredo entrou no stage
git diff --cached | grep -iE 'senha|password|api[_-]?key|secret|token' || true
git status
```

### 6.2 Checklist funcional (por tarefa)

1. Abrir a página no navegador e executar o fluxo feliz.
2. Executar os casos de erro previstos (campo vazio, id inexistente, acesso
   sem login, acesso a dado de outra usuária).
3. Recarregar a página (F5) e confirmar que nada foi duplicado.
4. Conferir o console do navegador (sem erros) e o log do PHP
   (`tail -f /var/log/apache2/error.log` ou equivalente).
5. Testar em largura de celular (360px) e desktop (1280px).

### 6.3 Testes de isolamento entre usuárias (obrigatório nas Fases 1–5)

Criar duas contas de teste, A e B, e confirmar:

- [ ] B não vê trilhas nem atividades de A no painel.
- [ ] B não abre `trilhas/ver.php?id=<de A>`.
- [ ] B não edita nem exclui dados de A (POST direto também é bloqueado).
- [ ] Após logout, URLs internas redirecionam para o login.

### 6.4 Testes de segurança mínimos

- [ ] Nome com `<script>alert(1)</script>` aparece como texto.
- [ ] `id` com `' OR 1=1 --` não altera resultado.
- [ ] POST sem token CSRF é recusado.
- [ ] Senha errada e e-mail inexistente dão a mesma mensagem.
- [ ] `includes/`, `database/` e `.env` não abrem pelo navegador.

### 6.5 Banco de dados

```bash
# Conferir estrutura
mariadb -u USUARIO -p techpath -e "SHOW TABLES;"

# Reproduzir o banco do zero em um banco de teste
mariadb -u USUARIO -p -e "CREATE DATABASE techpath_teste;"
mariadb -u USUARIO -p techpath_teste < database/schema.sql
mariadb -u USUARIO -p techpath_teste < database/seed.sql
```

### 6.6 Registro do teste

No commit ou no PR, escrever em uma linha o que foi testado, por exemplo:

```text
teste: cadastro valido, e-mail repetido e acesso ao painel deslogada
```

## 7. Git

- Commits pequenos, um por tarefa concluída, em português no imperativo
  (`adiciona cadastro de usuaria`, `corrige validacao de e-mail`).
- Não commitar `.env`, dumps, `vendor/`, `node_modules/` ou arquivos de IDE.
- Não reescrever histórico já publicado sem combinar com o time.
- Antes de começar uma tarefa: `git status` limpo. Ao terminar: commit e
  verificação registrada.

## 8. Definição de pronto

Uma alteração está pronta quando:

1. Cumpre o critério de aceitação correspondente no `PRODUCT.md`.
2. Passa `php -l`, sem warnings no log e sem erro no console.
3. Foi testada nos cenários de erro e de isolamento entre usuárias.
4. Não introduz dependência, tecnologia ou segredo fora das regras acima.
5. Tem commit pequeno e descritivo.

## 9. Quando parar e perguntar

Pare e peça confirmação antes de:

- adicionar qualquer dependência ou serviço externo;
- alterar o schema de forma destrutiva (dropar coluna/tabela com dados);
- mexer em configuração de servidor, Apache ou banco compartilhado;
- implementar algo fora do escopo do MVP;
- lidar com dados reais de pessoas.
