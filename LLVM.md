# **Manual Técnico: Geração de Código Final com LLVM para Ensino de Compiladores**

## **Sumário**

1. Introdução ao LLVM  
   1.1. O que é o LLVM: sua missão, filosofia e arquitetura  
   1.2. Vantagens de utilizar LLVM como backend de um compilador  
   1.3. Visão geral do ecossistema LLVM  
2. A Representação Intermediária (IR) do LLVM  
   2.1. Características principais da LLVM IR  
   2.2. Sintaxe textual da LLVM IR  
   2.3. Instruções comuns da LLVM IR  
   2.4. Como a LLVM IR facilita otimizações e o retargeting para diferentes arquiteturas  
3. Processo de Geração de Código LLVM a partir de uma Representação de Alto Nível  
   3.1. Mapeamento de construções de linguagens de programação para LLVM IR  
   3.2. Uso da API C++ do LLVM  
   3.3. O papel crucial da Tabela de Símbolos na tradução para LLVM IR  
4. O Pipeline de Backend do LLVM  
   4.1. Visão geral das fases do backend do LLVM após a geração da IR  
   4.2. Como o LLVM abstrai a complexidade dessas fases para o desenvolvedor do frontend  
5. Utilizando as Ferramentas LLVM  
   5.1. llc: O compilador estático do LLVM  
   5.2. lli: O interpretador de LLVM IR  
   5.3. llvm-as e llvm-dis: Montador e desmontador de LLVM bitcode  
   5.4. Outras ferramentas relevantes para análise e depuração  
6. Estudo de Caso: Integração com o Compilador Buriti  
   6.1. Análise da estrutura do Compilador Buriti  
   6.2. Proposta de tradução da saída do gerador de código intermediário do Buriti para LLVM IR  
   6.3. Exemplos de como as construções específicas da linguagem do Buriti seriam representadas em LLVM IR  
   6.4. Desafios e decisões de design ao integrar LLVM a um compilador existente ou novo  
7. Fundamentação Teórica e Referências  
   7.1. Conectar os conceitos apresentados com os princípios discutidos em livros clássicos de compiladores  
   7.2. Incentivar a consulta à documentação oficial do LLVM  
8. Guia Prático e Exercícios  
   8.1. Diretrizes sobre como configurar um ambiente de desenvolvimento com LLVM  
   8.2. Propor exercícios práticos

**Nota**: Este manual foi elaborado com o auxílio de ferramentas de inteligência artificial para compilar e estruturar informações relevantes sobre a geração de código com LLVM. Recomenda-se a consulta às fontes primárias e à documentação oficial do LLVM para um aprofundamento completo.

## **1\. Introdução ao LLVM**

O LLVM é muito mais do que um simples compilador; é uma infraestrutura modular e reutilizável de tecnologias de compilação e toolchain. Entender sua arquitetura e filosofia é fundamental para aproveitar seu potencial.

### **1.1. O que é o LLVM: sua missão, filosofia e arquitetura**

O **LLVM** (originalmente Low Level Virtual Machine) é uma coleção de bibliotecas e ferramentas modulares e reutilizáveis de compilador e toolchain. Sua **missão** principal é fornecer uma infraestrutura robusta, flexível e de alto desempenho para a construção de compiladores, otimizadores e geradores de código para diversas linguagens de programação e arquiteturas de hardware.

A **filosofia** do LLVM gira em torno da ideia de uma representação intermediária (IR) bem definida e poderosa, que serve como um "idioma universal" entre frontends de linguagens e backends de arquiteturas. Isso promove a reusabilidade e a separação de preocupações.

A **arquitetura** do LLVM é classicamente dividida em três fases principais:

1. **Frontend:** Responsável por analisar o código fonte de uma linguagem específica (como C, C++, Swift, Rust), verificar erros de sintaxe e semântica, e traduzi-lo para a Representação Intermediária (IR) do LLVM. O exemplo mais conhecido de frontend que utiliza LLVM é o Clang (para C/C++/Objective-C).  
   * \[Diagrama: Código Fonte (e.g., C++) \-\> Frontend (e.g., Clang) \-\> LLVM IR\]  
2. **Otimizador de Passes (Mid-end):** Opera sobre a LLVM IR. Consiste em uma série de "passes" de otimização que analisam e transformam o código IR para melhorá-lo (e.g., redução de redundâncias, inlining de funções, otimizações de laço). Esses passes são em grande parte independentes da linguagem fonte e da arquitetura alvo.  
   * \[Diagrama: LLVM IR \-\> Sequência de Passes de Otimização \-\> LLVM IR Otimizada\]  
3. **Backend (Code Generator):** Pega a LLVM IR (otimizada ou não) e a traduz para código de máquina específico de uma arquitetura alvo (e.g., x86, ARM, MIPS). Isso envolve seleção de instruções, alocação de registradores e emissão de código assembly ou objeto.  
   * \[Diagrama: LLVM IR Otimizada \-\> Backend (Seleção de Instruções, Alocação de Registradores, etc.) \-\> Código de Máquina\]

\[Figura: Visão geral da arquitetura do LLVM com as três fases principais e o fluxo de dados entre elas.\]

### **1.2. Vantagens de utilizar LLVM como backend de um compilador**

A adoção do LLVM como backend oferece inúmeras vantagens:

* **Reusabilidade:** Desenvolvedores de compiladores podem focar no frontend de sua nova linguagem, aproveitando toda a infraestrutura de otimização e geração de código para múltiplas arquiteturas já existente no LLVM.  
* **Otimizações de Alta Qualidade:** O LLVM possui um conjunto extenso e maduro de passes de otimização que são continuamente aprimorados pela comunidade global. Esses passes são, em sua maioria, independentes da linguagem e da arquitetura.  
* **Portabilidade para Múltiplas Arquiteturas:** Uma vez que o frontend gera LLVM IR, o backend do LLVM pode ser usado para "retarget" o código para uma vasta gama de arquiteturas de processadores (x86, ARM, PowerPC, SPARC, etc.) sem necessidade de reescrever o gerador de código para cada uma delas.  
* **API Bem Definida:** O LLVM oferece APIs robustas (principalmente em C++) para interagir programaticamente com a IR, passes de otimização e o processo de geração de código.  
* **Licença Permissiva:** A licença do LLVM (Apache 2.0 com exceções LLVM) permite seu uso em projetos comerciais e de código aberto.  
* **Comunidade Ativa:** Uma grande e ativa comunidade de desenvolvedores e pesquisadores contribui para o LLVM, garantindo sua evolução e suporte.

### **1.3. Visão geral do ecossistema LLVM**

O LLVM é mais do que apenas um conjunto de bibliotecas; é um ecossistema vibrante de ferramentas e projetos:

* **Clang:** Frontend para C, C++, Objective-C e Objective-C++ que utiliza o LLVM como backend. Conhecido por seus diagnósticos de erro detalhados e rápida compilação.  
* **llc:** O compilador estático do LLVM. Converte LLVM IR (arquivos .ll ou .bc) para código assembly específico da arquitetura (arquivos .s) ou código objeto (arquivos .o).  
* **lli:** O interpretador de LLVM IR. Executa diretamente código LLVM IR, útil para debugging e testes rápidos sem a necessidade de compilação para código de máquina.  
* **opt:** Ferramenta para aplicar passes de otimização da LLVM IR. Permite executar passes individualmente ou em sequência sobre um arquivo .ll ou .bc.  
* **llvm-as:** Montador de LLVM. Converte a forma textual da LLVM IR (arquivos .ll) para a forma binária (bitcode, arquivos .bc).  
* **llvm-dis:** Desmontador de LLVM. Converte bitcode LLVM (arquivos .bc) de volta para a forma textual (arquivos .ll).  
* **LLDB:** O depurador do projeto LLVM, com suporte para C, C++, Objective-C e Swift.  
* **Bibliotecas Principais:** libSupport, libCore, libIR, libCodeGen, libTarget, entre outras, que fornecem a funcionalidade central do LLVM.

## **2\. A Representação Intermediária (IR) do LLVM**

A LLVM IR é o coração do LLVM. É uma linguagem de montagem de baixo nível, mas abstrata e fortemente tipada, projetada para ser um formato universal para análise e transformação de programas.

### **2.1. Características principais da LLVM IR**

* **Forma SSA (Static Single Assignment):** Na LLVM IR, cada variável (registrador virtual) é atribuída exatamente uma vez. Se uma variável precisa receber diferentes valores em diferentes caminhos de controle, são usadas funções $ \\Phi $ (nós PHI) para mesclar os valores. A forma SSA simplifica muitas otimizações.  
* **Tipagem Forte:** Todo valor na LLVM IR possui um tipo. Isso permite verificações de consistência e previne erros de tipo em tempo de compilação da IR. Os tipos incluem inteiros de largura arbitrária, tipos de ponto flutuante, ponteiros, vetores, estruturas e funções.  
* **Conjunto de Instruções Abstrato:** A IR possui um conjunto de instruções que se assemelha a uma linguagem de montagem RISC (Reduced Instruction Set Computer), mas é independente de qualquer arquitetura de hardware específica. Inclui instruções aritméticas, lógicas, de memória, de controle de fluxo e de chamada de função.

### **2.2. Sintaxe textual da LLVM IR**

A LLVM IR pode ser representada em um formato textual legível por humanos (arquivos .ll) ou em um formato binário denso chamado bitcode (arquivos .bc).

Elementos chave da sintaxe textual:

* **Módulos (Modules):** A unidade de compilação de mais alto nível. Um módulo LLVM contém funções, variáveis globais e metadados.  
  ; Um comentário no módulo  
  target datalayout \= "e-m:e-p270:32:32-p271:32:32-p272:64:64-i64:64-f80:128-n8:16:32:64-S128"  
  target triple \= "x86\_64-pc-linux-gnu"

  @global\_var \= global i32 10, align 4

* **Funções (Functions):** Definidas com define ou declaradas com declare.  
  define i32 @minha\_funcao(i32 %a, i32 %b) {  
    ; corpo da função  
    ret i32 0  
  }

  declare i32 @printf(ptr nocapture readonly, ...)

* **Blocos Básicos (Basic Blocks):** Sequências de instruções sem desvios internos, terminadas por uma instrução de controle de fluxo (e.g., br, ret, switch). Cada bloco básico possui um rótulo (label). O primeiro bloco de uma função é o entry (por convenção, mas pode ter outro nome se for o primeiro da lista).  
  entry:  
    %tmp1 \= add i32 %a, %b  
    br label %corpo\_loop

  corpo\_loop:  
    ; ...  
    br label %fim\_loop

  fim\_loop:  
    ret i32 %tmp1

