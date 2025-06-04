Manual Técnico: Geração de Código de Três Endereços (TAC)Nota: Este manual foi elaborado com o auxílio de inteligência artificial para fornecer uma base de conhecimento abrangente. Recomenda-se a consulta às obras de referência e a supervisão de um instrutor para um aprendizado completo e aprofundado.1. Introdução ao Código de Três Endereços1.1 Definição e Propósito do TACO Código de Três Endereços (TAC) é uma representação intermediária (Intermediate Representation - IR) linearizada usada por compiladores. Ele se caracteriza por decompor as construções complexas da linguagem fonte em uma sequência de instruções simples, onde cada instrução possui, no máximo, três "endereços". Um endereço pode ser um nome (como o de uma variável da linguagem fonte), uma constante, ou uma variável temporária gerada pelo compilador.O propósito fundamental do TAC é servir como uma ponte entre a análise semântica (frontend do compilador) e a geração de código objeto (backend do compilador). Ele oferece um nível de abstração que é mais próximo do código de máquina do que a Árvore Sintática Abstrata (AST), mas ainda é suficientemente independente da máquina alvo para permitir otimizações de código genéricas e facilitar a portabilidade do compilador para diferentes arquiteturas.1.2 Vantagens da Utilização do TAC em CompiladoresA adoção do TAC como representação intermediária traz diversas vantagens significativas:Independência de Máquina (Parcial): Embora mais detalhado que a AST, o TAC ainda abstrai muitos detalhes específicos da arquitetura da máquina alvo, como registradores e modos de endereçamento complexos. Isso permite que otimizações sejam escritas de forma mais genérica.Facilidade de Otimização: A estrutura regular e explícita das instruções TAC simplifica a identificação de padrões e a aplicação de diversas técnicas de otimização de código, como eliminação de subexpressões comuns, propagação de constantes, redução de força de operadores, e remoção de código morto.Simplificação da Geração de Código Objeto: A tradução de TAC para código de máquina (ou assembly) é geralmente mais direta do que a tradução a partir de representações de mais alto nível como a AST. Cada instrução TAC pode ser mapeada para uma pequena sequência de instruções de máquina.Clareza e Depuração: A forma explícita das operações e o uso de variáveis temporárias para resultados intermediários podem tornar o fluxo de computação mais claro, auxiliando na análise e depuração tanto do código gerado quanto do próprio compilador.Modularidade: Permite uma separação clara entre o frontend (que produz TAC) e o backend (que consome TAC), facilitando o desenvolvimento e a manutenção do compilador.1.3 Formato Geral das Instruções TACA forma mais comum de uma instrução TAC é a de uma atribuição com uma operação binária:resultado := operando1 op operando2Onde:resultado: É o endereço (variável ou temporária) que armazena o resultado da operação.operando1, operando2: São os endereços dos operandos. Podem ser variáveis, constantes ou temporárias.op: É um operador (e.g., aritmético como +, -, *, /; lógico; relacional).Existem variações e outros formatos importantes:Atribuição com Operação Unária: resultado := op operando1 (e.g., x := -y)Cópia (Atribuição Simples): resultado := operando1 (e.g., x := y)Desvio Incondicional: GOTO L (onde L é um rótulo que identifica uma instrução)Desvio Condicional: IF condicao GOTO L (e.g., IF x < y GOTO L1) ou IF_FALSE condicao GOTO LChamada de Procedimento/Função: PARAM x (passa o parâmetro x), CALL p, n (chama o procedimento p com n parâmetros), y := CALL p, n (se a função p retorna um valor que é atribuído a y).Acesso a Arrays Indexados: x := y[i] (lê o valor do elemento i do array y para x), x[i] := y (escreve o valor de y no elemento i do array x).Declaração de Rótulos (Labels): LABEL L (define o rótulo L para a próxima instrução).1.4 Tipos Comuns de Instruções TACAtribuição:x := y (Cópia)x := op y (Operação unária, e.g., x := -y, x := NOT z)Operações Aritméticas Binárias:x := y + zx := y - zx := y * zx := y / zx := y MOD zOperações Lógicas: (Frequentemente usadas na avaliação de expressões booleanas e desvios)x := y AND zx := y OR z(O operador NOT é geralmente unário, como em x := NOT y)Operações Relacionais: (Frequentemente usadas em desvios condicionais)t1 := x < y (o resultado booleano é armazenado em t1)Seguido por: IF t1 GOTO L ou IF_FALSE t1 GOTO LAlternativamente, a forma combinada: IF x < y GOTO LDesvios Incondicionais:GOTO LDesvios Condicionais:IF x GOTO L (Se x é verdadeiro/diferente de zero, desvia para L)IF_FALSE x GOTO L (Se x é falso/igual a zero, desvia para L)IF x relop y GOTO L (e.g., IF x < y GOTO L1)Chamada e Retorno de Procedimento/Função:PARAM x (Define x como um parâmetro para a próxima chamada)CALL p, n (Chama o procedimento p que tem n parâmetros)y := CALL f, n (Chama a função f com n parâmetros e armazena o resultado em y)RETURN (Retorna de um procedimento sem valor)RETURN x (Retorna de uma função com o valor x)Acesso a Array Indexado:x := y[i] (Lê y[i] em x. Isso pode ser decomposto se i for uma expressão: t1 := i; t2 := y[t1]; x := t2)x[i] := y (Escreve y em x[i]. Similarmente, pode ser decomposto: t1 := i; x[t1] := y)Declaração de Rótulos (Labels):LABEL L ou L: (Marca a instrução seguinte com o rótulo L)2. Fundamentação TeóricaA geração de Código de Três Endereços é uma etapa consolidada na teoria e prática de compiladores. Ela se baseia em conceitos extensivamente abordados em obras de referência na área:"Compilers: Principles, Techniques, and Tools" por Aho, Sethi e Ullman (frequentemente chamado de "Livro do Dragão"): Este é um texto clássico que detalha profundamente as representações intermediárias, incluindo o código de três endereços. O livro introduz esquemas de tradução direcionada por sintaxe (Syntax-Directed Translation - SDT) para gerar TAC a partir de construções da linguagem fonte. Conceitos como atributos sintetizados e herdados dos nós da árvore sintática são fundamentais para guiar a geração de código."Modern Compiler Implementation in Java/C/ML" por Andrew Appel: Appel oferece uma abordagem prática e moderna para a construção de compiladores. Embora ele utilize uma representação intermediária baseada em árvores (IR Trees) que é posteriormente linearizada, os princípios de decomposição de expressões e controle de fluxo são muito similares aos do TAC. O livro enfatiza a importância de uma IR bem definida para facilitar as otimizações."Engineering a Compiler" por Cooper e Torczon: Este livro fornece uma perspectiva de engenharia sobre o design e implementação de compiladores, com uma discussão robusta sobre as várias formas de representações intermediárias, incluindo o TAC (e suas variantes como quádruplas e triplas), e como elas suportam as fases de análise e transformação de programas.Fluxo do Compilador e o Papel do TACO TAC se encaixa no fluxo geral de um compilador como uma etapa intermediária crucial, posicionada entre a análise semântica e a geração de código objeto.Diagrama Conceitual do Fluxo de Compilação:        Código Fonte (Ex: programa.txt)
              |
              V
      +---------------------+
      |   Análise Léxica    |  (Scanner)
      | (Gera Tokens)       |
      +---------------------+
              |
              V
      +---------------------+
      |  Análise Sintática  |  (Parser)
      | (Gera Árvore        |
      |  Sintática Abstrata |
      |  - AST)             |
      +---------------------+
              |
              V
      +---------------------+
      |  Análise Semântica  |
      | (Verifica Tipos,   |
      |  Escopos, etc.     |
      |  AST Anotada)       |
      +---------------------+
              |
              V
