# Resumo: Programação Concorrente, Processos e Threads

**Autores:** Arthur Lago e Gabriel de Oliveira

---

## 1. Introdução à Programação Concorrente
A programação concorrente refere-se à execução de múltiplos fluxos de instrução de forma sobreposta no tempo. Diferente do paralelismo — que exige múltiplos núcleos executando tarefas exatamente no mesmo instante —, a concorrência permite que o sistema operativo gerencie a alternância rápida de tarefas (*context switching*), dando a ilusão de execução simultânea e otimizando o uso da Unidade Central de Processamento (UCP).

---

## 2. Criação e Gerência de Processos
Um **processo** é uma instância de um programa em execução, possuindo o seu próprio espaço de endereçamento de memória isolado, variáveis globais, descritores de ficheiros e contexto de hardware.

* **Bloco de Controlo de Processos (PCB):** Estrutura de dados utilizada pelo sistema operativo para armazenar informações vitais sobre o processo (estado atual, contador de programa, registos da UCP e informações de gestão de memória).
* **Criação de Processos:** Em sistemas baseados em Unix/Linux, a criação ocorre tipicamente através da chamada de sistema `fork()`, que duplica o processo chamador, gerando um processo filho com uma cópia do espaço de memória.
* **Gerência e Estados:** O sistema operativo gere os processos através de transições entre estados fundamentais: *Novo*, *Pronto* (ready), *Em execução* (running), *Espera/Bloqueado* (waiting) e *Terminado* (terminated).

---

## 3. Criação e Gerência de Threads
Uma **thread** (ou processo leve) é a menor unidade de execução dentro de um processo. Várias threads podem coexistir dentro do mesmo processo, partilhando o mesmo espaço de memória e recursos, mas mantendo a sua própria pilha de execução (*stack*) e registos.

* **Vantagens face aos Processos:** A criação e a troca de contexto entre threads são significativamente mais rápidas do que entre processos, uma vez que o espaço de endereçamento de memória é comum.
* **Criação (ex: Pthreads):** Em C/C++, a biblioteca POSIX Threads (`pthreads`) disponibiliza funções como `pthread_create()` para instanciar uma nova thread e associá-la a uma rotina de execução específica.
* **Sincronização:** Como partilham o mesmo espaço de memória, a programação com threads exige mecanismos de sincronização rigorosos (como *mutexes*, semáforos e variáveis de condição) para evitar problemas críticos como condições de corrida (*race conditions*) e *deadlocks*.