* **Tipos de Dados:**  
  * **Primitivos:** void, iN (inteiro de N bits, e.g., i1, i8, i32, i64), float, double, half (ponto flutuante de 16 bits), label, metadata, token.  
  * **Agregados:**  
    * \[\<N\> x \<tipo\>\]: Arranjo de N elementos do tipo \<tipo\>. Ex: \[10 x i32\]  
    * { \<tipo1\>, \<tipo2\>, ... }: Estrutura (struct) anônima. Ex: { i32, ptr, \[4 x float\] }  
    * %nome\_struct \= type { ... }: Estrutura nomeada.  
  * **Ponteiros:** ptr (ponteiro opaco, forma moderna). Anteriormente, \<tipo\>\* (e.g., i32\*). A forma ptr to \<type\> também pode ser vista em documentação ou IR mais antiga.  
* **Identificadores:**  
  * @identificador\_global: Variáveis globais e funções.  
  * %identificador\_local: Registradores virtuais (resultados de instruções) e parâmetros de função.  
  * label\_nome: (ou \<nome\>:) Rótulos de blocos básicos (sem o % ou @).  
* **Metadados (Metadata):** Informações adicionais anexadas a instruções ou funções, como informações de debug (\!dbg), tipos (\!tbaa \- type-based alias analysis), etc. São prefixados com \!. Ex: \!5 \= \!{...}.

### **2.3. Instruções comuns da LLVM IR**

A LLVM IR possui um rico conjunto de instruções. Algumas das mais comuns incluem:

* **Instruções Terminadoras (Control Flow):**  
  * ret \<tipo\> %valor ou ret void: Retorna de uma função.  
  * br label %destino: Desvio incondicional.  
  * br i1 %condicao, label %se\_verdadeiro, label %se\_falso: Desvio condicional.  
  * switch \<tipo\_int\> %valor, label %default\_dest \[ \<tipo\_int\> \<const\_val\>, label %dest ... \]: Desvio multi-caminho.  
  * indirectbr ptr %endereco, \[label %dest1, label %dest2, ...\]: Desvio indireto.  
  * invoke ... to label %normal\_dest unwind label %exception\_dest: Chamada de função com tratamento de exceção.  
  * unreachable: Indica que este ponto do código não deve ser alcançado.  
* **Operações Binárias Aritméticas:**  
  * add, sub, mul  
  * udiv (divisão sem sinal), sdiv (divisão com sinal)  
  * urem (resto sem sinal), srem (resto com sinal)  
  * Para ponto flutuante: fadd, fsub, fmul, fdiv, frem.  
  * Flags opcionais: nsw (no signed wrap), nuw (no unsigned wrap), exact.

%soma \= add nsw i32 %x, %y  
%produto\_fp \= fmul float %val1, %val2

* **Operações Binárias Lógicas (Bitwise):**  
  * shl (shift left), lshr (logical shift right), ashr (arithmetic shift right).  
  * and, or, xor.

%mascara \= and i8 %byte, 15 ; 0b00001111  
%shifted \= shl i32 %val, 2

* **Operações de Memória:**  
  * alloca \<tipo\>, \[align \<alinhamento\>\]: Aloca memória na stack frame da função atual. Retorna um ponteiro (ptr).  
  * load \<tipo\>, ptr %ponteiro, \[align \<alinhamento\>\]: Carrega um valor da memória.  
  * store \<tipo\> %valor, ptr %ponteiro, \[align \<alinhamento\>\]: Armazena um valor na memória.  
  * getelementptr \[inbounds\] \<tipo\_base\>, ptr %ptr\_base, \<tipo\_indice1\> \<indice1\> \[, \<tipo\_indiceN\> \<indiceN\> ...\]: Calcula o endereço de um elemento de uma estrutura ou arranjo. **Importante:** GEP *não* acessa a memória; ele apenas realiza aritmética de ponteiros. inbounds é uma flag que indica que o endereço calculado está dentro dos limites alocados.

%ptr\_var \= alloca i32, align 4  
store i32 100, ptr %ptr\_var, align 4  
%val\_lido \= load i32, ptr %ptr\_var, align 4

; int arr\[10\]; &(arr\[i\])  
; %arr\_ptr é ptr, apontando para \[10 x i32\]  
; %idx é i32  
%elemento\_ptr \= getelementptr inbounds \[10 x i32\], ptr %arr\_ptr, i64 0, i32 %idx

* **Instruções de Conversão:**  
  * trunc \<tipo\_origem\> %valor to \<tipo\_destino\>: Trunca um inteiro para um tipo menor.  
  * zext \<tipo\_origem\> %valor to \<tipo\_destino\>: Zero-extends um inteiro para um tipo maior.  
  * sext \<tipo\_origem\> %valor to \<tipo\_destino\>: Sign-extends um inteiro para um tipo maior.  
  * fptrunc \<tipo\_fp\_origem\> %valor to \<tipo\_fp\_destino\>: Trunca um float para um tipo float menor.  
  * fpext \<tipo\_fp\_origem\> %valor to \<tipo\_fp\_destino\>: Estende um float para um tipo float maior.  
  * fptoui \<tipo\_fp\> %valor to \<tipo\_int\_destino\>: Converte float para inteiro sem sinal.  
  * fptosi \<tipo\_fp\> %valor to \<tipo\_int\_destino\>: Converte float para inteiro com sinal.  
  * uitofp \<tipo\_int\> %valor to \<tipo\_fp\_destino\>: Converte inteiro sem sinal para float.  
  * sitofp \<tipo\_int\> %valor to \<tipo\_fp\_destino\>: Converte inteiro com sinal para float.  
  * ptrtoint ptr %valor to \<tipo\_int\_destino\>: Converte ponteiro para inteiro.  
  * inttoptr \<tipo\_int\> %valor to ptr: Converte inteiro para ponteiro.  
  * bitcast \<tipo\_origem\> %valor to \<tipo\_destino\>: Converte o valor para um tipo diferente com o mesmo número de bits (e.g., i32 para float, ptr para ptr).

%char\_val \= load i8, ptr %char\_ptr  
%int\_val \= sext i8 %char\_val to i32  
%float\_val \= load float, ptr %float\_ptr  
%double\_val \= fpext float %float\_val to double

* **Outras Instruções:**  
  * icmp \<cond\> \<tipo\> %op1, %op2: Comparação de inteiros ou ponteiros. \<cond\> pode ser eq (igual), ne (diferente), ugt (maior sem sinal), sgt (maior com sinal), ule, sle, etc. Retorna i1.  
  * fcmp \<cond\> \<tipo\> %op1, %op2: Comparação de ponto flutuante. \<cond\> pode ser oeq (ordenado e igual), ogt, oge, olt, ole, one, ueq (não ordenado ou igual), etc. Retorna i1.  
  * phi \<tipo\> \[ %val0, %label0 \], \[ %val1, %label1 \], ...: Nó PHI, usado para implementar a forma SSA. Seleciona um valor com base no bloco básico predecessor pelo qual o controle fluiu para o bloco atual.  
  * call \<tipo\_retorno\> @nome\_funcao(\<tipo\_arg1\> %arg1, ...): Chamada de função.  
  * select i1 %cond, \<tipo\> %val\_true, \<tipo\> %val\_false: Seleciona um valor com base em uma condição (similar a um operador ternário).

### **2.4. Como a LLVM IR facilita otimizações e o retargeting para diferentes arquiteturas**

* **Otimizações:**  
  * A forma **SSA** simplifica drasticamente muitas análises de fluxo de dados e transformações, como propagação de constantes, eliminação de subexpressões comuns e movimentação de código. Os nós PHI explicitam a fusão de valores de diferentes caminhos.  
  * A **tipagem forte** permite que otimizações sejam mais agressivas e seguras, pois as propriedades dos dados são conhecidas.  
  * O conjunto de instruções **abstrato, mas de baixo nível,** expõe muitas oportunidades de otimização que seriam ocultadas em representações de nível mais alto, ao mesmo tempo que não se compromete prematuramente com detalhes de uma máquina específica.  
  * **Metadados** podem guiar otimizações (e.g., \!tbaa para alias analysis, \!loop para informações sobre laços).  
* **Retargeting:**  
  * A LLVM IR atua como uma "interface" bem definida. Frontends de linguagens precisam apenas gerar IR.  
  * Os backends do LLVM são responsáveis por traduzir essa IR genérica para o código de máquina específico de cada arquitetura. Isso significa que, para suportar uma nova arquitetura, "apenas" um novo backend LLVM precisa ser escrito (ou adaptado). O frontend da linguagem não precisa ser modificado.  
  * Da mesma forma, uma nova linguagem que gera LLVM IR automaticamente ganha suporte a todas as arquiteturas para as quais o LLVM já possui um backend.

## **3\. Processo de Geração de Código LLVM a partir de uma Representação de Alto Nível**

A tradução de construções de uma linguagem de programação (ou de uma IR de mais alto nível, como Código de Três Endereços \- CTE) para LLVM IR é uma etapa crucial no desenvolvimento de um compilador. Este processo envolve mapear conceitos semânticos da linguagem fonte para as instruções e estruturas da LLVM IR.

### **3.1. Mapeamento de construções de linguagens de programação para LLVM IR**

O objetivo é converter a estrutura e a semântica do programa fonte, geralmente já processada por um frontend e possivelmente convertida para uma IR de alto/médio nível, na LLVM IR.

* **Declaração de variáveis globais e locais (alloca)**  
  * **Variáveis Globais:** São declaradas no nível do módulo usando a sintaxe @nome\_var \= \[linkage\] \[visibility\] global \<tipo\> \<valor\_inicial\>, \[align \<alinhamento\>\].  
    @contador\_global \= global i32 0, align 4  
    @mensagem\_erro \= constant \[13 x i8\] c"Erro fatal\!\\0A\\00", align 1

  * **Variáveis Locais (na stack):** A instrução alloca \<tipo\>, \[align \<alinhamento\>\] é usada para alocar espaço na stack frame da função atual. alloca deve, idealmente, ser usada no bloco de entrada da função para facilitar otimizações como mem2reg (que promove alocações de memória para registradores SSA). O resultado de alloca é um ponteiro (ptr) para o espaço alocado.  
    define i32 @exemplo\_var\_local() {  
    entry:  
      %ptr\_x \= alloca i32, align 4  ; aloca espaço para um inteiro x  
      %ptr\_array \= alloca \[10 x float\], align 16 ; aloca espaço para um array de 10 floats  
      store i32 42, ptr %ptr\_x, align 4 ; armazena 42 em x  
      ; ...  
      ret i32 0  
    }

