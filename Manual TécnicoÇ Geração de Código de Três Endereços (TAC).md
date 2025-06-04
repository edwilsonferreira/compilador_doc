## **Manual Técnico: Geração de Código de Três Endereços (TAC)**

**Nota:** Este manual foi elaborado com o auxílio de inteligência artificial para fornecer uma base de conhecimento abrangente. Recomenda-se a consulta às obras de referência e a supervisão de um instrutor para um aprendizado completo e aprofundado.

### **1\. Introdução ao Código de Três Endereços**

#### **1.1 Definição e Propósito do TAC**

O **Código de Três Endereços (TAC)** é uma representação intermediária (Intermediate Representation \- IR) linearizada usada por compiladores. Ele se caracteriza por decompor as construções complexas da linguagem fonte em uma sequência de instruções simples, onde cada instrução possui, no máximo, três "endereços". Um endereço pode ser um nome (como o de uma variável da linguagem fonte), uma constante, ou uma variável temporária gerada pelo compilador.

O **propósito** fundamental do TAC é servir como uma ponte entre a análise semântica (frontend do compilador) e a geração de código objeto (backend do compilador). Ele oferece um nível de abstração que é mais próximo do código de máquina do que a Árvore Sintática Abstrata (AST), mas ainda é suficientemente independente da máquina alvo para permitir otimizações de código genéricas e facilitar a portabilidade do compilador para diferentes arquiteturas.

#### **1.2 Vantagens da Utilização do TAC em Compiladores**

A adoção do TAC como representação intermediária traz diversas vantagens significativas:

* **Independência de Máquina (Parcial):** Embora mais detalhado que a AST, o TAC ainda abstrai muitos detalhes específicos da arquitetura da máquina alvo, como registradores e modos de endereçamento complexos. Isso permite que otimizações sejam escritas de forma mais genérica.  
* **Facilidade de Otimização:** A estrutura regular e explícita das instruções TAC simplifica a identificação de padrões e a aplicação de diversas técnicas de otimização de código, como eliminação de subexpressões comuns, propagação de constantes, redução de força de operadores, e remoção de código morto.  
* **Simplificação da Geração de Código Objeto:** A tradução de TAC para código de máquina (ou assembly) é geralmente mais direta do que a tradução a partir de representações de mais alto nível como a AST. Cada instrução TAC pode ser mapeada para uma pequena sequência de instruções de máquina.  
* **Clareza e Depuração:** A forma explícita das operações e o uso de variáveis temporárias para resultados intermediários podem tornar o fluxo de computação mais claro, auxiliando na análise e depuração tanto do código gerado quanto do próprio compilador.  
* **Modularidade:** Permite uma separação clara entre o frontend (que produz TAC) e o backend (que consome TAC), facilitando o desenvolvimento e a manutenção do compilador.

#### **1.3 Formato Geral das Instruções TAC**

A forma mais comum de uma instrução TAC é a de uma atribuição com uma operação binária:  
resultado := operando1 op operando2  
Onde:

* resultado: É o endereço (variável ou temporária) que armazena o resultado da operação.  
* operando1, operando2: São os endereços dos operandos. Podem ser variáveis, constantes ou temporárias.  
* op: É um operador (e.g., aritmético como \+, \-, \*, /; lógico; relacional).

Existem variações e outros formatos importantes:

* **Atribuição com Operação Unária:** resultado := op operando1 (e.g., x := \-y)  
* **Cópia (Atribuição Simples):** resultado := operando1 (e.g., x := y)  
* **Desvio Incondicional:** GOTO L (onde L é um rótulo que identifica uma instrução)  
* **Desvio Condicional:** IF condicao GOTO L (e.g., IF x \< y GOTO L1) ou IF\_FALSE condicao GOTO L  
* **Chamada de Procedimento/Função:** PARAM x (passa o parâmetro x), CALL p, n (chama o procedimento p com n parâmetros), y := CALL p, n (se a função p retorna um valor que é atribuído a y).  
* **Acesso a Arrays Indexados:** x := y\[i\] (lê o valor do elemento i do array y para x), x\[i\] := y (escreve o valor de y no elemento i do array x).  
* **Declaração de Rótulos (Labels):** LABEL L (define o rótulo L para a próxima instrução).

#### **1.4 Tipos Comuns de Instruções TAC**

1. **Atribuição:**  
   * x := y (Cópia)  
   * x := op y (Operação unária, e.g., x := \-y, x := NOT z)  
2. **Operações Aritméticas Binárias:**  
   * x := y \+ z  
   * x := y \- z  
   * x := y \* z  
   * x := y / z  
   * x := y MOD z  
3. **Operações Lógicas:** (Frequentemente usadas na avaliação de expressões booleanas e desvios)  
   * x := y AND z  
   * x := y OR z  
   * (O operador NOT é geralmente unário, como em x := NOT y)  
4. **Operações Relacionais:** (Frequentemente usadas em desvios condicionais)  
   * t1 := x \< y (o resultado booleano é armazenado em t1)  
   * Seguido por: IF t1 GOTO L ou IF\_FALSE t1 GOTO L  
   * Alternativamente, a forma combinada: IF x \< y GOTO L  
5. **Desvios Incondicionais:**  
   * GOTO L  
6. **Desvios Condicionais:**  
   * IF x GOTO L (Se x é verdadeiro/diferente de zero, desvia para L)  
   * IF\_FALSE x GOTO L (Se x é falso/igual a zero, desvia para L)  
   * IF x relop y GOTO L (e.g., IF x \< y GOTO L1)  
7. **Chamada e Retorno de Procedimento/Função:**  
   * PARAM x (Define x como um parâmetro para a próxima chamada)  
   * CALL p, n (Chama o procedimento p que tem n parâmetros)  
   * y := CALL f, n (Chama a função f com n parâmetros e armazena o resultado em y)  
   * RETURN (Retorna de um procedimento sem valor)  
   * RETURN x (Retorna de uma função com o valor x)  
8. **Acesso a Array Indexado:**  
   * x := y\[i\] (Lê y\[i\] em x. Isso pode ser decomposto se i for uma expressão: t1 := i; t2 := y\[t1\]; x := t2)  
   * x\[i\] := y (Escreve y em x\[i\]. Similarmente, pode ser decomposto: t1 := i; x\[t1\] := y)  
9. **Declaração de Rótulos (Labels):**  
   * LABEL L ou L: (Marca a instrução seguinte com o rótulo L)

### **2\. Fundamentação Teórica**

A geração de Código de Três Endereços é uma etapa consolidada na teoria e prática de compiladores. Ela se baseia em conceitos extensivamente abordados em obras de referência na área:

* **"Compilers: Principles, Techniques, and Tools" por Aho, Sethi e Ullman (frequentemente chamado de "Livro do Dragão"):** Este é um texto clássico que detalha profundamente as representações intermediárias, incluindo o código de três endereços. O livro introduz esquemas de tradução direcionada por sintaxe (Syntax-Directed Translation \- SDT) para gerar TAC a partir de construções da linguagem fonte. Conceitos como atributos sintetizados e herdados dos nós da árvore sintática são fundamentais para guiar a geração de código.  
* **"Modern Compiler Implementation in Java/C/ML" por Andrew Appel:** Appel oferece uma abordagem prática e moderna para a construção de compiladores. Embora ele utilize uma representação intermediária baseada em árvores (IR Trees) que é posteriormente linearizada, os princípios de decomposição de expressões e controle de fluxo são muito similares aos do TAC. O livro enfatiza a importância de uma IR bem definida para facilitar as otimizações.  
* **"Engineering a Compiler" por Cooper e Torczon:** Este livro fornece uma perspectiva de engenharia sobre o design e implementação de compiladores, com uma discussão robusta sobre as várias formas de representações intermediárias, incluindo o TAC (e suas variantes como quádruplas e triplas), e como elas suportam as fases de análise e transformação de programas.

