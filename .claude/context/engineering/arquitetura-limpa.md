# Arquitetura Limpa (Clean Architecture) — Guia de Referência para Codar

> **Fonte:** *Arquitetura Limpa: O Guia do Artesão para Estrutura e Design de Software* — Robert C. Martin (Uncle Bob), 2018.
> **Propósito deste arquivo:** servir como base de conhecimento permanente para o Claude Code. Sempre que for projetar, escrever, refatorar ou revisar código, aplique os princípios, padrões e regras abaixo. Este documento é prescritivo: trate as regras como diretrizes ativas de engenharia, não como teoria.

---

## COMO USAR ESTE DOCUMENTO (instruções para o assistente de código)

1. **Antes de escrever qualquer módulo novo**, identifique a que camada ele pertence (Entidade, Caso de Uso, Adaptador de Interface, Framework/Driver) e garanta que as dependências apontem **para dentro** (rumo à política de alto nível).
2. **Regra de ouro permanente — A Regra da Dependência:** dependências de código-fonte só podem apontar para dentro, na direção das políticas de mais alto nível. Nada em um círculo interno pode mencionar o nome de algo em um círculo externo.
3. **Separe SEMPRE política de detalhe.** Regras de negócio são a joia da coroa; banco de dados, web, UI e frameworks são detalhes plugáveis.
4. **Prefira deixar decisões em aberto.** Não acople o núcleo a banco de dados, framework web, ORM ou biblioteca cedo demais. Maximize o número de decisões NÃO tomadas.
5. **Use o compilador / o sistema de tipos / a estrutura de pastas para impor a arquitetura**, não apenas disciplina e revisão de código. Restrinja visibilidade (package-private, `internal`, módulos) sempre que possível.
6. **Ao revisar código**, percorra as checklists e a lista de "red flags" no fim deste documento.
7. **Ao explicar decisões ao usuário**, fundamente-as nos princípios nomeados (SRP, OCP, DIP, Regra da Dependência etc.) para que a escolha seja rastreável.

---

## 1. FILOSOFIA CENTRAL — POR QUE ARQUITETURA IMPORTA

### 1.1 Design e Arquitetura são a mesma coisa
Não há diferença real entre "design" e "arquitetura". "Arquitetura" costuma ser usada para decisões de alto nível e "design" para detalhes de baixo nível, mas eles formam um **tecido contínuo**: há uma linha ininterrupta de decisões que vai do nível mais alto ao mais baixo. Detalhes de baixo nível e estrutura de alto nível fazem parte do mesmo todo. Você não pode ter um sem o outro.

### 1.2 O objetivo da arquitetura
> **O objetivo da arquitetura de software é minimizar os recursos humanos necessários para construir e manter um determinado sistema.**

A medida da qualidade do design é o esforço necessário para atender às demandas do cliente:
- Esforço **baixo e estável** ao longo da vida do sistema → design **bom**.
- Esforço que **cresce a cada release** → design **ruim**.

### 1.3 A marca registrada de uma bagunça
Quando o sistema é desenvolvido às pressas, com o número de programadores como único motor de resultado e pouca preocupação com a limpeza do código, a produtividade despenca release após release, tendendo assintoticamente a zero — enquanto o custo por linha de código explode (no estudo de caso do livro, **40x** entre o release 1 e o 8). Os desenvolvedores trabalham cada vez mais e entregam cada vez menos, porque o esforço é desviado da criação de recursos para a gestão da bagunça.

### 1.4 A maior mentira do desenvolvimento
> "Podemos limpar tudo depois, primeiro temos que colocar no mercado."

Isso é falso. As coisas **nunca** são limpas depois, porque a pressão do mercado nunca diminui. Fazer bagunça é **sempre mais lento** que manter tudo limpo — em qualquer escala de tempo. Recomeçar do zero também é uma ilusão (o "excesso de confiança da Lebre"): a mesma equipe recriará a mesma bagunça.

> **A única maneira de ir rápido é ir bem.**
> (Confirmado empiricamente: TDD foi ~10% mais rápido que codar sem disciplina, e o dia mais lento *com* TDD foi mais rápido que o dia mais rápido *sem* TDD.)

### 1.5 Os dois valores do software
Todo sistema fornece **dois valores** aos stakeholders. Desenvolvedores são responsáveis por manter ambos altos — e frequentemente focam no menor, esvaziando o valor do sistema.

| Valor | O que é | Urgência | Importância |
|---|---|---|---|
| **Comportamento** | fazer a máquina cumprir os requisitos / ganhar ou economizar dinheiro | Urgente | Nem sempre importante |
| **Estrutura** (arquitetura) | manter o software *soft* — fácil de mudar | Nunca urgente | Sempre importante |

**Por que a estrutura é o valor MAIOR:** software = "soft" + "ware". Foi inventado para ser fácil de mudar. A dificuldade de uma mudança deve ser proporcional ao **escopo** da mudança, não à sua **forma**. Quando a arquitetura privilegia uma forma, novos recursos com forma diferente ficam cada vez mais caros de encaixar ("peças quadradas em buracos redondos"). Por isso a arquitetura deve ser tão **agnóstica de forma** quanto possível.

**Prova lógica de que a estrutura vale mais (pelos extremos):**
- Programa que funciona perfeitamente mas é **impossível de mudar** → torna-se inútil quando os requisitos mudam.
- Programa que **não funciona** mas é **fácil de mudar** → pode ser feito funcionar e mantido funcionando. Permanece útil.

### 1.6 Matriz de Eisenhower (importante vs. urgente)
"O urgente não é importante, e o importante não é urgente." Ordene as prioridades:
1. Urgente **e** importante
2. **Não** urgente **e** importante ← *aqui mora a arquitetura*
3. Urgente e **não** importante
4. Não urgente e não importante

O erro clássico: tratar itens da posição 3 (urgentes, não importantes / recursos comportamentais) como se fossem posição 1, ignorando a arquitetura (posições 1 e 2). Gestores de negócio **não estão equipados** para avaliar a importância da arquitetura — essa é a responsabilidade da equipe de desenvolvimento.

### 1.7 Lute pela arquitetura
Como desenvolvedor, você é um **stakeholder** do software: tem uma participação a proteger. Isso é parte do seu papel. Se a arquitetura vier por último, o sistema fica cada vez mais caro e a mudança se torna inviável — sinal de que a equipe não lutou o suficiente.

---

## 2. PARADIGMAS DE PROGRAMAÇÃO — CADA UM TIRA UM PODER

A arquitetura começa no código. Existem **três** paradigmas (todos descobertos entre 1958 e 1968; é improvável que surjam outros). O traço comum surpreendente: **cada paradigma REMOVE uma capacidade do programador**. Eles dizem o que *não* fazer, impondo disciplina com intenção negativa.

| Paradigma | O que disciplina/remove | Uso arquitetural |
|---|---|---|
| **Estruturada** | disciplina sobre a **transferência direta** de controle (remove `goto`) | base algorítmica dos módulos; funções comprováveis e testáveis |
| **Orientada a Objetos** | disciplina sobre a **transferência indireta** de controle (remove ponteiros de função "soltos") | cruzar limites arquiteturais via polimorfismo; inversão de dependência; arquitetura de plug-in |
| **Funcional** | disciplina sobre a **atribuição** (remove/restringe mutabilidade) | gestão de localização e acesso a dados; concorrência segura |

Alinhamento com as três grandes preocupações da arquitetura: **função** (estruturada), **separação de componentes** (OO), **gerenciamento de dados** (funcional).

### 2.1 Programação Estruturada → testes provam erro, não acerto
> "Os testes mostram a presença, não a ausência, de bugs." — Dijkstra

Software é como **ciência**, não matemática: demonstramos correção ao **falhar em provar a incorreção**, depois de muito esforço. Um teste pode provar que um programa está errado, nunca que está 100% certo — apenas "suficientemente correto para nossos propósitos". A estrutura nos força a decompor o programa recursivamente em **pequenas funções comprováveis/refutáveis**. Por isso a decomposição funcional continua sendo uma das melhores práticas, e por isso linguagens modernas evitam `goto` irrestrito.

