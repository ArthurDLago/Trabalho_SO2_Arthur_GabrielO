# Questões oficiais — Concorrência, Threads, Sincronização e Processos

---

# 1. ENADE 2017 — Criação, execução e sincronização de duas threads

**Tema:** Threads, memória compartilhada, concorrência e `pthread_join`.

**Fonte:** Indagação — questão do ENADE 2017.

[Questão original no Indagação](https://www.indagacao.com.br/2023/06/enade-2017-considere-o-programa-seguir-que-ilustra-a-criacao-execucao-e-sincronizacao-de-duas-threads.html)

## Contexto da questão

O programa em C utiliza POSIX Threads (`pthread`). Ele possui duas variáveis globais compartilhadas, `x` e `y`, cria duas threads por meio de `pthread_create()` e, ao final, a função principal utiliza `pthread_join()` para aguardar as duas threads.

A questão pergunta quais valores podem ser impressos ao final da execução.

O código utiliza, de forma simplificada:

```c
int x = 0, y = 0;

void funcao1(...) {
    x = 1;
    ...
    if (y == 0)
        printf("1 ");
}

void funcao2(...) {
    y = 1;
    ...
    if (x == 0)
        printf("2 ");
}
```

A questão explora principalmente o fato de que as duas threads executam concorrentemente e que a ordem relativa entre suas instruções não é determinada pelo código.

### Alternativas

A) ambos os valores “1” e “2”.  
B) o valor “1”, necessariamente.  
C) o valor “2”, necessariamente.  
D) o valor “1”, ou o valor “2”, mas nunca ambos.  
E) o valor “1”, ou o valor “2”, ou nenhum valor, mas nunca ambos.

### Gabarito oficial indicado na fonte

**E**

## Resolução comentada

O ponto central é analisar as possíveis intercalações das instruções das duas threads.

As variáveis `x` e `y` pertencem ao espaço de memória compartilhado pelo processo. Portanto, ambas as threads podem ler e modificar essas variáveis.

As duas threads executam aproximadamente:

```text
Thread 1                    Thread 2
---------                    ---------
x = 1                        y = 1
verifica y                   verifica x
```

Como o escalonador pode intercalar a execução, diferentes situações podem ocorrer.

### Caso 1 — Thread 1 verifica `y` antes de Thread 2 executar `y = 1`

Nesse caso, `y` ainda pode ser `0` e a Thread 1 pode imprimir:

```text
1
```

Depois disso, a Thread 2 poderá executar e verificar `x`.

### Caso 2 — Thread 2 verifica `x` antes de Thread 1 executar `x = 1`

Nesse caso, `x` ainda pode ser `0` e a Thread 2 pode imprimir:

```text
2
```

### Caso 3 — As atribuições ocorrem antes das verificações

Se:

```text
x = 1
y = 1
```

forem executadas antes das respectivas verificações, nenhuma das condições poderá ser satisfeita e nenhum valor será impresso.

### Conclusão

A sincronização feita por:

```c
pthread_join(t1, NULL);
pthread_join(t2, NULL);
```

garante que a função principal aguarde o término das threads, mas **não determina a ordem interna das instruções entre elas**.

Portanto, a alternativa **E** é a indicada pela fonte: pode aparecer `1`, `2` ou nenhum dos dois, mas não ambos.

### Conceitos que a questão cobra

- concorrência;
- escalonamento;
- memória compartilhada;
- `pthread_create`;
- `pthread_join`;
- interleaving de instruções;
- sincronização de término.

---

# 2. ENADE 2017 — Sistema multithread e deadlock

**Tema:** Deadlock, semáforos e ordem de aquisição de recursos.

**Fonte:** Indagação — questão do ENADE 2017.

[Questão original no Indagação](https://www.indagacao.com.br/2023/06/enade-2017-um-programador-inexperiente-esta-desenvolvendo-um-sistema-multithread-que-possui-duas-estruturas-de-dados-diferentes-el-e-e2.html)

## Contexto da questão

O problema apresenta duas estruturas de dados compartilhadas, `E1` e `E2`, protegidas por mecanismos de sincronização, e duas threads que podem solicitar os recursos em ordens diferentes.

A situação relevante é:

```text
Thread A: M1 → M2

Thread B: M2 → M1
```

A questão pergunta qual situação pode ocorrer e como ela pode ser evitada.

### Alternativas

A) Não ocorre deadlock porque a sequência de alocação impede naturalmente o problema.

B) Pode ocorrer deadlock, mas ele pode ser evitado simplesmente eliminando cálculos entre os pedidos de alocação.

C) Pode ocorrer deadlock, mas sua baixa probabilidade e consequência inócua não comprometem o programa.

D) Não ocorre deadlock porque o uso de semáforos é suficiente para impedir o problema.

E) Pode ocorrer deadlock e ele pode ser evitado solicitando os recursos na mesma ordem nas duas threads.

### Gabarito

**E**

## Resolução comentada

O problema é causado pela aquisição dos recursos em ordens diferentes.

Considere a seguinte situação:

