# Buffer Overflow

## O que é Buffer Overflow?

Buffer Overflow é uma vulnerabilidade clássica de corrupção de memória que acontece quando um programa escreve mais dados em um buffer do que ele consegue armazenar.

Um *buffer* é basicamente uma região da memória reservada para armazenar dados temporariamente, como:

- Strings
- Pacotes de rede
- Dados enviados pelo usuário
- Arquivos
- Entradas de teclado
- Argumentos de funções

Quando o software não valida corretamente o tamanho da entrada recebida, os dados excedentes podem sobrescrever regiões adjacentes da memória.

Isso pode causar:

- Crash da aplicação
- Corrupção de memória
- Execução arbitrária de código
- Escalada de privilégios
- Controle completo do processo
- Execução remota de código (RCE)

---

# Conceito Básico

Imagine um buffer de 16 bytes:

```c
char buffer[16];
````

Se o programa copiar 40 bytes para ele sem validação:

```c
strcpy(buffer, input);
```

Os bytes excedentes irão sobrescrever partes da memória próximas ao buffer.

Dependendo da arquitetura e da organização da stack, isso pode sobrescrever:

* Variáveis locais
* Ponteiros
* Registradores salvos
* Base Pointer (EBP/RBP)
* Return Address (EIP/RIP)

E é exatamente aí que ataques de Buffer Overflow acontecem.

---

# Estrutura da Stack

Em arquiteturas x86/x86_64, funções normalmente utilizam a stack.

Exemplo simplificado:

```text
+-------------------+
| Return Address    |
+-------------------+
| Saved Base Pointer|
+-------------------+
| Local Variables   |
| Buffer            |
+-------------------+
```

Se um buffer local recebe dados demais, os bytes podem atingir o endereço de retorno.

Quando a função terminar e executar:

```asm
ret
```

O processador irá pular para o endereço sobrescrito.

Ou seja:

O atacante consegue controlar o fluxo de execução.

---

# Exemplo Vulnerável

## Código

```c
#include <stdio.h>
#include <string.h>

void vulnerable(char *input)
{
    char buffer[32];

    strcpy(buffer, input);

    printf("Você digitou: %s\n", buffer);
}

int main(int argc, char *argv[])
{
    if(argc < 2)
        return 1;

    vulnerable(argv[1]);

    return 0;
}
```

---

# Problema

A função:

```c
strcpy()
```

não verifica tamanho.

Se forem enviados mais de 32 bytes:

```bash
./program AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

o conteúdo excedente irá sobrescrever a stack.

---

# Stack Overflow

O tipo mais conhecido de Buffer Overflow é o Stack Buffer Overflow.

Acontece quando:

* O buffer vulnerável está na stack
* Dados excedentes sobrescrevem o endereço de retorno

Isso pode permitir:

* Shellcode execution
* Return Oriented Programming (ROP)
* Ret2libc
* Desvio de fluxo
* Execução remota

---

# Heap Overflow

Também existem buffer overflows na heap.

Nesse caso:

* A corrupção acontece em memória alocada dinamicamente
* Pode sobrescrever metadados do heap
* Pode afetar ponteiros e estruturas internas do allocator

Exemplo:

```c
char *buf = malloc(32);

memcpy(buf, input, 128);
```

---

# Integer Overflow + Buffer Overflow

Muitas vezes o Buffer Overflow começa com Integer Overflow.

Exemplo:

```c
int size = user_size * 4;
char *buf = malloc(size);
```

Se `user_size` causar overflow inteiro:

```text
0xFFFFFFFF * 4
```

o programa pode alocar menos memória do que deveria.

Depois disso:

* cópias grandes de memória
* loops
* memcpy()
* recv()

acabam causando Buffer Overflow.

---

# Shellcode

Shellcode é um payload em Assembly usado para executar ações após exploração.

Exemplo clássico:

* abrir shell
* baixar malware
* executar comandos
* criar reverse shell

Historicamente o atacante colocava:

1. Shellcode dentro do buffer
2. Sobrescrevia o Return Address
3. Fazia o programa retornar para o próprio buffer

---

# NOP Sled

Antigamente era comum utilizar:

```asm
NOP
NOP
NOP
NOP
```

para criar uma região "escorregadia".

O endereço de retorno não precisava cair exatamente no shellcode.

Bastava cair em qualquer região NOP.

---

# Técnicas Modernas

Hoje sistemas modernos possuem proteções.

Então ataques modernos usam técnicas como:

* ROP (Return Oriented Programming)
* JOP (Jump Oriented Programming)
* ret2libc
* Sigreturn Oriented Programming
* Stack Pivoting

---

# Proteções Modernas

## Stack Canaries

O compilador adiciona um valor sentinela antes do endereço de retorno.

Se ocorrer overwrite:

```text
*** stack smashing detected ***
```

o programa aborta.

---

## NX / DEP

Marca regiões da memória como não executáveis.

Impede execução direta de shellcode na stack.

---

## ASLR

Randomiza endereços de memória:

* Stack
* Heap
* Bibliotecas
* Binários

Dificulta previsibilidade.

---

## PIE

Randomiza o próprio executável.

---