* **Tradução de expressões aritméticas, lógicas e relacionais**  
  * **Expressões Aritméticas:** Mapeiam diretamente para as instruções aritméticas da LLVM IR (add, sub, mul, sdiv, udiv, etc.). Os operandos devem ser valores SSA (resultados de instruções anteriores, constantes ou argumentos de função).  
    ; Código Fonte (hipotético): c \= (a \+ 5\) \* b;  
    ; Supondo que %a\_val, %b\_val são os valores SSA de a e b  
    %soma\_temp \= add i32 %a\_val, 5  
    %c\_val \= mul i32 %soma\_temp, %b\_val

  * **Expressões Lógicas:** Mapeiam para instruções lógicas (and, or, xor, shl, etc.).  
    ; Código Fonte (hipotético): flags \= (flags | MASK\_X) & \~MASK\_Y;  
    ; Supondo %flags\_val, %mask\_x\_val, %mask\_y\_val (i32)  
    %or\_temp \= or i32 %flags\_val, %mask\_x\_val  
    %not\_mask\_y \= xor i32 %mask\_y\_val, \-1 ; \~MASK\_Y (complemento de um para not bitwise)  
    %flags\_novo\_val \= and i32 %or\_temp, %not\_mask\_y

  * **Expressões Relacionais:** Usam a instrução icmp (para inteiros/ponteiros) ou fcmp (para ponto flutuante) para produzir um valor booleano (i1). Esse valor é então tipicamente usado em uma instrução br condicional.  
    ; Código Fonte (hipotético): if (x \>= y) ...  
    ; Supondo %x\_val, %y\_val (i32)  
    %condicao \= icmp sge i32 %x\_val, %y\_val ; sge: signed greater or equal  
    br i1 %condicao, label %bloco\_then, label %bloco\_else

* Implementação de comandos de atribuição  
  A atribuição variavel \= expressao envolve:  
  1. Calcular o valor da expressao (resultando em um valor SSA %resultado\_expr).  
  2. Obter o ponteiro para a variavel (que foi retornada por alloca para locais ou é um @global para globais).  
  3. Usar a instrução store \<tipo\> %resultado\_expr, ptr %ponteiro\_variavel.

; Código Fonte (hipotético): x \= a \+ b; (onde x, a, b são locais)  
; %ptr\_x, %ptr\_a, %ptr\_b são os ponteiros de alloca  
entry:  
  %ptr\_x \= alloca i32, align 4  
  %ptr\_a \= alloca i32, align 4  
  %ptr\_b \= alloca i32, align 4  
  ; ... inicializações de a e b ...  
  store i32 10, ptr %ptr\_a, align 4  ; a \= 10  
  store i32 20, ptr %ptr\_b, align 4  ; b \= 20

  %val\_a \= load i32, ptr %ptr\_a, align 4  
  %val\_b \= load i32, ptr %ptr\_b, align 4  
  %soma \= add i32 %val\_a, %val\_b  
  store i32 %soma, ptr %ptr\_x, align 4 ; x \= soma  
\[Figura: Fluxograma mostrando avaliação da expressão e store na memória para uma atribuição.\]

* Geração de código para estruturas de controle de fluxo  
  O controle de fluxo é gerenciado por blocos básicos e instruções de desvio (br, switch).  
  * **if-then-else**:  
    1. Gere código para a condição, resultando em um i1 (%cond).  
    2. Use br i1 %cond, label %then\_block, label %else\_block.  
    3. Crie três novos blocos básicos: %then\_block, %else\_block e %merge\_block.  
    4. Popule %then\_block com o código do then e termine com br label %merge\_block.  
    5. Popule %else\_block com o código do else (se houver) e termine com br label %merge\_block. Se não houver else, o desvio falso da condição vai direto para %merge\_block.  
    6. O %merge\_block é para onde o controle flui após o if.

  ; if (a \< b) { x \= 1; } else { x \= 2; }  
       ; %val\_a, %val\_b são os valores carregados de a e b  
       ; %ptr\_x é o ponteiro para x  
         %cond \= icmp slt i32 %val\_a, %val\_b  
         br i1 %cond, label %then\_bb, label %else\_bb

       then\_bb:  
         store i32 1, ptr %ptr\_x, align 4  
         br label %merge\_bb

       else\_bb:  
         store i32 2, ptr %ptr\_x, align 4  
         br label %merge\_bb

       merge\_bb:  
         ; código após o if-else  
         %val\_x\_final \= load i32, ptr %ptr\_x, align 4 ; Se x for usado depois  
       \[Figura: Estrutura de blocos básicos para if-then-else, mostrando os desvios.\]

  * Laços (while, for):  
    Geralmente envolvem um bloco de cabeçalho/condição (loop\_header), um bloco de corpo (loop\_body), um bloco de atualização/latch (loop\_latch), e um bloco de saída (loop\_exit).  
    * **while (cond) { corpo }**:  
      1. loop\_header\_bb: Avalia cond. Se falso, desvia para loop\_exit\_bb. Se verdadeiro, desvia para loop\_body\_bb.  
      2. loop\_body\_bb: Contém o código do corpo do laço. Ao final, desvia incondicionalmente para loop\_latch\_bb (ou diretamente para loop\_header\_bb).  
      3. loop\_latch\_bb (opcional, mas comum para for ou para manter o header limpo): Contém código de incremento/atualização, depois desvia para loop\_header\_bb.  
      4. loop\_exit\_bb: Código após o laço.

  ; while (i \< 10\) { i \= i \+ 1; }  
         ; %ptr\_i é o ponteiro para i, inicializado com 0 antes do laço  
           br label %loop\_header\_bb ; Salto inicial para a condição

         loop\_header\_bb:  ; Também chamado de bloco de condição  
           %val\_i\_atual\_phi \= phi i32 \[ 0, %entry\_block\_antes\_do\_loop \], \[ %val\_i\_prox, %loop\_latch\_bb \] ; Exemplo com PHI  
           %val\_i\_carregado \= load i32, ptr %ptr\_i, align 4 ; Exemplo sem PHI (requer mem2reg)  
           %cond \= icmp slt i32 %val\_i\_carregado, 10  
           br i1 %cond, label %loop\_body\_bb, label %loop\_exit\_bb

         loop\_body\_bb:  
           ; ... corpo do loop ...  
           ; Exemplo: i \= i \+ 1  
           %val\_i\_antes\_inc \= load i32, ptr %ptr\_i, align 4  
           %val\_i\_inc \= add i32 %val\_i\_antes\_inc, 1  
           store i32 %val\_i\_inc, ptr %ptr\_i, align 4  
           br label %loop\_latch\_bb ; Ou diretamente para loop\_header\_bb

         loop\_latch\_bb: ; Bloco de atualização  
           %val\_i\_prox \= load i32, ptr %ptr\_i, align 4 ; Para o PHI node  
           br label %loop\_header\_bb ; Volta para checar a condição

         loop\_exit\_bb:  
           ; código após o while  
         \[Figura: Estrutura de blocos básicos para um laço while, destacando o header, corpo e saída.\]Nota sobre nós $ \\Phi $: Em laços, variáveis modificadas dentro do laço e usadas em iterações subsequentes ou após o laço frequentemente requerem nós $ \\Phi $ no cabeçalho do laço para manter a forma SSA, especialmente após a otimização mem2reg.

  * **switch**: Mapeia para a instrução switch do LLVM.  
    ; switch (val) { case 0: ...; case 1: ...; default: ... }  
    ; %val\_loaded é o valor de val  
      switch i32 %val\_loaded, label %default\_bb \[  
        i32 0, label %case0\_bb  
        i32 1, label %case1\_bb  
        i32 2, label %case2\_bb  
      \]  
    case0\_bb:  
      ; ... código para case 0 ...  
      br label %end\_switch\_bb  
    case1\_bb:  
      ; ... código para case 1 ...  
      br label %end\_switch\_bb  
    case2\_bb:  
      ; ... código para case 2 ...  
      br label %end\_switch\_bb  
    default\_bb:  
      ; ... código para default ...  
      br label %end\_switch\_bb  
    end\_switch\_bb:  
      ; código após o switch

* **Definição e chamada de funções**  
  * **Definição de Funções:** Usa a palavra-chave define.  
    define \<tipo\_retorno\> @nome\_funcao(\<tipo\_arg1\> %arg1\_nome, \<tipo\_arg2\> %arg2\_nome, ...) \[attributes\] {  
      entry:  
        ; corpo da função  
        ; Os argumentos (%arg1\_nome, %arg2\_nome) já são valores SSA.  
        ; Se precisar armazená-los em memória (e.g. para passar endereço ou modificar), use alloca \+ store.  
        ret \<tipo\_retorno\> %valor\_retorno  
    }

  * **Chamada de Funções:** Usa a instrução call (ou invoke para tratamento de exceções).  
    %resultado \= call \<tipo\_retorno\> @nome\_funcao(\<tipo\_arg1\> %valor\_arg1, ...)  
    ; Se a função retorna void:  
    call void @funcao\_sem\_retorno()

  * **Convenções de Chamada (Calling Conventions):** O LLVM abstrai a maioria das convenções de chamada específicas da arquitetura. Pode-se especificar uma convenção (e.g., ccc para C, fastcc para rápida, webkit\_jscc) na definição e chamada da função se necessário, usando atributos de função.  
  * **Passagem de Parâmetros:** Os argumentos são passados como valores SSA para a instrução call. Dentro da função chamada, os parâmetros são recebidos como valores SSA.  
  * **Tratamento de Valor de Retorno:** A instrução ret é usada para retornar um valor. Se a função chamada retorna um valor, a instrução call produzirá um novo valor SSA.  
* Manipulação de arranjos, estruturas e ponteiros (getelementptr)  
  A instrução getelementptr (GEP) é fundamental. Ela calcula endereços de elementos dentro de tipos agregados ou a partir de um ponteiro base.  
  * **Arranjos:**  
    ; C: int arr\[5\]; int val \= arr\[2\];  
    ; %arr\_ptr é ptr, apontando para \[5 x i32\] (resultado de um alloca, por exemplo)  
    %elemento\_ptr \= getelementptr inbounds \[5 x i32\], ptr %arr\_ptr, i64 0, i32 2  
    ; O primeiro índice (i64 0\) é para "desreferenciar" o ponteiro para o array, acessando o array em si.  
    ; O segundo índice (i32 2\) é o índice do elemento dentro do array.  
    %val \= load i32, ptr %elemento\_ptr, align 4

  * **Estruturas:**  
    ; C: struct Ponto { int x; float y; }; struct Ponto p; p.y \= 3.0f;  
    %struct.Ponto \= type { i32, float }  
    ; %p\_ptr é ptr, apontando para %struct.Ponto (resultado de um alloca)  
    %campo\_y\_ptr \= getelementptr inbounds %struct.Ponto, ptr %p\_ptr, i32 0, i32 1  
    ; O primeiro índice (i32 0\) é para o struct em si (se %p\_ptr fosse um array de structs, seria o índice do struct no array).  
    ; O segundo índice (i32 1\) é o índice do campo 'y' (0 para 'x', 1 para 'y').  
    store float 3.0, ptr %campo\_y\_ptr, align 4