#### **Fluxo do Compilador e o Papel do TAC**

O TAC se encaixa no fluxo geral de um compilador como uma etapa intermediária crucial, posicionada entre a análise semântica e a geração de código objeto.

**Diagrama Conceitual do Fluxo de Compilação:**

        Código Fonte (Ex: programa.txt)  
              |  
              V  
      \+---------------------+  
      |   Análise Léxica           |  (Scanner)  
      | (Gera Tokens)             |  
      \+---------------------+  
              |  
              V  
      \+---------------------+  
      |  Análise Sintática        |  (Parser)  
      | (Gera Árvore               |  
      |  Sintática Abstrata     |  
      |  \- AST)                         |  
      \+---------------------+  
              |  
              V  
      \+---------------------+  
      |  Análise Semântica     |  
      | (Verifica Tipos,            |  
      |  Escopos, etc.              |  
      |  AST Anotada)             |  
      \+---------------------+  
              |  
              V  
\+-----------------------------------+  
| Geração de Código Intermediário   |  
| (Gera Código de Três Endereços    |  
|  \- TAC)                                                  |  
\+-----------------------------------+  
              |  
              V  
      \+---------------------+  
      | Otimização de Código|  
      | (Transforma TAC          |  
      |  para TAC Otimizado)|  
      \+---------------------+  
              |  
              V  
      \+---------------------+  
      | Geração de Código   |  
      | Objeto (Assembly ou |  
      | Código de Máquina)  |  
      \+---------------------+  
              |  
              V  
        Código Executável

1. **Análise Léxica (Scanner):** O código fonte é lido e dividido em uma sequência de tokens (palavras-chave, identificadores, operadores, literais).  
2. **Análise Sintática (Parser):** Os tokens são organizados em uma estrutura hierárquica, geralmente uma Árvore Sintática Abstrata (AST), que representa a estrutura gramatical do programa.  
3. **Análise Semântica:** A AST é percorrida para verificar a consistência semântica do programa, como a checagem de tipos, a resolução de nomes (escopos) e outras regras da linguagem. A AST pode ser anotada com informações de tipo e outras informações semânticas.  
4. **Geração de Código Intermediário (TAC):** A AST (geralmente anotada) é traduzida para uma representação linear de mais baixo nível, o Código de Três Endereços. É nesta fase que expressões complexas são quebradas em operações simples e estruturas de controle são convertidas em desvios e rótulos.  
5. **Otimização de Código:** O TAC é analisado e transformado para melhorar a eficiência do código resultante (e.g., torná-lo mais rápido ou menor) sem alterar sua semântica. Muitas otimizações são mais fáceis de aplicar sobre o TAC do que sobre a AST ou sobre o código de máquina final.  
6. **Geração de Código Objeto:** O TAC (otimizado) é traduzido para a linguagem de máquina da arquitetura alvo ou para uma linguagem de montagem (Assembly).

O TAC, portanto, desacopla o frontend (fases 1-4) do backend (fases 5-6), permitindo, por exemplo, que um mesmo frontend que produz TAC seja usado com diferentes backends para gerar código para múltiplas arquiteturas, ou que diferentes frontends para linguagens distintas possam compartilhar um backend otimizador comum se ambos produzirem uma forma compatível de TAC.

### **3\. Geração de TAC para Construções da Linguagem Fonte**

A geração de TAC é tipicamente realizada através de uma travessia da Árvore Sintática Abstrata (AST). Para cada tipo de nó na AST, existem regras específicas para gerar as instruções TAC correspondentes.

#### **3.1 Expressões**

##### **3.1.1 Expressões Aritméticas**

Expressões aritméticas são decompostas em uma sequência de operações binárias ou unárias, utilizando variáveis temporárias para armazenar resultados intermediários. A ordem de geração deve respeitar a precedência e associatividade dos operadores.

**Exemplo:** x \= (a \+ b) \* c \- d / e;

**AST Conceitual (simplificada):**

        :=  
       /  \\  
      x    \-  
          / \\  
         \* /  
        / \\ / \\  
       \+   c d e  
      / \\  
     a   b

**Geração de TAC (considerando precedência usual: \*,/ antes de \+,-, da esquerda para a direita):**

T1 := a \+ b       // Resultado de (a \+ b)  
T2 := T1 \* c      // Resultado de (a \+ b) \* c  
T3 := d / e       // Resultado de d / e  
T4 := T2 \- T3     // Resultado final da expressão  
x  := T4          // Atribuição a x

* T1, T2, T3, T4 são variáveis temporárias geradas pelo compilador. Uma função como nova\_temp() seria usada para criar essas temporárias.

##### **3.1.2 Expressões Booleanas (Curto-Circuito)**

Para operadores lógicos como AND (&&) e OR (||), muitos compiladores implementam a avaliação em **curto-circuito**.

* Para E1 AND E2: Se E1 for falso, E2 não é avaliado, e a expressão toda é falsa.  
* Para E1 OR E2: Se E1 for verdadeiro, E2 não é avaliado, e a expressão toda é verdadeira.

Isso é implementado usando desvios condicionais.

**Exemplo: if (a \< b AND c \> d) then S;**

TAC com Curto-Circuito para E1 AND E2 (usado em if):  
Seja L\_false\_expr o rótulo para onde se desvia se a expressão booleana inteira for falsa, e L\_next o rótulo para a instrução após o if (ou para o bloco else).  
      // Código para avaliar a \< b  
      T1 := a \< b  
      IF\_FALSE T1 GOTO L\_false\_expr  // Se (a \< b) é falso, toda a AND é falsa  
      // Código para avaliar c \> d  
      T2 := c \> d  
      IF\_FALSE T2 GOTO L\_false\_expr  // Se (c \> d) é falso, toda a AND é falsa  
      // Se chegou aqui, (a \< b AND c \> d) é verdadeiro  
      // Código para S (bloco then)  
      GOTO L\_next\_instr\_after\_if  
L\_false\_expr:  
      // Se houvesse um ELSE, seu código estaria aqui, ou simplesmente continua  
L\_next\_instr\_after\_if:  
      // Próxima instrução após o if

**Exemplo: if (a \< b OR c \> d) then S;**

TAC com Curto-Circuito para E1 OR E2 (usado em if):  
Seja L\_true\_expr o rótulo para onde se desvia se a expressão booleana inteira for verdadeira.  
      // Código para avaliar a \< b  
      T1 := a \< b  
      IF T1 GOTO L\_true\_expr     // Se (a \< b) é verdadeiro, toda a OR é verdadeira  
      // Código para avaliar c \> d  
      T2 := c \> d  
      IF T2 GOTO L\_true\_expr     // Se (c \> d) é verdadeiro, toda a OR é verdadeira  
      // Se chegou aqui, (a \< b OR c \> d) é falso  
      GOTO L\_next\_instr\_after\_if // Pula o bloco then  
