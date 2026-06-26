# Código Limpo (Clean Code) — Guia de Referência para Codar

> **Fonte:** *Código Limpo: Habilidades Práticas do Agile Software* — Robert C. Martin (Uncle Bob) e colaboradores (Michael Feathers, Jeff Langr, Brett L. Schuchert, James Grenning), 2008.
> **Propósito deste arquivo:** base de conhecimento permanente para o Claude Code. Ao escrever, refatorar ou revisar **qualquer** trecho de código, aplique as regras, padrões e heurísticas abaixo. Este documento é prescritivo: trate-o como diretrizes ativas de engenharia no nível do código (linhas, funções, classes), complementando o guia de Arquitetura Limpa (que cobre o nível dos componentes e do sistema).

---

## COMO USAR ESTE DOCUMENTO (instruções para o assistente de código)

1. **Toda vez que escrever código**, aplique por padrão: nomes reveladores de intenção, funções pequenas que fazem uma coisa, sem duplicação, sem comentários que poderiam ser código, e tratamento de erro via exceções (não códigos de erro nem `null`).
2. **A Regra do Escoteiro é permanente:** *deixe o código mais limpo do que você o encontrou*. Em toda edição, faça pelo menos uma pequena melhoria (renomear uma variável, extrair uma função, eliminar uma duplicação, achatar um `if` aninhado).
3. **Prefira clareza a esperteza.** Código é lido muito mais vezes do que é escrito; otimize para o leitor (que provavelmente será você no futuro).
4. **Ao revisar/refatorar**, percorra a checklist de "Odores e Heurísticas" (seção 15) — ela é a referência canônica de sinais de código ruim, com códigos (C, E, F, G, J, N, T) que você pode citar.
5. **Fundamente decisões nos princípios nomeados** (SRP, DRY, Lei de Deméter, Separação Comando-Consulta, F.I.R.S.T, 4 regras do design simples) para que sejam rastreáveis.
6. **Não invente comentários nem documentação supérflua.** O melhor comentário é aquele que você não precisou escrever porque o código já se explica.
7. Estes princípios são independentes de linguagem. Os exemplos do livro são em Java; aplique o espírito à linguagem em uso (Python, JS/TS, Go, C#, etc.).

---

## 1. FILOSOFIA CENTRAL — O QUE É E POR QUE IMPORTA

### 1.1 O custo de um código ruim
Código ruim atrasa todo mundo. Equipes que começam rápido podem, em um ou dois anos, andar "a passos de tartaruga": cada mudança quebra duas ou três outras partes, nenhuma alteração é trivial. Conforme a bagunça cresce, **a produtividade tende assintoticamente a zero**. Adicionar gente ao projeto só piora (novos membros não conhecem o design e produzem mais bagunça sob pressão). Empresas já faliram por causa de código ruim.

### 1.2 A maior mentira e o Grande Replanejamento
- A mentira: *"vamos limpar depois"* (a **Lei de LeBlanc: "Depois nunca chega" / "Nunca é tarde"**). A pressão do mercado nunca diminui, então "depois" nunca vem.
- O Grande Replanejamento (reescrever do zero) raramente funciona: vira uma corrida de 10 anos entre o time da bagunça antiga e o time do sistema novo — que, quando termina, **já é uma nova bagunça**.

### 1.3 A única maneira de ir rápido
> **A única forma de cumprir prazos — a única maneira de ir rápido — é manter o código sempre o mais limpo possível.**

Fazer bagunça **reduz** a velocidade imediatamente e faz você perder o prazo. Manter limpo é uma questão de **sobrevivência profissional**, não estética.

### 1.4 Atitude e profissionalismo
A bagunça é responsabilidade nossa, não dos requisitos, prazos ou gerentes. Somos profissionais: gerentes e clientes contam conosco para as informações técnicas. **Assim como um médico não para de lavar as mãos porque o paciente pediu**, um programador não cede à pressão de gerar código sujo. A sua função é proteger o código com a mesma paixão com que o gerente protege o prazo.

### 1.5 O que é código limpo (visões dos mestres)
Coletânea de definições citadas no livro, úteis como bússola:
- **Bjarne Stroustrup:** elegante e eficiente; lógica direta para o bug não se esconder; dependências mínimas para facilitar manutenção; tratamento de erro completo; desempenho próximo do ótimo; **faz bem uma coisa**.
- **Grady Booch:** legível, como prosa bem escrita; deve conter **abstrações nítidas** e linhas diretas de controle.
- **Dave Thomas:** pode ser lido e melhorado por outro dev; tem testes; nomes significativos; **mínimo de dependências, explícitas**; API clara e mínima. Código deve ser *literário*.
- **Michael Feathers:** parece ter sido escrito por **alguém que se importa**; nada óbvio a melhorar.
- **Ron Jeffries:** sem duplicação; **expressa todas as ideias de design** do sistema; mínimo de entidades (classes, métodos, funções); nomes expressivos.
- **Ward Cunningham:** você sabe que está limpo quando **cada rotina que você lê é basicamente o que você esperava**. O código faz a linguagem parecer ter sido feita para o problema.

### 1.6 A Regra do Escoteiro (princípio operacional)
> **Deixe a área do acampamento mais limpa do que como você a encontrou.**

Se cada um deixar o código um pouco mais limpo a cada commit, ele não degrada — melhora com o tempo. A limpeza pode ser pequena: um nome melhor, uma função grande dividida, uma repetição eliminada, um `if` aninhado reduzido.

### 1.7 "Sensibilidade ao código" (code-sense)
Saber distinguir código bom de ruim não é o mesmo que saber **transformar** ruim em bom. O objetivo é desenvolver a disciplina e o repertório de pequenas técnicas (este documento) para, ao ver um módulo confuso, **enxergar alternativas** e a sequência de transformações que o limpam.

---

## 2. NOMES SIGNIFICATIVOS

Nomes estão em todo lugar (variáveis, funções, argumentos, classes, pacotes, arquivos). São ~90% responsáveis pela legibilidade. **Não tenha pressa ao nomear; reavalie nomes conforme o código evolui.**

### 2.1 Use nomes que revelem a intenção
O nome deve responder: por que existe, o que faz, como é usado. Se precisa de um comentário para explicar, o nome falhou.
- Ruim: `int d; // tempo decorrido em dias`
- Bom: `int diasDecorridos`, `int diasDesdeCriacao`, `int tamanhoArquivoEmDias`.
Substitua "números mágicos" e índices crus por nomes: um trecho com `list1`, `x[0]`, `4` é impenetrável; com `quadroPreenchido(quadro)`, `celula`, `BANDEIRA` fica óbvio.

### 2.2 Evite informação errada (disinformation)
- Não use nomes que escondam o sentido. Não chame de `accountList` algo que não é uma `List` (use `accounts` ou `accountGroup`).
- Evite nomes que variam minimamente (`XYZControllerForEfficientHandlingOfStrings` vs `...StorageOfStrings`).
- Cuidado com letras que parecem números: `l`, `O`, `1`, `0`.

### 2.3 Faça distinções significativas
- Não resolva colisões com ruído: `a1, a2, ... aN` ou `aZork` porque `zork` já existe.
- **Palavras de ruído são redundantes:** `Info`, `Data`, `Object`, `the`, `a`, `variable`, `table`. `NameString` não é melhor que `Name`. `Customer` vs `CustomerObject` não diz nada. `getActiveAccount()`, `getActiveAccounts()`, `getActiveAccountInfo()` — como escolher? Faça distinções que **o leitor compreenda**.

### 2.4 Use nomes pronunciáveis e pesquisáveis
- **Pronunciáveis:** `genymdhms` é impronunciável; prefira `dataGeracao`. Programação é atividade social — você precisa conversar sobre os nomes.
- **Pesquisáveis:** nomes de uma letra e números mágicos não são "grepáveis". Use nomes/constantes com nome para que possam ser localizados. **O tamanho do nome deve ser proporcional ao tamanho do escopo:** `i` num laço de 3 linhas é ótimo; em escopo grande, use nomes longos e precisos.

### 2.5 Evite codificações (notação húngara, prefixos de membro)
- Ambientes modernos tornam supérfluas as codificações de tipo/escopo. **Não use** prefixos `m_`, `f`, ou notação húngara (`strNome`, `iContador`). Não codifique subsistemas (`siv...`). Mantenha os nomes livres de "poluição húngara".
- Não prefixe interfaces com `I`; se precisar codificar, prefira codificar a **implementação** (`...Imp`) a poluir a interface.

### 2.6 Nomes de classes, métodos e conceitos
- **Classes/objetos:** substantivos ou frases nominais — `Cliente`, `PaginaWiki`, `Conta`, `AnaliseEndereco`. **Evite** `Gerente`, `Processador`, `Dados`, `Info`, `Super` (palavras vagas atraem responsabilidades demais). Não use verbo.
- **Métodos:** verbos ou frases verbais — `postarPagamento`, `excluirPagina`, `salvar`. Acessores/mutadores/predicados seguem o padrão JavaBean: `getNome`, `setNome`, `isPostado`.
- **Construtores sobrecarregados → métodos fábrica estáticos** com nomes descritivos: `Complex.fromRealNumber(23.0)` é melhor que `new Complex(23.0)`. Torne o construtor privado para forçar o uso.
- **Não dê uma de espertinho:** evite piadas, gírias e coloquialismos (`HolyHandGrenade` → `DeleteItems`; `comerFora()` → `abortar()`). **Clareza acima de diversão.**
- **Uma palavra por conceito:** escolha um termo e mantenha-o. Não misture `pegar`/`recuperar`/`obter`/`get` como equivalentes em classes diferentes. Não tenha `controlador`, `gerenciador` e `driver` para a mesma ideia. Um léxico consistente é uma grande vantagem.
- **Não faça trocadilhos:** não use a mesma palavra para dois propósitos. Se `add` em toda a base significa "somar/concatenar", não chame de `add` um método que só insere um item numa coleção — use `inserir`/`adicionar`.

### 2.7 Domínio da solução vs. domínio do problema
- Use termos **do domínio da solução** (CS, algoritmos, padrões, matemática) quando apropriado — quem lê é programador (`AccountVisitor` para quem conhece o padrão Visitor, `JobQueue`).
- Use termos **do domínio do problema** quando não houver termo técnico — assim quem mantém pode perguntar a um especialista. Separar os dois domínios é parte do trabalho de design.

### 2.8 Adicione contexto significativo
- Variáveis soltas como `state` não dizem que fazem parte de um endereço. Dê contexto colocando-as em **classes/funções/namespaces bem nomeados** (ex.: classe `Endereco`). Só como último recurso, use prefixos (`addrState`).
- Não adicione contexto **gratuito**: numa app "Gas Station Deluxe", não prefixe tudo com `GSD`. Nomes devem ser claros, não mais longos que o necessário.

---

## 3. FUNÇÕES

O capítulo mais importante para o nível do código. Funções são a primeira linha de organização.

### 3.1 Pequenas!
> A **primeira** regra das funções é: devem ser pequenas. A **segunda**: devem ser **menores ainda**.
- Alvo prático: **~20 linhas no máximo**, idealmente bem menos (2–5 linhas é excelente).
- **Blocos e endentação:** o corpo de `if`/`else`/`while` deve ter **uma linha** — geralmente uma chamada de função (que recebe um nome descritivo). O **nível de endentação** de uma função deve ser **1 ou 2, no máximo**. Sem estruturas profundamente aninhadas.

### 3.2 Faça apenas uma coisa
> **As funções devem fazer uma coisa. Devem fazê-la bem. Devem fazer apenas ela.**
- Uma função faz "uma coisa" se todos os seus passos estão **um nível de abstração abaixo** do nome dela (descrevível como um parágrafo "PARA [nome], fazemos X, depois Y...").
- **Teste decisivo:** se você consegue **extrair outra função** dela com um nome que **não seja apenas uma reformulação** da implementação, então ela fazia mais de uma coisa.
- **Seções dentro de uma função** (declarações / inicialização / seleção) são sinal de que ela faz mais de uma coisa.

### 3.3 Um nível de abstração por função
Misturar níveis de abstração (ex.: `getHtml()` de alto nível ao lado de `.append("\n")` de baixo nível) confunde: o leitor não distingue conceito essencial de detalhe. E, como "janelas quebradas", mistura atrai mais mistura.

### 3.4 Regra decrescente (Stepdown Rule)
O código deve ser lido **de cima para baixo**, como uma narrativa: cada função é seguida pelas do próximo nível de abstração abaixo. "Para fazer A, fazemos B e C. Para fazer B, fazemos D...". Manter isso é o segredo para funções curtas e com um só nível de abstração.

### 3.5 Estruturas `switch` (e `if/else` em cadeia)
- Um `switch` faz inerentemente **N coisas** e viola SRP e OCP (cresce a cada novo tipo; multiplica-se em outras funções com a mesma estrutura).
- **Regra:** tolere `switch` apenas se aparecer **uma única vez**, para **criar objetos polimórficos**, escondido atrás de uma **Abstract Factory**, de modo que o resto do sistema nunca o veja. Caso contrário, **prefira polimorfismo** (ver heurística G23).

### 3.6 Use nomes descritivos
- Não tema nomes longos: um nome longo e descritivo é melhor que um curto e enigmático, e melhor que um comentário longo. Experimente vários nomes e leia o código com cada um (IDEs facilitam renomear).
- **Seja consistente** na fraseologia: `includeSetupPage`, `includeSetupAndTeardownPages`, `includeSuiteSetupPage` — a sequência é dedutível.

### 3.7 Argumentos de função
Ordem de preferência por aridade: **0 (niládica) > 1 (monádica) > 2 (diádica) > 3 (triádica, evite) > mais (poliádica, precisa de justificativa muito forte e em geral não deve existir).**
- Argumentos custam esforço conceitual e **dificultam testes** (combinações a cobrir crescem).
- **Formas monádicas comuns (ok):** perguntar sobre o argumento (`boolean fileExists("MyFile")`); transformar e retornar (`InputStream fileOpen("MyFile")`); evento (`void passwordAttemptFailedNtimes(int)`). Evite outras formas monádicas.
- **Não use argumentos de saída (output arguments).** O leitor espera info entrando por argumentos e saindo pelo retorno. Se a função transforma algo, retorne o resultado (`StringBuffer transform(StringBuffer in)`, não `void transform(StringBuffer out)`). Para mudar estado, mude o estado **do objeto dono** do método.
- **Não use argumentos lógicos (flag arguments).** Passar `true`/`false` declara que a função faz mais de uma coisa. `render(true)` é confuso → divida em `renderForSuite()` e `renderForSingleTest()`.
- **Díades:** ok quando os dois args são componentes de um valor (`new Point(0, 0)`); ruins quando não têm ordem natural (`assertEquals(expected, actual)` — fácil inverter). Considere transformar em monádica (tornar o objeto membro, extrair classe).
- **Objetos-argumento:** se uma função precisa de muitos args, alguns provavelmente formam um objeto coeso: `Circle makeCircle(double x, double y, double radius)` → `Circle makeCircle(Point center, double radius)`.
- **Listas variádicas** contam como um argumento (`String.format(fmt, ...)` é diádica).
- **Verbos e palavras-chave:** o nome + argumentos deve formar um par verbo/substantivo (`write(name)`); embuta nomes de args no nome (`assertExpectedEqualsActual(expected, actual)`).

### 3.8 Evite efeitos colaterais
> **Efeitos colaterais são mentiras.** A função promete uma coisa, mas faz outra escondida (muda variáveis da classe, dos argumentos, ou globais).
- Ex.: `checkPassword(user, pass)` que também chama `Session.initialize()` — cria **acoplamento temporal** (só pode ser chamada em certos momentos). Se o efeito for inevitável, deixe-o explícito no nome (`checkPasswordAndInitializeSession`).

### 3.9 Separação Comando-Consulta (Command-Query Separation)
Uma função ou **faz** algo (comando, muda estado) **ou responde** algo (consulta, retorna info) — **nunca as duas**. `if (set("username", "bob"))` é ambíguo. Separe: `if (attributeExists("username")) { setAttribute("username", "bob"); ... }`.

### 3.10 Prefira exceções a retornar códigos de erro
- Retornar códigos de erro entope o chamador com verificações imediatas (`if`s aninhados) e é fácil de esquecer. **Lance exceções** — o caminho feliz fica limpo, separado do tratamento de erro.
- **Extraia os corpos de try/catch** para funções próprias. Tratamento de erro **é** uma coisa; uma função que trata erros não deve fazer mais nada (o `try` deve ser a primeira palavra e não deve haver nada após o `catch/finally`).
- (Mais detalhes na seção 8.)

### 3.11 Não se repita (DRY) e Programação Estruturada
- **DRY:** duplicação é a raiz de muitos males; elimine-a. Muitos princípios e padrões existem só para controlar duplicação.
- **Programação estruturada (Dijkstra):** idealmente uma entrada e uma saída — um `return` por função, sem `break`/`continue` no meio de laços, **nunca `goto`**. Em funções pequenas isso raramente atrapalha; múltiplos `return`s em funções minúsculas e expressivas são aceitáveis.

### 3.12 Como escrever funções assim
Você não acerta de primeira. Escreva a função feia e funcional (com testes), depois **refatore**: divida, renomeie, elimine duplicação, reduza args, até que cada função faça uma coisa em um nível de abstração e leia como narrativa.

---

## 4. COMENTÁRIOS

> **Comentários, no melhor caso, são um mal necessário.** Eles compensam nossa falha em nos expressar no código. O comentário verdadeiramente bom é aquele que você **encontrou um jeito de não escrever**.

- **Comentários mentem** com o tempo: o código muda, o comentário não. Comentários antigos viram "ilhas flutuantes de irrelevância" que desinformam.
- **Explique-se no código, não em comentários.** Em vez de `// verifica se elegível a benefícios`, escreva `if (employee.isEligibleForFullBenefits())`. Quase sempre basta criar uma função com nome que diga o que o comentário diria.

### 4.1 Comentários bons (use com parcimônia)
- **Legais:** copyright/licença no topo do arquivo (prefira referenciar um documento externo).
- **Informativos:** explicar um retorno/formato quando o nome não basta (mas prefira renomear). Ex.: explicar o formato de uma regex.
- **Explicação de intenção:** o *porquê* de uma decisão (não o *o quê*).
- **Esclarecimento:** traduzir argumento/retorno obscuro de uma API que você **não pode** alterar (risco: pode ficar incorreto).
- **Alerta de consequências:** ex.: "não thread-safe, crie uma instância por uso"; teste que demora muito.
- **TODO:** tarefas pendentes (`// TODO ...`). IDEs localizam; revise e elimine regularmente. **TODO não justifica deixar código ruim.**
- **Amplificação:** destacar a importância de algo aparentemente trivial (ex.: por que aquele `.trim()` é crucial).
- **Javadoc em APIs públicas:** documentação de API pública é valiosa (mas sujeita aos mesmos riscos de qualquer comentário).

### 4.2 Comentários ruins (a maioria — evite)
- **Murmúrio/redundante/enganoso:** comentar por obrigação, ou descrever o que o código já diz (`i++; // incrementa i`), ou um Javadoc que só repete a assinatura.
- **Comentários obrigatórios por regra** (Javadoc em toda função/variável): geram ruído e desinformação.
- **Diário de mudanças** no topo do arquivo: o controle de versão já faz isso.
- **Ruído:** `// construtor padrão`, `// o dia do mês`.
- **Marcadores de posição banais**, chaves comentadas (`} // end if`) — reescreva/encurte em vez disso.
- **Atribuições/assinaturas** (`/* adicionado por Bob */`): o VCS sabe.
- **Código comentado:** **abominação.** Apodrece, ninguém ousa apagar. **Apague-o** — o VCS lembra.
- **Comentário não-local** (descreve algo distante), **informação demais**, **conexão obscura** (precisa de outro comentário para explicar o comentário), **cabeçalhos de função** (uma função pequena bem nomeada não precisa).

---

## 5. FORMATAÇÃO

A formatação **é comunicação** — a primeira responsabilidade de um profissional. A funcionalidade de hoje muda; o **estilo e a legibilidade** sobrevivem e afetam toda manutenção futura. Escolha um conjunto de regras simples e aplique-o **consistentemente em equipe**; use um **formatador automático**.

### 5.1 Formatação vertical (a "metáfora do jornal")
- Arquivos pequenos são mais fáceis de entender que grandes. **Mire em arquivos curtos** (FitNesse: média ~65 linhas, máximo ~500). Não é regra fixa, mas prefira < 200–500 linhas.
- O arquivo deve se ler como um jornal: **nome no topo**, conceitos de alto nível primeiro, detalhes de baixo nível conforme se desce.
- **Linhas em branco separam conceitos:** entre pacote, imports e cada função/grupo de pensamento. Remover essas linhas obscurece muito a leitura.
- **Densidade vertical:** linhas fortemente relacionadas ficam juntas (sem comentários inúteis no meio).
- **Distância vertical:** declare variáveis **perto do uso**; variáveis locais no topo do seu bloco; variáveis de instância no topo da classe; funções que se chamam ficam **próximas verticalmente**, com a chamadora **acima** da chamada (segue a Regra Decrescente). Conceitos relacionados ficam juntos verticalmente.

### 5.2 Formatação horizontal
- **Linhas curtas:** limite ~100–120 colunas; nunca role horizontalmente.
- **Espaçamento horizontal** para associar/dissociar: espaços ao redor de operadores de baixa precedência, juntos os de alta (`b*b - 4*a*c`); espaço após vírgula.
- **Não alinhe** declarações em colunas artificialmente (atrai o olho para a coluna errada).
- **Endentação** reflete a hierarquia de escopos — nunca a "quebre" para encurtar (evite `if (x) return;` colado; mantenha a estrutura). Endente até blocos/classes vazios de forma visível.

### 5.3 Regras de equipe
Um time deve concordar em **um único estilo** (chaves, endentação, nomes, onde declarar variáveis) e o código deve parecer escrito por **uma só pessoa**. O estilo serve à comunicação, então a consistência vence a preferência individual.

---

## 6. OBJETOS E ESTRUTURAS DE DADOS

### 6.1 Abstração de dados (não só getters/setters)
Esconder a implementação **não é** apenas pôr getters/setters sobre variáveis privadas. É expor **interfaces abstratas** que permitam manipular a *essência* dos dados sem conhecer a implementação.
- Concreto (expõe implementação): `getX()/setX()`, `getFuelTankCapacity()/getGallons()`.
- Abstrato (esconde): `setCartesian/setPolar/getR/getTheta`; `getPercentFuelRemaining()`.
Pense bem na **melhor forma de representar** os dados; adicionar getters/setters cegamente é a pior opção.

### 6.2 A anti-simetria objeto vs. estrutura de dados
> **Objetos** escondem dados atrás de abstrações e expõem **funções** que operam nesses dados. **Estruturas de dados** expõem **dados** e não têm funções significativas. São praticamente opostos.

Consequência fundamental (a "dicotomia"):
- **Código procedimental (estruturas de dados):** fácil adicionar **novas funções** sem mexer nas estruturas existentes; difícil adicionar **novos tipos de dados** (toda função precisa mudar).
- **Código OO:** fácil adicionar **novos tipos/classes** sem mexer nas funções existentes; difícil adicionar **novas funções** (toda classe precisa mudar).

> **Não existe "tudo é objeto".** Quando você prevê **novos tipos de dados**, use OO/polimorfismo. Quando prevê **novas operações** sobre tipos estáveis, estruturas de dados + procedimentos são mais adequados. Escolha conforme o eixo de mudança esperado — sem preconceito.

### 6.3 A Lei de Deméter (LoD)
> Um módulo **não deve conhecer o interior** dos objetos que manipula. "Fale apenas com amigos, não com estranhos."

Mais precisamente, um método `f` de uma classe `C` só deve chamar métodos de:
- o próprio objeto (`this`),
- um objeto **criado** por `f`,
- um objeto **passado como argumento** a `f`,
- um objeto **mantido em uma variável de instância** de `C`.

**Não** chame métodos de objetos **retornados** por esses métodos.

- **"Train wrecks" (carrinhos de trem):** `ctxt.getOptions().getScratchDir().getAbsolutePath()` — encadeamento que revela o mapa de navegação do sistema. Divida em variáveis **se** forem estruturas de dados; mas se forem **objetos**, isso viola a LoD.
- **Cuidado com híbridos:** metade objeto, metade estrutura de dados (funções significativas **e** getters/setters públicos). Dificultam adicionar tanto funções quanto dados — **o pior dos dois mundos**. Evite.
- **Estruturas ocultas:** se `ctxt` é objeto, **diga a ele o que fazer**, não pergunte sobre seu interior. Em vez de obter o caminho para criar um arquivo, peça `ctxt.createScratchFileStream(name)`.

### 6.4 DTOs e Active Record
- **DTO (Data Transfer Object):** classe com variáveis públicas e nenhuma função — a forma "pura" de estrutura de dados. Útil para comunicação com BD, parsing de sockets, etc. (A forma "bean", com getters/setters privados, geralmente não agrega valor.)
- **Active Record:** DTO especial com métodos de navegação (`save`, `find`), tradução direta de tabelas. **Trate-o como estrutura de dados.** Não coloque regras de negócio nele (vira híbrido). Coloque as regras de negócio em **objetos separados** que escondem seus dados (que podem ser instâncias do Active Record).

---

## 7. (reservado — ver Arquitetura Limpa para o nível de sistema/componentes)

---

## 8. TRATAMENTO DE ERROS

O tratamento de erro é necessário, mas **não pode obscurecer a lógica**. Mantenha-o separado e elegante.

### 8.1 Use exceções, não códigos de retorno
Códigos de erro forçam o chamador a verificar imediatamente (entope a lógica, fácil esquecer). Lançar exceções separa o **algoritmo** do **tratamento de erro** — cada um pode ser lido e testado isoladamente.

### 8.2 Escreva o `try-catch-finally` primeiro
Blocos `try` são como **transações**: o `catch` deve deixar o programa em estado consistente. Comece pelo `try-catch-finally` (guiado por um teste que espera a exceção) para definir o escopo e o contrato desde o início.

### 8.3 Use exceções não-verificadas (unchecked)
Exceções verificadas (checked) violam o OCP: uma exceção lançada lá embaixo obriga a alterar assinaturas em toda a cadeia de chamadas (acoplamento). Para a maioria das aplicações, **prefira unchecked exceptions**.

### 8.4 Forneça contexto com as exceções
Cada exceção deve trazer informação suficiente para localizar a origem e a causa: a operação que falhou e o tipo de falha. Crie mensagens informativas; inclua a causa (`cause`).

### 8.5 Defina classes de exceção pela necessidade do chamador
Agrupe/empacote APIs de terceiros e **traduza** suas muitas exceções para **uma classe** adequada ao seu código, quando o tratamento for o mesmo. Isso reduz o acoplamento e simplifica o chamador (geralmente um único `catch`).

### 8.6 Defina o fluxo normal (Special Case Pattern)
Não polua a lógica com casos especiais via exceções. Em vez de `try { total += dao.getMeals(id).getTotal(); } catch (NotFound) { total += getMealPerDiem(); }`, faça o DAO **sempre retornar um objeto** (`PerDiemMealExpenses`) que encapsula o comportamento do caso especial. O cliente não precisa tratar a exceção — o **Special Case Object** cuida disso.

### 8.7 Não retorne `null`
- Retornar `null` cria verificações por toda parte e `NullPointerException`s. Em vez disso, **lance uma exceção** ou retorne um **Special Case Object** (ex.: `Collections.emptyList()` em vez de `null`). Empacote APIs de terceiros que retornam `null`.

### 8.8 Não passe `null`
- Passar `null` para um método é pior ainda. A menos que a API espere `null`, **evite passá-lo**; por padrão, **proíba `null` como argumento**. Não há boa forma de tratar `null` recebido acidentalmente (nem exceção nem `assert` resolvem de verdade), então previna na origem.

> **Limpo e robusto não conflitam.** Tratar erro como uma preocupação **separada**, independente da lógica principal, dá ambos.

---

## 9. LIMITES (BOUNDARIES) — código de terceiros

Há tensão natural: o fornecedor da biblioteca quer amplitude; você quer foco nas suas necessidades.

### 9.1 Encapsule interfaces de terceiros
- Não passe interfaces amplas de terceiros (ex.: `Map`) por todo o sistema. `Map` expõe `clear()`, aceita qualquer tipo, etc. — capacidade demais e ponto de mudança em muitos lugares (lembre da migração para genéricos no Java 5).
- **Embrulhe-as** numa classe sua (ex.: `Sensors` contendo um `Map` privado, expondo só `getById`). Benefícios: a interface no limite fica **oculta** (mudanças têm pouco impacto), você expõe só o que precisa, e pode impor regras do domínio. **Evite retornar ou aceitar interfaces de fronteira em APIs públicas.**

### 9.2 Testes de aprendizagem (Learning Tests)
- Para aprender uma biblioteca nova, escreva **testes** que exercitam como você pretende usá-la, em vez de só ler a doc e experimentar no código de produção. Vantagens: você aprende de forma isolada e, quando a biblioteca atualizar, os **learning tests** detectam mudanças de comportamento que afetam você.

### 9.3 Limites ainda não existentes
- Se um sistema/API de que você depende ainda não existe, **defina a interface que você gostaria de ter** (do seu lado), e implemente um **adapter** quando a API real chegar. Isso mantém seu código limpo, testável (com fakes) e isolado do detalhe externo.

---

## 10. TESTES DE UNIDADE

Testes são **tão importantes quanto** o código de produção — talvez mais, pois preservam flexibilidade, manutenibilidade e reusabilidade. **Código de teste sujo é pior que nenhum teste**, porque apodrece e é abandonado, e aí o código de produção também degrada (você perde a coragem de mudá-lo).

### 10.1 As Três Leis do TDD
1. **Não escreva código de produção** sem antes ter um **teste de unidade que falha**.
2. **Não escreva mais de um teste** do que o suficiente para falhar — **não compilar já é falhar**.
3. **Não escreva mais código de produção** do que o suficiente para fazer o teste atual passar.
Isso gera ciclos de ~30 segundos; testes e produção crescem juntos, com os testes alguns segundos à frente.

### 10.2 Testes limpos
- **Legibilidade é tudo** num teste: clareza, simplicidade, densidade de expressão. Use o padrão **BUILD-OPERATE-CHECK** (montar dados, operar, verificar).
- Crie uma **API/DSL de teste** específica do domínio para tornar os testes expressivos (não use as APIs cruas do sistema diretamente).
- O código de teste **não** segue os mesmos padrões de eficiência do produção, mas **deve** ser simples, sucinto e expressivo.

### 10.3 Uma asserção / um conceito por teste
- Minimize asserções por teste; **teste um único conceito por função de teste.** Se um teste verifica três coisas, divida em três.
- Nomes de teste descritivos (estilo `given/when/then` ou `método_condição_resultado`).

### 10.4 F.I.R.S.T — as cinco regras de testes limpos
- **F — Fast (Rápidos):** testes lentos não são executados; rode-os com frequência.
- **I — Independent (Independentes):** um teste não prepara estado para outro; rode em qualquer ordem. Dependência causa efeito dominó e esconde defeitos.
- **R — Repeatable (Repetíveis):** rodam em qualquer ambiente (produção, QA, notebook offline). Senão, você terá desculpas para falhas.
- **S — Self-Validating (Autovalidáveis):** saída **booleana** (passou/falhou). Nada de comparar logs/arquivos manualmente.
- **T — Timely (Pontuais):** escreva o teste **imediatamente antes** do código de produção. Se escrever depois, o código tende a ficar difícil de testar.

> **Mantenha os testes limpos.** Se os testes degradarem, o código de produção degrada junto.

---

## 11. CLASSES

### 11.1 Organização e encapsulamento
- Ordem padrão (Java): constantes públicas/estáticas → variáveis estáticas privadas → variáveis de instância privadas → funções públicas → funções privadas logo **abaixo** de quem as chama (Regra Decrescente; lê-se como jornal).
- **Variáveis públicas: raramente.** Prefira privado. Afrouxe a visibilidade (protected/package) **só** como último recurso, geralmente para testes.

### 11.2 Classes pequenas! (medidas por responsabilidades)
- Função: meça por **linhas**. Classe: meça por **responsabilidades**.
- Uma "classe de Deus" (ex.: 70 métodos públicos) é grande demais. Mas **mesmo 5 métodos** podem ser demais se a classe tem várias responsabilidades.
- **O nome revela o tamanho:** se você não consegue um nome **conciso** e específico, a classe faz coisas demais. Nomes vagos (`Processor`, `Manager`, `Super`) denunciam acúmulo de responsabilidades. Você deve conseguir descrever a classe em **~25 palavras sem usar "se", "e", "ou", "mas"**. Um "e" na descrição é sinal de responsabilidade demais.

### 11.3 SRP — Princípio da Responsabilidade Única
> Uma classe/módulo deve ter **um, e apenas um, motivo para mudar**.
- É um dos conceitos OO mais importantes — e o **mais ignorado**, porque "fazer funcionar" ≠ "deixar limpo". Não pare quando funciona; volte e separe responsabilidades.
- Prefira **muitas classes pequenas** de propósito único a poucas classes grandes. Um sistema de muitas peças pequenas, bem rotuladas (como gavetas organizadas), não tem mais "o que aprender" que um de peças grandes — e você só precisa entender a parte relevante no momento.

### 11.4 Coesão
- Classes devem ter **poucas variáveis de instância**; cada método deve usar **uma ou mais** delas. Quanto mais variáveis um método usa, mais coeso ele é. **Alta coesão** é o objetivo (métodos e variáveis interdependentes formando um todo lógico).
- **Quando a coesão cai, divida a classe.** Manter funções pequenas e listas de args curtas tende a proliferar variáveis de instância compartilhadas por poucos métodos — sinal de que **uma nova classe quer emergir**. Extrair funções de uma função grande quase sempre permite extrair **classes** também, melhorando a organização.

### 11.5 Organizar para a mudança e isolar da mudança
- Toda mudança traz risco de quebrar algo. Organize de modo que cada classe mude por **um motivo**, e que mudanças fiquem **localizadas**.
- Em vez de `switch`/`if` que cresce, use **subclasses/polimorfismo** (OCP).
- **Isole das dependências concretas com DIP:** dependa de **abstrações** (interfaces), não de detalhes concretos. Ex.: em vez da classe depender diretamente de uma API de cotações da bolsa, dependa de uma interface `StockExchange` e injete a implementação — assim você testa com um fake e protege as regras de negócio das mudanças externas. (Ver guia de Arquitetura Limpa.)

---

## 12. SISTEMAS

> *Construir* é diferente de *usar*. (Você não constrói um hotel e mora nele ao mesmo tempo.)

### 12.1 Separe a construção do uso
- Separe o **processo de inicialização** (criar objetos e conectar dependências) da **lógica de runtime**. Misturar os dois (ex.: *lazy initialization* `if (service == null) service = new MyServiceImpl(...)` espalhada) cria dependências concretas codificadas, dificulta testes (precisa de mocks no lugar certo) e espalha a configuração global pela aplicação (viola SRP, sem modularidade, com duplicação).
- **Separação no `main`:** ponha **toda** a construção no `main` (ou módulos chamados por ele). O resto do sistema assume que tudo já foi construído e injetado; as setas de dependência apontam **para fora** do `main` (o app não conhece o `main` nem a construção). (Ver "componente Main" no guia de Arquitetura Limpa.)
- **Factories:** quando o app precisa controlar **quando** um objeto é criado, use **Abstract Factory** — o app decide o momento, mas os detalhes da construção ficam separados.

### 12.2 Injeção de Dependência (DI / Inversion of Control)
- A classe **não** instancia nem localiza suas dependências; ela apenas as recebe (por construtor/setter) de um mecanismo de montagem (o `main` ou um container como Spring). Isso é a aplicação do SRP à construção: a responsabilidade de resolver dependências sai da classe.

### 12.3 Cresça incrementalmente; separe preocupações transversais
- Sistemas limpos, como cidades, **evoluem** com níveis adequados de abstração e modularidade — não exigem big design up front, mas exigem **separação de preocupações** em todos os níveis.
- **Preocupações transversais** (persistência, segurança, transações, logging) tendem a se espalhar. Trate-as de forma modular (POA/aspectos, proxies, ou decorators/interceptors) para não poluir as regras de negócio.
- **Adie decisões:** uma boa separação permite adiar escolhas (BD, framework) até ter mais informação — e tornar essas escolhas reversíveis. **Use a abordagem mais simples que funcione agora.**

---

## 13. EMERGÊNCIA — As 4 Regras do Design Simples (Kent Beck)

Um design é "simples" (e bom design **emerge**) se seguir, **nesta ordem de prioridade**:

1. **Executa todos os testes.** Um sistema precisa funcionar e ser **verificável**. Tornar testável força classes pequenas, de propósito único (SRP), baixo acoplamento (DIP, injeção de dependência, interfaces) e alta coesão. **Criar testes leva a designs melhores.**
2. **Não contém duplicação (DRY).** Duplicação é o inimigo nº 1: trabalho, risco e complexidade extras. Inclui duplicação óbvia (linhas iguais), de implementação (`isEmpty()` pode ser `return 0 == size()`), e algoritmos parecidos. Elimine com extração de métodos, **Template Method**, Strategy, polimorfismo. A "pequena reutilização" reduz muito a complexidade.
3. **Expressa a intenção do programador.** O maior custo é a manutenção de longo prazo; código claro reduz defeitos e custo. Expresse-se com: **bons nomes**; **funções e classes pequenas** (fáceis de nomear); **nomenclatura padrão** (nomes de Design Patterns como `Command`, `Visitor`); **testes bem escritos** (servem de documentação). Acima de tudo: **tente** — não pare quando "funciona"; tenha orgulho do trabalho.
4. **Minimiza o número de classes e métodos.** A regra de **menor** prioridade. Não caia em dogmatismo (interface para toda classe; separar sempre dados e comportamento). Mantenha o sistema pequeno **sem** sacrificar as regras 1–3. Pragmatismo sobre dogma.

---

## 14. CONCORRÊNCIA

Escrever código concorrente limpo é **difícil**. Código com bug de concorrência funciona "na maioria das vezes" e falha sob carga.

### 14.1 Por que e os mitos
- **Concorrência é uma estratégia de desacoplamento:** separa **o quê** de **quando**. Melhora throughput e estrutura (vários "minicomputadores" em vez de um `main()` gigante).
- **Mitos a derrubar:** "concorrência sempre melhora desempenho" (só com muita espera divisível); "o design não muda" (muda muito); "não precisa entender concorrência usando containers Web/EJB" (precisa). Verdades: concorrência tem custo; é complexa mesmo para problemas simples; bugs **não se repetem** (e são ignorados como "casos únicos"); costuma exigir mudança de design.

### 14.2 Princípios de defesa
- **SRP — separe o código de concorrência** do resto. Ele tem ciclo de vida, desafios e otimizações próprios. (`Mantenha o código de concorrência separado.`)
- **Limite o escopo dos dados:** proteja seções críticas (`synchronized`/locks) e **minimize** a quantidade de lugares que acessam dados compartilhados. Leve o **encapsulamento de dados a sério**.
- **Use cópias dos dados:** evite compartilhar — copie objetos como somente-leitura, ou processe em cópias e una os resultados em uma thread. Muitas vezes compensa o custo de criar objetos.
- **Threads o mais independentes possíveis:** cada thread em seu próprio mundo, com dados em **variáveis locais** de fonte não compartilhada (como um servlet que recebe tudo por parâmetros). Particione os dados em subsistemas independentes.
- **Conheça sua biblioteca:** use coleções thread-safe (`java.util.concurrent`, ex.: `ConcurrentHashMap`), o framework `Executor`, soluções **non-blocking**. (Em outras linguagens: as primitivas e bibliotecas concorrentes equivalentes.)

### 14.3 Modelos de execução (aprenda-os)
- **Producer-Consumer:** produtores enfileiram tarefas; consumidores as retiram; fila é recurso limitado; coordenam por sinalização.
- **Readers-Writers:** recurso lido frequentemente e atualizado ocasionalmente; equilibrar throughput vs. starvation; escritores não devem bloquear leitores indefinidamente (e vice-versa).
- **Dining Philosophers (Filósofos):** processos competindo por recursos; sem cuidado, geram **deadlock**, **livelock**, **starvation** e queda de throughput.
- Termos: **recursos limitados**, **exclusão mútua**, **starvation** (espera indefinida), **deadlock** (espera circular), **livelock** (avançam mas se atrapalham).

### 14.4 Cuidados práticos
- **Evite chamar mais de um método** em um objeto compartilhado. Se precisar, use **lock no cliente**, **lock no servidor** ou um **servidor intermediário** (adapted server).
- **Mantenha seções sincronizadas pequenas:** locks causam atrasos e contenção. Proteja só o mínimo crítico; não estenda a sincronização além do necessário.
- **Desligamento correto é difícil:** threads esperando sinais que nunca chegam (pai esperando filhas em deadlock; producer encerra e consumer fica bloqueada). Planeje o shutdown com cuidado desde cedo.
- **Teste código concorrente:** trate falhas "espúrias" como **prováveis bugs de threading** (não as ignore); faça o código **não-concorrente** funcionar primeiro; torne o código de threads **plugável e ajustável** (nº de threads); rode com **mais threads que processadores**; rode em **plataformas diferentes**; **instrumente** o código (inserindo `sleep`/`yield`) para forçar a aparição de falhas.

---

## 15. ODORES E HEURÍSTICAS (Smells & Heuristics) — A CHECKLIST CANÔNICA

> Esta é a lista de referência de sinais de código ruim e boas práticas (do capítulo final do livro, combinando os "code smells" de Martin Fowler com os do Uncle Bob). **Use-a em toda revisão.** Cada item tem um código (citável, ex.: `[G5]`).

### Comentários (C)
- **C1 — Informação inapropriada:** comentário com info que pertence ao VCS, issue tracker ou outro registro (históricos, autores, datas). Comentários só devem ter notas técnicas sobre código/projeto.
- **C2 — Comentário obsoleto:** velho, irrelevante ou incorreto. Atualize ou apague imediatamente.
- **C3 — Comentário redundante:** descreve o que o código já diz (`i++; // incrementa i`; Javadoc que repete a assinatura).
- **C4 — Comentário mal escrito:** se vale escrever, escreva bem — conciso, correto, sem o óbvio.
- **C5 — Código comentado:** **apague.** O VCS lembra. Código comentado apodrece.

### Ambiente (E)
- **E1 — Build exige mais de um passo:** build deve ser **um comando** (`checkout` + um comando).
- **E2 — Testes exigem mais de um passo:** rodar **todos** os testes deve ser **um comando**/um botão.

### Funções (F)
- **F1 — Argumentos demais:** zero é melhor; depois 1, 2, 3. Mais que isso é questionável.
- **F2 — Argumentos de saída:** antinatural; o leitor espera saída pelo retorno. Mude o estado do objeto dono.
- **F3 — Argumentos lógicos (flags):** booleano declara que a função faz mais de uma coisa. Elimine (divida a função).
- **F4 — Função morta:** método nunca chamado. Apague.

### Geral (G)
- **G1 — Múltiplas linguagens em um arquivo:** minimize linguagens por arquivo (idealmente uma).
- **G2 — Comportamento óbvio não implementado:** siga o **Princípio da Menor Surpresa**; implemente o que o leitor espera (ex.: conversão de "Segunda" e abreviações, case-insensitive). Senão, o leitor perde a confiança no código.
- **G3 — Comportamento incorreto nos limites:** não confie na intuição; teste **cada condição de limite/canto**.
- **G4 — Seguranças anuladas:** não desligue avisos do compilador, testes que falham, etc. ("Chernobyl").
- **G5 — Duplicação (DRY):** uma das regras mais importantes. Elimine toda duplicação — extraia métodos, use polimorfismo (em vez de `switch`/`if` repetidos), Template Method/Strategy. Muitos padrões e a própria OO existem para isso.
- **G6 — Código no nível errado de abstração:** separe conceitos de alto e baixo nível **completamente** (ex.: `percentFull()` não pertence a uma interface `Stack` genérica). Não misture; não "minta" (retornar 0) para forçar uma abstração mal posicionada.
- **G7 — Classe base depende das derivadas:** em geral, a base não deve mencionar/saber das derivadas (permite implantar em componentes separados).
- **G8 — Informação em excesso:** interfaces **pequenas**; exponha pouco. Quanto menos métodos/variáveis exibidos, melhor (acoplamento fraco). Esconda dados, utilitários, constantes, temporárias.
- **G9 — Código morto:** `if` impossível, `catch` que nunca dispara, métodos nunca chamados. Apague.
- **G10 — Separação vertical:** declare variáveis e funções **perto** de onde são usadas.
- **G11 — Inconsistência:** faça coisas similares da mesma forma (Princípio da Menor Surpresa). Mesmo nome de variável para o mesmo conceito; mesma convenção de nomes de métodos.
- **G12 — Entulho (clutter):** construtores vazios, variáveis/funções não usadas, comentários sem info. Remova.
- **G13 — Acoplamento artificial:** não acople coisas que não dependem entre si (ex.: enum genérico dentro de classe específica; função estática de uso geral em classe específica).
- **G14 — Feature Envy:** método mais interessado nos dados de **outra** classe que na própria (usa muitos getters/setters de outro objeto). Mova o comportamento para perto dos dados (com exceções, ex.: relatórios).
- **G15 — Argumentos seletores:** `false`/`true`/enum/int pendurado para escolher comportamento. Prefira **várias funções** a um parâmetro seletor.
- **G16 — Propósito obscuro:** evite expressões compactas demais, notação húngara, números mágicos. Vale o tempo de tornar a intenção visível.
- **G17 — Responsabilidade mal posicionada:** ponha o código onde o leitor **espera** (Princípio da Menor Surpresa). `PI` perto da trigonometria; `OVERTIME_RATE` no `HourlyPayCalculator`. Use os nomes das funções como guia de onde colocar a lógica.
- **G18 — `static` inadequado:** prefira métodos **não-estáticos**. Use `static` só quando não houver chance de querer comportamento polimórfico (ex.: `Math.max`). Na dúvida, não-estático.
- **G19 — Use variáveis explicativas:** separe cálculos em variáveis intermediárias bem nomeadas — uma das formas mais poderosas de tornar código legível.
- **G20 — Nomes de função devem dizer o que fazem:** se `date.add(5)` não deixa claro o que faz, renomeie (`addDaysTo`, `daysLater`). Se precisa olhar a implementação para entender, o nome é ruim.
- **G21 — Entenda o algoritmo:** não "empurre" `if`s e flags até "funcionar". Entenda **por que** funciona; refatore até ficar óbvio. ("Não ter certeza do algoritmo é normal; não saber o que seu código faz é preguiça.")
- **G22 — Torne dependências lógicas em físicas:** se um módulo depende de outro, peça a info **explicitamente** em vez de assumir (ex.: `HourlyReporter` não deve "saber" o `PAGE_SIZE` do formatter — peça `getMaxPageSize()`).
- **G23 — Prefira polimorfismo a `if/else`/`switch`:** considere polimorfismo **antes** de usar `switch`. Regra "UM SWITCH": no máximo um `switch` por tipo de seleção, para criar objetos polimórficos.
- **G24 — Siga convenções padrão:** a equipe segue um padrão (nomes, chaves, posição de declarações). O código é o exemplo das convenções.
- **G25 — Substitua números mágicos por constantes nomeadas:** `86400` → `SECONDS_PER_DAY`. Vale também para qualquer token não autoexplicativo (`7777`, `"John Doe"` → `HOURLY_EMPLOYEE_ID`, `HOURLY_EMPLOYEE_NAME`). Exceções: fórmulas onde o número é mais claro (`feet/5280.0`).
- **G26 — Seja preciso:** não seja desleixado. Verifique `null`, trate concorrência, use o tipo certo (não `float` para dinheiro — use inteiros/`Money`), não declare `ArrayList` quando `List` basta, etc.
- **G27 — Estrutura acima de convenção:** estruturas que **forçam** cumprimento (classes base com métodos abstratos) são superiores a convenções que dependem de disciplina (`switch` com enums).
- **G28 — Encapsule condicionais:** extraia funções que expliquem o booleano: `if (shouldBeDeleted(timer))` em vez de `if (timer.hasExpired() && !timer.isRecurrent())`.
- **G29 — Evite condicionais negativas:** `if (buffer.shouldCompact())` é melhor que `if (!buffer.shouldNotCompact())`.
- **G30 — Funções devem fazer uma coisa só.** Divida funções com múltiplas seções/operações.
- **G31 — Acoplamentos temporais ocultos:** se a ordem das chamadas importa, **exponha** isso (ex.: cada função produz o que a próxima recebe — "bucket brigade"). Não confie que o leitor adivinhe a ordem.
- **G32 — Não seja arbitrário:** tenha um motivo para a estrutura e que ele seja evidente. Estrutura consistente é preservada; arbitrária é alterada. (Classes públicas usadas de fora não devem ficar aninhadas em outra por conveniência.)
- **G33 — Encapsule condições de limite:** processe `+1`/`-1` num só lugar, numa variável nomeada (`int nextLevel = level + 1;`). Não espalhe `+1`/`-1`.
- **G34 — Funções devem descer apenas um nível de abstração:** todas as instruções de uma função no **mesmo** nível, um nível abaixo do nome dela. Uma das refatorações mais importantes (e difíceis).
- **G35 — Mantenha dados configuráveis em níveis altos:** constantes de configuração/padrão ficam em **nível alto** (ex.: topo da classe `Arguments`) e são passadas para baixo; não as enterre em funções de baixo nível.
- **G36 — Evite navegação transitiva (Lei de Deméter):** não `a.getB().getC().doSomething()`. Fale só com vizinhos imediatos; peça a eles o serviço (`meuColaborador.facaAlgo()`). Caso contrário a arquitetura fica rígida.

### Java (J) — adapte a outras linguagens
- **J1 — Evite listas longas de import usando wildcards:** `import package.*;` (imports com curinga não criam dependência rígida e reduzem ruído). *(Em outras linguagens, vale o espírito: imports concisos e organizados.)*
- **J2 — Não herde constantes:** não esconda constantes no topo de uma hierarquia de herança via interface. Use **import estático**.
- **J3 — Constantes vs enum:** use **enum** (têm nome, métodos e campos) em vez do truque `public static final int`.

### Nomes (N)
- **N1 — Escolha nomes descritivos:** nomes são ~90% da legibilidade. Reavalie-os com frequência.
- **N2 — Nomes no nível certo de abstração:** reflitam o nível da classe/função, não a implementação (ex.: `Modem.connect(connectionLocator)` em vez de `dial(phoneNumber)`).
- **N3 — Use nomenclatura padrão quando possível:** use nomes de padrões (`...Decorator`), `toString`, e a **linguagem ubíqua** do domínio (Eric Evans).
- **N4 — Nomes não ambíguos:** evite nomes vagos. `renamePageAndOptionallyAllReferences` é longo, mas claro (e só usado num lugar).
- **N5 — Nomes longos para escopos grandes:** `i`/`j` ok em laços curtos; nomes longos e precisos em escopos grandes.
- **N6 — Evite codificações:** sem notação húngara, sem prefixos `m_`/`f`. Ambientes modernos fornecem o contexto.
- **N7 — Nomes devem descrever efeitos colaterais:** se `getOos()` também **cria** o objeto, chame de `createOrReturnOos()`. Não esconda o que a função faz.

### Testes (T)
- **T1 — Testes insuficientes:** teste **tudo que pode falhar**. "Parece suficiente" não é medida.
- **T2 — Use uma ferramenta de cobertura:** revela lacunas (linhas verdes/vermelhas na IDE).
- **T3 — Não pule testes triviais:** são baratos e documentam.
- **T4 — Teste ignorado = pergunta sobre ambiguidade:** expresse dúvidas de requisito como teste com `@Ignore` (ou comentado).
- **T5 — Teste condições de limite:** entendemos o "meio" do algoritmo e erramos os limites.
- **T6 — Teste exaustivamente perto de bugs:** bugs se agrupam; ao achar um, teste a fundo a função.
- **T7 — Padrões de falha são reveladores:** a forma como os testes falham (ordenados) revela a causa.
- **T8 — Padrões de cobertura são reveladores:** o que é/não é executado pelos testes que passam dá pistas das falhas.
- **T9 — Testes devem ser rápidos:** teste lento é teste que não roda; mantenha-os rápidos.

> O objetivo final desta lista não é completude, mas transmitir o **sistema de valores** por trás do código limpo. Não se cria código limpo só seguindo regras — desenvolve-se "sensibilidade ao código".

---

## 16. REFERÊNCIA RÁPIDA PARA CODAR (cheat sheet)

### 16.1 Padrões a aplicar por defeito (ao escrever qualquer código)
- **Nomes:** reveladores de intenção, pronunciáveis, pesquisáveis; tamanho proporcional ao escopo; sem codificações; uma palavra por conceito; classes = substantivos, métodos = verbos.
- **Funções:** pequenas (≤20 linhas), uma coisa, um nível de abstração, ≤2 níveis de endentação, ≤3 argumentos, sem flags, sem efeitos colaterais, comando **ou** consulta.
- **Erros:** exceções (não códigos), sem `null` de entrada/saída, Special Case Pattern, contexto nas exceções.
- **Comentários:** prefira código autoexplicativo; nunca deixe código comentado; nada de comentários redundantes/obsoletos.
- **Duplicação:** elimine sempre (DRY).
- **Classes:** pequenas (por responsabilidade), SRP, alta coesão, dependa de abstrações.
- **Testes:** TDD quando possível; F.I.R.S.T; um conceito por teste; mantenha-os limpos.
- **Regra do Escoteiro:** deixe o código mais limpo do que encontrou, em toda edição.

### 16.2 Checklist de revisão antes de finalizar (pre-commit)
- [ ] Nomes revelam intenção e seguem o léxico do projeto? (N1–N7)
- [ ] Cada função faz **uma coisa**, é pequena e tem um só nível de abstração? (G30, G34, F1–F4)
- [ ] Há **duplicação** a eliminar? (G5)
- [ ] Há `switch`/`if-else` que deveria ser **polimorfismo**? (G23)
- [ ] Algum **número mágico** / token não autoexplicativo? (G25)
- [ ] Condicionais e condições de limite **encapsuladas**? (G28, G33)
- [ ] Algum **comentário** que deveria ser código, ou obsoleto, ou código comentado? (C1–C5)
- [ ] Tratamento de erro via **exceções**, sem retornar/passar `null`? (seção 8)
- [ ] Alguma violação da **Lei de Deméter** / train wreck? (G36)
- [ ] Classe respeita **SRP** e tem **alta coesão**? Nome conciso? (seção 11)
- [ ] Há **testes**, e eles são limpos e F.I.R.S.T? (seções 10, T1–T9)
- [ ] As **4 regras do design simples** estão satisfeitas, na ordem? (seção 13)
- [ ] Deixei o código **mais limpo** do que encontrei?

### 16.3 Red flags (pare e refatore)
- 🚩 Função longa, com muitos níveis de endentação ou várias "seções".
- 🚩 Argumento booleano / argumento de saída.
- 🚩 `if (x == null)` espalhado; método que retorna `null`.
- 🚩 `switch`/cadeia de `if-else` por tipo, repetida em vários lugares.
- 🚩 `a.getB().getC().doSomething()` (train wreck).
- 🚩 Classe com nome vago (`Manager`, `Processor`, `Data`, `Info`, `Util` genérico) ou com dezenas de métodos.
- 🚩 Getters/setters cegos expondo todos os campos; objeto híbrido (dados + comportamento + acessores públicos).
- 🚩 Código comentado; comentário que repete o código; comentário desatualizado.
- 🚩 Número mágico, nome de uma letra fora de laço curto, notação húngara.
- 🚩 Efeito colateral escondido (função que muda estado além do que o nome diz).
- 🚩 Duplicação (linhas iguais, algoritmos parecidos).
- 🚩 Teste acoplado à estrutura, lento, ou que depende de outro teste.
- 🚩 Construção de objetos (lazy init, `new ConcreteImpl()`) misturada à lógica de negócio.
- 🚩 Código de concorrência misturado ao código de aplicação.
- 🚩 "Vou limpar depois" / "vamos reescrever do zero".

### 16.4 Heurísticas de decisão
- **"Objeto ou estrutura de dados?"** → Vai adicionar **tipos** no futuro? Objeto (polimorfismo). Vai adicionar **operações** sobre tipos estáveis? Estrutura de dados + funções. Não force "tudo é objeto".
- **"Comentar isto?"** → Primeiro tente expressar no código (nome de função/variável). Comente só se for legal, intenção, alerta, esclarecimento de API externa ou amplificação.
- **"Criar uma exceção, retornar null, ou Special Case?"** → Nunca `null`. Erro de verdade → exceção. Ausência esperada → Special Case Object / coleção vazia.
- **"Esta função está grande?"** → Se você consegue extrair outra função com nome que **não** é só reformular o código, ela faz mais de uma coisa: extraia.
- **"Onde colocar este código/constante?"** → Onde o leitor espera (Princípio da Menor Surpresa); use os nomes das funções como guia (G17).
- **"Quantos argumentos?"** → Reduza: torne o objeto dono do método, extraia classe, ou agrupe args correlatos em um objeto.

---

## 17. GLOSSÁRIO E MANTRAS

### Princípios e siglas
- **Regra do Escoteiro:** deixe o código mais limpo do que o encontrou.
- **SRP (Single Responsibility Principle):** uma classe/módulo, um único motivo para mudar.
- **DRY (Don't Repeat Yourself):** não duplique conhecimento/código.
- **OCP (Open-Closed):** aberto a extensão, fechado a modificação (use polimorfismo). *(detalhes no guia de Arquitetura Limpa)*
- **DIP (Dependency Inversion):** dependa de abstrações, não de concretos. *(idem)*
- **Lei de Deméter (LoD):** fale só com amigos imediatos; não navegue pelo interior de objetos.
- **Separação Comando-Consulta:** uma função faz **ou** responde, nunca os dois.
- **Special Case Pattern:** objeto que encapsula o comportamento de um caso especial, eliminando `if`/exceção no cliente.
- **Stepdown Rule (Regra Decrescente):** leia o código de cima para baixo, cada função um nível abaixo da anterior.
- **Princípio da Menor Surpresa:** o código deve se comportar como o leitor espera.
- **3 Leis do TDD:** teste falho antes do código; só o teste necessário para falhar; só o código necessário para passar.
- **F.I.R.S.T:** testes Fast, Independent, Repeatable, Self-validating, Timely.
- **4 Regras do Design Simples (em ordem):** passa nos testes → sem duplicação → expressa intenção → menos classes/métodos.
- **Lei de LeBlanc:** "Depois nunca chega."
- **DTO / Active Record / Híbrido:** estrutura de dados pura / DTO com `save`/`find` / mistura ruim de objeto e estrutura de dados.
- **Train wreck (carrinho de trem):** cadeia `a.getB().getC()...` que viola a LoD.
- **Feature Envy:** método mais interessado nos dados de outra classe que nos da própria.
- **Aridade:** niládica (0) > monádica (1) > diádica (2) > triádica (3, evitar) > poliádica.
- **Code smell / heurística:** sinal de código ruim / boa prática (lista da seção 15, códigos C/E/F/G/J/N/T).

### Mantras para lembrar
- *A única maneira de ir rápido é ir bem.*
- *Deixe o acampamento mais limpo do que você o encontrou.*
- *Funções devem ser pequenas, fazer uma coisa e fazê-la bem.*
- *Não se repita (DRY).*
- *O melhor comentário é o que você não precisou escrever.*
- *Apague código comentado — o controle de versão lembra.*
- *Não retorne `null`; não passe `null`.*
- *Use exceções, não códigos de erro.*
- *Prefira polimorfismo a `if/else`/`switch`.*
- *Fale só com amigos (Lei de Deméter).*
- *Nomes revelam intenção; clareza acima de esperteza.*
- *Mantenha os testes limpos — se eles degradarem, o código degrada.*
- *Deixar o código limpo não é opcional: é sobrevivência profissional.*

> **Relação com a Arquitetura Limpa:** este guia cobre o **micro** (linhas, funções, classes); o guia de Arquitetura Limpa cobre o **macro** (componentes, fronteiras, regra da dependência, camadas). Use os dois juntos: código limpo dentro de uma arquitetura limpa.