**Implicação para codar:** escreva funções pequenas, com uma só saída lógica clara, fáceis de testar pela negação. Em todos os níveis (da função ao componente), busque testabilidade via disciplinas restritivas.

### 2.2 Orientação a Objetos → poder sobre as dependências
OO **não** é só "dados + funções" (isso já existia antes de 1966), nem só "modelar o mundo real" (vago). Para o arquiteto, **OO é a capacidade de obter controle absoluto, via polimorfismo, sobre a direção de toda dependência de código-fonte do sistema.**

- O polimorfismo já existia (ex.: `STDIN`/`STDOUT` em C via ponteiros de função na struct `FILE`). OO apenas o tornou **seguro e conveniente**, eliminando as convenções manuais perigosas de ponteiros de função.
- **Inversão de dependência:** normalmente a dependência de código-fonte segue o fluxo de controle (quem chama menciona quem é chamado). Com polimorfismo, ao inserir uma **interface** entre os módulos, a dependência de código-fonte do módulo de baixo nível passa a apontar **contra** o fluxo de controle, rumo à abstração.
- **Consequência:** o arquiteto pode fazer o banco de dados e a UI **dependerem** das regras de negócio (e não o contrário). UI e BD viram **plug-ins** das regras de negócio. Isso habilita **implantação independente** e **desenvolvimento independente** por equipes diferentes.

> **Regra prática:** sempre que precisar que um módulo de alto nível use um de baixo nível sem depender dele, coloque uma interface (de propriedade do alto nível) no meio e inverta a dependência.

### 2.3 Programação Funcional → imutabilidade e concorrência
- **Todos** os problemas de concorrência (race conditions, deadlocks, atualizações concorrentes) decorrem de **variáveis mutáveis**. Sem mutação, esses problemas não existem.
- A imutabilidade total exigiria armazenamento e CPU infinitos; na prática, faça **concessões**:
  - **Segregação de mutabilidade:** divida a aplicação em componentes **imutáveis** (puramente funcionais) e **mutáveis**. Empurre o máximo de processamento para os imutáveis e o máximo de código para fora dos mutáveis. Proteja o estado mutável com **memória transacional** (ex.: `atom`/`swap!` em Clojure, compare-and-swap).
  - **Event Sourcing:** armazene **transações**, não estado. O estado é derivado replicando as transações desde o início (com snapshots periódicos como atalho). Sem updates/deletes → sem problemas de concorrência. Aplicações viram **CR** em vez de CRUD (é assim que o controle de versão funciona).

> **Regra prática:** prefira estruturas imutáveis e funções puras no núcleo. Concentre a mutação de estado em fronteiras bem delimitadas e protegidas.

---

## 3. PRINCÍPIOS SOLID — DESIGN DE CLASSES E MÓDULOS

Os princípios SOLID dizem como organizar funções e estruturas de dados em classes e como essas classes devem se interconectar. Objetivo: criar estruturas de software de nível médio que **tolerem mudança**, sejam **fáceis de entender** e sejam **a base de componentes reutilizáveis**. Eles se aplicam a qualquer agrupamento coeso de código (classe em OO, módulo em geral).

### 3.1 SRP — Princípio da Responsabilidade Única (Single Responsibility Principle)
> **Um módulo deve ser responsável por um, e apenas um, ator.**

**O que NÃO é:** "uma função/classe deve fazer só uma coisa". Isso é um princípio de baixo nível (refatorar funções grandes em pequenas), **não** o SRP.

**O que É:** um módulo deve ter **uma, e apenas uma, razão para mudar** — onde "razão para mudar" = um **ator** (um grupo de usuários/stakeholders que demanda mudanças pelos mesmos motivos). Coesão = a força que amarra o código responsável a um único ator.

**Sintomas de violação (procure por eles):**
1. **Duplicação acidental / acoplamento de atores:** uma classe (ex.: `Employee` com `calculatePay()` [CFO/contabilidade], `reportHours()` [COO/RH], `save()` [CTO/DBA]) junta código de atores diferentes. Um método compartilhado (`regularHours()`) alterado para o ator A quebra silenciosamente o ator B.
2. **Fusões (merges):** vários devs de equipes/atores diferentes editam o mesmo arquivo por razões diferentes e colidem.

**Soluções:**
- Separe dados das funções: uma estrutura de dados sem métodos (`EmployeeData`) compartilhada por classes distintas (`PayCalculator`, `HourReporter`, `EmployeeSaver`), que **não conhecem umas às outras**.
- Use o padrão **Facade** para reduzir o atrito de instanciar/rastrear várias classes, ou mantenha a regra mais importante na classe original que age como facade para as menores.

**Como aplicar ao codar:**
- Antes de adicionar um método a uma classe, pergunte: *qual ator pede essa mudança?* Se for um ator diferente do resto da classe, **separe**.
- Não coloque na mesma classe código que muda por razões de departamentos/atores diferentes.

**Escala maior:** no nível de componente o SRP vira **CCP** (Common Closure); no nível arquitetural vira o **Eixo da Mudança** que cria limites.

---

### 3.2 OCP — Princípio Aberto/Fechado (Open-Closed Principle)
> **Um artefato de software deve ser aberto para extensão, mas fechado para modificação.**

O comportamento deve ser **extensível adicionando código novo**, não alterando código existente. Quando extensões simples nos requisitos forçam mudanças massivas, a arquitetura falhou. **Zero** mudança no código antigo é o ideal.

**Como concretizar:**
1. **Separe** o que muda por razões diferentes (aplique **SRP**).
2. **Organize as dependências** entre essas partes (aplique **DIP**) de modo que a mudança em uma não force mudança na outra.
3. Particione em **componentes** organizados numa **hierarquia de proteção**: componentes de nível mais alto (que contêm as regras de negócio) ficam **protegidos** de mudanças nos de nível mais baixo.

**Mecânica:** para proteger o componente A de mudanças em B, faça **B depender de A** (as setas de dependência apontam para o que se quer proteger). Use interfaces para **inverter dependências** e direcioná-las corretamente. Use também **ocultação de informação** (ex.: interfaces que impedem dependências transitivas) para que entidades não dependam de coisas que não usam diretamente.

**Como aplicar ao codar:**
- Ao prever um eixo de variação (ex.: vários formatos de saída, vários provedores de pagamento), introduza uma abstração e implemente cada variação como uma nova classe/plug-in.
- Nunca espalhe `if/else`/`switch` por tipo concreto pelo código quando polimorfismo resolveria.

---

### 3.3 LSP — Princípio de Substituição de Liskov (Liskov Substitution Principle)
> Subtipos devem ser **substituíveis** por seus tipos base sem alterar a corretude do programa. Para construir sistemas a partir de partes intercambiáveis, essas partes devem aderir a um **contrato** que permita substituí-las umas pelas outras.

**Exemplo de conformidade:** `License` com subtipos `PersonalLicense`/`BusinessLicense` — o cliente (`Billing`) funciona com qualquer um via `calcFee()`.

**Exemplo canônico de violação — Quadrado/Retângulo:** `Square` não é subtipo válido de `Rectangle` porque altura e largura do retângulo são mutáveis independentemente, mas no quadrado mudam juntas. O cliente que faz `setW(5); setH(2); assert(area()==10)` quebra. A "defesa" exigiria `if (rectangle is Square)` no cliente → os tipos deixam de ser substituíveis.

**LSP é arquitetural, não só sobre herança.** Aplica-se a interfaces Java, classes Ruby com mesma assinatura e **serviços REST** com a mesma interface.
- **Exemplo (agregador de táxis):** se um serviço usa `dest` em vez de `destination` no URI, surge a tentação de `if (uri.startsWith("acme.com"))` no código → bugs, falhas de segurança, e mais `if`s a cada fusão de empresas. A violação **contamina a arquitetura** com mecanismos extras (ex.: uma base de configuração que mapeia formato de despacho por URI).

**Como aplicar ao codar:**
- Toda implementação de uma interface deve honrar **plenamente** o contrato (pré/pós-condições, semântica, exceções). Se uma subclasse precisa enfraquecer o contrato ou lançar "não suportado", o design está errado.
- Nunca force o cliente a checar o tipo concreto por trás de uma abstração.

---