+-----------------------------------+
| Geração de Código Intermediário   |
| (Gera Código de Três Endereços    |
|  - TAC)                           |
+-----------------------------------+
              |
              V
      +---------------------+
      | Otimização de Código|
      | (Transforma TAC     |
      |  para TAC Otimizado)|
      +---------------------+
              |
              V
      +---------------------+
      | Geração de Código   |
      | Objeto (Assembly ou |
      | Código de Máquina)  |
      +---------------------+
              |
              V
        Código Executável

Análise Léxica (Scanner): O código fonte é lido e dividido em uma sequência de tokens (palavras-chave, identificadores, operadores, literais).Análise Sintática (Parser): Os tokens são organizados em uma estrutura hierárquica, geralmente uma Árvore Sintática Abstrata (AST), que representa a estrutura gramatical do programa.Análise Semântica: A AST é percorrida para verificar a consistência semântica do programa, como a checagem de tipos, a resolução de nomes (escopos) e outras regras da linguagem. A AST pode ser anotada com informações de tipo e outras informações semânticas.Geração de Código Intermediário (TAC): A AST (geralmente anotada) é traduzida para uma representação linear de mais baixo nível, o Código de Três Endereços. É nesta fase que expressões complexas são quebradas em operações simples e estruturas de controle são convertidas em desvios e rótulos.Otimização de Código: O TAC é analisado e transformado para melhorar a eficiência do código resultante (e.g., torná-lo mais rápido ou menor) sem alterar sua semântica. Muitas otimizações são mais fáceis de aplicar sobre o TAC do que sobre a AST ou sobre o código de máquina final.Geração de Código Objeto: O TAC (otimizado) é traduzido para a linguagem de máquina da arquitetura alvo ou para uma linguagem de montagem (Assembly).O TAC, portanto, desacopla o frontend (fases 1-4) do backend (fases 5-6), permitindo, por exemplo, que um mesmo frontend que produz TAC seja usado com diferentes backends para gerar código para múltiplas arquiteturas, ou que diferentes frontends para linguagens distintas possam compartilhar um backend otimizador comum se ambos produzirem uma forma compatível de TAC.3. Geração de TAC para Construções da Linguagem FonteA geração de TAC é tipicamente realizada através de uma travessia da Árvore Sintática Abstrata (AST). Para cada tipo de nó na AST, existem regras específicas para gerar as instruções TAC correspondentes.3.1 Expressões3.1.1 Expressões AritméticasExpressões aritméticas são decompostas em uma sequência de operações binárias ou unárias, utilizando variáveis temporárias para armazenar resultados intermediários. A ordem de geração deve respeitar a precedência e associatividade dos operadores.Exemplo: x = (a + b) * c - d / e;AST Conceitual (simplificada):        :=
       /  \
      x    -
          / \
         * /
        / \ / \
       +   c d e
      / \
     a   b

Geração de TAC (considerando precedência usual: *,/ antes de +,-, da esquerda para a direita):T1 := a + b       // Resultado de (a + b)
T2 := T1 * c      // Resultado de (a + b) * c
T3 := d / e       // Resultado de d / e
T4 := T2 - T3     // Resultado final da expressão
x  := T4          // Atribuição a x

T1, T2, T3, T4 são variáveis temporárias geradas pelo compilador. Uma função como nova_temp() seria usada para criar essas temporárias.3.1.2 Expressões Booleanas (Curto-Circuito)Para operadores lógicos como AND (&&) e OR (||), muitos compiladores implementam a avaliação em curto-circuito.Para E1 AND E2: Se E1 for falso, E2 não é avaliado, e a expressão toda é falsa.Para E1 OR E2: Se E1 for verdadeiro, E2 não é avaliado, e a expressão toda é verdadeira.Isso é implementado usando desvios condicionais.Exemplo: if (a < b AND c > d) then S;TAC com Curto-Circuito para E1 AND E2 (usado em if):Seja L_false_expr o rótulo para onde se desvia se a expressão booleana inteira for falsa, e L_next o rótulo para a instrução após o if (ou para o bloco else).      // Código para avaliar a < b
      T1 := a < b
      IF_FALSE T1 GOTO L_false_expr  // Se (a < b) é falso, toda a AND é falsa
      // Código para avaliar c > d
      T2 := c > d
      IF_FALSE T2 GOTO L_false_expr  // Se (c > d) é falso, toda a AND é falsa
      // Se chegou aqui, (a < b AND c > d) é verdadeiro
      // Código para S (bloco then)
      GOTO L_next_instr_after_if
L_false_expr:
      // Se houvesse um ELSE, seu código estaria aqui, ou simplesmente continua
L_next_instr_after_if:
      // Próxima instrução após o if

Exemplo: if (a < b OR c > d) then S;TAC com Curto-Circuito para E1 OR E2 (usado em if):Seja L_true_expr o rótulo para onde se desvia se a expressão booleana inteira for verdadeira.      // Código para avaliar a < b
      T1 := a < b
      IF T1 GOTO L_true_expr     // Se (a < b) é verdadeiro, toda a OR é verdadeira
      // Código para avaliar c > d
      T2 := c > d
      IF T2 GOTO L_true_expr     // Se (c > d) é verdadeiro, toda a OR é verdadeira
      // Se chegou aqui, (a < b OR c > d) é falso
      GOTO L_next_instr_after_if // Pula o bloco then