L\_true\_expr:  
      // Código para S (bloco then)  
L\_next\_instr\_after\_if:  
      // Próxima instrução após o if

##### **3.1.3 Expressões Relacionais**

Expressões como a \< b são frequentemente o núcleo de condições em desvios.  
O resultado de uma expressão relacional (true ou false) pode ser armazenado em uma temporária, ou a expressão pode ser usada diretamente em uma instrução de desvio condicional.  
**Exemplo:** flag := (x \== y);

T1 := x \== y  
flag := T1

Ou, se usada em um if: if (x \== y) then ...

IF x \== y GOTO L\_then\_block

##### **3.1.4 Uso de Variáveis Temporárias**

Variáveis temporárias (e.g., T0, T1, T2, ...) são cruciais. Elas são geradas pelo compilador sempre que um resultado intermediário de uma subexpressão precisa ser armazenado antes de ser usado em outra operação. O gerenciamento de temporárias (criação de novas, e potencialmente reutilização) é uma tarefa importante do gerador de TAC.

#### **3.2 Comandos de Atribuição**

Para um comando de atribuição id := expr;:

1. Gerar código TAC para a expressão expr. Seja E\_place a variável ou temporária que contém o valor de expr após sua avaliação.  
2. Emitir a instrução de cópia: id := E\_place.

Exemplo: count := (i \+ j) \* 2;  
AST (Conceitual):  
      :=  
     /  \\  
 count   \*  
        / \\  
       \+   2  
      / \\  
     i   j

**TAC Gerado:**

T1 := i \+ j  
T2 := T1 \* 2  
count := T2

#### **3.3 Estruturas de Controle de Fluxo**

Estruturas de controle de fluxo (condicionais, laços) são traduzidas usando rótulos e instruções de desvio (condicionais e incondicionais). Funções como novo\_rotulo() são usadas para gerar rótulos únicos.

##### **3.3.1 if-then**

Fonte: if (condicao) then BlocoComandos;  
TAC:  
      // Código para avaliar 'condicao'  
      // Seja T\_cond o resultado (booleano) da condição  
      // Ex: T\_cond := expr\_cond  
L\_inicio\_if:  
      IF\_FALSE T\_cond GOTO L\_fim\_if  
      // Código TAC para BlocoComandos  
L\_fim\_if:  
      // Próxima instrução após o if

**Exemplo:** if (x \> 0\) then y := 1;

T0 := x \> 0  
IF\_FALSE T0 GOTO L0  
y := 1  
L0:

##### **3.3.2 if-then-else**

Fonte: if (condicao) then BlocoComandos1 else BlocoComandos2;  
TAC:  
      // Código para avaliar 'condicao'  
      // Seja T\_cond o resultado  
L\_inicio\_if\_else:  
      IF\_FALSE T\_cond GOTO L\_else  
      // Código TAC para BlocoComandos1 (bloco then)  
      GOTO L\_fim\_if\_else  
L\_else:  
      // Código TAC para BlocoComandos2 (bloco else)  
L\_fim\_if\_else:  
      // Próxima instrução

**Exemplo:** if (x \> 0\) then y := 1; else y := \-1;

T0 := x \> 0  
IF\_FALSE T0 GOTO L\_ELSE  
y := 1  
GOTO L\_FIM\_IF  
L\_ELSE:  
y := \-1  
L\_FIM\_IF:

##### **3.3.3 while**

Fonte: while (condicao) do BlocoComandos;  
TAC:  
L\_inicio\_while:  
      // Código para avaliar 'condicao'  
      // Seja T\_cond o resultado  
      IF\_FALSE T\_cond GOTO L\_fim\_while  
      // Código TAC para BlocoComandos (corpo do loop)  
      GOTO L\_inicio\_while  
L\_fim\_while:  
      // Próxima instrução

**Exemplo:** while (i \< 10\) do i := i \+ 1;

L\_INICIO\_WHILE:  
T0 := i \< 10  
IF\_FALSE T0 GOTO L\_FIM\_WHILE  
T1 := i \+ 1  
i := T1  
GOTO L\_INICIO\_WHILE  
L\_FIM\_WHILE:

##### **3.3.4 do-while (ou repeat-until em algumas linguagens)**

Fonte: do BlocoComandos while (condicao); (A condição é verificada no final)  
TAC:  
L\_inicio\_do\_while:  
      // Código TAC para BlocoComandos  
      // Código para avaliar 'condicao'  
      // Seja T\_cond o resultado  
      IF T\_cond GOTO L\_inicio\_do\_while  
L\_fim\_do\_while: // Opcional, pode não haver desvio explícito para fora se for a última instrução do bloco  
      // Próxima instrução

**Exemplo:** do x := x \- 1; while (x \> 0);

L\_INICIO\_DO\_WHILE:  
T0 := x \- 1  
x := T0  
T1 := x \> 0  
IF T1 GOTO L\_INICIO\_DO\_WHILE

##### **3.3.5 for**

Um loop for é frequentemente traduzido para uma estrutura while equivalente antes da geração do TAC.  
Fonte: for (inicializacao; condicao; incremento) BlocoComandos;  
Equivalente a:  
inicializacao;  
while (condicao) {  
  BlocoComandos;  
  incremento;  
}

**TAC (baseado na equivalência while):**

      // Código TAC para 'inicializacao'  
L\_inicio\_for:  
      // Código para avaliar 'condicao'  
      // Seja T\_cond o resultado  
      IF\_FALSE T\_cond GOTO L\_fim\_for  
      // Código TAC para BlocoComandos  
      // Código TAC para 'incremento'  
      GOTO L\_inicio\_for  
L\_fim\_for:  
      // Próxima instrução

**Exemplo:** for (i := 0; i \< 5; i := i \+ 1\) sum := sum \+ i;

i := 0  
L\_FOR\_INICIO:  
T0 := i \< 5  
IF\_FALSE T0 GOTO L\_FOR\_FIM  
T1 := sum \+ i  
sum := T1  
T2 := i \+ 1  
i := T2  
GOTO L\_FOR\_INICIO  
L\_FOR\_FIM:

#### **3.4 Geração de Rótulos (Labels) e Desvios**

Funções como novo\_rotulo() são essenciais para gerar nomes de rótulos únicos (e.g., L0, L1, L2, ...) que servem como destinos para as instruções de desvio (GOTO, IF ... GOTO). A gestão desses rótulos garante que os fluxos de controle sejam corretamente implementados.

### **4\. Tradução Direcionada por Sintaxe (SDD) para Geração de TAC**

A Tradução Direcionada por Sintaxe (SDD) é uma técnica formal que associa regras semânticas (ações) às produções de uma gramática livre de contexto. Essas regras definem como os atributos dos nós da árvore sintática são calculados e como o código intermediário (ou outras informações semânticas) é gerado.

**Atributos dos Nós da Árvore Sintática:**

* **Atributos Sintetizados:** São calculados a partir dos atributos dos filhos de um nó. A informação flui de baixo para cima na árvore. Exemplos: E.code (o código TAC para a expressão E), E.place (a variável temporária ou nome que armazena o resultado de E).  
* **Atributos Herdados:** São calculados a partir dos atributos do pai e/ou dos irmãos de um nó. A informação flui de cima para baixo ou lateralmente na árvore. Exemplos: rótulos para desvios em estruturas de controle (S.next indicando o rótulo para o qual pular após a execução do comando S).

