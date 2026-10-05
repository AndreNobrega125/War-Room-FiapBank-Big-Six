# FICHA DE DECISÕES · War Room FiapBank (CP6 · 3 aulas)

> Este arquivo é o **README.md do repositório do grupo** (`cp6-warroom-<nome-do-grupo>`).
> Vale **5,0 pontos** (rodadas 0,5 · relâmpagos 0,3), e a nota é pela
> **justificativa**, não pela letra. Preencham após cada aula e commitem até
> **23h59 do mesmo dia** (regras completas na seção 5 do enunciado).
>
> **Os incidentes da madrugada são revelados só em aula.** O título de cada registro
> será **ditado pelo professor na hora**; ninguém se antecipe.

**Grupo (nome da equipe plantonista):** Big Six

**Turma:** 2CCPG **Repo:** `cp6-warroom-BIGSIX`

| Integrante | RM | Turma |
|---|---|---|
| Guilherme Vasques Tamai | RM563276 | 2CCPG |
| Mirella Mascarenhas | RM562092 | 2CCPG |
| Caio Castelão Carminato | RM563630 | 2CCPG |
| Vitor Komura de Freitas | RM563694 | 2CCPG |
| André Ayello de Nóbrega | RM561754 | 2CCPG |

## 0. Setup do repositório (antes da 1ª aula; podem apagar esta seção depois)

1. Um integrante cria o repo **público** no GitHub: `cp6-warroom-<nome-do-grupo>`
   (ex.: `cp6-warroom-debugadores`), com um README qualquer
2. Substituam o conteúdo do `README.md` por este template (no navegador, pelo próprio
   GitHub, ou clonando):
   ```bash
   git clone https://github.com/<conta>/cp6-warroom-<nome-do-grupo>.git
   cd cp6-warroom-<nome-do-grupo>
   # substitua o conteúdo do README.md por este template e:
   git add .
   git commit -m "chore: ficha em branco do grupo"
   git push
   ```
3. Postem o **link no Teams** (o mesmo link para todo o grupo)

**Convenção de commits** (1 commit por rodada; relâmpagos podem ir junto com a
rodada seguinte):

```
decisao: R1 - opcao C (<resumo da justificativa em uma frase>)
decisao: R2 - opcao A (<resumo em uma frase>)
pos-mortem: relatorio de incidente da madrugada
```

---

## 📁 Dossiê técnico do FiapBank (MVP em produção)

**Stack:** Java 17 + Spring Boot + Spring Data JPA + Oracle. API com endpoints em
`/api/contas` e `/api/transferencias` (cenário visto desde a Aula 13).

**Contrato e regras de negócio que o banco prometeu aos clientes e aos reguladores:**

| Regra | Como deve ser |
|---|---|
| Transferência **PIX** | taxa **R$ 0,00** |
| Transferência **TED** | taxa fixa **R$ 5,00** |
| Saldo | **nunca fica negativo**: transferência/saque sem saldo é recusado com erro claro |
| Número de conta | **sequencial e único** (1001, 1002, 1003...), gerado pelo sistema |
| Extrato de transferência | grava **quem pagou** e **quem recebeu**, na ordem certa |
| CPF | **dado sensível**: nunca aparece nas respostas da API |
| Consultas ao banco | sempre parametrizadas, e cada operação **usa e libera** a conexão |
| Suíte de testes | roda antes de todo deploy; **verde** é pré-requisito pra subir |

**Como escrever a justificativa:** nomeie o **mecanismo técnico** em jogo (o
conceito das Aulas 11 a 15 que explica o incidente) e o **trade-off** (velocidade ×
segurança × faturamento × dívida). "Porque é mais seguro" não é justificativa.

---

# 📝 REGISTRO DE DECISÕES

> A cada incidente, o professor dita o título (ex.: "Rodada 1"). **Copiem o modelo
> abaixo, colem no fim desta seção** e preencham com o rascunho feito em aula, junto
> com o **placar do grupo** após a consequência. Commitem 1 commit por rodada até
> 23h59 do dia.
>
> **Eventos relâmpago:** registrem apenas se o grupo for **afetado** (o professor
> chama quem for; não ser chamado é bom sinal).

**Modelo (copiar para cada decisão):**

```
## <título ditado pelo professor>

**Tipo:** ( rodada / relâmpago ) · **Voto:** ( A / B / C / D )

**Justificativa:**

<2 a 3 linhas: mecanismo técnico + trade-off>

**Placar do grupo após esta decisão:** 🔥 __ · 💰 R$ __ mil · 🧹 __
```

*(as decisões entram aqui, na ordem em que a madrugada as trouxer; placar inicial:
🔥 7 · 💰 0 · 🧹 2)*

## Rodada 1: O saldo que virou negativo (19h12)

**Tipo:** rodada · **Voto:** D

**Justificativa:**

Escolhemos o **rollback para a versão de terça** porque o incidente estava ativo (clientes com saldo negativo) e a Black Friday abre às 08h. O rollback é a mitigação mais rápida e previsível: usa o mecanismo de versionamento/deploy já existente, sem escrever código novo às 19h, com o autor do MVP fora da equipe. O bug está no `catch (SaldoInsuficienteException)` de `TransferenciaService.realizar()`, que engole a exceção, e o `debitar()` roda sempre, deixando o saldo negativo. Assumimos o trade-off (velocidade de mitigação × correção definitiva): o rollback perde as entregas posteriores e não ataca a causa raiz, então a correção real (remover o catch + teste `assertThrows` + suíte verde) fica como dívida técnica.