```text
Thread A
   ↓
obtém M1
   ↓
tenta obter M2
```

Enquanto isso:

```text
Thread B
   ↓
obtém M2
   ↓
tenta obter M1
```

Agora temos:

```text
A possui M1 e espera M2
B possui M2 e espera M1
```

Nenhuma consegue prosseguir.

Isso caracteriza um **deadlock**.

## Como evitar?

Uma estratégia clássica é estabelecer uma ordem global de aquisição:

```text
Thread A: M1 → M2
Thread B: M1 → M2
```

Assim, uma thread pode esperar pela outra, mas não se forma o ciclo de espera causado pela ordem inversa.

### Importante

O simples uso de mutexes ou semáforos **não elimina automaticamente deadlocks**.

Esses mecanismos controlam acesso a recursos, mas a forma como os recursos são adquiridos e liberados também precisa ser analisada.

### Conceitos que a questão cobra

- deadlock;
- exclusão mútua;
- espera e retenção;
- ordem de aquisição;
- sincronização;
- hierarquia de locks.

---

# 3. POSCOMP 2022 — Questão 46 — `fork()` e variáveis após criação de processo

**Tema:** Criação de processos, `fork()` e espaço de memória.

**Fonte:** POSCOMP 2022.

A questão 46 apresenta um programa em C executado em um sistema UNIX. O programa realiza um `fork()`, incrementa uma variável global em ambos os fluxos de execução e depois imprime o valor dessa variável.

[Discussão/referência visual enviada para este trabalho no Reddit](https://www.reddit.com/media?url=https%3A%2F%2Fpreview.redd.it%2Ffundamentos-de-computa%C3%A7%C3%A3o-prova-poscomp-2022-se%C3%A7%C3%A3o-resolvida-v0-waih914ene3b1.png%3Fwidth%3D606%26format%3Dpng%26auto%3Dwebp%26s%3D18ba14b66565572c42391097429ef8c6ca4de3a7)

[Discussão da seção resolvida do POSCOMP 2022 no Reddit](https://www.reddit.com/r/brdev/comments/13xh35q)

[Prova POSCOMP 2022 — referência consultada](https://pt.scribd.com/document/704656748/Prova-2022)

## Enunciado resumido

Após o `fork()` bem-sucedido, existem dois processos: pai e filho.

Cada processo possui sua própria cópia lógica da variável global `i`. O programa incrementa `i` uma vez em cada ramo e depois realiza outro incremento antes de imprimir.

### Alternativas

A) `1 1`  
B) `2 2`  
C) `3 3`  
D) `4 4`  
E) Indeterminado Indeterminado

### Gabarito

**B — `2 2`**

O gabarito definitivo do POSCOMP 2022 registra a alternativa **B** para a questão 46, classificada em Sistemas Operacionais.

## Resolução comentada

Antes do `fork()`:

```text
i = 0
```

O `fork()` cria dois processos.

Depois da criação:

```text
Processo pai  → i = 0
Processo filho → i = 0
```

Cada processo trabalha sobre sua própria cópia do espaço de memória.

O código executa um incremento em cada ramo:

```text
pai   → i = 1
filho → i = 1
```

Depois do `if/else`, ambos executam mais um incremento:

```text
pai   → i = 2
filho → i = 2
```

Portanto, cada processo imprime:

```text
2
```

O resultado observado é:

```text
2 2
```

A ordem entre as duas impressões pode variar, mas o valor produzido por cada processo é 2.

## Ponto conceitual importante

O `fork()` não transforma pai e filho em duas threads que compartilham a mesma variável global.

Após a criação, cada processo possui seu próprio espaço de endereçamento lógico. Por isso, o incremento realizado pelo pai não altera diretamente a cópia da variável pertencente ao filho, e vice-versa.

### Conceitos que a questão cobra

- `fork()`;
- processo pai;
- processo filho;
- espaço de endereçamento;
- cópia do estado após `fork()`;
- concorrência;
- diferença entre processo e thread.

---

# Quadro de revisão das três questões

| Questão | Tema principal | Conceito-chave |
|---|---|---|
| ENADE 2017 — Threads | Concorrência | Ordem de execução não determinística |
| ENADE 2017 — Deadlock | Sincronização | Ordem de aquisição dos recursos |
| POSCOMP 2022 — `fork()` | Processos | Pai e filho possuem espaços de memória separados |

## O que revisar depois de resolver

### Threads

Pergunte:

> A criação de threads determina a ordem de execução?

**Não.**

### `join()`

Pergunte:

> `pthread_join()` controla a ordem das instruções entre as threads?

**Não.** Ele faz a thread chamadora aguardar o término da thread especificada.

### Deadlock

Pergunte:

> Usar mutex/semafóro automaticamente elimina deadlock?

**Não.** A organização da aquisição dos recursos também importa.

### `fork()`

Pergunte:

> Pai e filho compartilham automaticamente a mesma variável global?

**Não.** Após o `fork()`, cada processo possui seu próprio espaço de endereçamento lógico.