Esquemas de Tradução:  
Um esquema de tradução é uma gramática com ações semânticas embutidas nas produções.  
Exemplo de SDD para Expressões Aritméticas:  
Considere a produção E \-\> E1 op E2.

* E.place: Atributo sintetizado que guarda o nome da temporária que armazena o resultado de E.  
* E.code: Atributo sintetizado que guarda a sequência de instruções TAC para calcular E.  
* gen(...): Função hipotética que emite uma instrução TAC e a anexa a uma lista global de instruções.

Produção: E \-\> E1 \+ E2  
Regras Semânticas (Ações):  
{  
  E.place \= nova\_temp(); // Cria uma nova temporária para o resultado de E  
  // E.code é a concatenação do código de E1, E2, e a nova instrução  
  // Assumindo que E1.code e E2.code já foram gerados (pela ordem de visitação da AST)  
  gen(E.place, ':=', E1.place, '+', E2.place); // Emite: E.place := E1.place \+ E2.place  
}

A geração de E.code pode ser implícita pela ordem de chamada da função gen. O atributo E.code acumularia as instruções.

Exemplo de AST com Atributos (para a \+ b \* c):  
Suponha que a AST seja percorrida em pós-ordem para expressões.  
              E (place=T3)  \<-- T3 := T1 \+ T2  
             / \\  
            /   \\  
  E (place=T1)   E (place=T2) \<-- T2 := b \* c  
 /             / \\  
a             b   c

**Geração de Código (ordem de gen chamadas):**

1. Ao visitar o nó \*: T2 \= nova\_temp(); gen(T2, ':=', 'b', '\*', 'c'); (E(b\*c).place \= T2)  
2. Ao visitar o nó \+: T1 \= 'a'; (supondo que a é E1.place) T3 \= nova\_temp(); gen(T3, ':=', T1, '+', T2); (E(a \+ b\*c).place \= T3)

SDD para if (Cond) then S1 (simplificado):  
Atributos:

* Cond.true: Rótulo para onde pular se Cond for verdadeira (herdado ou gerado).  
* Cond.false: Rótulo para onde pular se Cond for falsa (herdado ou gerado).  
* S.code: Código TAC para o comando S.

Produção: Stmt \-\> IF ( Cond ) THEN S1  
Regras Semânticas:  
{  
  Cond.true \= novo\_rotulo();   // Rótulo para o início do bloco S1  
  Cond.false \= S1.next;        // Rótulo para depois de S1 (herdado ou igual a Stmt.next)  
                               // (S1.next seria o rótulo para onde pular após S1)  
  // Stmt.code \= código\_de\_Cond (usando Cond.true, Cond.false) ||  
  //             gen('LABEL', Cond.true) ||  
  //             S1.code  
  // O código para Cond já usaria Cond.true e Cond.false para os desvios.  
  // Exemplo para Cond (se Cond é E1 \< E2):  
  //    gen('IF', E1.place, '\<', E2.place, 'GOTO', Cond.true)  
  //    gen('GOTO', Cond.false)  
}

A implementação real envolve passar os rótulos corretos para as sub-árvores (Cond e S1) e concatenar o código gerado. Para IF\_FALSE Cond.place GOTO L\_false, L\_false seria S1.next.

### **5\. Estruturas de Dados para TAC**

As instruções de três endereços podem ser armazenadas na memória do compilador de várias formas. As mais comuns são quádruplas, triplas e triplas indiretas.

#### **5.1 Quádruplas**

Esta é a representação mais comum e direta. Cada instrução TAC é representada como um objeto ou registro com quatro campos: (op, arg1, arg2, result).

| Campo | Descrição |
| :---- | :---- |
| op | O operador da instrução (e.g., \+, :=, GOTO, IF\_LT). |
| arg1 | O primeiro operando (um ponteiro para a tabela de símbolos, uma constante, ou uma temporária). |
| arg2 | O segundo operando (similar a arg1). Pode ser nulo para operações unárias ou cópias. |
| result | O destino do resultado (similar a arg1). Para desvios, pode ser um rótulo. |

**Exemplo: x := y \+ z**

(ADD, y\_ptr, z\_ptr, x\_ptr)

**Exemplo: IF a \< b GOTO L1**

(IF\_LT, a\_ptr, b\_ptr, L1\_label\_obj)

**Exemplo: t1 := \-y**

(UMINUS, y\_ptr, null, t1\_temp\_obj)

**Vantagens das Quádruplas:**

* **Clareza:** A instrução é auto-contida e fácil de entender.  
* **Facilidade de Otimização:** As instruções podem ser movidas e reorganizadas durante a otimização de forma relativamente simples, pois o resultado é nomeado explicitamente.  
* **Depuração:** Mais fáceis de inspecionar durante a depuração do compilador.

**Desvantagens das Quádruplas:**

* **Espaço:** Podem ocupar mais espaço devido ao armazenamento explícito de nomes de temporárias.

#### **5.2 Triplas**

Nas triplas, cada instrução tem apenas três campos: (op, arg1, arg2). Os resultados de subexpressões não são armazenados em temporárias explícitas na instrução, mas são referenciados pelo número (índice) da instrução que os calcula.

Exemplo: x := a \* b \+ c \* d  
TAC correspondente:  
T1 := a \* b  
T2 := c \* d  
T3 := T1 \+ T2  
x  := T3

Representação em Triplas:  
| \# | op | arg1 | arg2 | Comentário (Não faz parte da tripla) |  
|:-:|:----|:-----|:-----|:-----------------------------------|  
|(0)| \* | a | b | Calcula a \* b |  
|(1)| \* | c | d | Calcula c \* d |  
|(2)| \+ | (0)| (1)| Calcula resultado(0) \+ resultado(1)|  
|(3)| :=| x | (2)| Atribui resultado(2) a x |

* (0), (1), (2) são referências aos resultados das instruções nos índices 0, 1 e 2, respectivamente.

**Vantagens das Triplas:**

* **Compactação:** Mais compactas, pois não há necessidade de gerar e armazenar nomes para todas as variáveis temporárias. O resultado é implícito pela posição da tripla.  
* **Detecção de Subexpressões Comuns:** A forma como os resultados são referenciados pode facilitar a detecção de subexpressões comuns (se duas triplas idênticas forem geradas, elas calculam o mesmo valor).

**Desvantagens das Triplas:**

* **Dificuldade de Otimização:** Mover instruções durante a otimização é complicado, pois todas as referências a elas (os índices) precisariam ser atualizadas. Isso torna otimizações que reordenam código mais difíceis.

#### **5.3 Triplas Indiretas**

As triplas indiretas tentam combinar as vantagens das triplas (compactação) com a flexibilidade das quádruplas para otimizações. Consistem em duas estruturas:

1. Uma **lista de ponteiros** para as triplas.  
2. As **próprias triplas** (como definidas acima).

As otimizações que envolvem reordenamento de código manipulam a lista de ponteiros, em vez de mover as triplas em si. Isso evita a necessidade de atualizar referências de índice dentro das triplas.

Exemplo:  
Lista de Instruções (Ponteiros):  
Instrucao\[0\] \-\> Ponteiro para Triple\[idx\_A\]  
Instrucao\[1\] \-\> Ponteiro para Triple\[idx\_B\]  
...

**Triplas (armazenadas separadamente):**