### 3.4 ISP — Princípio da Segregação de Interface (Interface Segregation Principle)
> **Não force um cliente a depender de métodos/coisas que ele não usa.**

Se `User1` usa só `op1` de uma classe `OPS` que também tem `op2`/`op3`, ele fica **acoplado inadvertidamente** a `op2`/`op3`. Em linguagens estaticamente tipadas, uma mudança em `op2` força recompilação/reimplantação de `User1` mesmo sem nada essencial ter mudado. **Solução:** segregar em interfaces específicas (`U1Ops`, `U2Ops`, `U3Ops`) para que cada cliente dependa só do que usa.

**Nuance de linguagem:** o problema de recompilação é mais agudo em linguagens estaticamente tipadas (Java, C#). Em dinamicamente tipadas (Ruby, Python) as dependências são inferidas em runtime, reduzindo o acoplamento — por isso são mais flexíveis nesse aspecto.

**É arquitetural:** depender de um módulo "gordo" com mais do que você precisa é prejudicial. Ex.: `S` depende de `F` (framework) que depende de `D` (banco). Recursos de `D` que `F` não usa, ao mudarem ou falharem, podem forçar reimplantação ou quebrar `S`.

> **Lição:** depender de algo que contém itens desnecessários causa problemas inesperados. (Versão genérica disso no nível de componente: **CRP**.)

**Como aplicar ao codar:** crie interfaces **pequenas e focadas no papel** (role interfaces). Não obrigue implementadores a fornecer métodos vazios/irrelevantes.

---

### 3.5 DIP — Princípio da Inversão de Dependência (Dependency Inversion Principle)
> **Os sistemas mais flexíveis são aqueles em que as dependências de código-fonte se referem apenas a abstrações, não a itens concretos.**

`import`/`use`/`include` devem apontar para interfaces, classes abstratas ou declarações abstratas — **não** para módulos concretos voláteis.

**Estabilidade primeiro:** ignoramos o DIP para itens concretos **estáveis** (ex.: `java.lang.String`, SO, plataforma). O alvo do DIP são os **concretos voláteis** — os módulos que você desenvolve ativamente e que mudam com frequência. Interfaces são menos voláteis que implementações; bons designers trabalham para reduzir ainda mais a volatilidade das interfaces (adicionar funcionalidade às implementações sem mexer na interface).

**Práticas de codificação específicas do DIP:**
1. **Não se refira a classes concretas voláteis.** Refira-se a interfaces abstratas. (Isso força o uso de **Fábricas Abstratas** para criar objetos.)
2. **Não derive de classes concretas voláteis.** Herança é o acoplamento de código-fonte mais forte; use com extremo cuidado.
3. **Não sobrescreva funções concretas.** Sobrescrever não elimina as dependências da função concreta — você as herda. Em vez disso, torne a função **abstrata** e crie múltiplas implementações.
4. **Nunca mencione o nome de algo concreto e volátil.**

**Fábricas (Factories) e o limite abstrato/concreto:**
- Criar um objeto concreto exige, em quase toda linguagem, uma dependência de código-fonte na definição concreta. Para evitar isso, use **Abstract Factory**: a `Application` usa `ConcreteImpl` via interface `Service`, e cria instâncias chamando `ServiceFactory.makeSvc()` (implementada por `ServiceFactoryImpl`).
- Isso divide o sistema em um **componente abstrato** (regras de negócio de alto nível) e um **componente concreto** (detalhes). A "linha curva" que separa os dois é um **limite arquitetural**.
- **O fluxo de controle cruza a linha na direção OPOSTA às dependências de código-fonte.** As dependências de código-fonte estão **invertidas** em relação ao fluxo de controle — daí o nome do princípio.

**Componentes concretos / `main`:** violações do DIP não podem ser 100% eliminadas; **concentre-as** em poucos componentes concretos, separados do resto. O componente `main` é onde os concretos voláteis são instanciados (geralmente via injeção de dependência).

> **O DIP é o princípio organizador mais visível nos diagramas de arquitetura.** A forma como as dependências cruzam a linha curva sempre em direção às entidades mais abstratas é a **Regra da Dependência** (ver seção de Arquitetura).

---

## 4. PRINCÍPIOS DOS COMPONENTES

Se SOLID organiza os "tijolos" em paredes e salas, os princípios de componentes organizam as salas em prédios. **Componente** = unidade de implantação (jar em Java, gem em Ruby, DLL em .NET, etc.). Bem projetados, componentes mantêm a capacidade de serem **implantados e desenvolvidos independentemente**.

### 4.1 Coesão de componentes — quais classes pertencem a qual componente

#### REP — Princípio da Equivalência do Reúso/Release (Reuse/Release Equivalence Principle)
> **A granularidade do reúso é a granularidade do release.**
Para reutilizar um componente, ele precisa ser **rastreado por um processo de release** com números de versão e notas de release. As classes/módulos de um componente devem formar um **grupo coeso** com um tema/propósito comum — devem fazer sentido para serem liberadas **juntas**.

#### CCP — Princípio do Fechamento Comum (Common Closure Principle)
> **Reúna no mesmo componente as classes que mudam pelas mesmas razões e nos mesmos momentos. Separe em componentes diferentes as que mudam por razões/momentos diferentes.**
É o **SRP no nível de componente**. Na maioria das aplicações, **manutenção > reúso**: se uma mudança fica confinada a um único componente, só ele precisa ser revalidado/reimplantado. Está ligado ao OCP — agrupa classes "fechadas" para os mesmos tipos de mudança.

#### CRP — Princípio do Reúso Comum (Common Reuse Principle)
> **Não force os usuários de um componente a depender de coisas que eles não precisam.**
Classes que tendem a ser **reutilizadas juntas** (ex.: container + iteradores) pertencem ao mesmo componente. Mais importante: classes **sem forte ligação** NÃO devem ficar no mesmo componente, pois qualquer dependência (mesmo que use só uma classe do componente) força recompilação/revalidação/reimplantação quando o componente usado muda. É o **ISP no nível de componente**: *não dependa de coisas que você não precisa.*

#### O diagrama de tensão da coesão (equilíbrio dinâmico)
Os três princípios brigam entre si:
- **REP** e **CCP** são **inclusivos** → aumentam os componentes.
- **CRP** é **exclusivo** → diminui os componentes.

```
                 REP (reusabilidade)
                  /\
                 /  \
   abandonar REP/    \abandonar CRP
   = muitos releases  = muitos componentes
   desnecessários      impactados por uma
                       mudança
                /        \
           CCP /__________\ CRP
   (manutenção)            (não depender do desnecessário)
```
- Foco só em REP+CRP → mudanças simples impactam muitos componentes.
- Foco só em CCP+REP → muitos releases desnecessários.
- **No início do projeto**, CCP > REP (capacidade de desenvolvimento importa mais que reúso). Projetos começam à direita do triângulo (sacrificando reúso) e migram para a esquerda conforme amadurecem. **A estrutura de componentes evolui com o tempo.**

### 4.2 Acoplamento de componentes — relações entre componentes

#### ADP — Princípio das Dependências Acíclicas (Acyclic Dependencies Principle)
> **Não permita ciclos no grafo de dependência dos componentes.**
Evita a **"síndrome da manhã seguinte"** (alguém mudou algo do qual você dependia e nada mais funciona). O grafo de dependências deve ser um **DAG** (grafo acíclico direcionado).

**Benefícios do DAG:**
- Impacto de um release é rastreável seguindo as setas de trás para frente.
- Para testar um componente, só é preciso buildar suas dependências diretas.
- O release do sistema é feito **de baixo para cima**, em ordem clara.

**Efeito de um ciclo:** componentes do ciclo viram, na prática, **um único componente gigante** — todos precisam usar exatamente o mesmo release; testes exigem buildar tudo junto; pode não existir ordem de build correta.

**Como quebrar um ciclo:**
1. **Aplique DIP:** crie uma interface com os métodos que o componente A precisa, coloque-a em A e faça B implementá-la — inverte a dependência.
2. **Crie um novo componente** do qual ambos dependam, movendo para lá as classes compartilhadas.

> **Corolário — a estrutura de componentes NÃO é projetada de cima para baixo.** Ela **evolui** com o sistema. Diagramas de dependência de componentes não descrevem a função da aplicação; são um **mapa de build e manutenção** e de **isolamento de volatilidade** (proteger componentes estáveis de alto valor dos voláteis). Monitore ciclos continuamente e quebre-os quando surgirem.

#### SDP — Princípio das Dependências Estáveis (Stable Dependencies Principle)
> **Dependa na direção da estabilidade.**
Um componente difícil de mudar não deve depender de um componente volátil (senão o volátil também fica difícil de mudar). Estabilidade ≠ frequência de mudança; é a **quantidade de trabalho necessária para mudar**. Um componente do qual **muitos dependem** é estável (muitas razões para não mudar) e **independente**.

**Métricas de estabilidade:**
- **Fan-in:** nº de classes externas que dependem deste componente (dependências que chegam).
- **Fan-out:** nº de classes deste componente que dependem de classes externas (dependências que saem).
- **I (Instabilidade) = Fan-out / (Fan-in + Fan-out)**, varia de 0 a 1.
  - **I = 0** → máxima estabilidade (responsável e independente).
  - **I = 1** → máxima instabilidade (irresponsável e dependente).
- **Regra:** a métrica I de um componente deve ser **maior** que a dos componentes dos quais ele depende. **I deve decrescer na direção da dependência.**

Nem todo componente deve ser estável — se todos fossem, o sistema seria imutável. Coloque componentes **instáveis no topo** do diagrama (qualquer seta apontando para cima viola SDP/ADP). Corrija violações com DIP (extraia uma interface para um componente abstrato estável).

#### SAP — Princípio das Abstrações Estáveis (Stable Abstractions Principle)
> **Um componente deve ser tão abstrato quanto estável.**
- Componente **estável** (I=0) deve ser **abstrato** (interfaces/classes abstratas) para que a estabilidade não impeça extensão (resolve a tensão via OCP). É onde ficam as **políticas de alto nível**.
- Componente **instável** (I=1) deve ser **concreto**, pois sua instabilidade permite mudar o código concreto facilmente.

**Métrica de abstração:**
- **Nc** = nº total de classes do componente; **Na** = nº de classes abstratas/interfaces.
- **A = Na / Nc**, varia de 0 (nenhuma abstração) a 1 (só abstrações).

**SDP + SAP = DIP no nível de componente:** dependências apontam para a estabilidade (SDP) e estabilidade implica abstração (SAP) → dependências apontam para a **abstração**.

#### A Sequência Principal e as zonas de exclusão
Plote A (vertical) × I (horizontal). Bons componentes ficam em (0,1) — abstrato e estável — ou (1,0) — concreto e instável.
- **Zona da Dor** (perto de (0,0)): concreto, estável e rígido. Não pode ser estendido (não abstrato) e é difícil de mudar (estável). Ex.: **esquema de banco de dados** (volátil + concreto + muito dependido) e bibliotecas utilitárias concretas. Componentes **não voláteis** ali (ex.: `String`) são inofensivos.
- **Zona da Inutilidade** (perto de (1,1)): abstrato sem dependentes — restos de classes abstratas que ninguém implementou. Detrito.
- **Sequência Principal:** a linha entre (1,0) e (0,1). Componentes nela não são "abstratos demais" para sua estabilidade nem "instáveis demais" para sua abstração.
- **D (Distância) = |A + I − 1|**, varia de 0 (na Sequência Principal) a 1 (o mais longe possível). Componentes com D longe de zero devem ser reexaminados/reestruturados. Pode-se acompanhar D ao longo do tempo para detectar degradação (ex.: um componente que se afasta da Sequência Principal a cada release).

> **Aviso do autor:** métricas não são deuses; são medidas contra um padrão arbitrário, imperfeitas, mas úteis.

---

## 5. ARQUITETURA

### 5.1 O que é (e o que faz) a arquitetura
A arquitetura é a **forma** dada ao sistema: como ele é dividido em componentes, como eles são organizados e como se comunicam. **Propósito:** facilitar **desenvolvimento, implantação, operação e manutenção** — e, principalmente, **minimizar o custo de vida útil** do sistema e maximizar a produtividade.

> **Estratégia central: deixar o máximo de opções em aberto, pelo máximo de tempo possível.**

- A arquitetura tem **pouca** relação com fazer o sistema *funcionar* (muitos sistemas funcionam apesar de arquiteturas terríveis). Os problemas de má arquitetura aparecem na **implantação, manutenção e desenvolvimento contínuo**.
- **Um arquiteto de software CONTINUA programando.** Não abandone o código — você não consegue projetar bem se não vivenciar os problemas que cria para os demais.

**Impacto por dimensão:**
- **Desenvolvimento:** a estrutura tende a espelhar a organização das equipes (Lei de Conway). Equipes pequenas toleram monólitos; muitas equipes exigem componentes bem definidos com interfaces estáveis.
- **Implantação:** mire em **deploy com uma única ação**. Considere a implantação **cedo** (não decida por microsserviços só porque facilita o dev — pode virar pesadelo de configuração).
- **Operação:** problemas de operação muitas vezes se resolvem com mais hardware (barato) — então a arquitetura pende para dev/deploy/manutenção. Ainda assim, a arquitetura deve **comunicar/revelar a operação** (elevar casos de uso e comportamentos a entidades de primeira classe).
- **Manutenção:** é o custo mais alto (exploração + risco). Separar em componentes isolados por interfaces estáveis reduz o custo de achar onde mexer e o risco de quebrar.

### 5.2 Manter as opções abertas: POLÍTICA vs. DETALHE
Todo sistema se decompõe em:
- **Política:** regras e procedimentos de negócio. **É onde está o valor real.**
- **Detalhes:** o que permite humanos/sistemas/programadores se comunicarem com a política, mas não afeta o comportamento dela — **IO, banco de dados, web, servidores, frameworks, protocolos**.

O trabalho do arquiteto é criar uma forma que reconheça a política como o essencial e torne os **detalhes irrelevantes** para ela, **adiando** as decisões de detalhe:
- Não escolha o **banco de dados** cedo — a política de alto nível não deve saber se é relacional, distribuído, hierárquico ou arquivos simples.
- Não escolha o **servidor/framework web** cedo — a política não deve saber de HTML/AJAX/JSP/JSF; nem precisa decidir se será entregue pela web.
- Não adote **REST/SOA/microsserviços** cedo — a política deve ser agnóstica à interface com o mundo externo.
- Não adote **framework de DI** cedo — a política não deve se preocupar com resolução de dependências.

> Se as decisões já foram tomadas por terceiros, **finja que não foram** e molde o sistema como se ainda pudessem ser adiadas/trocadas.
> **Um bom arquiteto maximiza o número de decisões NÃO tomadas.**

**Lição histórica (independência de dispositivo):** acoplar código a dispositivos IO específicos foi um erro caro. Abstrair IO em serviços do SO (registros de unidade) deu origem ao OCP — o mesmo programa lê/escreve cartões ou fita sem mudança. A política (formatação de nomes/endereços) ficou desacoplada do detalhe (o dispositivo).

### 5.3 Independência, desacoplamento e duplicação
Uma boa arquitetura suporta:
- **Casos de uso** como cidadãos de primeira classe (a estrutura "grita" o que o sistema faz).
- **Independência de:** frameworks, UI, banco de dados, e qualquer agência externa.
- **Testabilidade** sem UI/BD/servidor.

**Desacople em camadas E por caso de uso.** Casos de uso são "fatias verticais" que cortam as camadas horizontais (UI, regra de negócio, BD). Isso permite adicionar/alterar um caso de uso sem afetar os outros.

**Modos de desacoplamento (escolha conforme a necessidade, e a decisão pode mudar no tempo):**
- **Nível de código-fonte:** controlar dependências entre módulos no mesmo executável (monólito).
- **Nível de implantação:** controlar dependências entre unidades implantáveis (jars, DLLs).
- **Nível de serviço:** dependências no nível de dados/comunicação (serviços/microsserviços).
Comece simples (código-fonte) e **promova** para deploy/serviço só quando necessário; saiba também **regredir** se a separação por serviço se mostrar custosa demais.

**Duplicação — distinga as duas:**
- **Duplicação verdadeira:** mesma coisa, mudam sempre juntas → **elimine** (DRY).
- **Duplicação acidental/falsa:** parecem iguais agora, mas evoluem por razões diferentes (ex.: tela de um caso de uso vs. de outro; estruturas de request/response vs. entidades) → **NÃO unifique**. Unificar cria acoplamento que mais tarde força condicionais e "tramp data". *Cuidado especial:* não una as estruturas de request/response aos objetos Entidade só porque compartilham dados — eles mudam por razões diferentes.

### 5.4 Política e Nível
> **Nível = distância das entradas e saídas.** Quanto mais longe do IO, **mais alto** o nível.

- Coloque o componente de mais alto nível (ex.: a transformação central) o mais distante possível do IO. As dependências de código-fonte devem ser **desacopladas do fluxo de dados e acopladas ao nível**.
- **Anti-exemplo:** `writeChar(translate(readChar()))` numa função `encrypt()` de alto nível faz o alto nível depender de funções de baixo nível (`readChar`/`writeChar`). Corrija com interfaces (`CharReader`/`CharWriter`) cujas implementações concretas (console etc.) são de baixo nível e apontam **para dentro**.
- Políticas de **alto nível** mudam **com menos frequência** e por razões mais importantes; as de **baixo nível** mudam com frequência e urgência, mas por razões menos importantes. Mantê-las separadas, com dependências apontando para o alto nível, reduz o impacto das mudanças.
- **Componentes de baixo nível devem ser plug-ins dos de alto nível.**

### 5.5 Regras de Negócio: Entidades e Casos de Uso
**Regras de negócio** = regras/procedimentos que **geram ou economizam dinheiro**, independentemente de existir um computador.

#### Entidades — Regras Cruciais de Negócio (Critical Business Rules)
- Objeto (ou conjunto de funções + dados) que encapsula as **Regras Cruciais de Negócio** e os **Dados Cruciais de Negócio** da empresa. Existiriam mesmo sem automação.
- **Puramente negócio.** Não sabe nada sobre banco de dados, UI, frameworks. Reutilizável por **muitas aplicações** da empresa.
- Não precisa ser OO — basta reunir dados e regras cruciais em um módulo único e separado.
- **Nível mais alto.** Não deve ser impactada por mudanças de navegação, segurança, ou qualquer mudança operacional de uma aplicação específica.

#### Casos de Uso — Regras de Negócio Específicas da Aplicação
- Descrevem **como** um sistema automatizado é usado: entrada do usuário, saída ao usuário e os passos de processamento. Definem **como e quando** as Regras Cruciais (das Entidades) são invocadas — "controlam a dança das Entidades".
- **NÃO descrevem a UI.** A partir de um caso de uso é impossível saber se a app é web, console, thick client ou serviço. O modo como os dados entram/saem é irrelevante.
- **Direção da dependência:** casos de uso **dependem** das Entidades; Entidades **não** sabem dos casos de uso (DIP). Por quê? Casos de uso são específicos de uma app (mais perto do IO, nível mais baixo); Entidades são generalizações (mais longe do IO, nível mais alto).

#### Modelos de Requisição e Resposta (Request/Response Models)
- O caso de uso recebe uma **estrutura de dados de request simples** e devolve uma **estrutura de response simples**. Essas estruturas **não dependem de nada** — nada de `HttpRequest`/`HttpResponse`, nada de HTML/SQL.
- **Não** coloque referências a objetos Entidade nesses modelos (mesmo que compartilhem dados): eles mudam por razões diferentes → violaria CCP/SRP e geraria condicionais/tramp data.

> **As regras de negócio são as joias da família.** Devem permanecer puras, imaculadas por UI/BD, e ser o código **mais independente e reutilizável** do sistema, contendo as preocupações menores como plug-ins.

### 5.6 Arquitetura Gritante (Screaming Architecture)
A estrutura de pastas/pacotes de topo deve **gritar o domínio**, não o framework. Ao abrir o repositório de um sistema de saúde, a primeira impressão deve ser "isto é um sistema de saúde", não "isto é um app Spring/Rails". Casos de uso devem ser óbvios; UI, controllers e views são detalhes decididos depois. Frameworks são **ferramentas, não modos de vida** — use-os com ceticismo e mantenha-os sob rédea curta. Arquiteturas centradas em casos de uso são **testáveis sem framework**.

### 5.7 A ARQUITETURA LIMPA (o diagrama dos círculos concêntricos)
Integra ideias de **Arquitetura Hexagonal (Ports & Adapters)** de Cockburn, **DCI** (Coplien/Reenskaug) e **BCE** (Jacobson). Todas buscam o mesmo: **separação de preocupações** dividindo o software em camadas, com pelo menos uma camada de regras de negócio e uma de interfaces.

**Características produzidas:**
- Independência de **frameworks** (use-os como ferramentas).
- **Testabilidade** (regras testáveis sem UI/BD/web).
- Independência da **UI** (trocar web por console sem mexer nas regras).
- Independência do **banco de dados** (trocar Oracle/SQL Server por Mongo/CouchDB/etc.).
- Independência de **qualquer agência externa**.

**Os quatro círculos (de dentro para fora — nível decrescente):**

```
        ┌─────────────────────────────────────────────┐
        │  Frameworks & Drivers (DB, Web, UI, devices) │  ← detalhes, o mais externo
        │  ┌───────────────────────────────────────┐   │
        │  │  Adaptadores de Interface             │   │  ← Controllers, Presenters,
        │  │  (Controllers, Gateways, Presenters)  │   │     Gateways, MVC, conversão de dados
        │  │  ┌─────────────────────────────────┐  │   │
        │  │  │  Casos de Uso                   │  │   │  ← regras de negócio DA APLICAÇÃO
        │  │  │  (Application Business Rules)   │  │   │
        │  │  │  ┌───────────────────────────┐  │  │   │
        │  │  │  │  Entidades                │  │  │   │  ← Regras Cruciais de Negócio
        │  │  │  │  (Enterprise Bus. Rules)  │  │  │   │     (o mais interno, mais alto nível)
        │  │  │  └───────────────────────────┘  │  │   │
        │  │  └─────────────────────────────────┘  │   │
        │  └───────────────────────────────────────┘   │
        └─────────────────────────────────────────────┘
                 Dependências de código-fonte → SEMPRE apontam para DENTRO
```

| Camada | Conteúdo | Conhece... |
|---|---|---|
| **Entidades** | Regras Cruciais de Negócio da empresa; objetos de negócio mais gerais e estáveis | nada externo |
| **Casos de Uso** | regras específicas da aplicação; orquestram o fluxo de/para Entidades | só Entidades |
| **Adaptadores de Interface** | convertem dados entre o formato dos casos de uso/entidades e o formato de agentes externos; contém MVC (Presenters, Views, Controllers), Gateways. Todo o SQL fica restrito aqui. | camadas internas |
| **Frameworks & Drivers** | BD, framework web, ferramentas; código de "cola". **A web é um detalhe. O BD é um detalhe.** | a próxima camada interna |

> **Não precisam ser exatamente quatro círculos.** A regra inviolável é a **Regra da Dependência**.

#### A REGRA DA DEPENDÊNCIA (a regra primordial)
> **As dependências de código-fonte devem apontar apenas para dentro, na direção das políticas de nível mais alto.**

- Nada num círculo interno pode **saber** de nada num círculo externo: nenhum nome (função, classe, variável) declarado externamente pode ser mencionado por código interno.
- **Formatos de dados** declarados externamente (ex.: estruturas geradas por um framework/ORM, "row structures") **não** podem ser usados internamente.

#### Cruzando os limites (como inverter o controle)
Quando o fluxo de controle precisa ir de dentro para fora (ex.: o caso de uso precisa entregar resultado ao Presenter), você **não** chama o externo diretamente (violaria a Regra da Dependência). Em vez disso:
1. O caso de uso chama uma **interface** definida no círculo interno (ex.: "Output Boundary" / "porta de output do caso de uso").
2. O componente externo (Presenter) **implementa** essa interface.
Assim você usa **polimorfismo dinâmico** para criar uma dependência de código-fonte que se **opõe** ao fluxo de controle. Vale para qualquer travessia de limite, em qualquer direção.

#### Quais dados cruzam os limites
- Apenas **estruturas de dados simples e isoladas** (structs básicas, DTOs, ou argumentos de função).
- **Nunca** passe objetos Entidade nem registros/row structures do banco através do limite.
- Os dados devem estar sempre na forma **mais conveniente para o círculo interno**.

#### Cenário típico (Java/web/BD) — o fluxo completo
1. O servidor web entrega a entrada ao **Controller**.
2. O Controller empacota num objeto Java simples e o passa via **InputBoundary** ao **UseCaseInteractor**.
3. O Interactor interpreta os dados, controla a **dança das Entities** e usa a **DataAccessInterface** (gateway) para carregar/salvar dados no **Database**.
4. O Interactor monta o **OutputData** (objeto simples) e o passa via **OutputBoundary** ao **Presenter**.
5. O **Presenter** reempacota em um **ViewModel** (Strings, flags, valores já formatados — ex.: `Date`→String, `Currency` com casas decimais, flag booleana para número negativo em vermelho, nomes de botões, flags de "acinzentar").
6. A **View** apenas move o ViewModel para a tela/HTML — não processa nada.

**Todas as dependências cruzam os limites apontando para dentro.**

### 5.8 Apresentadores e Objetos Humble (Humble Object Pattern)
**Padrão de Objeto Humble:** separe comportamentos **difíceis de testar** dos **fáceis de testar** em dois módulos. O "humble" contém o mínimo possível, difícil de testar; o testável contém a lógica.

**Aplicação clássica — Presenter/View:**
- **View** = objeto **humble**, o mais simples possível. Só move dados do ViewModel para a GUI. Não processa.
- **Presenter** = objeto **testável**. Recebe dados da aplicação e os formata no **ViewModel** (tudo vira String/boolean/enum). Toda decisão de apresentação fica aqui.

**A separação testável/não-testável quase sempre coincide com um limite arquitetural.** Outros exemplos de Humble Object:
- **Gateways de banco de dados:** interfaces polimórficas com métodos CRUD específicos da aplicação (ex.: `getLastNamesOfUsersWhoLoggedInAfter(Date)`). **Sem SQL na camada de casos de uso.** A implementação concreta (com SQL) na camada de BD é o objeto humble; os interactors não são humble (contêm regras), mas são testáveis porque os gateways podem ser **mockados**.
- **Mapeadores de dados (ORM):** o ORM pertence à camada de BD (entre os Gateways e o BD), não ao núcleo.
- **Service Listeners:** convertem dados do/para o formato de serviços externos.

> Testabilidade é atributo de boa arquitetura. O padrão Humble Object define onde traçar muitos limites.

### 5.9 Limites parciais, camadas e fronteiras
Limites arquiteturais completos são **caros** (interfaces de input/output recíprocas, structs, gestão de dependências em ambas as direções). Quando o custo total não se justifica agora mas você quer preservar a opção, use **limites parciais**:
- **Pule o último passo:** faça todo o trabalho de um limite completo, mas mantenha tudo no mesmo componente (não separe em dois componentes implantáveis).
- **Limites unidimensionais:** uma única interface apontando numa direção (ex.: Strategy), mais barato que o limite recíproco completo.
- **Fachadas (Facades):** ainda mais simples — uma classe Facade lista os serviços e delega; o cliente não acessa as classes de serviço diretamente, mas há acoplamento transitivo.

> **Onde traçar limites:** entre coisas que importam (regras de negócio) e coisas que **não** importam (UI, BD, frameworks) — o **eixo da mudança** (SRP). Não trace todos os limites possíveis prematuramente (over-engineering), nem ignore-os (sub-engineering). É uma decisão a ser **reavaliada** ao longo do tempo; trace quando o custo de não tê-los superar o custo de tê-los.

### 5.10 O componente Main
> **`Main` é o detalhe final — a política de nível mais baixo, o ponto de entrada.** Nada além do SO depende dele. É **o mais sujo dos componentes sujos.**

- Cria todas as **Factories, Strategies e utilitários globais**, **injeta dependências** (é aqui que o framework de DI atua) e então **entrega o controle** às porções abstratas de alto nível.
- Carrega strings/configurações que o núcleo não deve conhecer.
- **Pense em `Main` como um plug-in da aplicação.** Pode haver **vários `Main`** — um por configuração (Dev/Teste/Produção, por país/cliente/local). Isso torna a configuração trivial de gerenciar.

### 5.11 Serviços, microsserviços e limites
- Serviços **parecem** desacoplados e independentes, mas isso é só **parcialmente** verdade. Eles compartilham dados e podem ficar fortemente acoplados ("o problema do gato": adicionar um comportamento transversal pode forçar mudanças em múltiplos serviços).
- **Limites arquiteturais NÃO caem necessariamente entre serviços.** Frequentemente eles **atravessam** os serviços, dividindo-os em **componentes** internos.
- Projete cada serviço com design de componentes interno conforme **SOLID** e a **Regra da Dependência**, permitindo adicionar recursos como **novas classes/jars** (OCP) sem reimplantar o serviço.
- **A arquitetura é definida pelos limites e pelas dependências que os cruzam — não pelos mecanismos físicos de comunicação/execução.** Microsserviços/SOA ajudam em escalabilidade e desenvolvimento, mas não são, por si, arquiteturalmente significantes. Trate-os como um modo de **desacoplamento** (que você pode adotar ou regredir).

### 5.12 O limite de teste (testes fazem parte da arquitetura)
- Do ponto de vista arquitetural, **todos os testes são iguais** (de unidade a Cucumber/FitNesse): seguem a Regra da Dependência e são o **círculo mais externo**. Nada no sistema depende dos testes; os testes dependem do sistema. São o componente mais isolado e independentemente implantável.
- **Problema dos Testes Frágeis:** testes acoplados à estrutura do sistema quebram a cada mudança trivial, tornando o sistema **rígido** (devs evitam mudar para não quebrar 1000 testes). Causa: **acoplamento**, sobretudo o **acoplamento estrutural** (uma classe de teste por classe de produção, um método de teste por método de produção).
- **Primeira regra do design (vale para testes também): não dependa de coisas voláteis.** GUIs são voláteis → não teste regras de negócio através da GUI.
- **API de Teste:** crie uma API específica para os testes verificarem regras de negócio, com "superpoderes" (contornar segurança, evitar BD, forçar estados testáveis). Ela **desacopla a estrutura dos testes da estrutura da aplicação**, permitindo que código de produção (que tende a ficar mais abstrato/geral) e testes (que tendem a ficar mais concretos/específicos) evoluam separadamente. Se os superpoderes forem perigosos em produção, mantenha a API de teste num componente separado e independentemente implantável.

---

## 6. DETALHES (tudo isto é plugável — mantenha fora do núcleo)

### 6.1 O banco de dados é um detalhe
- O **modelo de dados** é arquiteturalmente significante; o **SGBD** (Oracle, MySQL, Mongo, etc.) é um **detalhe** — uma utilidade para mover bytes entre disco e RAM. Não deixe esse detalhe poluir a arquitetura.
- SGBDs relacionais são ótimos, mas a forma tabular **não** deve vazar para o núcleo. Mantenha o SQL e o esquema restritos à camada externa, atrás de **gateways**.
- Performance é uma preocupação **isolável** (encapsule consultas/caches/índices atrás de interfaces). Não justifique acoplar o núcleo ao BD em nome de desempenho.

### 6.2 A web é um detalhe
- A web é apenas mais um dispositivo de IO. O "pêndulo" computação-no-cliente vs. computação-no-servidor oscila sem parar — não amarre suas regras de negócio à moda atual.
- Mantenha a lógica de negócio ignorante da web; a UI web é um detalhe atrás dos adaptadores de interface.

### 6.3 Frameworks são detalhes — NÃO case com o framework
**Casamento assimétrico:** você se compromete totalmente com o framework, mas o autor dele não se compromete com você. A documentação te empurra a acoplar fortemente (herdar das classes-base do framework, importar utilitários nos objetos de negócio).

**Riscos de acoplar o núcleo a um framework:**
- A arquitetura do framework muitas vezes não é limpa e **viola a Regra da Dependência** (ele quer entrar no círculo mais interno — nas suas Entidades).
- Seu produto pode **superar** o framework com o tempo.
- O framework pode **evoluir numa direção inútil** para você; atualizações dolorosas.
- Pode surgir um framework melhor.

**A solução: namore, não case.**
- Trate o framework como um **detalhe** nos círculos externos. Não o deixe entrar no núcleo.
- Se ele pede que você derive seus objetos de negócio das classes-base dele, **diga não**. Crie **proxies/adaptadores** em componentes que sejam **plug-ins** das regras de negócio.
- **Exemplo (Spring):** use-o para DI, mas **não** espalhe `@Autowired`/anotações pelos objetos de negócio. Faça a injeção apenas no componente **`Main`** (que pode conhecer o Spring, por ser o mais sujo).
- Alguns frameworks você **precisa** casar (STL em C++, biblioteca padrão do Java) — tudo bem, mas que seja uma **decisão consciente** (compromisso para o resto da vida da aplicação).

---

## 7. ORGANIZAÇÃO DE CÓDIGO NA PRÁTICA (o "Capítulo Perdido")

> **O diabo está nos detalhes de implementação.** Um ótimo design morre se a estrutura de código e a visibilidade não forem cuidadas. Use o **compilador/sistema de tipos/módulos** para impor a arquitetura — não confie só em disciplina e revisão.

### 7.1 Quatro formas de organizar o código (ranqueadas)

**a) Pacote por Camada (Package by Layer) — evite como destino.**
Fatias horizontais técnicas: `web` / `business logic` / `persistence` (ex.: `OrdersController`, `OrdersService`, `OrdersRepository`). Bom só para *começar*. Problemas: não diz nada sobre o **domínio** (dois sistemas diferentes ficam idênticos); três "baldes" não escalam; e permite **camadas relaxadas** (controller pulando o service e acessando o repository direto → "grande bola de lama").