\[Figura: Ilustração de como GEP indexa um array e um struct, mostrando os índices e o tipo base.\]

### **3.2. Uso da API C++ do LLVM**

Em vez de gerar IR textual, um compilador normalmente usa a API C++ do LLVM para construir programaticamente o módulo, funções, blocos básicos e instruções.

Conceitualmente, isso envolve:

1. **LLVMContext**: Um objeto opaco que armazena informações globais do LLVM, como tipos únicos e constantes.  
   \#include "llvm/IR/LLVMContext.h"  
   // ...  
   llvm::LLVMContext TheContext;

2. **Module**: Criação de um objeto Module para representar a unidade de compilação.  
   \#include "llvm/IR/Module.h"  
   // ...  
   std::unique\_ptr\<llvm::Module\> TheModule \= std::make\_unique\<llvm::Module\>("meu\_modulo", TheContext);

3. **IRBuilder\<\>**: Um helper class para facilitar a criação e inserção de instruções no ponto correto do bloco básico atual.  
   \#include "llvm/IR/IRBuilder.h"  
   // ...  
   llvm::IRBuilder\<\> Builder(TheContext);

4. **FunctionType e Function**: Para cada função na linguagem fonte, criar um FunctionType (especificando tipos de retorno e parâmetros) e depois um objeto Function dentro do Module.  
   \#include "llvm/IR/Function.h"  
   \#include "llvm/IR/Type.h"  
   // ...  
   // Ex: Função int add(int a, int b)  
   std::vector\<llvm::Type\*\> Ints(2, llvm::Type::getInt32Ty(TheContext));  
   llvm::FunctionType\* FT \= llvm::FunctionType::get(llvm::Type::getInt32Ty(TheContext), Ints, false);  
   llvm::Function\* F \= llvm::Function::Create(FT, llvm::Function::ExternalLinkage, "add", TheModule.get());

5. **BasicBlock**: Criar objetos BasicBlock dentro de cada Function. O IRBuilder é posicionado em um bloco para inserir instruções.  
   \#include "llvm/IR/BasicBlock.h"  
   // ...  
   llvm::BasicBlock\* BB \= llvm::BasicBlock::Create(TheContext, "entry", F);  
   Builder.SetInsertPoint(BB);

6. **Criação de Instruções com IRBuilder**: O IRBuilder possui métodos para criar todas as instruções LLVM IR (e.g., CreateAdd, CreateAlloca, CreateLoad, CreateStore, CreateBr, CreateCall, CreateRet).  
   // Supondo que os argumentos da função F são %a e %b  
   llvm::Argument \*ArgX \= F-\>arg\_begin();  
   llvm::Argument \*ArgY \= (F-\>arg\_begin() \+ 1);  
   ArgX-\>setName("a");  
   ArgY-\>setName("b");

   llvm::Value\* Sum \= Builder.CreateAdd(ArgX, ArgY, "sumtmp");  
   Builder.CreateRet(Sum);

7. **Value**: A maioria das instruções, operandos, constantes e argumentos são subclasses de llvm::Value.

A API é extensa e permite controle fino sobre todos os aspectos da IR. O tutorial "Kaleidoscope" é um excelente ponto de partida para aprender a usar a API C++.

### **3.3. O papel crucial da Tabela de Símbolos na tradução para LLVM IR**

Durante a tradução do código fonte (ou de uma IR de alto nível) para LLVM IR, a **Tabela de Símbolos** do compilador desempenha um papel vital. Ela armazena informações sobre os identificadores (variáveis, funções, tipos) e seus atributos, incluindo seus correspondentes llvm::Value\* na IR.

Ao gerar LLVM IR:

* Para **variáveis locais**, a tabela de símbolos mapeia o nome da variável para o llvm::AllocaInst\* (que é um llvm::Value\*) que representa o ponteiro para a memória alocada na stack. Quando a variável é usada em uma expressão, seu valor é carregado (CreateLoad) usando este ponteiro. Quando é atribuída, o novo valor é armazenado (CreateStore) no endereço do ponteiro.  
* Para **variáveis globais**, a tabela de símbolos mapeia o nome para o llvm::GlobalVariable\* correspondente no módulo LLVM.  
* Para **funções**, a tabela de símbolos mapeia o nome da função para o llvm::Function\* correspondente no módulo LLVM. Isso é usado para gerar instruções CreateCall.  
* Para **parâmetros de função**, a tabela de símbolos mapeia o nome do parâmetro para o llvm::Argument\* correspondente (que é um llvm::Value\*). Estes são diretamente utilizáveis como operandos SSA.  
* **Tipos:** A tabela de símbolos também pode ajudar a resolver nomes de tipos definidos pelo usuário (como structs) para seus correspondentes llvm::StructType\* na LLVM IR.

Sem uma tabela de símbolos eficaz, que gerencie escopos e mapeie nomes para seus Value\*s LLVM, seria impossível rastrear onde os dados estão armazenados ou quais funções chamar. Ela faz a ponte entre os nomes simbólicos da linguagem fonte e os operandos/endereços da LLVM IR.

## **4\. O Pipeline de Backend do LLVM**

Após a geração da LLVM IR (possivelmente já otimizada por passes de IR), o backend do LLVM entra em ação para transformar essa IR em código de máquina executável para uma arquitetura específica. Este é um processo complexo, multifásico, que o LLVM abstrai consideravelmente para o desenvolvedor do frontend.

### **4.1. Visão geral das fases do backend do LLVM após a geração da IR**

O pipeline de backend típico do LLVM, também conhecido como "Code Generation" (geração de código), envolve as seguintes etapas principais:

1. **Otimizações da LLVM IR (Opcional, mas Comum nesta Fase):**  
   * Embora muitas otimizações possam ser aplicadas antes do backend (usando opt ou passes programáticos), alguns passes de otimização específicos do alvo ou que se beneficiam de informações de mais baixo nível podem ser executados aqui.  
   * **Ferramenta opt:** Pode ser usada para aplicar passes de otimização em arquivos .ll ou .bc. Exemplos de passes:  
     * mem2reg: Promove alocações de memória (criadas por alloca) para registradores SSA, eliminando loads e stores desnecessários. Essencial para obter bom desempenho.  
     * instcombine: Combina sequências de instruções em instruções mais eficientes.  
     * gvn (Global Value Numbering): Elimina redundâncias.  
     * licm (Loop Invariant Code Motion): Move código invariante para fora de laços.  
     * loop-unroll: Desenrola laços.  
     * inline: Substitui chamadas de função pelo corpo da função (inlining).  
     * constprop (Constant Propagation): Propaga constantes.  
     * deadargelim: Elimina argumentos de função não utilizados.  
     * dce (Dead Code Elimination): Remove código morto.  
   * \[Diagrama: LLVM IR \-\> Passes de Otimização (opt ou programáticos) \-\> LLVM IR Otimizada\]  
2. **Seleção de Instruções (Instruction Selection \- ISel):**  
   * Esta é uma das fases mais críticas. A LLVM IR, que é genérica, é traduzida para uma forma de IR específica da máquina alvo, geralmente usando um grafo de seleção de instruções (DAG \- Directed Acyclic Graph), resultando em MachineInstrs.  
   * O LLVM usa um sistema baseado em tabelas (TableGen) para gerar grande parte do código de seleção de instruções a partir de descrições da arquitetura alvo (arquivos .td).  
   * Padrões de LLVM IR são pareados com sequências de instruções da máquina alvo. Por exemplo, uma instrução add i32 %a, %b na LLVM IR pode ser mapeada para uma instrução ADD R1, R2, R3 em uma máquina RISC ou ADD EAX, EBX em x86.  
   * O objetivo é escolher as instruções da máquina que implementam corretamente a semântica da IR e, idealmente, são eficientes.  
   * \[Diagrama: LLVM IR Otimizada \-\> SelectionDAG ISel \-\> MachineInstrs (ainda com registradores virtuais)\]  
3. **Agendamento de Instruções Pré-Alocação de Registradores (Pre-RA Scheduling):**  
   * Reordena as MachineInstrs para melhorar o desempenho, explorando o paralelismo em nível de instrução e minimizando stalls, antes da alocação de registradores.  
4. **Alocação de Registradores (Register Allocation):**  
   * As instruções da máquina agora operam sobre um número infinito de "registradores virtuais". Esta fase mapeia esses registradores virtuais para o conjunto finito de registradores físicos disponíveis na arquitetura alvo.  
   * Se não houver registradores físicos suficientes, alguns valores são "derramados" (spilled) para a memória (stack), exigindo loads e stores adicionais.  
   * O LLVM inclui vários algoritmos de alocação de registradores (e.g., Basic, Greedy, Linear Scan, PBQP).  
   * Esta é uma tarefa NP-completa, então heurísticas são usadas.  
   * \[Diagrama: MachineInstrs (virtuais) \-\> Alocador de Registradores \-\> MachineInstrs (físicos ou com spills)\]  
5. **Agendamento de Instruções Pós-Alocação de Registradores (Post-RA Scheduling):**  
   * Reordena novamente as instruções da máquina após a alocação de registradores (e a inserção de código de spill) para otimizar o desempenho, preencher slots de atraso (delay slots) e acomodar as restrições impostas pelos registradores físicos.  
6. **Otimizações de Código Máquina (Peephole, etc.):**  
   * Passes finais de otimização que operam diretamente nas MachineInstrs, como otimizações "peephole" que procuram por pequenas sequências de instruções que podem ser substituídas por sequências mais eficientes.  
7. **Emissão de Código Máquina (Assembly ou código objeto):**  
   * A sequência final de instruções da máquina é emitida.  
   * **Emissão de Assembly:** Gera um arquivo de texto em linguagem de montagem (e.g., .s).  
   * **Emissão de Código Objeto:** Gera diretamente um arquivo objeto (e.g., .o, .obj) contendo código de máquina binário e metadados para o linker. O LLVM tem seu próprio MC (Machine Code) layer para isso.  
   * \[Diagrama: MachineInstrs (físicos, reordenados) \-\> Emissor de Código \-\> Código Assembly (.s) OU Código Objeto (.o)\]

