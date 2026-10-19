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

## Rodada 4: O comprovante que aponta pra pessoa errada (00h47)

**Tipo:** rodada · **Voto:** B (corrigir + reprocessar os extratos antigos)

**Justificativa:**

O extrato de transferência é gravado com `origem` e `destino` **trocados** na chamada do construtor `new Transferencia(tipo, destino.getNumero(), origem.getNumero(), valor, taxa)`: como os argumentos são posicionais e do mesmo tipo, a troca compila e passa sem erro, e só aparece no comprovante. Os saldos estão certos; o defeito é só no **registro**. O contrato exige que o extrato grave quem pagou e quem recebeu, na ordem certa, e o comprovante é a prova perante o cliente e a auditoria. Escolhemos **corrigir a gravação e reprocessar o histórico**: corrigir só daqui para frente (A) deixaria comprovantes já emitidos errados, uma dívida de dados que a auditoria encontraria. Comunicar e reemitir pela 2ª via (C) deixa o sistema errado e joga o custo no SAC. Deixar para depois (D) mantém o erro público na Black Friday. O trade-off é gastar cerca de 2h e arcar com mais prejuízo agora, em troca de integridade total dos dados e menos dívida técnica.

**Consequência (opção B):** corrige a gravação e reprocessa o histórico (~2h): os comprovantes já emitidos voltam certos, com integridade total. Efeitos: 🔥 +1 · 💰 +25 · 🧹 −1.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 50 mil · 🧹 1

## Rodada 5: A fábrica que devolve null (01h30)

**Tipo:** rodada · **Voto:** A (fail-fast: exceção clara na factory)

**Justificativa:**

O `ContaFactory.criar()` devolve `null` no `default` do `switch` quando o tipo é desconhecido (`UNIVERSITARIA`), e o `null` só explode muito depois, no `save()` do repository, com a mensagem "Entity must not be null", que não cita a causa real. Escolhemos o **fail-fast**: lançar uma exceção clara na própria factory ("Tipo de conta inválido: UNIVERSITARIA"), de modo que o erro nasce e morre no mesmo lugar, perto da causa, e o cliente e o suporte recebem uma mensagem que explica o problema. Validar só no controller (B) deixa a validação espalhada: outro endpoint que esquecer de validar traz o `null` de volta. Devolver CORRENTE por padrão (C) esconde o erro e abre contas erradas para os clientes. Tirar a conta da campanha (D) faz o erro sumir sem corrigir a causa no código. O trade-off é cerca de 30 min e uma pequena queda de reputação agora (o cadastro da Universitária continua recusado, mas com erro claro), em troca de um sistema que falha de forma previsível e sem dívida técnica.

**Consequência (opção A):** a factory rejeita na hora com "Tipo de conta inválido: UNIVERSITARIA": o erro nasce e morre no mesmo lugar (~30 min). Efeitos: 🔥 −1 · 💰 0 · 🧹 0.

**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 50 mil · 🧹 1

## Evento Relâmpago C (02h20): O cadastro com metade dos campos

**Tipo:** relâmpago · **Voto:** B (Builder com valores padrão)

**Justificativa:**

O marketing quer cadastrar só com nome e saldo, mas o construtor atual exige todos os campos. Isso é o problema do **telescoping constructor** (Aula 14): cada combinação de campos opcionais pede um construtor novo, e a classe cresce sem controle. Escolhemos o **Builder (Builder Pattern)**, que concentra os valores padrão e a validação no `build()`, deixando a criação do objeto legível e cada campo opcional explícito. Mais um construtor sobrecarregado (A) só empurra o telescópio para frente, e aceitar `null` nos campos (C) espalha `NullPointerException` pelo sistema, o mesmo defeito da factory que devolvia `null` na Rodada 5. O trade-off é escrever uma classe a mais agora, em troca de não acumular dívida técnica a cada novo cadastro.

**Consequência (opção B):** o Builder concentra defaults e validação no `build()`. Efeitos: 🧹 0.

**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 50 mil · 🧹 1

## Rodada 6: O teste vermelho às 03h45 (a janela fecha às 04h)

**Tipo:** rodada · **Voto:** A (`@Disabled` no teste e subir na janela)

**Justificativa:**

O teste `tedDeveCobrarTaxaDe5Reais` falha porque o serviço usa `double taxa = 50.00`, um **número mágico** sem constante nem comentário, enquanto o contrato define a TED em R$ 5,00. A janela de deploy fecha às 04h (congelamento para a Black Friday) e a campanha abre às 08h sem exceções, e o fix da R1 mais as correções da noite precisam subir hoje. Escolhemos **subir na janela com o teste marcado `@Disabled("conferir taxa com o gerente")`**, priorizando a **velocidade de entrega** das correções críticas. Assumimos o risco de forma explícita: o teste desligado é um alarme silenciado e a taxa de R$ 50 segue no ar, dez vezes o valor do contrato. Subir com a suíte vermelha (C) normalizaria o vermelho no time, e ajustar o teste para 50.0 (D) faria a suíte confirmar o bug. Investigar agora (B) custaria a janela e adiaria as correções para depois da abertura. O trade-off é velocidade e correções no ar contra dívida técnica e uma taxa possivelmente errada em produção.