L_true_expr:
      // Código para S (bloco then)
L_next_instr_after_if:
      // Próxima instrução após o if

3.1.3 Expressões RelacionaisExpressões como a < b são frequentemente o núcleo de condições em desvios.O resultado de uma expressão relacional (true ou false) pode ser armazenado em uma temporária, ou a expressão pode ser usada diretamente em uma instrução de desvio condicional.Exemplo: flag := (x == y);T1 := x == y
flag := T1

Ou, se usada em um if: if (x == y) then ...IF x == y GOTO L_then_block

3.1.4 Uso de Variáveis TemporáriasVariáveis temporárias (e.g., T0, T1, T2, ...) são cruciais. Elas são geradas pelo compilador sempre que um resultado intermediário de uma subexpressão precisa ser armazenado antes de ser usado em outra operação. O gerenciamento de temporárias (criação de novas, e potencialmente reutilização) é uma tarefa importante do gerador de TAC.3.2 Comandos de AtribuiçãoPara um comando de atribuição id := expr;:Gerar código TAC para a expressão expr. Seja E_place a variável ou temporária que contém o valor de expr após sua avaliação.Emitir a instrução de cópia: id := E_place.Exemplo: count := (i + j) * 2;AST (Conceitual):      :=
     /  \
 count   *
        / \
       +   2
      / \
     i   j

TAC Gerado:T1 := i + j
T2 := T1 * 2
count := T2

3.3 Estruturas de Controle de FluxoEstruturas de controle de fluxo (condicionais, laços) são traduzidas usando rótulos e instruções de desvio (condicionais e incondicionais). Funções como novo_rotulo() são usadas para gerar rótulos únicos.3.3.1 if-thenFonte: if (condicao) then BlocoComandos;TAC:      // Código para avaliar 'condicao'
      // Seja T_cond o resultado (booleano) da condição
      // Ex: T_cond := expr_cond
L_inicio_if:
      IF_FALSE T_cond GOTO L_fim_if
      // Código TAC para BlocoComandos
L_fim_if:
      // Próxima instrução após o if

Exemplo: if (x > 0) then y := 1;T0 := x > 0
IF_FALSE T0 GOTO L0
y := 1
L0:

3.3.2 if-then-elseFonte: if (condicao) then BlocoComandos1 else BlocoComandos2;TAC:      // Código para avaliar 'condicao'
      // Seja T_cond o resultado
L_inicio_if_else:
      IF_FALSE T_cond GOTO L_else
      // Código TAC para BlocoComandos1 (bloco then)
      GOTO L_fim_if_else
L_else:
      // Código TAC para BlocoComandos2 (bloco else)
L_fim_if_else:
      // Próxima instrução

Exemplo: if (x > 0) then y := 1; else y := -1;T0 := x > 0
IF_FALSE T0 GOTO L_ELSE
y := 1
GOTO L_FIM_IF
L_ELSE:
y := -1
L_FIM_IF:

3.3.3 whileFonte: while (condicao) do BlocoComandos;TAC:L_inicio_while:
      // Código para avaliar 'condicao'
      // Seja T_cond o resultado
      IF_FALSE T_cond GOTO L_fim_while
      // Código TAC para BlocoComandos (corpo do loop)
      GOTO L_inicio_while
L_fim_while:
      // Próxima instrução

Exemplo: while (i < 10) do i := i + 1;L_INICIO_WHILE:
T0 := i < 10
IF_FALSE T0 GOTO L_FIM_WHILE
T1 := i + 1
i := T1
GOTO L_INICIO_WHILE
L_FIM_WHILE:

3.3.4 do-while (ou repeat-until em algumas linguagens)Fonte: do BlocoComandos while (condicao); (A condição é verificada no final)TAC:L_inicio_do_while:
      // Código TAC para BlocoComandos
      // Código para avaliar 'condicao'
      // Seja T_cond o resultado
      IF T_cond GOTO L_inicio_do_while
L_fim_do_while: // Opcional, pode não haver desvio explícito para fora se for a última instrução do bloco
      // Próxima instrução

Exemplo: do x := x - 1; while (x > 0);L_INICIO_DO_WHILE:
T0 := x - 1
x := T0
T1 := x > 0
IF T1 GOTO L_INICIO_DO_WHILE

3.3.5 forUm loop for é frequentemente traduzido para uma estrutura while equivalente antes da geração do TAC.Fonte: for (inicializacao; condicao; incremento) BlocoComandos;Equivalente a:inicializacao;
while (condicao) {
  BlocoComandos;
  incremento;
}

TAC (baseado na equivalência while):      // Código TAC para 'inicializacao'
L_inicio_for:
      // Código para avaliar 'condicao'
      // Seja T_cond o resultado
      IF_FALSE T_cond GOTO L_fim_for
      // Código TAC para BlocoComandos
      // Código TAC para 'incremento'
      GOTO L_inicio_for
L_fim_for:
      // Próxima instrução

Exemplo: for (i := 0; i < 5; i := i + 1) sum := sum + i;i := 0
L_FOR_INICIO:
T0 := i < 5
IF_FALSE T0 GOTO L_FOR_FIM
T1 := sum + i
sum := T1
T2 := i + 1
i := T2
GOTO L_FOR_INICIO
L_FOR_FIM:

3.4 Geração de Rótulos (Labels) e DesviosFunções como novo_rotulo() são essenciais para gerar nomes de rótulos únicos (e.g., L0, L1, L2, ...) que servem como destinos para as instruções de desvio (GOTO, IF ... GOTO). A gestão desses rótulos garante que os fluxos de controle sejam corretamente implementados.4. Tradução Direcionada por Sintaxe (SDD) para Geração de TACA Tradução Direcionada por Sintaxe (SDD) é uma técnica formal que associa regras semânticas (ações) às produções de uma gramática livre de contexto. Essas regras definem como os atributos dos nós da árvore sintática são calculados e como o código intermediário (ou outras informações semânticas) é gerado.Atributos dos Nós da Árvore Sintática:Atributos Sintetizados: São calculados a partir dos atributos dos filhos de um nó. A informação flui de baixo para cima na árvore. Exemplos: E.code (o código TAC para a expressão E), E.place (a variável temporária ou nome que armazena o resultado de E).Atributos Herdados: São calculados a partir dos atributos do pai e/ou dos irmãos de um nó. A informação flui de cima para baixo ou lateralmente na árvore. Exemplos: rótulos para desvios em estruturas de controle (S.next indicando o rótulo para o qual pular após a execução do comando S).Esquemas de Tradução:Um esquema de tradução é uma gramática com ações semânticas embutidas nas produções.Exemplo de SDD para Expressões Aritméticas:Considere a produção E -> E1 op E2.E.place: Atributo sintetizado que guarda o nome da temporária que armazena o resultado de E.E.code: Atributo sintetizado que guarda a sequência de instruções TAC para calcular E.gen(...): Função hipotética que emite uma instrução TAC e a anexa a uma lista global de instruções.Produção: E -> E1 + E2Regras Semânticas (Ações):{
  E.place = nova_temp(); // Cria uma nova temporária para o resultado de E
  // E.code é a concatenação do código de E1, E2, e a nova instrução
  // Assumindo que E1.code e E2.code já foram gerados (pela ordem de visitação da AST)
  gen(E.place, ':=', E1.place, '+', E2.place); // Emite: E.place := E1.place + E2.place
}