### **4.2. Como o LLVM abstrai a complexidade dessas fases para o desenvolvedor do frontend do compilador**

Para o desenvolvedor do frontend de um compilador, a principal responsabilidade é **gerar uma LLVM IR correta e semanticamente equivalente ao código fonte**. Uma vez que essa IR é entregue ao LLVM, o framework se encarrega das complexidades do backend:

* **Independência de Alvo:** O desenvolvedor do frontend não precisa se preocupar com os detalhes intrincados de cada arquitetura de máquina (conjunto de instruções, número de registradores, pipeline, etc.). A LLVM IR é o único alvo.  
* **Otimizações Genéricas e Específicas:** Muitas otimizações são realizadas na própria IR e são, portanto, independentes do alvo. O LLVM também aplica otimizações específicas do alvo durante o backend, mas isso é gerenciado internamente.  
* **Descrições de Alvo (Target Descriptions):** A complexidade de uma arquitetura específica é encapsulada em arquivos de descrição de alvo (.td). O LLVM usa o TableGen para processar esses arquivos e gerar grande parte do código do backend (seleção de instruções, informações de registradores, etc.). O desenvolvedor do frontend não interage diretamente com isso, a menos que esteja desenvolvendo um backend para uma nova arquitetura.  
* **Interface Clara:** A interface para invocar o backend é relativamente simples (e.g., através da API C++ do LLVM ou ferramentas de linha de comando como llc). O frontend fornece a IR e especifica a arquitetura alvo, e o LLVM produz o código de máquina.

Em resumo, o LLVM atua como uma "caixa preta" altamente sofisticada para a geração de código otimizado para múltiplas arquiteturas, permitindo que os desenvolvedores de compiladores se concentrem na análise da linguagem fonte e na tradução para uma IR bem definida.

## **5\. Utilizando as Ferramentas LLVM**

O ecossistema LLVM oferece um conjunto poderoso de ferramentas de linha de comando para compilar, analisar, otimizar e executar código LLVM IR. Dominar essas ferramentas é essencial para o desenvolvimento e depuração de compiladores que utilizam LLVM.

### **5.1. llc: O compilador estático do LLVM**

* **Função:** llc (LLVM static compiler) traduz LLVM IR (seja na forma textual .ll ou bitcode .bc) para código assembly específico da arquitetura alvo (arquivos .s) ou para código objeto (arquivos .o).  
* **Uso Básico:**  
  \# Compilar LLVM IR para assembly (alvo padrão: host)  
  llc meu\_codigo.ll \-o meu\_codigo.s  
  llc meu\_codigo.bc \-o meu\_codigo.s

  \# Compilar LLVM IR para código objeto  
  llc meu\_codigo.ll \-filetype=obj \-o meu\_codigo.o  
  llc meu\_codigo.bc \-filetype=obj \-o meu\_codigo.o

* Especificando Arquitetura Alvo:  
  Use a opção \-march=\<arquitetura\> para especificar a arquitetura (e.g., x86-64, arm, mips, riscv32, riscv64).  
  llc \-march=x86-64 meu\_codigo.ll \-o meu\_codigo\_x86\_64.s  
  llc \-march=arm meu\_codigo.ll \-o meu\_codigo\_arm.s

  Pode-se também especificar o "triple" completo com \-mtriple=\<target-triple\> (e.g., \-mtriple=armv7-none-linux-gnueabihf).  
* Níveis de Otimização:  
  llc aplica passes de otimização do backend. Pode-se controlar o nível com \-O\<nível\> (e.g., \-O0, \-O1, \-O2, \-O3, \-Os para tamanho, \-Oz para tamanho ainda menor).  
  llc \-O2 meu\_codigo.ll \-o meu\_codigo\_otimizado.s

* **Outras Opções Úteis:**  
  * \-relocation-model=\<modelo\>: static, pic (Position Independent Code), dynamic-no-pic, ropi-rwpi.  
  * \-mcpu=\<cpu\_especifica\>: Otimiza para uma CPU específica dentro da arquitetura (e.g., \-mcpu=cortex-a72).  
  * \-mattr=+feature,-feature: Habilita ou desabilita features específicas da CPU (e.g., \-mattr=+neon).  
  * \-stats: Exibe estatísticas sobre os passes do backend.  
  * \-verify-machineinstrs: Verifica as instruções de máquina geradas.

### **5.2. lli: O interpretador de LLVM IR**

* **Função:** lli (LLVM IR interpreter/JIT compiler) executa diretamente código LLVM IR. Pode operar como um interpretador puro ou usar um compilador Just-In-Time (JIT) para melhor desempenho. É extremamente útil para testar e depurar a IR gerada pelo frontend do seu compilador.  
* **Uso Básico:**  
  lli meu\_codigo.ll  
  lli meu\_codigo.bc

