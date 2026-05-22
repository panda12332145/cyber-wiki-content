---
title: Buffer Overflow
difficulty: advanced
starred: true
tags:
  - exploit
  - linux
  - memory-corruption
  - cybersecurity
---

# Buffer Overflow

> Classic Memory Corruption Vulnerability

<p align="center">
    <img
        src="https://cdn.prod.website-files.com/5ff66329429d880392f6cba2/67b43310ef6bef8402765c28_60618356ed0c90a97885a568_Stack%2520Overflow%2520Attack.jpeg"
        width="700">
</p>

---

# O que é Buffer Overflow?

Buffer Overflow é uma vulnerabilidade clássica de corrupção de memória que acontece quando um programa escreve mais dados em um buffer do que ele consegue armazenar.

Um buffer é uma região da memória reservada para armazenar dados temporariamente.

---

# Exemplos de Dados em Buffers

- Strings
- Pacotes de rede
- Entradas do usuário
- Arquivos
- Dados temporários
- Argumentos de funções

---

# Impactos

Quando o software não valida corretamente o tamanho da entrada recebida, os dados excedentes podem sobrescrever regiões adjacentes da memória.

## Possíveis consequências

- Crash da aplicação
- Corrupção de memória
- Execução arbitrária de código
- Escalada de privilégios
- Controle total do processo
- Execução remota de código (RCE)

---

# Conceito Básico

Imagine um buffer de 16 bytes:

```c
char buffer[16];
````

Se o programa copiar 40 bytes sem validação:

```c
strcpy(buffer, input);
```

Os bytes excedentes irão sobrescrever partes próximas da memória.

---

# Estrutura da Stack

Em arquiteturas x86/x86_64, funções normalmente utilizam a stack.

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

```asm
ret
```

O processador irá pular para o endereço sobrescrito.

## Resultado

O atacante pode controlar o fluxo de execução.

---

# Exemplo Vulnerável

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

```bash
./program AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

O conteúdo excedente irá sobrescrever a stack.

---

# Stack Buffer Overflow

<p align="center">
    <img
        src="https://cdn.prod.website-files.com/5ff66329429d880392f6cba2/67b43310ef6bef8402765c28_60618356ed0c90a97885a568_Stack%2520Overflow%2520Attack.jpeg"
        width="500">
</p>

* Buffer vulnerável na stack
* Overwrite do endereço de retorno
* Controle do fluxo de execução

---

# Heap Overflow

Também existem buffer overflows na heap.

```c
char *buf = malloc(32);

memcpy(buf, input, 128);
```

Nesse caso:

* Corrupção ocorre em memória dinâmica
* Pode sobrescrever metadados do heap
* Pode afetar ponteiros internos

---

# Shellcode

Shellcode é um payload em Assembly usado após exploração.

## Possíveis ações

* Abrir shell
* Executar comandos
* Reverse shell
* Download de payloads

---

# NOP Sled

```asm
NOP
NOP
NOP
NOP
```

Cria uma região "escorregadia" para facilitar o redirecionamento da execução.

---

# Proteções Modernas

## Stack Canaries

Detecta overwrite antes do endereço de retorno.

## NX / DEP

Impede execução de shellcode na stack.

## ASLR

Randomiza endereços de memória.

---

# Funções Perigosas

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

---

# Exploração Clássica

1. Encontrar vulnerabilidade
2. Descobrir offset
3. Controlar RIP/EIP
4. Encontrar gadgets
5. Construir payload
6. Obter execução de código

---

# Exemplo de Payload

```python
payload = b"A" * 72
payload += p64(0x4141414141414141)
```

Após 72 bytes:

* RIP será sobrescrito

---

# Ferramentas Utilizadas

* GDB
* pwndbg
* GEF
* x64dbg
* pwntools
* ROPgadget

---

# Imagem Responsiva

```html
<img 
    src="imagem.jpg"
    style="max-width:100%;">
```

<p align="center">
    <img 
        src="https://cdn.prod.website-files.com/5ff66329429d880392f6cba2/67b43310ef6bef8402765c28_60618356ed0c90a97885a568_Stack%2520Overflow%2520Attack.jpeg"
        width="100%">
</p>

---

# Vídeo

GitHub não suporta iframe do YouTube.

Vídeo:
[https://www.youtube.com/watch?v=1S0aBV-Waeo](https://www.youtube.com/watch?v=1S0aBV-Waeo)

---

# PDF

GitHub não suporta embed de PDF via iframe.

PDF:
[https://www.cs.cornell.edu/courses/cs513/2005fa/paper.alpeh1.stacksmashing.pdf](https://www.cs.cornell.edu/courses/cs513/2005fa/paper.alpeh1.stacksmashing.pdf)

---

# Compatibilidade

| Recurso     | Markdown | GitHub | HTML |
| ----------- | -------- | ------ | ---- |
| Imagem      | ✅        | ✅      | ✅    |
| Vídeo Embed | ❌        | ⚠️     | ✅    |
| PDF Embed   | ❌        | ❌      | ✅    |
| CSS Inline  | ⚠️       | ⚠️     | ✅    |
| iframe      | ❌        | ❌      | ✅    |

---

# Observação Final

Buffer Overflow não é apenas uma falha simples de programação.

Ele representa um problema estrutural relacionado ao gerenciamento manual de memória e ao controle inseguro de entradas.

Por isso linguagens modernas vêm adotando modelos mais seguros de memória, enquanto sistemas críticos continuam exigindo auditorias profundas em código de baixo nível.

---

# Referências

* [https://phrack.org/issues/49/14](https://phrack.org/issues/49/14)
* [https://owasp.org/www-community/vulnerabilities/Buffer_Overflow](https://owasp.org/www-community/vulnerabilities/Buffer_Overflow)
* [https://cwe.mitre.org/data/definitions/120.html](https://cwe.mitre.org/data/definitions/120.html)

```