Triple\[idx\_A\] \= (opA, arg1A, arg2A)  
Triple\[idx\_B\] \= (opB, arg1B, arg2B)  
...

Se a ordem das instruções idx\_A e idx\_B precisar ser trocada, apenas os ponteiros em Instrucao\[0\] e Instrucao\[1\] são trocados, sem modificar as triplas ou as referências dentro delas.

**Vantagens das Triplas Indiretas:**

* Mantêm a compactação das triplas.  
* Facilitam a reordenação de código para otimizações.

**Desvantagens das Triplas Indiretas:**

* **Indireção:** Adicionam um nível de indireção no acesso às instruções, o que pode ter um pequeno custo de desempenho.  
* **Complexidade:** Ligeiramente mais complexas de implementar do que quádruplas ou triplas simples.

A escolha da representação depende dos objetivos do compilador. **Quádruplas** são frequentemente uma escolha popular devido à sua simplicidade e flexibilidade para otimizações, apesar do potencial uso maior de espaço.

### **6\. O Papel da Tabela de Símbolos na Geração do TAC**

A **Tabela de Símbolos (TS)** é uma estrutura de dados essencial usada por quase todas as fases de um compilador. Durante a geração de Código de Três Endereços, ela desempenha papéis cruciais:

1. **Busca de Informações sobre Operandos:**  
   * Quando um identificador (nome de variável, constante, nome de função) aparece no código fonte (e, portanto, na AST que está sendo traduzida), o gerador de TAC precisa consultar a TS.  
   * **Tipo:** Para verificar a validade semântica das operações (e.g., não se pode somar um string com um booleano diretamente) e para determinar o tipo de variáveis temporárias que podem precisar ser criadas.  
   * **Escopo:** Para resolver qual declaração de um nome está sendo referenciada, especialmente em linguagens com escopos aninhados. A TS armazena informações sobre onde cada nome é válido.  
   * **"Endereço" Simbólico:** Para o TAC, o "endereço" de uma variável da linguagem fonte é o seu próprio nome (ou um identificador único associado a ele na TS). Para constantes, pode ser o valor literal ou um ponteiro para uma entrada na TS que descreve a constante.  
   * **Offset/Localização:** Embora o TAC em si seja mais abstrato, a TS pode já conter (ou estar em processo de calcular) informações sobre o offset de uma variável na pilha de ativação (para variáveis locais) ou seu endereço de memória (para variáveis globais). Essa informação é mais diretamente usada na geração de código objeto, mas o tipo e outras propriedades da TS são vitais para o TAC.  
2. **Armazenamento e Gerenciamento de Variáveis Temporárias:**  
   * Quando o gerador de TAC cria uma nova variável temporária (e.g., T1, T2), ele pode precisar registrar essa temporária.  
   * Informações como o **tipo** da temporária (inferido dos operandos da operação que a gerou) podem ser armazenadas. Isso é útil para fases subsequentes, como a otimização (que pode precisar saber o tipo para realizar certas transformações) ou a geração de código objeto (que pode precisar alocar registradores ou memória de tipos específicos).  
   * Alguns compiladores podem adicionar as temporárias à tabela de símbolos, tratando-as de forma similar a variáveis declaradas pelo usuário, mas com um escopo e tempo de vida controlados pelo compilador.  
3. **Gerenciamento de Rótulos (Labels):**  
   * Rótulos (L0, L1, ...) usados como destinos de desvios (GOTO, IF ... GOTO) também são símbolos.  
   * O gerador de TAC precisa garantir que os rótulos sejam únicos.  
   * A "definição" de um rótulo (a instrução TAC que ele marca) e seus "usos" (as instruções de desvio que pulam para ele) precisam ser consistentes.  
   * A TS ou uma estrutura de dados auxiliar pode ser usada para armazenar informações sobre rótulos, como o endereço (índice da instrução TAC) para o qual eles apontam. Isso é especialmente importante se a geração de código for feita em múltiplas passagens ou se os rótulos puderem ser referenciados antes de serem definidos (forward jumps).

**Exemplo de Interação com a TS ao gerar TAC para x := y \+ 10;:**

1. **Processar y:**  
   * Consultar TS: y existe? Qual seu tipo (e.g., inteiro, real)? (Suponha que y é um inteiro).  
   * y se torna arg1 na instrução TAC.  
2. **Processar 10:**  
   * Reconhecer 10 como uma constante inteira.  
   * 10 se torna arg2.  
3. **Gerar operação \+:**  
   * Criar uma nova temporária, T0 \= nova\_temp().  
   * Inferir o tipo de T0 (inteiro, baseado em y e 10). Opcionalmente, registrar T0 e seu tipo.  
   * Emitir: T0 := y \+ 10\.  
4. **Processar x (lado esquerdo da atribuição):**  
   * Consultar TS: x existe? Qual seu tipo? (Suponha que x é um inteiro).  
   * Verificar se o tipo de T0 é compatível com o tipo de x para a atribuição.  
5. **Gerar atribuição :=:**  
   * Emitir: x := T0.

A Tabela de Símbolos é, portanto, uma fonte de verdade indispensável para o gerador de TAC, fornecendo o contexto semântico necessário para traduzir corretamente as construções da linguagem fonte.

### **7\. Estudo de Caso: O Compilador Buriti**