**Consequência (opção A):** sobe na janela; o teste fica `@Disabled("conferir taxa com o gerente")`: alarme desligado. A taxa de R$ 50 segue no ar. Efeitos: 🔥 0 · 💰 0 · 🧹 +1.

**Placar do grupo após esta decisão:** 🔥 7 · 💰 R$ 50 mil · 🧹 2

## Evento Relâmpago E (05h58): "A taxa é 5 ou 50?"

**Tipo:** relâmpago · **Voto:** C (reproduzir com um teste antes de responder)

**Justificativa:**

O gerente tem um minuto para responder à imprensa, e o código (`taxa = 50.00`) contradiz o contrato (R$ 5,00). Como na R6 subimos com o teste `tedDeveCobrarTaxaDe5Reais` desabilitado, não temos evidência atual sobre qual valor está em produção. Escolhemos **reproduzir com um teste antes de responder**: o teste é a especificação executável do contrato e transforma opinião em evidência. Confiar no código (A) confirmaria publicamente uma cobrança indevida, e responder pela documentação sem conferir (B) é chutar com autoridade. O trade-off é gastar poucos minutos de um prazo apertado, em troca de uma resposta pública baseada em fato e de entrar no Go/No-Go sabendo da taxa.

**Consequência (opção C):** o teste dá evidência: a equipe entra no go/no-go sabendo da taxa. Efeitos: 🔥 +1.

**Placar do grupo após esta decisão:** 🔥 8 · 💰 R$ 50 mil · 🧹 2

## Rodada 7: Go/No-Go da Black Friday (06h15)

**Tipo:** rodada · **Voto:** B (corrigir o que a noite revelou + suíte verde, abrir 2h atrasado)

**Justificativa:**

Toda falha da noite passou pelo mesmo ponto: o `TransferenciaService`, com ~80 linhas que ninguém entende inteiro (saldo, taxa, extrato e numeração juntos). É uma classe com **muitas responsabilidades**, o que a torna difícil de testar e fácil de quebrar. Nossa suíte está doente desde a R6 (teste da taxa TED `@Disabled`, taxa de R$ 50 contra os R$ 5 do contrato) e o ⚡E confirmou a divergência. Escolhemos **corrigir o que a noite revelou, ter a suíte verde de verdade e abrir 2h atrasado**. Abrir 100% como está (A) colocaria no ar uma cobrança errada que já conhecemos, com risco de estornos e churn. Modo degradado (C) deixaria a taxa errada escondida até a TED voltar, e adiar para segunda (D) perde a campanha inteira. O trade-off é perder o início do pico e aceitar prejuízo agora, em troca de entrar na Black Friday com o contrato cumprido e menos dívida técnica.

**Consequência (opção B, suíte doente):** ao revisar o serviço, o time enxerga a taxa de R$ 50 e conserta antes de abrir: as 2h doeram menos que a taxa doeria. Efeitos: 🔥 +1 · 💰 +40 · 🧹 −1.

**Placar do grupo após esta decisão:** 🔥 9 · 💰 R$ 90 mil · 🧹 1

## 🌅 Epílogo: amanhece a Black Friday (07h00)

**Placar final (congelado):** 🔥 9 · 💰 R$ 90 mil · 🧹 1

**Perfil da noite: A Noite dos Heróis** (🔥 7 ou mais e 🧹 até 2): banco abriu, clientes confiam, time dorme (quase) em paz. Não se aplicam "O Preço da Pressa" (💰 abaixo de 400) nem "Firefighters Crônicos" (🧹 abaixo de 6).

---

## 🔎 O caminho do MEU grupo