## RELRO

Protege GOT/PLT contra sobrescrita.

---

# Funções Perigosas

Algumas funções clássicas associadas a Buffer Overflow:

```c
gets()
strcpy()
strcat()
sprintf()
scanf("%s")
memcpy()
```

---

# Alternativas Mais Seguras

```c
fgets()
strncpy()
snprintf()
memcpy_s()
```

Mesmo assim:

uso incorreto ainda pode causar vulnerabilidades.

---

# Exemplo Visual

## Buffer normal

```text
[AAAAAAAAAAAAAAAA]
```

## Overflow

```text
[AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA]
```

Sobrescrevendo:

```text
[BUFFER][EBP][RETURN ADDRESS]
```

---

# Exploração Clássica

Fluxo tradicional:

1. Encontrar vulnerabilidade
2. Descobrir offset
3. Controlar RIP/EIP
4. Encontrar gadgets
5. Construir payload
6. Obter execução de código

---

# Offset

O offset é a quantidade exata de bytes necessários para alcançar o endereço de retorno.

Ferramentas comuns:

```bash
pattern_create
pattern_offset
pwntools
gef
peda
pwndbg
```

---

# Exemplo de Offset

```python
payload = b"A" * 72
payload += p64(0x4141414141414141)
```

Após 72 bytes:

* RIP será sobrescrito

---

# Ferramentas Utilizadas em Exploração

## Debuggers

* GDB
* WinDbg
* x64dbg
* Immunity Debugger

---

## Frameworks

* pwntools
* Metasploit
* Ropper
* ROPgadget

---

# Tipos de Corrupção de Memória Relacionados

## Off-by-One

Escreve apenas 1 byte além do limite.

Mesmo assim pode ser explorável.

---

## Use-After-Free

Uso de memória já liberada.

---

## Double Free

Liberação dupla de memória.

---

## Format String

Vulnerabilidades com:

```c
printf(user_input);
```

Podem causar leitura/escrita arbitrária.

---

# Buffer Overflow em Drivers e Kernel

Buffer Overflow em kernel é extremamente crítico.

Pode resultar em:

* Ring0 execution
* Privilege escalation
* Kernel panic
* Escape de sandbox
* VM Escape

---

# Buffer Overflow em Protocolos

Muitos worms históricos exploraram buffer overflows:

* Code Red
* Slammer
* Blaster
* WannaCry (cadeia com corrupção de memória)

---

# Linguagens Mais Vulneráveis

Linguagens sem gerenciamento automático de memória:

* C
* C++
* Assembly

são mais suscetíveis.

---

# Linguagens Mais Seguras

Linguagens modernas tentam impedir isso:

* Rust
* Java
* Go
* Python

---

# Buffer Overflow em Firmware e IoT

Dispositivos embarcados frequentemente possuem:

* Sem ASLR
* Sem NX
* Sem canaries

o que facilita exploração.

Muito comum em:

* roteadores
* DVRs
* câmeras IP
* dispositivos industriais

---

# Secure Coding

Práticas importantes:

* validar tamanho de entrada
* usar funções seguras
* utilizar compiladores modernos
* ativar proteções do compilador
* fuzzing
* sanitizers
* revisão de código
* análise estática

---

# Flags de Compilação

## GCC/Clang

```bash
-fstack-protector
-D_FORTIFY_SOURCE=2
-fPIE
-pie
-Wall
-Wextra
```

---

# Address Sanitizer

```bash
-fsanitize=address
```

Ajuda a detectar:

* overflow
* use-after-free
* corrupção de memória

---

# Exemplo de Compilação Vulnerável

```bash
gcc vuln.c -o vuln -fno-stack-protector -z execstack -no-pie
```

---

# Exemplo de Compilação Protegida

```bash
gcc vuln.c -o vuln \
-fstack-protector-strong \
-D_FORTIFY_SOURCE=2 \
-fPIE -pie \
-Wl,-z,relro,-z,now
```

---

# Impacto Real

Buffer Overflow continua sendo uma das vulnerabilidades mais perigosas da computação.

Mesmo décadas após sua descoberta, ainda aparece em:

* kernels
* navegadores
* drivers
* firmware
* softwares industriais
* servidores
* aplicações embarcadas

---

# Resumo

Buffer Overflow ocorre quando:

```text
dados > tamanho do buffer
```

e isso permite corrupção de memória.

Dependendo do contexto, pode resultar em:

* crash
* execução arbitrária
* escalada de privilégios
* comprometimento total do sistema

---

# Referências Interessantes

## Livros

* Hacking: The Art of Exploitation
* Practical Binary Analysis
* The Shellcoder's Handbook
* Reversing: Secrets of Reverse Engineering

---

## Ferramentas

* GDB
* pwndbg
* GEF
* x64dbg
* pwntools

---

# Observação Final

Buffer Overflow não é apenas uma falha simples de programação.

Ele representa um problema estrutural relacionado ao gerenciamento manual de memória e ao controle inseguro de entradas.

Por isso linguagens modernas vêm adotando modelos mais seguros de memória, enquanto sistemas críticos continuam exigindo auditorias profundas em código de baixo nível.

```
```