O Compilador Buriti, disponível no GitHub ([https://github.com/edwilsonferreira/compilador\_buriti](https://github.com/edwilsonferreira/compilador_buriti)), é um projeto de compilador didático escrito em Java. Uma análise de sua estrutura pode fornecer insights sobre uma implementação prática da geração de Código de Três Endereços.

**Observação:** A análise a seguir é baseada na estrutura de arquivos e nomes de classes/métodos presentes no repositório, inferindo a lógica de geração de TAC. Uma análise profunda do código Java em si não foi realizada.

Localização Provável da Geração de TAC:  
No repositório, o pacote buriti.compilador.geracao e, especificamente, a classe GeradorDeCodigoTresEnderecos.java (ou um nome similar, se houver refatoração) seria o local principal onde a lógica de geração de TAC reside.  
**Abordagem Adotada pelo Buriti (Inferida):**

1. **Padrão Visitor:** É altamente provável que o compilador Buriti utilize o padrão de projeto *Visitor* para percorrer a Árvore Sintática Abstrata (AST). A classe GeradorDeCodigoTresEnderecos implementaria uma interface Visitor (ou estenderia uma classe base Visitor) com métodos como:  
   * visit(NoIf noIf)  
   * visit(NoWhile noWhile)  
   * visit(NoAtribuicao noAtribuicao)  
   * visit(NoExpressaoAritmetica noExpr)  
   * visit(NoIdentificador noId)  
   * visit(NoLiteral noLiteral)  
   * etc., para cada tipo de nó relevante na AST da linguagem Buriti.  
2. **Geração de Temporárias e Rótulos:**  
   * A classe GeradorDeCodigoTresEnderecos provavelmente possui métodos internos para gerar nomes únicos para variáveis temporárias (e.g., private String novaTemporaria() { return "T" \+ contadorTemp++; }) e rótulos (e.g., private String novoRotulo() { return "L" \+ contadorRotulo++; }). Contadores internos (contadorTemp, contadorRotulo) garantiriam a unicidade.  
3. **Representação do TAC:**  
   * As instruções TAC podem ser representadas como objetos de uma classe específica (e.g., InstrucaoTAC ou Quadrupla) ou, de forma mais simples, como Strings.  
   * Pela natureza didática, é comum que inicialmente se use uma List\<String\> para armazenar o código TAC gerado, onde cada String é uma instrução formatada.  
   * No código do Buriti, a classe GeradorDeCodigoTresEnderecos.java parece utilizar uma List\<String\> codigo; para armazenar as instruções geradas, o que confirma a abordagem de Strings.  
4. Tradução de Construções da Linguagem Buriti para TAC (Exemplos Hipotéticos da Lógica):  
   A linguagem Buriti possui palavras-chave em português (e.g., SE, ENTAO, SENAO, ENQUANTO, FACA).  
   * Comando SE (cond) ENTAO Bloco1 SENAO Bloco2 FIMSE:  
     O método visit(NoIf noIf) faria algo como:  
     // String condPlace \= (String) noIf.getCondicao().accept(this); // Visita a condição, obtém a temp com o resultado  
     // String rotuloElse \= novoRotulo();  
     // String rotuloFimSe \= novoRotulo();  
     // codigo.add("IF\_FALSE " \+ condPlace \+ " GOTO " \+ rotuloElse);  
     // noIf.getBlocoThen().accept(this); // Visita o bloco THEN  
     // codigo.add("GOTO " \+ rotuloFimSe);  
     // codigo.add("LABEL " \+ rotuloElse);  
     // if (noIf.getBlocoElse() \!= null) {  
     //     noIf.getBlocoElse().accept(this); // Visita o bloco ELSE  
     // }  
     // codigo.add("LABEL " \+ rotuloFimSe);

   * Comando ENQUANTO (cond) FACA Bloco FIMENQUANTO:  
     O método visit(NoWhile noWhile) seguiria a lógica:  
     // String rotuloInicioWhile \= novoRotulo();  
     // String rotuloFimWhile \= novoRotulo();  
     // codigo.add("LABEL " \+ rotuloInicioWhile);  
     // String condPlace \= (String) noWhile.getCondicao().accept(this);  
     // codigo.add("IF\_FALSE " \+ condPlace \+ " GOTO " \+ rotuloFimWhile);  
     // noWhile.getBlocoComandos().accept(this); // Visita o corpo do loop  
     // codigo.add("GOTO " \+ rotuloInicioWhile);  
     // codigo.add("LABEL " \+ rotuloFimWhile);

   * Expressão Aritmética expr1 op expr2 (e.g., a \+ b):  
     O método visit(NoExpressaoAritmetica noExpr):  
     // String place1 \= (String) noExpr.getOperandoEsquerda().accept(this);  
     // String place2 \= (String) noExpr.getOperandoDireita().accept(this);  
     // String tempResultado \= novaTemporaria();  
     // String operador \= noExpr.getOperador(); // ex: "+"  
     // codigo.add(tempResultado \+ " := " \+ place1 \+ " " \+ operador \+ " " \+ place2);  
     // return tempResultado; // Retorna o nome da temporária que guarda o resultado

   * Identificadores e Literais:  
     Os métodos visit(NoIdentificador noId) e visit(NoLiteral noLiteral) simplesmente retornariam o nome do identificador ou o valor do literal como uma String, para serem usados como operandos.  
     // visit(NoIdentificador noId) { return noId.getNome(); }  
     // visit(NoLiteralNumero noNum) { return String.valueOf(noNum.getValor()); }

Conexão com a Tabela de Símbolos:  
A classe GeradorDeCodigoTresEnderecos precisaria de acesso à Tabela de Símbolos (provavelmente passada no construtor ou acessível globalmente) para consultar tipos de variáveis, verificar declarações, etc., embora o foco principal da geração de TAC seja a estrutura do código.  
Conclusão sobre o Buriti:  
O Compilador Buriti parece adotar uma abordagem clássica de geração de TAC baseada no padrão Visitor, percorrendo a AST e emitindo instruções TAC como Strings. A lógica para cada construção da linguagem (SE-ENTAO-SENAO, ENQUANTO, atribuições, expressões) é encapsulada nos respectivos métodos visit. O uso de funções auxiliares para gerar temporárias e rótulos é uma prática padrão e esperada. Esta abordagem é didática e eficaz para ensinar os princípios da geração de código intermediário.

### **8\. Implementação Prática**

Para ilustrar a implementação, forneceremos diretrizes e exemplos de código em Python. Python é escolhido por sua clareza e sintaxe concisa, comum em ambientes de ensino de compiladores para prototipagem.

#### **8.1 Estruturas de Dados Básicas e Gerador (Python)**

class TACInstruction:  
    """Representa uma instrução TAC como uma quádrupla."""  
    def \_\_init\_\_(self, op, arg1=None, arg2=None, result=None):  
        self.op \= op         \# Operador (string, e.g., '+', ':=', 'GOTO', 'IF\_LT')  
        self.arg1 \= arg1     \# Primeiro operando (string: nome de var/temp, literal, ou None)  
        self.arg2 \= arg2     \# Segundo operando (string ou None)  
        self.result \= result \# Resultado/Destino (string: nome de var/temp, label, ou None)

    def \_\_str\_\_(self):  
        if self.op \== ':=':  
            return f"{self.result} := {self.arg1}"  
        elif self.op in \['+', '-', '\*', '/'\]:  
            return f"{self.result} := {self.arg1} {self.op} {self.arg2}"  
        elif self.op \== 'UMINUS': \# Exemplo de operador unário  
             return f"{self.result} := \-{self.arg1}"  
        elif self.op \== 'LABEL':  
            return f"{self.result}:"  
        elif self.op \== 'GOTO':  
            return f"GOTO {self.result}"  
        elif self.op.startswith('IF\_'): \# e.g., IF\_LT, IF\_EQ, IF\_FALSE  
            \# Para IF\_FALSE T1 GOTO L1: op='IF\_FALSE', arg1='T1', result='L1'  
            \# Para IF T1 \< T2 GOTO L1: op='IF\_LT', arg1='T1', arg2='T2', result='L1' (alternativa)  
            if self.arg2: \# Forma IF arg1 op\_cond arg2 GOTO result  
                op\_str \= self.op.split('\_')\[1\] \# Extrai LT de IF\_LT  
                return f"IF {self.arg1} {op\_str} {self.arg2} GOTO {self.result}"  
            else: \# Forma IF\_FALSE arg1 GOTO result  
                return f"{self.op} {self.arg1} GOTO {self.result}"  
        elif self.op \== 'PARAM':  
            return f"PARAM {self.arg1}"  
        elif self.op \== 'CALL':  
            if self.result: \# Função com retorno: result := CALL arg1, arg2(num\_params)  
                return f"{self.result} := CALL {self.arg1}, {self.arg2}"  
            else: \# Procedimento sem retorno: CALL arg1, arg2(num\_params)  
                return f"CALL {self.arg1}, {self.arg2}"  
        elif self.op \== 'RETURN':  
            return f"RETURN {self.arg1 if self.arg1 else ''}"  
        else:  
            return f"({self.op}, {self.arg1}, {self.arg2}, {self.result})" \# Fallback genérico

class TACGenerator:  
    def \_\_init\_\_(self):  
        self.instructions \= \[\]  \# Lista de objetos TACInstruction  
        self.temp\_counter \= 0  
        self.label\_counter \= 0  
        \# self.symbol\_table \= symbol\_table \# Referência à tabela de símbolos, se necessário

    def new\_temp(self):  
        temp\_name \= f"T{self.temp\_counter}"  
        self.temp\_counter \+= 1  
        \# Opcional: adicionar temp à tabela de símbolos com seu tipo  
        return temp\_name

    def new\_label(self):  
        label\_name \= f"L{self.label\_counter}"  
        self.label\_counter \+= 1  
        return label\_name

    def emit(self, op, arg1=None, arg2=None, result=None):  
        instr \= TACInstruction(op, arg1, arg2, result)  
        self.instructions.append(instr)  
        return instr \# Pode ser útil retornar a instrução

    def get\_code\_str(self):  
        return "\\n".join(map(str, self.instructions))

    \# Métodos de visitação da AST (simulados aqui)  
    \# Em um compilador real, estes seriam parte de um Visitor que percorre a AST.  
    \# Eles retornam o 'place' (nome da var/temp) onde o resultado da expressão está.

    def generate\_for\_expression(self, node):  
        \# Exemplo simplificado: nó da AST é um dicionário ou objeto  
        if node\['type'\] \== 'literal\_int':  
            return str(node\['value'\])  
        elif node\['type'\] \== 'identifier':  
            return node\['name'\]  
        elif node\['type'\] \== 'binary\_op':  
            \# Avalia recursivamente os operandos  
            place1 \= self.generate\_for\_expression(node\['left'\])  
            place2 \= self.generate\_for\_expression(node\['right'\])  
            \# Cria uma nova temporária para o resultado  
            result\_temp \= self.new\_temp()  
            \# Emite a instrução TAC  
            self.emit(node\['op'\], place1, place2, result\_temp)  
            return result\_temp  
        elif node\['type'\] \== 'unary\_op':  
            place1 \= self.generate\_for\_expression(node\['operand'\])  
            result\_temp \= self.new\_temp()  
            self.emit(node\['op'\], place1, None, result\_temp) \# e.g., op='UMINUS'  
            return result\_temp  
        \# Adicionar outros tipos de expressão (chamada de função, acesso a array, etc.)  
        raise NotImplementedError(f"Geração para expressão tipo {node\['type'\]} não implementada.")

    def generate\_for\_assignment(self, node):  
        \# node \= {'type': 'assignment', 'target': {'type':'identifier', 'name':'x'},   
        \#         'source': {'type':'binary\_op', ...}}  
        target\_name \= node\['target'\]\['name'\]  
        source\_place \= self.generate\_for\_expression(node\['source'\])  
        self.emit(':=', source\_place, None, target\_name)

    def generate\_for\_if\_then\_else(self, node):  
        \# node \= {'type': 'if\_stmt', 'condition': {...},   
        \#         'then\_block': \[{...}\], 'else\_block': \[{...}\] (opcional) }  
          
        \# Gerar código para a condição  
        \# Se a condição for simples como 'id \< literal', pode ser otimizado  
        \# Aqui, assumimos que generate\_for\_expression retorna uma temp com valor booleano (0 ou 1\)  
        \# ou que usamos uma forma de IF que avalia a expressão diretamente.  
        \# Para simplificar, vamos usar IF\_FALSE.  
          
        cond\_place \= self.generate\_for\_expression(node\['condition'\])  
          
        if 'else\_block' in node and node\['else\_block'\]:  
            label\_else \= self.new\_label()  
            label\_end\_if \= self.new\_label()  
              
            self.emit('IF\_FALSE', cond\_place, None, label\_else) \# Se cond\_place é falso, pule para o else  
              
            \# Gerar código para o bloco THEN  
            for stmt\_node in node\['then\_block'\]:  
                self.generate\_for\_statement(stmt\_node)  
            self.emit('GOTO', None, None, label\_end\_if) \# Pula o bloco ELSE  
              
            self.emit('LABEL', None, None, label\_else)  
            \# Gerar código para o bloco ELSE  
            for stmt\_node in node\['else\_block'\]:  
                self.generate\_for\_statement(stmt\_node)  
              
            self.emit('LABEL', None, None, label\_end\_if)  
        else: \# Apenas IF-THEN  
            label\_end\_if \= self.new\_label()  
            self.emit('IF\_FALSE', cond\_place, None, label\_end\_if) \# Se cond\_place é falso, pule o then  
              
            \# Gerar código para o bloco THEN  
            for stmt\_node in node\['then\_block'\]:  
                self.generate\_for\_statement(stmt\_node)  
              
            self.emit('LABEL', None, None, label\_end\_if)

    def generate\_for\_while(self, node):  
        \# node \= {'type': 'while\_stmt', 'condition': {...}, 'body': \[{...}\]}  
        label\_begin\_while \= self.new\_label()  
        label\_end\_while \= self.new\_label()

        self.emit('LABEL', None, None, label\_begin\_while)  
        cond\_place \= self.generate\_for\_expression(node\['condition'\])  
        self.emit('IF\_FALSE', cond\_place, None, label\_end\_while)

        \# Gerar código para o corpo do WHILE  
        for stmt\_node in node\['body'\]:  
            self.generate\_for\_statement(stmt\_node)  
          
        self.emit('GOTO', None, None, label\_begin\_while)  
        self.emit('LABEL', None, None, label\_end\_while)

    def generate\_for\_statement(self, stmt\_node):  
        \# Dispatcher para diferentes tipos de comandos  
        if stmt\_node\['type'\] \== 'assignment':  
            self.generate\_for\_assignment(stmt\_node)  
        elif stmt\_node\['type'\] \== 'if\_stmt':  
            self.generate\_for\_if\_then\_else(stmt\_node)  
        elif stmt\_node\['type'\] \== 'while\_stmt':  
            self.generate\_for\_while(stmt\_node)  
        \# Adicionar outros tipos de comandos (print, read, call, etc.)  
        else:  
            raise NotImplementedError(f"Geração para comando tipo {stmt\_node\['type'\]} não implementada.")

#### **8.2 Exemplo Completo de Tradução**

**Código Fonte (Hipotético):**

VAR x, y, z : INTEGER;  
BEGIN  
  x := 10;  
  y := 20;  
  IF x \< y THEN  
    z := x \+ y;  
  ELSE  
    z := y \- x;  
  ENDIF;  
  PRINT z; // Comando hipotético, não implementado no gerador acima  
END.

**Representação da AST (Simplificada, como dicionários Python para o gerador acima):**

ast\_example \= \[  
    {'type': 'assignment',   
     'target': {'type':'identifier', 'name':'x'},   
     'source': {'type':'literal\_int', 'value':10}},  
    {'type': 'assignment',   
     'target': {'type':'identifier', 'name':'y'},   
     'source': {'type':'literal\_int', 'value':20}},  
    {'type': 'if\_stmt',   
     'condition': {'type': 'binary\_op', 'op': '\<',  \# Usaremos '\<' diretamente no IF  
                   'left': {'type':'identifier', 'name':'x'},  
                   'right': {'type':'identifier', 'name':'y'}},  
     'then\_block': \[  
         {'type': 'assignment',  
          'target': {'type':'identifier', 'name':'z'},  
          'source': {'type': 'binary\_op', 'op': '+',  
                     'left': {'type':'identifier', 'name':'x'},  
                     'right': {'type':'identifier', 'name':'y'}}}  
     \],  
     'else\_block': \[  
         {'type': 'assignment',  
          'target': {'type':'identifier', 'name':'z'},  
          'source': {'type': 'binary\_op', 'op': '-',  
                     'left': {'type':'identifier', 'name':'y'},  
                     'right': {'type':'identifier', 'name':'x'}}}  
     \]}  
    \# Comando PRINT z não modelado no gerador simples acima  
\]

\# Modificação para a condição do IF para usar uma temporária explícita:  
\# A condição x \< y seria:  
\# cond\_expr\_node \= {'type': 'binary\_op', 'op': '\<',   
\#                   'left': {'type':'identifier', 'name':'x'},  
\#                   'right': {'type':'identifier', 'name':'y'}}  
\# E o nó 'if\_stmt' usaria o resultado disso.

\# Para o exemplo de IF\_FALSE T\_cond GOTO L\_else:  
\# Primeiro, a condição x \< y precisa ser traduzida:  
\# T0 := x \< y  (onde '\<' é um operador que retorna booleano, ou 0/1)  
\# E então 'cond\_place' seria T0.

\# Vamos redefinir a condição do if\_stmt para ser uma expressão que gera uma temporária  
ast\_example\_if\_cond\_as\_expr \= {  
    'type': 'binary\_op', 'op': '\<',   
    'left': {'type':'identifier', 'name':'x'},  
    'right': {'type':'identifier', 'name':'y'}  
}  
ast\_example\[2\]\['condition'\] \= ast\_example\_if\_cond\_as\_expr \# Substitui a condição

**Geração de TAC usando o Gerador Python:**

\# Supondo que a AST está corretamente estruturada para o gerador:  
tac\_gen \= TACGenerator()

\# Simulação de geração para a condição do IF primeiro, para que cond\_place seja uma temporária  
\# (generate\_for\_if\_then\_else espera que cond\_place já seja o nome de uma temp/var)  
\# Isso é um pouco artificial aqui, pois a lógica de visitação da AST cuidaria disso.

\# Vamos simular o processo:  
\# 1\. x := 10  
tac\_gen.generate\_for\_statement(ast\_example\[0\])  
\# 2\. y := 20  
tac\_gen.generate\_for\_statement(ast\_example\[1\])  
\# 3\. IF x \< y THEN ... ELSE ...  
\#    A chamada a generate\_for\_if\_then\_else internamente chamará  
\#    generate\_for\_expression para ast\_example\[2\]\['condition'\]  
tac\_gen.generate\_for\_statement(ast\_example\[2\])

print("--- Código TAC Gerado \---")  
print(tac\_gen.get\_code\_str())

**Saída TAC Esperada (Pode variar ligeiramente com a implementação exata do generate\_for\_expression para \<):**

\--- Código TAC Gerado \---  
x := 10  
y := 20  
T0 := x \< y  
IF\_FALSE T0 GOTO L0  
T1 := x \+ y  
z := T1  
GOTO L1  
L0:  
T2 := y \- x  
z := T2  
L1:

Nota: A implementação de generate\_for\_expression para operadores relacionais como \< precisaria ser definida para retornar uma temporária que guarda o resultado booleano (e.g., 0 ou 1), ou a instrução IF\_FALSE (ou IF\_LT, etc.) seria construída de forma mais direta se generate\_for\_expression pudesse retornar os operandos originais para serem usados em um IF op GOTO.  
O exemplo de TACInstruction.\_\_str\_\_ já prevê IF {self.arg1} {op\_str} {self.arg2} GOTO {self.result}. Se generate\_for\_expression para \< não gerasse T0 := x \< y mas passasse x, \<, y para generate\_for\_if\_then\_else, este poderia emitir IF\_LT x, y, GOTO L\_THEN (ou IF\_GE x, y, GOTO L\_ELSE). A abordagem com T0 := x \< y seguida de IF\_FALSE T0 GOTO ... é mais geral.

### **9\. Exercícios Propostos**

1. Expressões Aritméticas Complexas:  
   Traduza as seguintes expressões para Código de Três Endereços, mostrando explicitamente o uso de variáveis temporárias e respeitando a precedência e associatividade padrão dos operadores (multiplicação/divisão antes de adição/subtração, da esquerda para a direita para operadores de mesma precedência):  
   a. res \= (a \- b) \* (c \+ d / e) \- f  
   b. val \= a \* b \* c \+ d \- e / f / g  
   c. y \= \-x \+ (w / (z \- 2)) \* 5  
2. Expressões Booleanas com Curto-Circuito:  
   Gere TAC para os seguintes comandos if, implementando a avaliação em curto-circuito para os operadores lógicos AND e OR:  
   a. if (idade \>= 18 AND possui\_cnh \== true) then status := "apto"; endif;  
   b. if (nota \< 5 OR faltas \> 10\) then resultado := "reprovado"; else resultado := "aprovado"; endif;  
   c. if ((a \+ b \> c AND c \!= 0\) OR d \== 1\) then x := 1; endif;  
3. Estruturas de Repetição (while, for):  
   Traduza as seguintes estruturas de laço para TAC:  
   a. i := 0; soma := 0; while (i \< 10 AND soma \< 100\) do soma := soma \+ i; i := i \+ 1; endwhile;  
   b. (Assuma que for passo P significa incremento de P)  
   for k := 10 down\_to 1 step 2 do // (down\_to significa k := k \- P) print(k); endfor;  
   (Dica: Converta o for para uma estrutura while equivalente primeiro.)  
   c. fatorial := 1; num := 5; do fatorial := fatorial \* num; num := num \- 1; while (num \> 0);  
4. Aninhamento de Estruturas:  
   Gere o TAC para o seguinte trecho de código aninhado:  
   a := 10;  
   b := 20;  
   while (a \< b) do  
     print(a);  
     if (a % 2 \== 0\) then  
       b := b \- a / 2;  
     else  
       b := b \- 1;  
     endif;  
     a := a \+ 1;  
   endwhile;  
   resultado\_final := b;

5. Representações de TAC (Quádruplas vs. Triplas):  
   Considere a expressão: expr \= (x \+ y) \- (x \+ y \+ z);  
   a. Gere o TAC para esta expressão.  
   b. Represente o TAC gerado usando Quádruplas (formato (op, arg1, arg2, result)).  
   c. Represente o mesmo TAC usando Triplas (formato (op, arg1, arg2) com referências a resultados anteriores).  
   d. Discuta como a otimização de "eliminação de subexpressão comum" poderia simplificar o TAC antes da representação. Mostre o TAC otimizado.  
6. Chamadas de Função/Procedimento:  
   Gere TAC para o seguinte código, incluindo o tratamento de parâmetros e valor de retorno:  
   function adicionar(a: integer, b: integer) : integer;  
   begin  
     return a \+ b;  
   end;

   procedure principal();  
   var res, val1, val2 : integer;  
   begin  
     val1 := 5;  
     val2 := 7;  
     res := adicionar(val1, val2);  
     print(res);  
   end;

   (Assuma instruções TAC como PARAM, CALL nome, num\_params, RETORNO valor, e var := CALL nome, num\_params).

### **Aviso**

*Este manual foi gerado com o auxílio de inteligência artificial. Embora tenha sido feito um esforço para garantir a precisão técnica e didática, ele deve ser usado como um ponto de partida e complementado com os materiais de referência clássicos da área de compiladores e a orientação de um educador. A implementação prática de um gerador de TAC em um projeto de compilador real envolve muitos detalhes e decisões de design adicionais.*