**b) Pacote por Recurso (Package by Feature) — melhor, mas não ótimo.**
Fatias verticais por conceito de domínio: tudo de "orders" num pacote. A estrutura passa a **gritar o domínio**; fica mais fácil achar o código de um caso de uso. Mas ainda não impõe a separação interna.

**c) Portas e Adaptadores (Ports & Adapters).**
Um "**dentro**" (domínio, independente de tecnologia) e um "**fora**" (infra: UI, BD, integrações). **Regra: o fora depende do dentro, nunca o contrário.** Nomeie tudo no "dentro" na **linguagem ubíqua do domínio** (ex.: a interface chama-se `Orders`, não `OrdersRepository`). Cuidado com o **"antipadrão périphérique"**: se toda a infra está numa só árvore de código, um adaptador (controller web) pode chamar outro (repository) **sem passar pelo domínio** — especialmente se você esquecer os modificadores de acesso.

**d) Pacote por Componente (Package by Component) — recomendado por Simon Brown.**
Agrupa **toda** a funcionalidade relacionada a um componente de alta granularidade (lógica de negócio **+** persistência) atrás de **uma interface limpa** num único pacote (ex.: `OrdersComponent`). A separação de preocupações é mantida **dentro** do componente, como detalhe de implementação que os consumidores não veem. A UI fica separada (como em Ports & Adapters / microsserviços). **Benefício-chave:** ao escrever código sobre "orders", há **uma única opção** — o `OrdersComponent`. É um degrau natural rumo a microsserviços.

