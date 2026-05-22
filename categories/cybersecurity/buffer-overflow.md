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

<div align="center">

# <span style="color:#00ffcc;">Buffer Overflow</span>

<p>
    <span style="color:#aaaaaa;">
        Classic Memory Corruption Vulnerability
    </span>
</p>

<img
    src="https://cdn.prod.website-files.com/5ff66329429d880392f6cba2/67b43310ef6bef8402765c28_60618356ed0c90a97885a568_Stack%2520Overflow%2520Attack.jpeg"
    width="700"
    style="
        border-radius:20px;
        border:2px solid #00ffcc;
        box-shadow:
            0 0 10px #00ffcc,
            0 0 30px #00ffcc,
            0 0 60px #00ffcc;
    ">

</div>

---

# <span style="color:#00ffcc;">O que é Buffer Overflow?</span>

<p>
    <span style="color:#c792ea;">
        Buffer Overflow
    </span>

    é uma vulnerabilidade clássica de corrupção de memória que acontece quando um programa escreve mais dados em um buffer do que ele consegue armazenar.
</p>

<p>
    Um
    <span style="color:#00ffcc;">buffer</span>
    é basicamente uma região da memória reservada para armazenar dados temporariamente.
</p>

---

# <span style="color:#ff5555;">Exemplos de Dados em Buffers</span>

<ul>
    <li><span style="color:#50fa7b;">Strings</span></li>
    <li><span style="color:#50fa7b;">Pacotes de rede</span></li>
    <li><span style="color:#50fa7b;">Entradas do usuário</span></li>
    <li><span style="color:#50fa7b;">Arquivos</span></li>
    <li><span style="color:#50fa7b;">Dados temporários</span></li>
    <li><span style="color:#50fa7b;">Argumentos de funções</span></li>
</ul>

---

# <span style="color:#ff0066;">Impactos</span>

<p>
Quando o software não valida corretamente o tamanho da entrada recebida, os dados excedentes podem sobrescrever regiões adjacentes da memória.
</p>

<div style="
    background:#0d1117;
    border-left:4px solid #ff0066;
    padding:15px;
    border-radius:10px;
">

<ul>
    <li>Crash da aplicação</li>
    <li>Corrupção de memória</li>
    <li>Execução arbitrária de código</li>
    <li>Escalada de privilégios</li>
    <li>Controle total do processo</li>
    <li>Execução remota de código (RCE)</li>
</ul>

</div>

---

# <span style="color:#00ffcc;">Conceito Básico</span>

<p>
Imagine um buffer de 16 bytes:
</p>

```c
char buffer[16];
```

<p>
Se o programa copiar 40 bytes sem validação:
</p>

```c
strcpy(buffer, input);
```

<p>
Os bytes excedentes irão sobrescrever partes da memória próximas ao buffer.
</p>

---

# <span style="color:#ffaa00;">Estrutura da Stack</span>

<p>
Em arquiteturas
<span style="color:#50fa7b;">x86/x86_64</span>,
funções normalmente utilizam a stack.
</p>

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

<p>
Se um buffer local recebe dados demais, os bytes podem atingir o endereço de retorno.
</p>

```asm
ret
```

<p>
O processador irá pular para o endereço sobrescrito.
</p>

<div style="
    background:#1a1b26;
    padding:15px;
    border-radius:15px;
    border:1px solid #00ffcc;
">

<b style="color:#00ffcc;">
Resultado:
</b>

<p>
O atacante consegue controlar o fluxo de execução.
</p>

</div>

---

# <span style="color:#ff5555;">Exemplo Vulnerável</span>

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

# <span style="color:#ff0066;">Problema</span>

<p>
A função:
</p>

```c
strcpy()
```

<p>
não verifica tamanho.
</p>

```bash
./program AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
```

<p>
O conteúdo excedente irá sobrescrever a stack.
</p>

---

# <span style="color:#00ffcc;">Stack Buffer Overflow</span>

<div align="center">

<img
    src="https://cdn.prod.website-files.com/5ff66329429d880392f6cba2/67b43310ef6bef8402765c28_60618356ed0c90a97885a568_Stack%2520Overflow%2520Attack.jpeg"
    width="500"
    style="
        border-radius:15px;
        box-shadow:0 0 20px #00ffcc;
    ">

</div>

<ul>
    <li>Buffer vulnerável na stack</li>
    <li>Overwrite do endereço de retorno</li>
    <li>Controle do fluxo de execução</li>
</ul>

---

# <span style="color:#ffaa00;">Heap Overflow</span>

<p>
Também existem buffer overflows na heap.
</p>

```c
char *buf = malloc(32);

memcpy(buf, input, 128);
```

<p>
Nesse caso:
</p>

<ul>
    <li>Corrupção ocorre em memória dinâmica</li>
    <li>Pode sobrescrever metadados do heap</li>
    <li>Pode afetar ponteiros internos</li>
</ul>

---

# <span style="color:#ff5555;">Shellcode</span>

<p>
Shellcode é um payload em Assembly usado para executar ações após exploração.
</p>