A geração de E.code pode ser implícita pela ordem de chamada da função gen. O atributo E.code acumularia as instruções.Exemplo de AST com Atributos (para a + b * c):Suponha que a AST seja percorrida em pós-ordem para expressões.              E (place=T3)  <-- T3 := T1 + T2
             / \
            /   \
  E (place=T1)   E (place=T2) <-- T2 := b * c
 /             / \
a             b   c

Geração de Código (ordem de gen chamadas):Ao visitar o nó *: T2 = nova_temp(); gen(T2, ':=', 'b', '*', 'c'); (E(b*c).place = T2)Ao visitar o nó +: T1 = 'a'; (supondo que a é E1.place) T3 = nova_temp(); gen(T3, ':=', T1, '+', T2); (E(a + b*c).place = T3)SDD para if (Cond) then S1 (simplificado):Atributos:Cond.true: Rótulo para onde pular se Cond for verdadeira (herdado ou gerado).Cond.false: Rótulo para onde pular se Cond for falsa (herdado ou gerado).S.code: Código TAC para o comando S.Produção: Stmt -> IF ( Cond ) THEN S1Regras Semânticas:{
  Cond.true = novo_rotulo();   // Rótulo para o início do bloco S1
  Cond.false = S1.next;        // Rótulo para depois de S1 (herdado ou igual a Stmt.next)
                               // (S1.next seria o rótulo para onde pular após S1)
  // Stmt.code = código_de_Cond (usando Cond.true, Cond.false) ||
  //             gen('LABEL', Cond.true) ||
  //             S1.code
  // O código para Cond já usaria Cond.true e Cond.false para os desvios.
  // Exemplo para Cond (se Cond é E1 < E2):
  //    gen('IF', E1.place, '<', E2.place, 'GOTO', Cond.true)
  //    gen('GOTO', Cond.false)
}

A implementação real envolve passar os rótulos corretos para as sub-árvores (Cond e S1) e concatenar o código gerado. Para IF_FALSE Cond.place GOTO L_false, L_false seria S1.next.5. Estruturas de Dados para TACAs instruções de três endereços podem ser armazenadas na memória do compilador de várias formas. As mais comuns são quádruplas, triplas e triplas indiretas.5.1 QuádruplasEsta é a representação mais comum e direta. Cada instrução TAC é representada como um objeto ou registro com quatro campos: (op, arg1, arg2, result).| Campo | Descrição || op | O operador da instrução (e.g., +, :=, GOTO, IF_LT). || arg1 | O primeiro operando (um ponteiro para a tabela de símbolos, uma constante, ou uma temporária). || arg2 | O segundo operando (similar a arg1). Pode ser nulo para operações unárias ou cópias. || result | O destino do resultado (similar a arg1). Para desvios, pode ser um rótulo. |Exemplo: x := y + z(ADD, y_ptr, z_ptr, x_ptr)

Exemplo: IF a < b GOTO L1(IF_LT, a_ptr, b_ptr, L1_label_obj)

Exemplo: t1 := -y(UMINUS, y_ptr, null, t1_temp_obj)

Vantagens das Quádruplas:Clareza: A instrução é auto-contida e fácil de entender.Facilidade de Otimização: As instruções podem ser movidas e reorganizadas durante a otimização de forma relativamente simples, pois o resultado é nomeado explicitamente.Depuração: Mais fáceis de inspecionar durante a depuração do compilador.Desvantagens das Quádruplas:Espaço: Podem ocupar mais espaço devido ao armazenamento explícito de nomes de temporárias.5.2 TriplasNas triplas, cada instrução tem apenas três campos: (op, arg1, arg2). Os resultados de subexpressões não são armazenados em temporárias explícitas na instrução, mas são referenciados pelo número (índice) da instrução que os calcula.Exemplo: x := a * b + c * dTAC correspondente:T1 := a * b
T2 := c * d
T3 := T1 + T2
x  := T3

Representação em Triplas:| # | op  | arg1 | arg2 | Comentário (Não faz parte da tripla) ||:-:|:----|:-----|:-----|:-----------------------------------||(0)| * | a  | b  | Calcula a * b                    ||(1)| * | c  | d  | Calcula c * d                    ||(2)| + | (0)| (1)| Calcula resultado(0) + resultado(1)||(3)| :=| x  | (2)| Atribui resultado(2) a x       |(0), (1), (2) são referências aos resultados das instruções nos índices 0, 1 e 2, respectivamente.Vantagens das Triplas:Compactação: Mais compactas, pois não há necessidade de gerar e armazenar nomes para todas as variáveis temporárias. O resultado é implícito pela posição da tripla.Detecção de Subexpressões Comuns: A forma como os resultados são referenciados pode facilitar a detecção de subexpressões comuns (se duas triplas idênticas forem geradas, elas calculam o mesmo valor).Desvantagens das Triplas:Dificuldade de Otimização: Mover instruções durante a otimização é complicado, pois todas as referências a elas (os índices) precisariam ser atualizadas. Isso torna otimizações que reordenam código mais difíceis.5.3 Triplas IndiretasAs triplas indiretas tentam combinar as vantagens das triplas (compactação) com a flexibilidade das quádruplas para otimizações. Consistem em duas estruturas:Uma lista de ponteiros para as triplas.As próprias triplas (como definidas acima).As otimizações que envolvem reordenamento de código manipulam a lista de ponteiros, em vez de mover as triplas em si. Isso evita a necessidade de atualizar referências de índice dentro das triplas.Exemplo:Lista de Instruções (Ponteiros):Instrucao[0] -> Ponteiro para Triple[idx_A]
Instrucao[1] -> Ponteiro para Triple[idx_B]
...