### 7.2 Organização vs. Encapsulamento — pare de usar `public` por reflexo
- Se **todos** os tipos são `public`, os pacotes viram só **organização** (pastas), não **encapsulamento**. Aí o estilo arquitetural escolhido é **irrelevante** — as quatro abordagens acima ficam **sintaticamente idênticas** (todas as setas iguais), equivalendo a uma arquitetura em camadas tradicional.
- **Use a visibilidade mais restrita possível:**
  - **Java:** deixe as **interfaces de entrada** (as que têm dependências de fora do pacote) como `public`; deixe **implementações** (ex.: `OrdersServiceImpl`, `JdbcOrdersRepository`) **package-private** (protegidas por pacote). No "pacote por componente", só `OrdersComponent` é `public`; todo o resto package-private → o compilador **impede** acesso indevido ao repositório/implementação.
  - **.NET:** use `internal` e crie uma **assembly separada por componente**.
  - **Java 9+ / OSGi:** distinga tipos `public` de tipos **publicados** (exporte só um subconjunto do módulo).
- **Não trapaceie com reflection** para furar a visibilidade.

### 7.3 Outros modos de desacoplamento de código-fonte
- Separe o código em **árvores de código-fonte / módulos de build distintos** (Maven/Gradle/MSBuild). Ex. para Ports & Adapters:
  - **Domínio** (independente de tecnologia): `OrdersService`, `OrdersServiceImpl`, `Orders`.
  - **Web**: `OrdersController`.
  - **Persistência**: `JdbcOrdersRepository`.
  - Web e persistência têm dependência de **tempo de compilação** sobre o domínio; o domínio não sabe nada deles.