**Consequência (opção D):** o bug some, junto com as 3 correções feitas durante a semana (clientes percebem os bugs antigos voltando). Corrigir de novo na segunda.

**Placar do grupo após esta decisão:** 🔥 6 · 💰 R$ 15 mil · 🧹 3

## Evento Relâmpago A (20h10): GET por titular devolve lista vazia

**Tipo:** relâmpago · **Voto:** B (usar a derived query `findByTitular` do repository)

**Justificativa:**

O `==` entre Strings em Java compara **referências de objeto**, não conteúdo: o nome vindo da requisição e o `titular` carregado do banco são instâncias distintas, então o filtro manual no controller nunca casa e o GET devolve `[]`. Escolhemos a derived query `findByTitular`, que já existia no `ContaRepository` sem uso. Ela delega o filtro ao banco com SQL de **parâmetros vinculados**, respeita a regra de consultas sempre parametrizadas e evita carregar todas as contas em memória. O trade-off é uma mudança um pouco maior que trocar `==` por `equals`, mas corrige a causa (filtro no lugar errado) e não só o sintoma.

**Consequência (opção B):** o caminho da arquitetura: o banco filtra por parâmetros vinculados e mata um incidente que ainda não aconteceu (a injeção SQL da Rodada 2). 🔥 +1.

**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 15 mil · 🧹 3

## Rodada 2: A injeção que derreteu o banco (21h02)

**Tipo:** rodada · **Voto:** D (usar a derived query `findByTitular` do Spring Data JPA)

**Justificativa:**

O alerta mostrou `SELECT * FROM contas WHERE titular = '' OR '1'='1'` retornando todas as contas: **SQL Injection** causada por concatenação de String no JDBC puro (Aula 12). Nosso endpoint já usava `findByTitular` desde o Relâmpago A, que gera SQL com **parâmetros vinculados**: o valor digitado é tratado como dado, nunca como código, e `' OR '1'='1` vira só um nome sem correspondência. Escolhemos a D por ser a solução estrutural (abstração correta, repository já tinha a query pronta), em vez do `PreparedStatement` manual (A), da blocklist (C), que é contornável, ou de tirar a busca do ar (B), que derrubaria o atendimento na Black Friday. O trade-off é demorar ~50 min a mais que a A, em troca de eliminar a concatenação de vez e reduzir dívida técnica.

**Consequência (opção D):** o caminho da arquitetura: usa a abstração certa (o repository já tinha a query pronta). Demora ~50 min, mais que A. Efeitos: 🔥 0 · 💰 0 · 🧹 −1.

**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 15 mil · 🧹 2

## Rodada 3: A noite das contas gêmeas (22h50) · Aula 14

**Tipo:** rodada · **Voto:** D (delegar a numeração ao banco, sequence)

**Justificativa:**

O `getInstancia()` retorna `new GeradorNumeroConta()` a cada chamada: é um **Singleton quebrado** (Aula 14), então o contador recomeça em 1001 e dois clientes receberam o mesmo número de conta. Número de conta é identificador único perante o Bacen, então duplicidade é incidente de compliance. Escolhemos **delegar a numeração ao banco (sequence do Oracle)**: ela é atômica e persistente, não depende da memória da aplicação e atende ao contrato (sequencial, único, gerado pelo sistema). Corrigir só o singleton (A) manteria o contador em memória, que reinicia junto com o servidor. UUID (B) violaria a regra de numeração sequencial. Auditar as duplicatas (C) não impede que novas sejam geradas. O trade-off é demorar mais que o ajuste do singleton, em troca de resolver a causa raiz.

**Consequência (opção D):** arquitetura sólida (~1h de migração de madrugada): o banco garante a unicidade e a aplicação deixa de ser responsável pela numeração. Efeitos: 🔥 0 · 💰 +10 · 🧹 0.

**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 25 mil · 🧹 2

---

## 🔎 O caminho do MEU grupo (preencher na 3ª aula, quando o mapa for revelado)

Uma linha por decisão registrada acima (usem os títulos ditados em aula):

| # | Decisão | Nossa letra | Consequência que ELA teria tido |
|---|---|---|---|
| | | | |
| | | | |
| | | | |

*(adicionem linhas conforme as decisões da madrugada)*

---

# 📋 PÓS-MORTEM · relatório de incidente (montar em sala na 3ª aula)

> Rascunho em aula e commit final `pos-mortem:` até 23h59 do dia da 3ª aula.

## 1. Linha do tempo da madrugada

_______________________________________________________________________________________

_______________________________________________________________________________________

_______________________________________________________________________________________

## 2. Causa raiz de 2 incidentes (aula + mecanismo técnico)

**Incidente 1:** _______________________________________________

Aula/mecanismo:

_______________________________________________________________________________________

**Incidente 2:** _______________________________________________

Aula/mecanismo:

_______________________________________________________________________________________

## 3. O que faríamos diferente (2 rodadas + por quê)

_______________________________________________________________________________________

_______________________________________________________________________________________

## 4. A maior lição da equipe

_______________________________________________________________________________________