| # | Decisão | Nossa letra | Consequência que ELA teve |
|---|---|---|---|
| 1 | R1 · O saldo que virou negativo | D | O rollback fez o bug sumir, junto com as 3 correções da semana (clientes perceberam bugs antigos voltando). 🔥 −1 · 💰 15 · 🧹 +1 |
| 2 | ⚡A · A busca que não acha ninguém | B | `findByTitular` com parâmetros vinculados: matou um incidente que ainda não tinha acontecido. 🔥 +1 |
| 3 | R2 · A injeção que derreteu o banco | D | Abstração certa (o repository já tinha a query), ~50 min a mais que a A. 🔥 0 · 💰 0 · 🧹 −1 |
| 4 | R3 · A noite das contas gêmeas | D | Sequence no banco, ~1h de migração de madrugada: a unicidade passou a ser garantida pelo banco. 🔥 0 · 💰 +10 · 🧹 0 |
| 5 | R4 · O comprovante que aponta pra pessoa errada | B | Corrigiu a gravação e reprocessou o histórico (~2h): integridade total. 🔥 +1 · 💰 +25 · 🧹 −1 |
| 6 | R5 · A fábrica que devolve null | A | Fail-fast: o erro nasce e morre na factory (~30 min). 🔥 −1 · 💰 0 · 🧹 0 |
| 7 | ⚡C · O cadastro com metade dos campos | B | Builder concentrou defaults e validação no `build()`. 🧹 0 |
| 8 | R6 · O teste vermelho às 03h45 | A | Subiu na janela com o teste `@Disabled`: alarme desligado, taxa de R$ 50 no ar. 🔥 0 · 💰 0 · 🧹 +1 |
| 9 | ⚡E · A taxa é 5 ou 50? | C | O teste deu evidência: entramos no Go/No-Go sabendo da taxa. 🔥 +1 |
| 10 | R7 · Go/No-Go da Black Friday | B | Revisando o serviço, o time viu a taxa de R$ 50 e consertou antes de abrir. 🔥 +1 · 💰 +40 · 🧹 −1 |

**Placar final:** 🔥 9 · 💰 R$ 90 mil · 🧹 1 · **Perfil:** A Noite dos Heróis

---

# 📋 PÓS-MORTEM · relatório de incidente

## 1. Linha do tempo da madrugada

- **19h12 · R1:** clientes com saldo negativo. Causa: exceção de saldo insuficiente engolida por um `catch`. Optamos pelo rollback, que também desfez 3 correções da semana.
- **20h10 · ⚡A:** busca por titular devolvia lista vazia (`==` entre Strings). Passamos a usar `findByTitular`.
- **21h02 · R2:** tentativa de SQL Injection (`OR '1'='1'`) por concatenação de SQL. Nosso endpoint já usava parâmetros vinculados; consolidamos na derived query.
- **22h50 · R3:** duas contas com o número 1001 (Singleton sem instância guardada). Numeração delegada a uma sequence do banco.
- **00h47 · R4:** comprovante com origem e destino invertidos. Corrigimos a gravação e reprocessamos o histórico.
- **01h30 · R5:** `ContaFactory` devolvia `null` para tipo desconhecido. Adotamos fail-fast.
- **02h20 · ⚡C:** cadastro com só nome e saldo. Adotamos o Builder.
- **03h45 · R6:** teste da taxa TED vermelho (taxa 50 contra contrato 5). Subimos com o teste `@Disabled`.
- **05h58 · ⚡E:** a imprensa perguntou se a TED custa R$ 5 ou R$ 50. Reproduzimos com um teste antes de responder.
- **06h15 · R7:** Go/No-Go. Corrigimos o que a noite revelou, com a suíte verde, e abrimos 2h atrasados.
- **07h00 · Epílogo:** 🔥 9 · 💰 R$ 90 mil · 🧹 1, perfil A Noite dos Heróis.

## 2. Causa raiz de 2 incidentes (aula + mecanismo técnico)

**Incidente 1:** R1 · O saldo que virou negativo

Aula/mecanismo: **exceção lançada e engolida pelo `catch` no próprio método (Aula 11)**. O `if` lançava `SaldoInsuficienteException`, mas o `catch` vazio a engolia, e o `debitar()` ficava fora do `try`, então rodava sempre e o saldo ficava negativo, com a API devolvendo "sucesso" para o que devia ser recusado.

**Incidente 2:** R3 · A noite das contas gêmeas

Aula/mecanismo: **Singleton sem instância guardada (Aula 14)**. O `getInstancia()` retornava `new GeradorNumeroConta()` a cada chamada, então o contador recomeçava em 1001 e dois clientes receberam o mesmo número de conta, que é identificador único perante o Bacen.

## 3. O que faríamos diferente (2 rodadas + por quê)

- **R1 (voto D → C):** o rollback desfez 3 correções da semana e fez clientes verem bugs antigos voltando. Corrigir a causa (remover o `catch` engolindo a exceção) com um teste `assertThrows` teria barrado a regressão sem perder as correções.
- **R6 (voto A → B):** subir com o teste `@Disabled` silenciou o único alarme que apontava o contrato violado (taxa de R$ 50 contra R$ 5). O ⚡E e o Go/No-Go mostraram o custo de seguir com a suíte doente; investigar na hora teria custado a janela de deploy, mas evitaria a taxa errada no ar.

## 4. A maior lição da equipe

Teste é contrato executável: desligar o alarme (`@Disabled`) ou ajustar o teste ao código esconde o bug em vez de resolvê-lo, e quase todo incidente da noite nasceu de um erro silenciado (exceção engolida, `null`, teste ignorado).

## 5. Uma palavra para a madrugada

**Satisfeitos**