- Ideal: uma árvore por componente. Pragmático: às vezes só duas (domínio = "dentro", infraestrutura = "fora") — mas atenção ao antipadrão périphérique.

### 7.4 A recomendação perdida (resumo prático)
Ao mapear o design em código: pense em **como organizar o código**, **quais modos de desacoplamento** aplicar (compilação vs. execução), **deixe opções em aberto** onde fizer sentido, mas **seja pragmático** — considere tamanho/habilidade da equipe, complexidade, tempo e orçamento. **Use o compilador para impor o estilo** e fique atento a acoplamentos em outras áreas (ex.: modelos de dados).

---

## 8. REFERÊNCIA RÁPIDA PARA CODAR (cheat sheet acionável)

### 8.1 Regras invioláveis (sempre aplicar)
1. **Regra da Dependência:** dependências de código-fonte apontam **só para dentro** (rumo à política de alto nível). Núcleo nunca menciona nome de UI/BD/framework.
2. **Separe política de detalhe.** Regras de negócio (Entidades + Casos de Uso) no centro; UI/BD/web/frameworks como plug-ins externos.
3. **Inverta dependências com interfaces** sempre que o fluxo de controle precise ir de dentro para fora. A interface pertence ao lado interno; a implementação fica no lado externo.
4. **DTOs/structs simples cruzam limites.** Nunca passe Entidades nem registros do BD através de um limite.
5. **Nada de SQL/HTML/serialização de framework** dentro de Casos de Uso ou Entidades.
6. **Visibilidade mínima:** restrinja com package-private/`internal`/módulos. `public` só para o que precisa ser visto de fora.
7. **`Main` é o lugar dos detalhes sujos:** instanciação de concretos, configuração e DI.
8. **Mantenha decisões em aberto:** não acople cedo a BD/framework/UI específicos.