* Passando Argumentos para a Função main:  
  Se seu código LLVM IR define uma função main que espera argumentos (como argc, argv), você pode passá-los na linha de comando após o nome do arquivo.  
  ; exemplo\_main.ll  
  define i32 @main(i32 %argc, ptr %argv) {  
    ; ...  
    ret i32 0  
  }  
  \`\`\`bash  
  lli exemplo\_main.ll arg1 arg2

* **Opções:**  
  * \-force-interpreter=true: Força o uso do interpretador puro (em vez do JIT).  
  * \-jit-kind=\<tipo\>: Seleciona o tipo de JIT (e.g., mcjit ou orc). O padrão geralmente é orc.  
  * Opções de otimização como \-O1, \-O2, \-O3 também podem ser aplicadas antes da execução pelo JIT.

### **5.3. llvm-as e llvm-dis: Montador e desmontador de LLVM bitcode**

* **llvm-as (LLVM assembler):**  
  * **Função:** Converte a representação textual da LLVM IR (arquivos .ll) para a representação binária compacta chamada "bitcode" (arquivos .bc).  
  * **Uso Básico:**  
    llvm-as meu\_codigo.ll \-o meu\_codigo.bc

* **llvm-dis (LLVM disassembler):**  
  * **Função:** Converte arquivos de bitcode LLVM (arquivos .bc) de volta para a forma textual legível por humanos (arquivos .ll).  
  * **Uso Básico:**  
    llvm-dis meu\_codigo.bc \-o meu\_codigo\_convertido.ll  
    \# Se \-o não for especificado, imprime na saída padrão  
    llvm-dis meu\_codigo.bc

### **5.4. Outras ferramentas relevantes para análise e depuração**

* **opt:**  
  * **Função:** Ferramenta para executar passes de otimização e análise da LLVM IR. Permite aplicar transformações granulares na IR.  
  * **Uso Básico:**  
    \# Aplicar o passe de otimização mem2reg e depois dce, saída em formato textual .ll  
    opt \-passes=mem2reg,dce \-S meu\_codigo.ll \-o meu\_codigo\_otimizado.ll  
    \# O \-S é para saída em formato textual .ll, sem ele, a saída é .bc  
    \# Para listar passes disponíveis (novo PassManager): opt \--print-passes  
    \# Para passes legados: opt \-help | grep "\\-passname"

  * **Análise:** Muitos passes são apenas de análise (e.g., \-passes="print\<dom-tree\>" para imprimir a árvore de dominadores). Use \-disable-output para suprimir a IR e ver apenas a saída da análise.  
* **llvm-link:**  
  * **Função:** Liga múltiplos arquivos LLVM IR (sejam .ll ou .bc) em um único arquivo LLVM IR. Útil para compilação de múltiplos módulos.  
  * **Uso Básico:**  
    llvm-link modulo1.bc modulo2.bc \-o modulo\_final.bc  
    \# Para saída textual: llvm-link modulo1.ll modulo2.ll \-S \-o modulo\_final.ll

* **llvm-nm:**  
  * **Função:** Lista os símbolos em arquivos objeto LLVM ou bitcode. Semelhante à ferramenta nm do Unix.  
* **llvm-objdump:**  
  * **Função:** Exibe informações de arquivos objeto, incluindo desmontagem de seções de código.  
* **FileCheck:**  
  * **Função:** Uma ferramenta para verificar se a saída de um comando corresponde a um conjunto de padrões. Muito usada nos testes do LLVM (e útil para testar seu próprio gerador de código). Você escreve comentários especiais no arquivo de teste que descrevem o que esperar na saída.

; RUN: llc \< %s | FileCheck %s  
; CHECK: minha\_label:  
; CHECK-NEXT: addl %eax, %ebx  
; ...  
define void @minha\_func() {  
entry:  
  ; ... código LLVM IR ...  
  ret void  
}  
A linha RUN especifica como executar o teste. As linhas CHECK especificam o que procurar na saída.

Dominar essas ferramentas acelera o ciclo de desenvolvimento, permitindo inspecionar, transformar e testar a IR gerada em cada etapa do processo de compilação.

## **6\. Estudo de Caso: Integração com o Compilador Buriti**

Esta seção visa analisar conceitualmente a estrutura do Compilador Buriti, com base nas informações fornecidas e no repositório GitHub (https://github.com/edwilsonferreira/compilador\_buriti), e propor como sua saída de geração de código intermediário poderia ser traduzida para LLVM IR.

*Nota: A análise a seguir é baseada na estrutura e nos exemplos inferidos do repositório GitHub em junho de 2025 e em práticas comuns de construção de compiladores. A estrutura interna real do compilador pode variar ou evoluir.*

### **6.1. Análise da estrutura do Compilador Buriti**

Observando a estrutura de um compilador acadêmico típico como o Buriti e a menção à possibilidade de gerar Código de Três Endereços (CTE), podemos inferir:

* **Linguagem Fonte:** O Buriti provavelmente compila uma linguagem imperativa, possivelmente com foco didático, incluindo construções como:  
  * Declaração de variáveis e tipos (e.g., inteiro, real, booleano, talvez arranjos simples).  
  * Expressões aritméticas, lógicas e relacionais.  
  * Comandos de atribuição.  
  * Estruturas de controle de fluxo (e.g., if-then-else, laços while ou for).  
  * Definição e chamada de funções/procedimentos.  
* **Fases do Compilador (Prováveis):**  
  * **Análise Léxica:** Um lexer para tokenizar o código fonte.  
  * **Análise Sintática:** Um parser para construir uma árvore sintática (AST) ou verificar a gramática.  
  * **Análise Semântica:** Verificação de tipos, escopos, declarações, e construção/uso de uma tabela de símbolos.  
  * **Geração de Código Intermediário:** Se o Buriti gera CTE, esta fase traduz a AST (após análise semântica) para uma sequência de instruções de três endereços.  
  * **Geração de Código Final (Alvo para LLVM):** Atualmente, esta fase pode gerar código para uma máquina hipotética ou assembly específico. A integração com LLVM substituiria ou complementaria esta fase.

### **6.2. Proposta de tradução da saída do gerador de código intermediário do Buriti para LLVM IR**

Assumindo que o Compilador Buriti gera Código de Três Endereços (CTE) ou uma representação similar, a tradução para LLVM IR pode ser bastante sistemática. O CTE geralmente possui instruções como:

* x \= y op z (operações binárias)  
* x \= op y (operações unárias)  
* x \= y (cópia/atribuição)  
* goto L (salto incondicional)  
* if x relop y goto L (salto condicional)  
* param x / call p, n (chamada de procedimento/função)  
* x \= M\[y\] / M\[x\] \= y (acesso à memória, indexação de arranjos)  
* label L (definição de rótulo)

O processo de tradução para LLVM IR envolveria:

1. **Inicialização do Módulo LLVM:**  
   * Criar um llvm::LLVMContext.  
   * Criar um llvm::Module para conter todas as funções e variáveis globais.  
   * Inicializar um llvm::IRBuilder\<\>.  
2. **Tradução de Funções/Procedimentos:**  
   * Para cada função ou procedimento no CTE do Buriti:  
     * Determinar os tipos dos parâmetros e o tipo de retorno a partir da tabela de símbolos do Buriti.  
     * Criar um llvm::FunctionType.  
     * Criar um llvm::Function no módulo LLVM.  
     * Nomear os argumentos da função LLVM.  
     * Criar um bloco de entrada (entry) para a função LLVM. Posicionar o IRBuilder neste bloco.  
     * Alocar espaço na stack (alloca) para todas as variáveis locais da função Buriti no bloco de entrada. Armazenar esses AllocaInst\* em um mapa (associado à tabela de símbolos do Buriti) para fácil acesso.  
     * Se os parâmetros da função Buriti podem ser modificados ou seu endereço tomado, alocar espaço para eles também e armazenar os valores dos argumentos iniciais nessas alocações.  
3. **Identificação e Criação de Blocos Básicos LLVM:**  
   * Analisar o CTE para identificar os líderes de blocos básicos (primeira instrução, alvos de saltos, instruções após saltos).  
   * Para cada bloco básico do CTE, criar um llvm::BasicBlock correspondente na função LLVM.  
4. **Tradução de Instruções CTE para LLVM IR (dentro de cada bloco básico):**  
   * Iterar sobre as instruções CTE de um bloco básico. Para cada instrução:  
     * **x \= y op z**:  
       * Carregar os valores de y e z de suas alocações de memória usando CreateLoad (se y e z forem variáveis/temporários armazenados). Se forem constantes, usar llvm::ConstantInt::get(), etc.  
       * Usar o IRBuilder para criar a instrução LLVM correspondente (CreateAdd, CreateSub, CreateMul, CreateSDiv, CreateICmpEQ, etc.).  
       * Armazenar o resultado na alocação de memória de x usando CreateStore.  
     * **x \= y (cópia)**:  
       * Carregar o valor de y.  
       * Armazenar na alocação de x.  
     * **goto L**:  
       * Criar uma instrução CreateBr para o BasicBlock LLVM correspondente ao rótulo L.  
     * **if x relop y goto L\_true**:  
       * Carregar x e y.  
       * Gerar a comparação com CreateICmp \<cond\> (ou CreateFCmp).  
       * Criar uma instrução CreateCondBr para o BasicBlock LLVM de L\_true e para o BasicBlock da próxima instrução CTE (se o if for falso).  
     * **label L**: Já tratado na criação dos BasicBlocks. O IRBuilder deve ser posicionado no BasicBlock correto.  
     * **param x / call p, n**:  
       * Para cada param x: carregar o valor de x e adicioná-lo a um std::vector\<llvm::Value\*\>.  
       * Para call p, n: obter o llvm::Function\* para p (da tabela de símbolos/módulo). Criar uma instrução CreateCall com os argumentos coletados. Se a função p retorna um valor e ele é atribuído a um temporário no CTE (e.g., t \= call p, n), armazenar o resultado da CreateCall na alocação de t.  
     * **Acesso a Arranjos (e.g., x \= arr\[i\] ou arr\[i\] \= y)**:  
       * Obter o ponteiro base do arranjo (arr) da sua AllocaInst.  
       * Carregar o valor do índice i.  
       * Usar CreateGEP (GetElementPtr) para calcular o endereço do elemento.  
         * Para arranjos unidimensionais alocados como \[N x type\], o GEP geralmente precisa de dois índices: i64 0 (para desreferenciar o ponteiro do alloca para o início do array) e o índice i (convertido para i64 ou i32 conforme o tipo de índice esperado pelo GEP para o tipo do array).  
       * Para x \= arr\[i\]: usar CreateLoad a partir do endereço calculado pelo GEP. Armazenar em x.  
       * Para arr\[i\] \= y: carregar o valor de y. Usar CreateStore para o endereço calculado pelo GEP.  
5. **Finalização da Função:**  
   * Garantir que todos os blocos básicos terminem com uma instrução terminadora (e.g., CreateRet, CreateBr).  
   * Para funções que retornam valor, usar CreateRet com o valor a ser retornado (carregado da variável de retorno, se houver). Para procedimentos (void), usar CreateRetVoid().

### **6.3. Exemplos de como as construções específicas da linguagem do Buriti seriam representadas em LLVM IR**

**Exemplo 1: Atribuição e Expressão Aritmética**

Código Buriti (hipotético):

var a, b, c: integer;  
begin  
  a := 10;  
  b := 20;  
  c := a \+ b \* 2;  
end

Possível CTE:

t1 \= 10  
a \= t1  
t2 \= 20  
b \= t2  
t3 \= 2  
t4 \= b \* t3  
t5 \= a \+ t4  
c \= t5

LLVM IR Gerada (simplificada, sem otimização mem2reg ainda):

define void @buriti\_main\_function() {  
entry:  
  %ptr\_a \= alloca i32, align 4  
  %ptr\_b \= alloca i32, align 4  
  %ptr\_c \= alloca i32, align 4  
  %ptr\_t1 \= alloca i32, align 4 ; Temporários também podem ser alocados  
  %ptr\_t2 \= alloca i32, align 4  
  %ptr\_t3 \= alloca i32, align 4  
  %ptr\_t4 \= alloca i32, align 4  
  %ptr\_t5 \= alloca i32, align 4

  store i32 10, ptr %ptr\_t1, align 4       ; t1 \= 10  
  %val\_t1 \= load i32, ptr %ptr\_t1, align 4  
  store i32 %val\_t1, ptr %ptr\_a, align 4   ; a \= t1

  store i32 20, ptr %ptr\_t2, align 4       ; t2 \= 20  
  %val\_t2 \= load i32, ptr %ptr\_t2, align 4  
  store i32 %val\_t2, ptr %ptr\_b, align 4   ; b \= t2

  store i32 2, ptr %ptr\_t3, align 4        ; t3 \= 2  
  %val\_b\_loaded \= load i32, ptr %ptr\_b, align 4  
  %val\_t3\_loaded \= load i32, ptr %ptr\_t3, align 4  
  %mul\_res \= mul i32 %val\_b\_loaded, %val\_t3\_loaded  
  store i32 %mul\_res, ptr %ptr\_t4, align 4 ; t4 \= b \* t3

  %val\_a\_loaded \= load i32, ptr %ptr\_a, align 4  
  %val\_t4\_loaded \= load i32, ptr %ptr\_t4, align 4  
  %add\_res \= add i32 %val\_a\_loaded, %val\_t4\_loaded  
  store i32 %add\_res, ptr %ptr\_t5, align 4 ; t5 \= a \+ t4

  %val\_t5\_loaded \= load i32, ptr %ptr\_t5, align 4  
  store i32 %val\_t5\_loaded, ptr %ptr\_c, align 4 ; c \= t5

  ret void  
}

*Após o passe mem2reg, esta IR seria significativamente mais limpa, com temporários e variáveis locais promovidos a registradores SSA.*

**Exemplo 2: Estrutura de Controle if-then**

Código Buriti (hipotético):

var x, y: integer;  
// ... inicializar x, y ...  
if (x \> y) then  
  x := 0;  
// continua

LLVM IR Gerada:

; ... alloca para x e y, inicializações ...  
  %val\_x\_loaded \= load i32, ptr %ptr\_x, align 4  
  %val\_y\_loaded \= load i32, ptr %ptr\_y, align 4  
  %cond \= icmp sgt i32 %val\_x\_loaded, %val\_y\_loaded ; x \> y  
  br i1 %cond, label %then\_block, label %merge\_block

then\_block:  
  store i32 0, ptr %ptr\_x, align 4 ; x := 0  
  br label %merge\_block

merge\_block:  
  ; código após o if  
  ret void

### **6.4. Desafios e decisões de design ao integrar LLVM a um compilador existente ou novo**

**Desafios:**

1. **Mapeamento de Tipos:** Garantir que o sistema de tipos da linguagem fonte seja corretamente mapeado para os tipos da LLVM IR. Linguagens com tipagem dinâmica ou inferência de tipos complexa podem apresentar desafios adicionais.  
2. **Gerenciamento da Tabela de Símbolos:** A tabela de símbolos do compilador precisa ser robusta e capaz de fornecer as informações necessárias para a geração da LLVM IR (e.g., qual llvm::Value\* de alloca ou argumento corresponde a qual variável/parâmetro).  
3. **Forma SSA e Nós PHI:** Embora o frontend não precise gerar IR *diretamente* em SSA (o passe mem2reg pode converter alloca/load/store para registradores SSA e inserir nós PHI), entender as implicações da SSA é importante para gerar IR que otimiza bem. A inserção manual de nós PHI pode ser complexa.  
4. **Tratamento de Erros e Debugging:** Mapear informações de debug (números de linha, nomes de variáveis) da fonte para a LLVM IR (usando metadados de debug) é crucial para a depuração do código compilado. Isso adiciona complexidade ao gerador de IR.  
5. **Interface com a API C++ do LLVM:** A API C++ do LLVM é poderosa, mas também vasta e com uma curva de aprendizado.  
6. **Construções Específicas da Linguagem:** Algumas linguagens possuem características que não mapeiam trivialmente para LLVM IR (e.g., exceções com semântica específica, co-rotinas, certas formas de metaprogramação, strings com gerenciamento complexo). Isso pode exigir a implementação de "runtime support functions" (funções de suporte em tempo de execução escritas em C/C++ e linkadas) ou representações mais elaboradas na IR.  
7. **Convenções de Chamada e ABI:** Para interoperabilidade com código externo ou bibliotecas do sistema, é preciso respeitar a Application Binary Interface (ABI) da plataforma alvo, o que o LLVM geralmente ajuda a gerenciar, mas pode requerer atenção para tipos de dados complexos ou passagem de structs.

**Decisões de Design:**

1. **Nível da IR do Frontend:** O frontend gerará LLVM IR diretamente a partir da AST, ou gerará uma IR de mais alto nível (como CTE) primeiro e depois a converterá para LLVM IR? Para linguagens mais complexas, uma IR de frontend pode ser benéfica para realizar otimizações de alto nível antes de baixar para LLVM IR.  
2. **Uso da API vs. Geração Textual:** Para qualquer compilador sério, a API C++ do LLVM é a escolha. A geração textual é mais para aprendizado inicial, prototipagem muito rápida ou ferramentas simples.  
3. **Quando e Como aplicar mem2reg:** É comum gerar allocas para todas as variáveis locais e depois rodar o passe mem2reg (programaticamente, usando FunctionPassManager) por função para obter uma IR em forma SSA mais limpa.  
4. **Geração de Metadados de Debug:** Decidir o nível de detalhe dos metadados de debug. Incluir informações básicas (linha, arquivo) é um bom começo.  
5. **Abstração da Geração de IR:** Criar classes ou funções de ajuda (visitor pattern na AST, por exemplo) no frontend do compilador para encapsular padrões comuns de geração de LLVM IR (e.g., "gerar código para uma expressão binária", "gerar código para um if") torna o código do gerador de IR mais limpo, modular e gerenciável.  
6. **Tratamento de Erros de Compilação:** Como o gerador de código LLVM IR lida com erros detectados em fases anteriores (e.g., erros semânticos)? Geralmente, a geração de IR não prossegue se houver erros. O sistema de diagnóstico do compilador deve ser robusto.  
7. **Suporte a Runtime:** Identificar quais funcionalidades da linguagem necessitarão de uma biblioteca de runtime e como a LLVM IR fará interface com ela (e.g., para alocação de memória dinâmica, operações de string complexas, I/O).

A integração do LLVM como backend do Compilador Buriti, ou de qualquer novo compilador, é um investimento significativo que pode trazer grandes recompensas em termos de desempenho do código gerado, portabilidade e acesso a um ecossistema maduro de ferramentas de desenvolvimento.

## **7\. Fundamentação Teórica e Referências**

A compreensão da geração de código com LLVM é enriquecida pelo conhecimento dos princípios fundamentais da teoria de compiladores. Muitos dos conceitos implementados e abstraídos pelo LLVM são discutidos em profundidade em textos clássicos da área.

### **7.1. Conectar os conceitos apresentados com os princípios discutidos em livros clássicos de compiladores**

* **Representações Intermediárias (IRs):**  
  * Livros como **"Compilers: Principles, Techniques, and Tools" (Aho, Lam, Sethi, Ullman \- o "Dragon Book")** dedicam capítulos inteiros às diferentes formas de IRs, incluindo Código de Três Endereços, árvores sintáticas abstratas (ASAs), grafos de fluxo de controle e representações baseadas em stack. A LLVM IR pode ser vista como uma forma sofisticada de código de três endereços em formato SSA, com uma tipagem forte e um conjunto de instruções bem definido.  
  * **"Modern Compiler Implementation in C/Java/ML" (Appel)** também discute várias IRs e suas vantagens, com ênfase na praticidade da implementação e na facilitação de otimizações como a seleção de instruções.  
  * A necessidade de uma IR que facilite otimizações e retargeting é um tema central nesses textos, e o LLVM é um exemplo moderno proeminente dessa filosofia.  
* **Forma SSA (Static Single Assignment):**  
  * Embora o Dragon Book original não cobrisse SSA extensivamente (edições mais recentes sim), **"Engineering a Compiler" (Cooper & Torczon)** oferece uma excelente discussão sobre a forma SSA, sua construção (incluindo a colocação de funções $ \\Phi $ \- PHI nodes) e como ela beneficia diversas otimizações (e.g., propagação de constantes, eliminação de código morto). A LLVM IR é um exemplo prático proeminente do uso de SSA.  
  * **"Advanced Compiler Design and Implementation" (Muchnick)** também aborda SSA e suas aplicações em otimizações.  
* **Geração de Código para Construções de Linguagem:**  
  * Todos os principais livros de compiladores detalham algoritmos e técnicas para traduzir construções de linguagens de alto nível (expressões, comandos de atribuição, estruturas de controle de fluxo, chamadas de função) para código de máquina ou uma IR de baixo nível. Os exemplos de mapeamento para LLVM IR apresentados neste manual (e.g., if-then-else para blocos básicos com br condicional, laços para estruturas com nós PHI) são aplicações diretas desses princípios.  
* **Otimização de Código:**  
  * O "Dragon Book", Appel, Cooper & Torczon, e Muchnick cobrem uma vasta gama de técnicas de otimização, muitas das quais são implementadas como passes no LLVM (e.g., eliminação de subexpressão comum, propagação de constante, código invariante de laço, inlining, eliminação de código morto, otimizações de laço, análise de alias). A arquitetura de passes do LLVM é uma manifestação flexível dessas técnicas.  
* **Backend (Geração de Código Final):**  
  * **Seleção de Instruções:** Técnicas como pareamento de árvores (tree matching) ou cobertura de DAGs (DAG covering), discutidas em profundidade por Appel e no Dragon Book, são fundamentais para a fase de seleção de instruções do LLVM. O TableGen do LLVM automatiza parte da geração de seletores de instrução baseados nessas ideias.  
  * **Alocação de Registradores:** O problema da alocação de registradores (frequentemente modelado como coloração de grafos de interferência) é um tópico clássico. Livros como o de Appel, Cooper & Torczon, e Muchnick discutem algoritmos como coloração por grafos de interferência e abordagens mais lineares como o Linear Scan (usado no LLVM) ou baseadas em Program Building Quangos (PBQP).  
  * **Agendamento de Instruções:** O agendamento para explorar paralelismo em nível de instrução e minimizar stalls é coberto, especialmente em contextos de arquiteturas pipeline e superescalares.  
* **Tabela de Símbolos:**  
  * A importância da tabela de símbolos como uma estrutura de dados central que armazena informações sobre identificadores (nomes, tipos, escopos, atributos) é enfatizada desde os primeiros capítulos de qualquer livro de compiladores. Sua interação com a geração de IR é crucial para associar nomes a localizações de memória ou registradores virtuais.

### **7.2. Incentivar a consulta à documentação oficial do LLVM**

Embora os livros clássicos forneçam a base teórica, a documentação oficial do LLVM é indispensável para o uso prático e aprofundado da infraestrutura:

* **LLVM Language Reference Manual (LangRef.html):**  
  * A referência definitiva para a sintaxe e semântica da LLVM IR. Detalha cada tipo, instrução, atributo e metadado.  
  * Link (geralmente encontrado no site oficial llvm.org/docs/): [https://llvm.org/docs/LangRef.html](https://llvm.org/docs/LangRef.html)  
* **LLVM Programmer's Manual:**  
  * Guia para desenvolvedores que desejam usar as bibliotecas LLVM (e.g., a API C++). Contém informações sobre a arquitetura interna, classes importantes e como interagir com o sistema.  
  * Link: [https://llvm.org/docs/ProgrammersManual.html](https://llvm.org/docs/ProgrammersManual.html)  
* **Tutorials and How-Tos:**  
  * **"My First Language Frontend with LLVM" (Kaleidoscope Tutorial):** Um tutorial prático e famoso que guia o leitor na construção de um compilador para uma linguagem simples (Kaleidoscope) usando a API C++ do LLVM. Cobre análise léxica, parsing, construção de AST, geração de LLVM IR e JITing.  
    * Link: [https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html](https://llvm.org/docs/tutorial/MyFirstLanguageFrontend/index.html)  
  * **"Writing an LLVM Pass":** Explica como desenvolver passes de análise e transformação personalizados.  
    * Link: [https://llvm.org/docs/WritingAnLLVMPass.html](https://llvm.org/docs/WritingAnLLVMPass.html) (Nota: existem versões para o novo e o antigo PassManager).  
  * **"LLVM Essentials" (documentação do opt e llc):**  
    * Link: [https://llvm.org/docs/CommandGuide/opt.html](https://llvm.org/docs/CommandGuide/opt.html)  
    * Link: [https://llvm.org/docs/CommandGuide/llc.html](https://llvm.org/docs/CommandGuide/llc.html)  
  * Diversos outros tutoriais e guias sobre tópicos específicos (TableGen, Garbage Collection, Debugging Information, etc.).  
* **LLVM Blog:**  
  * Artigos e anúncios sobre novos desenvolvimentos, recursos e usos do LLVM.  
  * Link: [https://blog.llvm.org/](https://blog.llvm.org/)  
* **Documentação das Ferramentas de Linha de Comando:**  
  * Cada ferramenta ( llc, lli, opt, etc.) geralmente tem sua própria página de manual ou documentação detalhando suas opções (acessível via \--help ou no site).

A combinação do entendimento teórico dos princípios de compiladores com o estudo detalhado da documentação e exemplos práticos do LLVM é a chave para se tornar proficiente na geração de código com esta poderosa infraestrutura.

## **8\. Guia Prático e Exercícios**

Esta seção oferece diretrizes para configurar um ambiente de desenvolvimento com LLVM e propõe exercícios práticos para solidificar o aprendizado sobre a geração de código LLVM IR.

### **8.1. Diretrizes sobre como configurar um ambiente de desenvolvimento com LLVM**

A configuração do ambiente LLVM pode variar ligeiramente dependendo do sistema operacional (Linux, macOS, Windows). A abordagem mais comum para desenvolvimento é compilar o LLVM a partir do código fonte, o que garante acesso à versão mais recente e permite configurar quais componentes construir.

**Passos Gerais (Exemplo para Linux/macOS):**

1. **Pré-requisitos:**  
   * **Compilador C++ moderno:** Clang (recomendado, pois o LLVM é frequentemente construído com ele) ou GCC (versão recente que suporte C++17 ou superior).  
   * **CMake:** Sistema de build usado pelo LLVM (versão 3.13.4 ou mais recente, preferencialmente mais nova).  
   * **Ninja (opcional, mas recomendado):** Um sistema de build rápido que pode ser usado com CMake.  
   * **Python:** Necessário para alguns scripts (Python 3.6+).  
   * **Git:** Para clonar o repositório do LLVM.  
   * **Zlib, libffi, libxml2 (opcional):** Para funcionalidades adicionais.  
2. Obter o Código Fonte do LLVM:  
   O LLVM é um monorepositório contendo o LLVM core, Clang, LLD, LLDB, e outros subprojetos.  
   git clone \[https://github.com/llvm/llvm-project.git\](https://github.com/llvm/llvm-project.git)  
   cd llvm-project  
   \# Opcional: checkout uma release específica, e.g., git checkout llvmorg-17.0.1

3. Configurar o Build com CMake:  
   É recomendado criar um diretório de build separado (fora da árvore de fontes).  
   mkdir build  
   cd build

   Existem muitas opções de CMake. Algumas comuns para desenvolvimento:  
   cmake \-G Ninja ../llvm \\  
         \-DCMAKE\_BUILD\_TYPE=Debug \\ \# Ou Release, RelWithDebInfo  
         \-DLLVM\_ENABLE\_PROJECTS="clang;lld" \\ \# Quais projetos construir (e.g., clang, lld, lldb)  
         \-DLLVM\_TARGETS\_TO\_BUILD="X86;ARM;AArch64" \\ \# Arquiteturas alvo a serem suportadas (Native para apenas a do host)  
         \-DLLVM\_ENABLE\_ASSERTIONS=ON \\ \# Habilita asserções (bom para debug)  
         \-DCMAKE\_INSTALL\_PREFIX=/usr/local/llvm\_custom \# Opcional: onde instalar

   * CMAKE\_BUILD\_TYPE: Debug para desenvolvimento (mais lento, mais informações de debug), Release para desempenho, RelWithDebInfo (Release com informações de debug).  
   * LLVM\_ENABLE\_PROJECTS: Lista de subprojetos a serem construídos (e.g., clang, lld, compiler-rt, lldb). Separados por ponto e vírgula.  
   * LLVM\_TARGETS\_TO\_BUILD: Lista de backends de arquitetura a serem construídos (e.g., X86, ARM, AArch64, RISCV). Native constrói apenas para a arquitetura host.  
   * LLVM\_ENABLE\_ASSERTIONS=ON: Muito útil durante o desenvolvimento, pois ativa verificações internas que podem pegar erros mais cedo. Desabilitar para builds de Release para melhor desempenho.  
4. Compilar o LLVM:  
   Se estiver usando Ninja:  
   ninja  
   \# Para compilar um target específico, e.g., ninja clang llc opt

   Se estiver usando Make (padrão se \-G Ninja não for usado):  
   make \-jN \# N é o número de cores do processador para compilação paralela

   A compilação pode levar um tempo considerável (de dezenas de minutos a várias horas), especialmente com Debug e múltiplos projetos/alvos.  
5. Instalar (Opcional):  
   Se você especificou CMAKE\_INSTALL\_PREFIX:  
   ninja install  
   \# ou  
   make install

   Isso copiará os cabeçalhos, bibliotecas e executáveis para o diretório especificado. Caso contrário, os executáveis estarão em llvm-project/build/bin e as bibliotecas em llvm-project/build/lib.  
6. Configurar Variáveis de Ambiente (Opcional, mas Conveniente):  
   Adicione o diretório bin da sua instalação/build do LLVM ao seu PATH.  
   export PATH="/usr/local/llvm\_custom/bin:$PATH" \# Se instalado  
   \# ou  
   \# export PATH="/caminho/para/llvm-project/build/bin:$PATH" \# Se usando do diretório de build

   Pode ser necessário configurar LD\_LIBRARY\_PATH (Linux) ou DYLD\_LIBRARY\_PATH (macOS) se as bibliotecas não forem encontradas pelo sistema em tempo de execução, ou usar rpath durante a linkagem dos seus projetos.

Alternativa: Pacotes Pré-compilados:  
Para muitos sistemas operacionais, existem pacotes LLVM pré-compilados disponíveis através de gerenciadores de pacotes (e.g., apt no Debian/Ubuntu, yum/dnf no Fedora/CentOS, brew no macOS, ou downloads diretos do site do LLVM ou GitHub Releases). Esta é uma maneira mais rápida de começar, mas pode oferecer menos flexibilidade ou não ser a versão mais recente. Ex: sudo apt install llvm clang lld.

### **8.2. Propor exercícios práticos**

Os seguintes exercícios são projetados para aumentar a familiaridade com a LLVM IR e as ferramentas LLVM.

**Nível 1: Entendendo e Escrevendo LLVM IR Manualmente**

1. **Olá, Mundo em LLVM IR:**  
   * Escreva manualmente um arquivo .ll que defina uma função @main que chame a função puts (declarada a partir da libc) para imprimir "Ola, LLVM\!" na tela.  
   * Dica: Você precisará de uma string global para "Ola, LLVM\!", usar getelementptr para obter um ptr para ela (como ptr to i8 ou simplesmente ptr), e call para @puts.  
   * Compile com llc seu\_arquivo.ll \-o seu\_arquivo.s, depois monte e linke com gcc seu\_arquivo.s \-o programa e execute. Ou execute diretamente com lli seu\_arquivo.ll.  
2. **Funções Simples:**  
   * Escreva LLVM IR para uma função @soma\_int que receba dois inteiros de 32 bits e retorne o resultado da soma.  
   * Escreva LLVM IR para uma função @eh\_par que receba um inteiro de 32 bits e retorne 1 (tipo i1) se ele for par e 0 se for ímpar. Use srem e icmp eq.  
3. **Estruturas de Controle:**  
   * Traduza manualmente o seguinte pseudocódigo para LLVM IR em uma função @max\_abs:  
     funcao max\_abs(int a, int b) {  
       int res;  
       if (a \> b) {  
         res \= a;  
       } else {  
         res \= b;  
       }  
       if (res \< 0\) {  
         res \= \-res; // assumindo que \-res é 0 \- res  
       }  
       return res;  
     }

   * Traduza um laço while que calcule a soma dos números de 1 a N (N passado como argumento). Use alloca/load/store inicialmente.

**Nível 2: Usando as Ferramentas LLVM**

1. **Compilando e Otimizando:**  
   * Pegue o arquivo .ll do laço while do exercício anterior.  
   * Compile-o para assembly x86-64 usando llc. Examine o assembly.  
   * Use opt \-passes=mem2reg \-S seu\_arquivo.ll \-o seu\_arquivo.opt.ll. Compare a IR original com a otimizada (que agora deve usar nós PHI).  
   * Aplique mais otimizações com opt \-O2 \-S seu\_arquivo.opt.ll \-o seu\_arquivo.O2.ll. Examine as mudanças.  
   * Compile seu\_arquivo.O2.ll para assembly com llc e compare com o assembly não otimizado.  
2. **Interpretando:**  
   * Crie uma função @main que chame suas funções @soma\_int e @eh\_par com alguns valores e use printf (declarada) para imprimir os resultados. Execute com lli.  
3. **Analisando com opt:**  
   * Use opt com passes de análise como \-passes="print\<callgraph\>" ou \-passes="print\<dom-tree\>" em um de seus arquivos .ll (pode ser necessário adicionar \-disable-output para suprimir a escrita da IR e ver apenas a saída da análise).

**Nível 3: Geração de LLVM IR com a API (C++)**

Requer um ambiente de desenvolvimento C++ configurado para linkar com as bibliotecas LLVM (Core, IR, Support, etc.). Use o tutorial "Kaleidoscope" como guia principal e a documentação da API.

1. **Gerador de Expressões Simples:**  
   * Escreva um programa C++ que use a API do LLVM (LLVMContext, Module, IRBuilder) para gerar LLVM IR para uma função que calcula (a \+ b) \* 20, onde a e b são argumentos i32.  
   * Imprima a IR gerada para o console (TheModule-\>print(llvm::errs(), nullptr);) ou salve em um arquivo .ll.  
2. **Gerador para if-then-else:**  
   * Estenda o programa anterior para gerar LLVM IR para uma função que implemente:  
     int func(int a, int b) { if (a \< b) return a; else return b; }  
   * Você precisará criar múltiplos BasicBlocks e usar CreateCondBr e CreateBr.  
3. **(Avançado) Mini-Linguagem (Adaptado do Kaleidoscope):**  
   * Implemente um parser simples (pode ser manual para uma gramática muito restrita, ou usando ferramentas como Flex/Bison se o foco for o parser) para uma linguagem de expressões aritméticas simples (números, identificadores para variáveis locais simples, \+, \*).  
   * Faça seu programa gerar LLVM IR para as expressões parseadas dentro de uma função. Assuma que as variáveis são argumentos da função ou alocadas localmente.  
   * Adicione suporte para chamadas a uma função externa declarada (e.g., @print\_int(i32)).

**Dicas para os Exercícios com API:**

* **Comece Simples:** Não tente implementar tudo de uma vez. Compile e teste frequentemente.  
* **CMake para seu Projeto:** Use CMake para configurar seu projeto C++ para encontrar e linkar com as bibliotecas LLVM. Exemplo de CMakeLists.txt:  
  cmake\_minimum\_required(VERSION 3.10)  
  project(MeuGeradorLLVM)

  set(CMAKE\_CXX\_STANDARD 17\)

  find\_package(LLVM REQUIRED CONFIG)  
  \# Ou, se LLVM não foi instalado, mas buildado:  
  \# set(LLVM\_DIR /caminho/para/llvm-project/build/lib/cmake/llvm)  
  \# find\_package(LLVM REQUIRED CONFIG)

  include\_directories(${LLVM\_INCLUDE\_DIRS})  
  add\_definitions(${LLVM\_DEFINITIONS})

  add\_executable(meu\_gerador main.cpp)  
  llvm\_map\_components\_to\_libnames(llvm\_libs core ir support mc) \# Adicione componentes conforme necessário  
  target\_link\_libraries(meu\_gerador PRIVATE ${llvm\_libs})

* **Consulte a Documentação:** Mantenha LangRef.html, ProgrammersManual.html e a documentação Doxygen da API abertos.  
* **Inspire-se no Kaleidoscope:** O código do tutorial Kaleidoscope é uma excelente fonte de exemplos de uso da API.  
* **Verifique a IR Gerada:** Use llvm-as para validar a IR textual, llc para compilar e lli para executar.  
* **Use Module::print(errs(), nullptr);** em C++ para imprimir a IR gerada na saída de erro padrão para inspeção.  
* **Debug com Asserções LLVM:** Se você compilou o LLVM com \-DLLVM\_ENABLE\_ASSERTIONS=ON, ele pegará muitos erros de uso da API.

Estes exercícios devem fornecer uma base sólida para entender como a LLVM IR funciona e como as ferramentas e APIs do LLVM podem ser usadas para construir backends de compiladores. Boa sorte\!