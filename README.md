# java-kb


## Leak Memory


## Comportamento saudável (sem memory leak) – padrão “serra”:

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

- Padrão de serra estável.
- A memória é liberada regularmente.
- A linha de base (parte inferior do “V”) se mantém estável.

## Comportamento com memory leak – crescimento constante:

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
- Uso da Old Gen só cresce.
- GCs não conseguem liberar memória.
- Pode indicar objetos sendo retidos indevidamente.
  
## Comportamento com GC, mas com linha base crescente:

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

- A memória ainda sobe e desce.
- Mas a linha de base aumenta com o tempo → sinal claro de vazamento progressivo.