### 8.2 Checklist de design (antes de escrever um módulo)
- [ ] A que **camada** este módulo pertence? (Entidade / Caso de Uso / Adaptador / Framework)
- [ ] Quem é o **ator** (única razão de mudança)? Há mais de um misturado aqui? (SRP)
- [ ] As **dependências apontam para dentro**? Há alguma seta apontando para fora ou para cima? (Regra da Dependência / SDP)
- [ ] Existe um **eixo de variação** previsível? Se sim, há uma **abstração** para estendê-lo sem modificar o existente? (OCP)
- [ ] As **interfaces** são pequenas e focadas no papel do cliente? (ISP)
- [ ] Toda implementação **honra plenamente o contrato** da abstração? (LSP)
- [ ] Este módulo depende só de **abstrações** (não de concretos voláteis)? (DIP)
- [ ] É **testável sem** UI/BD/web/framework?
- [ ] Há **ciclos** de dependência? (ADP — quebre com DIP ou novo componente)
- [ ] Estou criando **duplicação acidental** (coisas que parecem iguais mas mudam por razões diferentes)? Se sim, **não unifique**.

### 8.3 Red flags (sinais de violação — pare e refatore)
- 🚩 `import`/`require`/`using` de um framework, ORM ou tipo de BD **dentro** de uma Entidade ou Caso de Uso.
- 🚩 `if (x is ConcreteTypeA) ... else if (x is ConcreteTypeB)` espalhado → falta polimorfismo (OCP/LSP).
- 🚩 Nome de tecnologia no código de negócio (`acme`, `purplecab`, `oracle`, `HttpRequest` no caso de uso).
- 🚩 Uma classe com métodos pedidos por **departamentos/atores diferentes** (SRP).
- 🚩 Setas de dependência apontando **para cima** num diagrama (instável → estável invertido) (SDP).
- 🚩 Objeto Entidade ou row/record do BD **atravessando um limite**.
- 🚩 SQL/HTML em camada interna.
- 🚩 Tudo marcado `public` (encapsulamento perdido; pacotes viram só pastas).
- 🚩 Controller acessando Repository **direto**, pulando a regra de negócio (camadas relaxadas).
- 🚩 Teste que navega pela **GUI** para verificar regra de negócio (testes frágeis).
- 🚩 Uma classe de teste por classe de produção + um método de teste por método (acoplamento estrutural).
- 🚩 Anotações de framework (`@Autowired`, `@Entity` de ORM) espalhadas pelos objetos de negócio.
- 🚩 "Vamos limpar depois" / "vamos reescrever do zero" como plano.
- 🚩 Generalidade especulativa: abstrações sem implementação/uso real (Zona da Inutilidade).
- 🚩 Componente concreto, estável e volátil ao mesmo tempo (Zona da Dor) — ex.: esquema de BD acoplado.

