# Relatório do Laboratório 2 - Chamadas de Sistema

---

## Exercício 1a - Observação printf() vs 1b - write()

### Comandos executados:
```bash
strace -e write ./ex1a_printf
strace -e write ./ex1b_write
```

### Análise

**1. Quantas syscalls write() cada programa gerou?**
- ex1a_printf: __8___ syscalls
- ex1b_write: ___7__ syscalls

**2. Por que há diferença entre os dois métodos? Consulte o docs/printf_vs_write.md**

```
A diferença ocorre porque o printf usa buffering, diferente do write que cada chamada vira uma chamada ao kernel, sem buffers adicionais.
```

**3. Qual método é mais previsível? Por quê?**

```
O método mais previsível é o write, pois cada chamada no código corresponde a um syscall ao kernel.
```

---

## Exercício 2 - Leitura de Arquivo

### Resultados da execução:
- File descriptor: __3___
- Bytes lidos: __127___

### Comando strace:
```bash
strace -e openat,read,close ./ex2_leitura
```

### Análise

**1. Qual file descriptor foi usado? Por que não 0, 1 ou 2?**

```
Foi usado o file descriptor 3. Isso ocorre pois o 0,1,2 são reservados para entrada, saída e erro, respectivamente.
```

**2. Como você sabe que o arquivo foi lido completamente?**

```
Pois o número de bytes lidos corresponde ao tamanho total do arquivo, e além disso read() retorna 0, indicando que chegou ao fim do arquivo.
```

**3. O que acontece se esquecer de fechar o arquivo?**

```
O file descriptor permanecerá aberto, consumindo recursos do sistema.
```

**4. Por que verificar retorno de cada syscall?**

```
Verificar o retorno de cada syscall garante que o programa trate os erros de maneira adequada.
```

---

## Exercício 3 - Contador com Loop

### Resultados (BUFFER_SIZE = 64):
- Linhas: ___25__ (esperado: 25)
- Caracteres: _1300____
- Chamadas read(): __21___
- Tempo: __0.002269___ segundos

### Experimentos com buffer:

| Buffer Size | Chamadas read() | Tempo (s) |
|-------------|-----------------|-----------|
| 16          |       87        |  0.002380 |
| 64          |       21        |  0.002269 |
| 256         |       6         |  0.000699 |
| 1024        |       2         |  0.000577 |

### Análise

**1. Como o tamanho do buffer afeta o número de syscalls?**

```
Quanto maior o buffer, menor o número de syscalls
```

**2. Como você detecta o fim do arquivo?**

```
O fim do arquivo é detectado quando read() retorna 0 bytes lidos.
```

**3. Todas as chamadas read() retornaram BUFFER_SIZE bytes?**

```
Não. O read() pode retornar menos que BUFFER_SIZE em qualquer chamada, dependendo de quanto o kernel conseguiu disponibilizar. Normalmente a última chamada retorna menos bytes porque só restam os finais do arquivo.

```

**4. Qual é a relação entre syscalls e performance?**

```
Quanto menor a quantidade de syscalls, mais rápida a perfomance. Isso ocorre pois cada chamada ao kernel tem um custo alto
```

---

## Exercício 4 - Cópia de Arquivo

### Resultados:
- Bytes copiados: _1364____
- Operações: __6___
- Tempo: _0.000912____ segundos
- Throughput: _1460.56____ KB/s

### Verificação:
```bash
diff dados/origem.txt dados/destino.txt
```
Resultado: [X] Idênticos [ ] Diferentes

### Análise

**1. Por que devemos verificar que bytes_escritos == bytes_lidos?**

```
Para garantir que todos bytes lidos do arquivo de origem foram escritos no arquivo destino.
```

**2. Que flags são essenciais no open() do destino?**

```
O_WRONLY, para abrir o arquivo para escrita, O_CREAT para criar o arquivo caso não exista, e O_TRUNC para zerar o arquivo caso ele já exista.
```

**3. O número de reads e writes é igual? Por quê?**

```
Não são iguais, tendo 14 writes e 8 reads. Um único read pode resultar em múltiplas reads dependendo de como o kernel gerencia os buffers.
```

**4. Como você saberia se o disco ficou cheio?**

```
Se um write retornar -1 e o erro for ENOSPC, indicando que não há espaço.
```

**5. O que acontece se esquecer de fechar os arquivos?**

```
O file descriptor permanecerá aberto, consumindo recursos do sistema.
```

---

## Análise Geral

### Conceitos Fundamentais

**1. Como as syscalls demonstram a transição usuário → kernel?**

```
Os syscalls permitem que os programas acessem arquivos de forma segura e controlada, realizando a transição do usuário para o kernel.
```

**2. Qual é o seu entendimento sobre a importância dos file descriptors?**

```
Eles são importantes pois cada um deles indica uma operação diferente, auxiliando o kernel a identificar os estados dos arquivos.
```

**3. Discorra sobre a relação entre o tamanho do buffer e performance:**

```
Um buffer maior faz menos chamadas de read/write pois ele lê e escreve mais dados de cada vez, diminuindo o tempo de execução.
```

### Comparação de Performance

```bash
# Teste seu programa vs cp do sistema
time ./ex4_copia
time cp dados/origem.txt dados/destino_cp.txt
```

**Qual foi mais rápido?** __Meu código___

**Por que você acha que foi mais rápido?**

```
Meu programa foi mais rápido. Isso aconteceu porque ele realiza a cópia de forma direta com syscalls simples, sem verificações extras ou camadas adicionais de otimização que o cp pode usar. 
```

---

## Entrega

Certifique-se de ter:
- [X] Todos os códigos com TODOs completados
- [X] Traces salvos em `traces/`
- [X] Este relatório preenchido como `RELATORIO.md`

```bash
strace -e write -o traces/ex1a_trace.txt ./ex1a_printf
strace -e write -o traces/ex1b_trace.txt ./ex1b_write
strace -o traces/ex2_trace.txt ./ex2_leitura
strace -c -o traces/ex3_stats.txt ./ex3_contador
strace -o traces/ex4_trace.txt ./ex4_copia
```