<ul>
    <li>Abrir shell</li>
    <li>Executar comandos</li>
    <li>Reverse shell</li>
    <li>Download de payloads</li>
</ul>

---

# <span style="color:#00ffcc;">NOP Sled</span>

```asm
NOP
NOP
NOP
NOP
```

<p>
Cria uma região "escorregadia" para facilitar o redirecionamento da execução.
</p>

---

# <span style="color:#ff0066;">Proteções Modernas</span>

<div style="
    display:flex;
    flex-direction:column;
    gap:15px;
">

<div style="
    background:#161b22;
    padding:15px;
    border-radius:15px;
    border-left:4px solid #00ffcc;
">

<h3>Stack Canaries</h3>

<p>
Detecta overwrite antes do endereço de retorno.
</p>

</div>

<div style="
    background:#161b22;
    padding:15px;
    border-radius:15px;
    border-left:4px solid #ffaa00;
">

<h3>NX / DEP</h3>

<p>
Impede execução de shellcode na stack.
</p>

</div>

<div style="
    background:#161b22;
    padding:15px;
    border-radius:15px;
    border-left:4px solid #ff5555;
">

<h3>ASLR</h3>

<p>
Randomiza endereços de memória.
</p>

</div>

</div>

---

# <span style="color:#00ffcc;">Funções Perigosas</span>

```c
gets()
strcpy()
strcat()
sprintf()
scanf("%s")
memcpy()
```

---

# <span style="color:#50fa7b;">Alternativas Mais Seguras</span>

```c
fgets()
strncpy()
snprintf()
memcpy_s()
```

---

# <span style="color:#ff0066;">Exploração Clássica</span>

<ol>
    <li>Encontrar vulnerabilidade</li>
    <li>Descobrir offset</li>
    <li>Controlar RIP/EIP</li>
    <li>Encontrar gadgets</li>
    <li>Construir payload</li>
    <li>Obter execução de código</li>
</ol>

---

# <span style="color:#00ffcc;">Exemplo de Payload</span>

```python
payload = b"A" * 72
payload += p64(0x4141414141414141)
```

<p>
Após 72 bytes:
</p>

<ul>
    <li>RIP será sobrescrito</li>
</ul>

---

# <span style="color:#ffaa00;">Ferramentas Utilizadas</span>

<ul>
    <li>GDB</li>
    <li>pwndbg</li>
    <li>GEF</li>
    <li>x64dbg</li>
    <li>pwntools</li>
    <li>ROPgadget</li>
</ul>

---

# <span style="color:#ff5555;">Exemplo de Imagem Responsiva</span>

```html
<img 
    src="https://cdn.prod.website-files.com/5ff66329429d880392f6cba2/67b43310ef6bef8402765c28_60618356ed0c90a97885a568_Stack%2520Overflow%2520Attack.jpeg"
    style="max-width:100%;">
```

<img 
    src="https://cdn.prod.website-files.com/5ff66329429d880392f6cba2/67b43310ef6bef8402765c28_60618356ed0c90a97885a568_Stack%2520Overflow%2520Attack.jpeg"
    style="max-width:100%; border-radius:15px;">

---

# <span style="color:#00ffcc;">Vídeo</span>

```html
<iframe 
    width="700"
    height="400"
    src="https://www.youtube.com/embed/1S0aBV-Waeo"
    frameborder="0"
    allowfullscreen>
</iframe>
```

---

# <span style="color:#ffaa00;">PDF Embed</span>

```html
<iframe
    src="https://www.cs.cornell.edu/courses/cs513/2005fa/paper.alpeh1.stacksmashing.pdf"
    width="800"
    height="500">
</iframe>
```

---

# <span style="color:#50fa7b;">Compatibilidade</span>

| Recurso | Markdown | GitHub | HTML |
|---|---|---|---|
| Imagem | ✅ | ✅ | ✅ |
| Vídeo Embed | ❌ | ⚠️ | ✅ |
| PDF Embed | ❌ | ❌ | ✅ |
| CSS Inline | ❌ | ⚠️ | ✅ |
| iframe | ❌ | ❌ | ✅ |

---

# <span style="color:#ff0066;">Observação Final</span>

<div style="
    background:#0d1117;
    padding:20px;
    border-radius:20px;
    border:1px solid #00ffcc;
">

<p>
Buffer Overflow não é apenas uma falha simples de programação.
</p>

<p>
Ele representa um problema estrutural relacionado ao gerenciamento manual de memória e ao controle inseguro de entradas.
</p>

<p>
Por isso linguagens modernas vêm adotando modelos mais seguros de memória, enquanto sistemas críticos continuam exigindo auditorias profundas em código de baixo nível.
</p>

</div>

---


# <span style="color:#00ffcc;">Referências</span>

<ul>
    <li>
        <a href="https://phrack.org/issues/49/14">
            Smashing The Stack For Fun And Profit
        </a>
    </li>

    <li>
        <a href="https://owasp.org/www-community/vulnerabilities/Buffer_Overflow">
            OWASP Buffer Overflow
        </a>
    </li>

    <li>
        <a href="https://cwe.mitre.org/data/definitions/120.html">
            CWE-120
        </a>
    </li>
</ul>