### 8.4 Heurísticas de decisão
- **"Devo criar uma interface aqui?"** → Sim, se: (a) o tipo concreto é **volátil** e você quer proteger o cliente; (b) precisa **inverter** uma dependência para cumprir a Regra da Dependência; (c) há **mais de uma implementação** real ou prevista. Não, se o concreto é **estável** (ex.: `String`, lib padrão) e não há eixo de variação.
- **"Monólito, modular ou microsserviços?"** → Comece com **monólito bem componentizado** (desacoplamento em nível de código-fonte). Promova para deploy/serviço só quando houver razão concreta (escala, equipes independentes). Saiba **regredir** se o custo de serviço não compensar. Serviço é **modo de desacoplamento**, não arquitetura.
- **"Eliminar esta duplicação?"** → Só se for **duplicação verdadeira** (muda sempre junta pela mesma razão). Se as cópias **evoluem por razões diferentes**, mantenha separadas.
- **"Onde colocar esta classe (qual componente)?"** → CCP (muda junto com quê?), CRP (é reutilizada junto com quê?), e evite ciclos (ADP). A estrutura **evolui** — reavalie.
- **"Quão abstrato/estável este componente deve ser?"** → Quanto mais **dependido** (estável), mais **abstrato** (SAP). Núcleo de políticas: estável + abstrato (I≈0, A≈1). Detalhes: instável + concreto (I≈1, A≈0). Fique perto da Sequência Principal.
- **"Adotar este framework?"** → Namore primeiro. Mantenha-o atrás de um limite, como plug-in. Não derive objetos de negócio das classes-base dele.

### 8.5 Receita de implementação de um caso de uso (template mental)
1. **Entidade(s):** objetos com Regras Cruciais de Negócio + dados, sem dependências externas.
2. **Caso de Uso (Interactor):** orquestra Entidades; recebe **RequestModel** (struct simples), devolve **ResponseModel** (struct simples).
3. **Input Boundary:** interface que o Controller usa para invocar o Interactor.
4. **Output Boundary:** interface que o Interactor usa para entregar resultado ao Presenter (inversão de dependência).
5. **DataAccess Gateway:** interface (definida no lado interno) para persistência; implementação concreta (SQL/ORM) na camada externa.
6. **Controller:** converte entrada externa em RequestModel.
7. **Presenter:** converte ResponseModel em **ViewModel** (Strings/flags formatadas).
8. **View:** humble — só desenha o ViewModel.
9. **Main:** instancia tudo, injeta as implementações concretas nas interfaces, entrega o controle.

---

## 9. GLOSSÁRIO E SIGLAS

**SOLID (classes/módulos):**
- **SRP** — Single Responsibility Principle (uma razão para mudar = um ator).
- **OCP** — Open-Closed Principle (aberto a extensão, fechado a modificação).
- **LSP** — Liskov Substitution Principle (subtipos substituíveis; contrato honrado).
- **ISP** — Interface Segregation Principle (não depender do que não usa).
- **DIP** — Dependency Inversion Principle (depender de abstrações, não de concretos voláteis).

**Coesão de componentes:**
- **REP** — Reuse/Release Equivalence Principle (granularidade do reúso = do release).
- **CCP** — Common Closure Principle (junte o que muda junto; SRP de componente).
- **CRP** — Common Reuse Principle (não force a depender do que não se usa; ISP de componente).

**Acoplamento de componentes:**
- **ADP** — Acyclic Dependencies Principle (sem ciclos; DAG).
- **SDP** — Stable Dependencies Principle (dependa rumo à estabilidade; I decrescente).
- **SAP** — Stable Abstractions Principle (estável ⇒ abstrato).

**Métricas:**
- **Fan-in** — dependências que chegam. **Fan-out** — dependências que saem.
- **I (Instabilidade)** = Fan-out / (Fan-in + Fan-out). 0 = estável, 1 = instável.
- **A (Abstração)** = Na / Nc (interfaces+abstratas / total de classes). 0 = concreto, 1 = abstrato.
- **D (Distância da Sequência Principal)** = |A + I − 1|. 0 = ideal.
- **Zona da Dor** ≈ (0,0): concreto+estável+rígido. **Zona da Inutilidade** ≈ (1,1): abstrato sem uso.
- **Sequência Principal** — linha (1,0)–(0,1); posição saudável.

**Conceitos arquiteturais:**
- **Regra da Dependência** — dependências de código-fonte apontam só para dentro (rumo à política de alto nível).
- **Política** — regras de negócio (o valor). **Detalhe** — IO/BD/web/framework (plugável).
- **Entidade** — Regras Cruciais de Negócio da empresa (nível mais alto).
- **Caso de Uso (Interactor)** — regras específicas da aplicação; orquestra Entidades.
- **Adaptadores de Interface** — Controllers, Presenters, Gateways; conversão de formato.
- **Frameworks & Drivers** — camada externa de detalhes.
- **Humble Object** — separa o difícil-de-testar (humble) do testável.
- **Boundary (Limite)** — fronteira entre o que importa e o que não importa (eixo da mudança).
- **Input/Output Boundary** — interfaces para entrar/sair de um caso de uso.
- **Request/Response Model** — structs simples que cruzam limites.
- **ViewModel** — dados já formatados (Strings/flags) que a View só exibe.
- **Main** — componente mais sujo; cria concretos, faz DI, entrega controle; é um plug-in.
- **Plug-in** — componente de baixo nível conectado à política de alto nível.
- **Ports & Adapters / Hexagonal** — "dentro" (domínio) vs. "fora" (infra); fora depende do dentro.
- **Síndrome da manhã seguinte** — quebra causada por ciclos/integração tardia.
- **Problema dos Testes Frágeis** — testes acoplados à estrutura quebram a cada mudança.
- **Casamento assimétrico** — acoplar-se demais a um framework que não se compromete com você.
- **Duplicação verdadeira vs. acidental** — eliminar a primeira; preservar a segunda.

---

## 10. MANTRAS PARA LEMBRAR
- *A única maneira de ir rápido é ir bem.*
- *Um bom arquiteto maximiza o número de decisões NÃO tomadas.*
- *Dependa na direção da estabilidade e da abstração.*
- *O banco de dados é um detalhe. A web é um detalhe. O framework é um detalhe.*
- *Não case com o framework — namore.*
- *As regras de negócio são as joias da família: mantenha-as puras e independentes.*
- *Separe o que muda por razões diferentes; junte o que muda pelas mesmas razões.*
- *Não dependa de coisas que você não precisa.*
- *Não dependa de coisas voláteis.*
- *Use o compilador para impor sua arquitetura.*
- *Os testes mostram a presença, não a ausência, de bugs.*
- *O diabo está nos detalhes de implementação.*