Triplas (armazenadas separadamente):Triple[idx_A] = (opA, arg1A, arg2A)
Triple[idx_B] = (opB, arg1B, arg2B)
...

Se a ordem das instruções idx_A e idx_B precisar ser trocada, apenas os ponteiros em Instrucao[0] e Instrucao[1] são trocados, sem modificar as triplas ou as referências dentro delas.Vantagens das Triplas Indiretas:Mantêm a compactação das triplas.Facilitam a reordenação de código para otimizações.Desvantagens das Triplas Indiretas:Indireção: Adicionam um nível de indireção no acesso às instruções, o que pode ter um pequeno custo de desempenho.Complexidade: Ligeiramente mais complexas de implementar do que quádruplas ou triplas simples.A escolha da representação depende dos objetivos do compilador. Quádruplas são frequentemente uma escolha popular devido à sua simplicidade e flexibilidade para otimizações, apesar do potencial uso maior de espaço.6. O Papel da Tabela de Símbolos na Geração do TACA Tabela de Símbolos (TS) é uma estrutura de dados essencial usada por quase todas as fases de um compilador. Durante a geração de Código de Três Endereços, ela desempenha papéis cruciais:Busca de Informações sobre Operandos:Quando um identificador (nome de variável, constante, nome de função) aparece no código fonte (e, portanto, na AST que está sendo traduzida), o gerador de TAC precisa consultar a TS.Tipo: Para verificar a validade semântica das operações (e.g., não se pode somar um string com um booleano diretamente) e para determinar o tipo de variáveis temporárias que podem precisar ser criadas.Escopo: Para resolver qual declaração de um nome está sendo referenciada, especialmente em linguagens com escopos aninhados. A TS armazena informações sobre onde cada nome é válido."Endereço" Simbólico: Para o TAC, o "endereço" de uma variável da linguagem fonte é o seu próprio nome (ou um identificador único associado a ele na TS). Para constantes, pode ser o valor literal ou um ponteiro para uma entrada na TS que descreve a constante.Offset/Localização: Embora o TAC em si seja mais abstrato, a TS pode já conter (ou estar em processo de calcular) informações sobre o offset de uma variável na pilha de ativação (para variáveis locais) ou seu endereço de memória (para variáveis globais). Essa informação é mais diretamente usada na geração de código objeto, mas o tipo e outras propriedades da TS são vitais para o TAC.Armazenamento e Gerenciamento de Variáveis Temporárias:Quando o gerador de TAC cria uma nova variável temporária (e.g., T1, T2), ele pode precisar registrar essa temporária.Informações como o tipo da temporária (inferido dos operandos da operação que a gerou) podem ser armazenadas. Isso é útil para fases subsequentes, como a otimização (que pode precisar saber o tipo para realizar certas transformações) ou a geração de código objeto (que pode precisar alocar registradores ou memória de tipos específicos).Alguns compiladores podem adicionar as temporárias à tabela de símbolos, tratando-as de forma similar a variáveis declaradas pelo usuário, mas com um escopo e tempo de vida controlados pelo compilador.Gerenciamento de Rótulos (Labels):Rótulos (L0, L1, ...) usados como destinos de desvios (GOTO, IF ... GOTO) também são símbolos.O gerador de TAC precisa garantir que os rótulos sejam únicos.A "definição" de um rótulo (a instrução TAC que ele marca) e seus "usos" (as instruções de desvio que pulam para ele) precisam ser consistentes.A TS ou uma estrutura de dados auxiliar pode ser usada para armazenar informações sobre rótulos, como o endereço (índice da instrução TAC) para o qual eles apontam. Isso é especialmente importante se a geração de código for feita em múltiplas passagens ou se os rótulos puderem ser referenciados antes de serem definidos (forward jumps).Exemplo de Interação com a TS ao gerar TAC para x := y + 10;:Processar y:Consultar TS: y existe? Qual seu tipo (e.g., inteiro, real)? (Suponha que y é um inteiro).y se torna arg1 na instrução TAC.Processar 10:Reconhecer 10 como uma constante inteira.10 se torna arg2.Gerar operação +:Criar uma nova temporária, T0 = nova_temp().Inferir o tipo de T0 (inteiro, baseado em y e 10). Opcionalmente, registrar T0 e seu tipo.Emitir: T0 := y + 10.Processar x (lado esquerdo da atribuição):Consultar TS: x existe? Qual seu tipo? (Suponha que x é um inteiro).Verificar se o tipo de T0 é compatível com o tipo de x para a atribuição.Gerar atribuição :=:Emitir: x := T0.A Tabela de Símbolos é, portanto, uma fonte de verdade indispensável para o gerador de TAC, fornecendo o contexto semântico necessário para traduzir corretamente as construções da linguagem fonte.7. Estudo de Caso: O Compilador BuritiO Compilador Buriti, disponível no GitHub (https://github.com/edwilsonferreira/compilador_buriti), é um projeto de compilador didático escrito em Java. Uma análise de sua estrutura pode fornecer insights sobre uma implementação prática da geração de Código de Três Endereços.Observação: A análise a seguir é baseada na estrutura de arquivos e nomes de classes/métodos presentes no repositório, inferindo a lógica de geração de TAC. Uma análise profunda do código Java em si não foi realizada.Localização Provável da Geração de TAC:No repositório, o pacote buriti.compilador.geracao e, especificamente, a classe GeradorDeCodigoTresEnderecos.java (ou um nome similar, se houver refatoração) seria o local principal onde a lógica de geração de TAC reside.Abordagem Adotada pelo Buriti (Inferida):Padrão Visitor: É altamente provável que o compilador Buriti utilize o padrão de projeto Visitor para percorrer a Árvore Sintática Abstrata (AST). A classe GeradorDeCodigoTresEnderecos implementaria uma interface Visitor (ou estenderia uma classe base Visitor) com métodos como:visit(NoIf noIf)visit(NoWhile noWhile)visit(NoAtribuicao noAtribuicao)visit(NoExpressaoAritmetica noExpr)visit(NoIdentificador noId)visit(NoLiteral noLiteral)etc., para cada tipo de nó relevante na AST da linguagem Buriti.Geração de Temporárias e Rótulos:A classe GeradorDeCodigoTresEnderecos provavelmente possui métodos internos para gerar nomes únicos para variáveis temporárias (e.g., private String novaTemporaria() { return "T" + contadorTemp++; }) e rótulos (e.g., private String novoRotulo() { return "L" + contadorRotulo++; }). Contadores internos (contadorTemp, contadorRotulo) garantiriam a unicidade.Representação do TAC:As instruções TAC podem ser representadas como objetos de uma classe específica (e.g., InstrucaoTAC ou Quadrupla) ou, de forma mais simples, como Strings.Pela natureza didática, é comum que inicialmente se use uma List<String> para armazenar o código TAC gerado, onde cada String é uma instrução formatada.No código do Buriti, a classe GeradorDeCodigoTresEnderecos.java parece utilizar uma List<String> codigo; para armazenar as instruções geradas, o que confirma a abordagem de Strings.Tradução de Construções da Linguagem Buriti para TAC (Exemplos Hipotéticos da Lógica):A linguagem Buriti possui palavras-chave em português (e.g., SE, ENTAO, SENAO, ENQUANTO, FACA).Comando SE (cond) ENTAO Bloco1 SENAO Bloco2 FIMSE:O método visit(NoIf noIf) faria algo como:// String condPlace = (String) noIf.getCondicao().accept(this); // Visita a condição, obtém a temp com o resultado
// String rotuloElse = novoRotulo();
// String rotuloFimSe = novoRotulo();
// codigo.add("IF_FALSE " + condPlace + " GOTO " + rotuloElse);
// noIf.getBlocoThen().accept(this); // Visita o bloco THEN
// codigo.add("GOTO " + rotuloFimSe);
// codigo.add("LABEL " + rotuloElse);
// if (noIf.getBlocoElse() != null) {
//     noIf.getBlocoElse().accept(this); // Visita o bloco ELSE
// }
// codigo.add("LABEL " + rotuloFimSe);

Comando ENQUANTO (cond) FACA Bloco FIMENQUANTO:O método visit(NoWhile noWhile) seguiria a lógica:// String rotuloInicioWhile = novoRotulo();
// String rotuloFimWhile = novoRotulo();
// codigo.add("LABEL " + rotuloInicioWhile);
// String condPlace = (String) noWhile.getCondicao().accept(this);
// codigo.add("IF_FALSE " + condPlace + " GOTO " + rotuloFimWhile);
// noWhile.getBlocoComandos().accept(this); // Visita o corpo do loop
// codigo.add("GOTO " + rotuloInicioWhile);
// codigo.add("LABEL " + rotuloFimWhile);

Expressão Aritmética expr1 op expr2 (e.g., a + b):O método visit(NoExpressaoAritmetica noExpr):// String place1 = (String) noExpr.getOperandoEsquerda().accept(this);
// String place2 = (String) noExpr.getOperandoDireita().accept(this);
// String tempResultado = novaTemporaria();
// String operador = noExpr.getOperador(); // ex: "+"
// codigo.add(tempResultado + " := " + place1 + " " + operador + " " + place2);
// return tempResultado; // Retorna o nome da temporária que guarda o resultado

Identificadores e Literais:Os métodos visit(NoIdentificador noId) e visit(NoLiteral noLiteral) simplesmente retornariam o nome do identificador ou o valor do literal como uma String, para serem usados como operandos.// visit(NoIdentificador noId) { return noId.getNome(); }
// visit(NoLiteralNumero noNum) { return String.valueOf(noNum.getValor()); }

Conexão com a Tabela de Símbolos:A classe GeradorDeCodigoTresEnderecos precisaria de acesso à Tabela de Símbolos (provavelmente passada no construtor ou acessível globalmente) para consultar tipos de variáveis, verificar declarações, etc., embora o foco principal da geração de TAC seja a estrutura do código.Conclusão sobre o Buriti:O Compilador Buriti parece adotar uma abordagem clássica de geração de TAC baseada no padrão Visitor, percorrendo a AST e emitindo instruções TAC como Strings. A lógica para cada construção da linguagem (SE-ENTAO-SENAO, ENQUANTO, atribuições, expressões) é encapsulada nos respectivos métodos visit. O uso de funções auxiliares para gerar temporárias e rótulos é uma prática padrão e esperada. Esta abordagem é didática e eficaz para ensinar os princípios da geração de código intermediário.8. Implementação PráticaPara ilustrar a implementação, forneceremos diretrizes e exemplos de código em Python. Python é escolhido por sua clareza e sintaxe concisa, comum em ambientes de ensino de compiladores para prototipagem.8.1 Estruturas de Dados Básicas e Gerador (Python)class TACInstruction:
    """Representa uma instrução TAC como uma quádrupla."""
    def __init__(self, op, arg1=None, arg2=None, result=None):
        self.op = op         # Operador (string, e.g., '+', ':=', 'GOTO', 'IF_LT')
        self.arg1 = arg1     # Primeiro operando (string: nome de var/temp, literal, ou None)
        self.arg2 = arg2     # Segundo operando (string ou None)
        self.result = result # Resultado/Destino (string: nome de var/temp, label, ou None)

    def __str__(self):
        if self.op == ':=':
            return f"{self.result} := {self.arg1}"
        elif self.op in ['+', '-', '*', '/']:
            return f"{self.result} := {self.arg1} {self.op} {self.arg2}"
        elif self.op == 'UMINUS': # Exemplo de operador unário
             return f"{self.result} := -{self.arg1}"
        elif self.op == 'LABEL':
            return f"{self.result}:"
        elif self.op == 'GOTO':
            return f"GOTO {self.result}"
        elif self.op.startswith('IF_'): # e.g., IF_LT, IF_EQ, IF_FALSE
            # Para IF_FALSE T1 GOTO L1: op='IF_FALSE', arg1='T1', result='L1'
            # Para IF T1 < T2 GOTO L1: op='IF_LT', arg1='T1', arg2='T2', result='L1' (alternativa)
            if self.arg2: # Forma IF arg1 op_cond arg2 GOTO result
                op_str = self.op.split('_')[1] # Extrai LT de IF_LT
                return f"IF {self.arg1} {op_str} {self.arg2} GOTO {self.result}"
            else: # Forma IF_FALSE arg1 GOTO result
                return f"{self.op} {self.arg1} GOTO {self.result}"
        elif self.op == 'PARAM':
            return f"PARAM {self.arg1}"
        elif self.op == 'CALL':
            if self.result: # Função com retorno: result := CALL arg1, arg2(num_params)
                return f"{self.result} := CALL {self.arg1}, {self.arg2}"
            else: # Procedimento sem retorno: CALL arg1, arg2(num_params)
                return f"CALL {self.arg1}, {self.arg2}"
        elif self.op == 'RETURN':
            return f"RETURN {self.arg1 if self.arg1 else ''}"
        else:
            return f"({self.op}, {self.arg1}, {self.arg2}, {self.result})" # Fallback genérico

class TACGenerator:
    def __init__(self):
        self.instructions = []  # Lista de objetos TACInstruction
        self.temp_counter = 0
        self.label_counter = 0
        # self.symbol_table = symbol_table # Referência à tabela de símbolos, se necessário

    def new_temp(self):
        temp_name = f"T{self.temp_counter}"
        self.temp_counter += 1
        # Opcional: adicionar temp à tabela de símbolos com seu tipo
        return temp_name

    def new_label(self):
        label_name = f"L{self.label_counter}"
        self.label_counter += 1
        return label_name

    def emit(self, op, arg1=None, arg2=None, result=None):
        instr = TACInstruction(op, arg1, arg2, result)
        self.instructions.append(instr)
        return instr # Pode ser útil retornar a instrução

    def get_code_str(self):
        return "\n".join(map(str, self.instructions))

    # Métodos de visitação da AST (simulados aqui)
    # Em um compilador real, estes seriam parte de um Visitor que percorre a AST.
    # Eles retornam o 'place' (nome da var/temp) onde o resultado da expressão está.

    def generate_for_expression(self, node):
        # Exemplo simplificado: nó da AST é um dicionário ou objeto
        if node['type'] == 'literal_int':
            return str(node['value'])
        elif node['type'] == 'identifier':
            return node['name']
        elif node['type'] == 'binary_op':
            # Avalia recursivamente os operandos
            place1 = self.generate_for_expression(node['left'])
            place2 = self.generate_for_expression(node['right'])
            # Cria uma nova temporária para o resultado
            result_temp = self.new_temp()
            # Emite a instrução TAC
            self.emit(node['op'], place1, place2, result_temp)
            return result_temp
        elif node['type'] == 'unary_op':
            place1 = self.generate_for_expression(node['operand'])
            result_temp = self.new_temp()
            self.emit(node['op'], place1, None, result_temp) # e.g., op='UMINUS'
            return result_temp
        # Adicionar outros tipos de expressão (chamada de função, acesso a array, etc.)
        raise NotImplementedError(f"Geração para expressão tipo {node['type']} não implementada.")

    def generate_for_assignment(self, node):
        # node = {'type': 'assignment', 'target': {'type':'identifier', 'name':'x'}, 
        #         'source': {'type':'binary_op', ...}}
        target_name = node['target']['name']
        source_place = self.generate_for_expression(node['source'])
        self.emit(':=', source_place, None, target_name)

    def generate_for_if_then_else(self, node):
        # node = {'type': 'if_stmt', 'condition': {...}, 
        #         'then_block': [{...}], 'else_block': [{...}] (opcional) }
        
        # Gerar código para a condição
        # Se a condição for simples como 'id < literal', pode ser otimizado
        # Aqui, assumimos que generate_for_expression retorna uma temp com valor booleano (0 ou 1)
        # ou que usamos uma forma de IF que avalia a expressão diretamente.
        # Para simplificar, vamos usar IF_FALSE.
        
        cond_place = self.generate_for_expression(node['condition'])
        
        if 'else_block' in node and node['else_block']:
            label_else = self.new_label()
            label_end_if = self.new_label()
            
            self.emit('IF_FALSE', cond_place, None, label_else) # Se cond_place é falso, pule para o else
            
            # Gerar código para o bloco THEN
            for stmt_node in node['then_block']:
                self.generate_for_statement(stmt_node)
            self.emit('GOTO', None, None, label_end_if) # Pula o bloco ELSE
            
            self.emit('LABEL', None, None, label_else)
            # Gerar código para o bloco ELSE
            for stmt_node in node['else_block']:
                self.generate_for_statement(stmt_node)
            
            self.emit('LABEL', None, None, label_end_if)
        else: # Apenas IF-THEN
            label_end_if = self.new_label()
            self.emit('IF_FALSE', cond_place, None, label_end_if) # Se cond_place é falso, pule o then
            
            # Gerar código para o bloco THEN
            for stmt_node in node['then_block']:
                self.generate_for_statement(stmt_node)
            
            self.emit('LABEL', None, None, label_end_if)

    def generate_for_while(self, node):
        # node = {'type': 'while_stmt', 'condition': {...}, 'body': [{...}]}
        label_begin_while = self.new_label()
        label_end_while = self.new_label()

        self.emit('LABEL', None, None, label_begin_while)
        cond_place = self.generate_for_expression(node['condition'])
        self.emit('IF_FALSE', cond_place, None, label_end_while)

        # Gerar código para o corpo do WHILE
        for stmt_node in node['body']:
            self.generate_for_statement(stmt_node)
        
        self.emit('GOTO', None, None, label_begin_while)
        self.emit('LABEL', None, None, label_end_while)

    def generate_for_statement(self, stmt_node):
        # Dispatcher para diferentes tipos de comandos
        if stmt_node['type'] == 'assignment':
            self.generate_for_assignment(stmt_node)
        elif stmt_node['type'] == 'if_stmt':
            self.generate_for_if_then_else(stmt_node)
        elif stmt_node['type'] == 'while_stmt':
            self.generate_for_while(stmt_node)
        # Adicionar outros tipos de comandos (print, read, call, etc.)
        else:
            raise NotImplementedError(f"Geração para comando tipo {stmt_node['type']} não implementada.")


8.2 Exemplo Completo de TraduçãoCódigo Fonte (Hipotético):VAR x, y, z : INTEGER;
BEGIN
  x := 10;
  y := 20;
  IF x < y THEN
    z := x + y;
  ELSE
    z := y - x;
  ENDIF;
  PRINT z; // Comando hipotético, não implementado no gerador acima
END.

Representação da AST (Simplificada, como dicionários Python para o gerador acima):ast_example = [
    {'type': 'assignment', 
     'target': {'type':'identifier', 'name':'x'}, 
     'source': {'type':'literal_int', 'value':10}},
    {'type': 'assignment', 
     'target': {'type':'identifier', 'name':'y'}, 
     'source': {'type':'literal_int', 'value':20}},
    {'type': 'if_stmt', 
     'condition': {'type': 'binary_op', 'op': '<',  # Usaremos '<' diretamente no IF
                   'left': {'type':'identifier', 'name':'x'},
                   'right': {'type':'identifier', 'name':'y'}},
     'then_block': [
         {'type': 'assignment',
          'target': {'type':'identifier', 'name':'z'},
          'source': {'type': 'binary_op', 'op': '+',
                     'left': {'type':'identifier', 'name':'x'},
                     'right': {'type':'identifier', 'name':'y'}}}
     ],
     'else_block': [
         {'type': 'assignment',
          'target': {'type':'identifier', 'name':'z'},
          'source': {'type': 'binary_op', 'op': '-',
                     'left': {'type':'identifier', 'name':'y'},
                     'right': {'type':'identifier', 'name':'x'}}}
     ]}
    # Comando PRINT z não modelado no gerador simples acima
]

# Modificação para a condição do IF para usar uma temporária explícita:
# A condição x < y seria:
# cond_expr_node = {'type': 'binary_op', 'op': '<', 
#                   'left': {'type':'identifier', 'name':'x'},
#                   'right': {'type':'identifier', 'name':'y'}}
# E o nó 'if_stmt' usaria o resultado disso.

# Para o exemplo de IF_FALSE T_cond GOTO L_else:
# Primeiro, a condição x < y precisa ser traduzida:
# T0 := x < y  (onde '<' é um operador que retorna booleano, ou 0/1)
# E então 'cond_place' seria T0.

# Vamos redefinir a condição do if_stmt para ser uma expressão que gera uma temporária
ast_example_if_cond_as_expr = {
    'type': 'binary_op', 'op': '<', 
    'left': {'type':'identifier', 'name':'x'},
    'right': {'type':'identifier', 'name':'y'}
}
ast_example[2]['condition'] = ast_example_if_cond_as_expr # Substitui a condição

Geração de TAC usando o Gerador Python:# Supondo que a AST está corretamente estruturada para o gerador:
tac_gen = TACGenerator()

# Simulação de geração para a condição do IF primeiro, para que cond_place seja uma temporária
# (generate_for_if_then_else espera que cond_place já seja o nome de uma temp/var)
# Isso é um pouco artificial aqui, pois a lógica de visitação da AST cuidaria disso.

# Vamos simular o processo:
# 1. x := 10
tac_gen.generate_for_statement(ast_example[0])
# 2. y := 20
tac_gen.generate_for_statement(ast_example[1])
# 3. IF x < y THEN ... ELSE ...
#    A chamada a generate_for_if_then_else internamente chamará
#    generate_for_expression para ast_example[2]['condition']
tac_gen.generate_for_statement(ast_example[2])


print("--- Código TAC Gerado ---")
print(tac_gen.get_code_str())

Saída TAC Esperada (Pode variar ligeiramente com a implementação exata do generate_for_expression para <):--- Código TAC Gerado ---
x := 10
y := 20
T0 := x < y
IF_FALSE T0 GOTO L0
T1 := x + y
z := T1
GOTO L1
L0:
T2 := y - x
z := T2
L1:

Nota: A implementação de generate_for_expression para operadores relacionais como < precisaria ser definida para retornar uma temporária que guarda o resultado booleano (e.g., 0 ou 1), ou a instrução IF_FALSE (ou IF_LT, etc.) seria construída de forma mais direta se generate_for_expression pudesse retornar os operandos originais para serem usados em um IF op GOTO.O exemplo de TACInstruction.__str__ já prevê IF {self.arg1} {op_str} {self.arg2} GOTO {self.result}. Se generate_for_expression para < não gerasse T0 := x < y mas passasse x, <, y para generate_for_if_then_else, este poderia emitir IF_LT x, y, GOTO L_THEN (ou IF_GE x, y, GOTO L_ELSE). A abordagem com T0 := x < y seguida de IF_FALSE T0 GOTO ... é mais geral.9. Exercícios PropostosExpressões Aritméticas Complexas:Traduza as seguintes expressões para Código de Três Endereços, mostrando explicitamente o uso de variáveis temporárias e respeitando a precedência e associatividade padrão dos operadores (multiplicação/divisão antes de adição/subtração, da esquerda para a direita para operadores de mesma precedência):a.  res = (a - b) * (c + d / e) - fb.  val = a * b * c + d - e / f / gc.  y = -x + (w / (z - 2)) * 5Expressões Booleanas com Curto-Circuito:Gere TAC para os seguintes comandos if, implementando a avaliação em curto-circuito para os operadores lógicos AND e OR:a.  if (idade >= 18 AND possui_cnh == true) then status := "apto"; endif;b.  if (nota < 5 OR faltas > 10) then resultado := "reprovado"; else resultado := "aprovado"; endif;c.  if ((a + b > c AND c != 0) OR d == 1) then x := 1; endif;Estruturas de Repetição (while, for):Traduza as seguintes estruturas de laço para TAC:a.  i := 0; soma := 0; while (i < 10 AND soma < 100) do soma := soma + i; i := i + 1; endwhile;b.  (Assuma que for passo P significa incremento de P)for k := 10 down_to 1 step 2 do // (down_to significa k := k - P) print(k); endfor;(Dica: Converta o for para uma estrutura while equivalente primeiro.)c.  fatorial := 1; num := 5; do fatorial := fatorial * num; num := num - 1; while (num > 0);Aninhamento de Estruturas:Gere o TAC para o seguinte trecho de código aninhado:a := 10;
b := 20;
while (a < b) do
  print(a);
  if (a % 2 == 0) then
    b := b - a / 2;
  else
    b := b - 1;
  endif;
  a := a + 1;
endwhile;
resultado_final := b;

Representações de TAC (Quádruplas vs. Triplas):Considere a expressão: expr = (x + y) - (x + y + z);a.  Gere o TAC para esta expressão.b.  Represente o TAC gerado usando Quádruplas (formato (op, arg1, arg2, result)).c.  Represente o mesmo TAC usando Triplas (formato (op, arg1, arg2) com referências a resultados anteriores).d.  Discuta como a otimização de "eliminação de subexpressão comum" poderia simplificar o TAC antes da representação. Mostre o TAC otimizado.Chamadas de Função/Procedimento:Gere TAC para o seguinte código, incluindo o tratamento de parâmetros e valor de retorno:function adicionar(a: integer, b: integer) : integer;
begin
  return a + b;
end;

procedure principal();
var res, val1, val2 : integer;
begin
  val1 := 5;
  val2 := 7;
  res := adicionar(val1, val2);
  print(res);
end;

(Assuma instruções TAC como PARAM, CALL nome, num_params, RETORNO valor, e var := CALL nome, num_params).
