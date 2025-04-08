# java-kb

# Siglas e Conceitos da JVM

Este documento reúne as siglas e termos mais importantes usados no universo da JVM (Java Virtual Machine), com explicações claras e práticas para facilitar o entendimento e o uso no dia a dia.

---

## Conceitos Básicos

| Sigla / Termo | Significado                  | Explicação                                                                 |
|---------------|------------------------------|----------------------------------------------------------------------------|
| **JVM**       | Java Virtual Machine         | Máquina virtual que executa bytecode Java.                                |
| **JRE**       | Java Runtime Environment     | JVM + bibliotecas básicas para rodar aplicações Java.                     |
| **JDK**       | Java Development Kit         | JRE + ferramentas de desenvolvimento (javac, javadoc, etc).               |
| **JIT**       | Just-In-Time Compiler        | Compila bytecode para código nativo durante a execução.                   |
| **GC**        | Garbage Collector            | Coletor de lixo responsável por liberar memória automaticamente.          |
| **JITC**      | JIT Compiler                 | Outro nome para o compilador JIT.                                         |

---

## Estrutura de Memória da JVM

| Área / Sigla      | Explicação                                                                 |
|-------------------|----------------------------------------------------------------------------|
| **Heap**          | Onde os objetos vivem. Dividida em Young Gen e Old Gen.                    |
| **Young Gen**     | Objetos recém-criados.                                                     |
| **Eden Space**    | Objetos "nascem" aqui.                                                     |
| **Survivor Space**| Objetos que sobrevivem a GCs jovens.                                       |
| **Old Gen**       | Objetos promovidos (de longa duração).                                     |
| **Metaspace**     | Metadados de classes (substitui o PermGen a partir do Java 8).             |
| **PermGen**       | Área para classes (obsoleta desde Java 8).                                 |
| **Code Cache**    | Armazena código compilado pelo JIT.                                        |
| **Thread Stack**  | Pilha de execução de cada thread.                                          |

---

## Parâmetros e Flags da JVM

| Parâmetro / Flag                   | Explicação                                                                 |
|-----------------------------------|----------------------------------------------------------------------------|
| `-Xmx`                            | Tamanho máximo da heap.                                                    |
| `-Xms`                            | Tamanho inicial da heap.                                                  |
| `-Xmn`                            | Tamanho da Young Gen.                                                     |
| `-XX:+UseG1GC`                    | Usa o coletor de lixo G1.                                                 |
| `-XX:+UseZGC`                     | Usa o coletor de lixo ZGC.                                                |
| `-XX:+PrintGCDetails`            | Ativa logs detalhados de GC.                                              |
| `-Xlog:gc*`                       | Novo formato de logging (Java 9+).                                        |
| `-XX:+HeapDumpOnOutOfMemoryError`| Gera um heap dump ao estourar a memória.                                  |
| `-XX:MaxMetaspaceSize`           | Limita o tamanho da Metaspace.                                            |

---

## Tipos de Garbage Collection

| Tipo / GC         | Explicação                                                                 |
|-------------------|----------------------------------------------------------------------------|
| **Minor GC**      | Coleta a Young Gen. Rápido e frequente.                                   |
| **Major GC**      | Coleta a Old Gen. Mais custoso.                                            |
| **Full GC**       | Coleta toda a heap e Metaspace.                                            |
| **CMS**           | Concurrent Mark-Sweep. Coletor obsoleto (até Java 14).                    |
| **G1GC**          | Garbage First. Equilibra pausas e desempenho. Padrão atual recomendado.   |
| **ZGC**           | Coletor com pausas muito curtas (ideal para latência).                    |
| **Shenandoah**    | Coletor alternativo com pausas baixas.                                     |

---

## Ferramentas e Diagnóstico

| Ferramenta / Sigla | Explicação                                                                 |
|--------------------|----------------------------------------------------------------------------|
| **JVisualVM**       | Ferramenta gráfica para monitorar heap, threads, GC etc.                 |
| **JFR**             | Java Flight Recorder – coleta eventos de performance em runtime.         |
| **JMC**             | Java Mission Control – análise de gravações JFR.                         |
| **MAT**             | Memory Analyzer Tool – analisa heap dumps (.hprof).                      |
| **JStack**          | Mostra stack trace de threads da JVM.                                    |
| **JMap**            | Mostra uso da heap ou gera heap dump.                                    |
| **JStat**           | Exibe estatísticas da JVM em tempo real.                                 |
| **JCmd**            | Interface para comandos da JVM (ex: dump, GC, etc).                      |

---

# Leak Memory

## Comportamento saudável (sem memory leak) – padrão “serra”:

```text
Memória (Old Gen)
▲
│         /\        /\        /\
│        /  \      /  \      /  \
│  /\   /    \    /    \    /    \
│ /  \_/      \__/      \__/      \___
│
└────────────────────────────────────▶ Tempo
        ↑         ↑         ↑
     GC roda   GC roda   GC roda
```

- Padrão de serra estável.
- A memória é liberada regularmente.
- A linha de base (parte inferior do “V”) se mantém estável.

## Comportamento com memory leak – crescimento constante:

```text
Memória (Old Gen)
▲
│
│                  /‾‾‾‾‾‾‾‾‾
│                /
│              /
│            /
│          /
│        /
│      /
│    /
│  /
└──────────────────────────────▶ Tempo
```

- Uso da Old Gen só cresce.
- GCs não conseguem liberar memória.
- Pode indicar objetos sendo retidos indevidamente.
  
## Comportamento com GC, mas com linha base crescente:

```text
Memória (Old Gen)
▲
│         /\         /\        /\
│        /  \       /  \______/  \
│  /\   /    \____/
│ /  \_/            
│
│
└────────────────────────────────────▶ Tempo
      ↑          ↑          ↑
   GC roda    GC roda    GC roda
   mas cada vez libera menos memória
```

- A memória ainda sobe e desce.
- Mas a linha de base aumenta com o tempo → sinal claro de vazamento progressivo.
