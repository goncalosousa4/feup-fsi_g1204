# CTF Semana #11 (Weak Encryption)

#### Objetivo:
> explorar uma geração inadequada de chaves para decifrar um criptograma, sem acesso à chave simétrica usada na sua criação.

O problema no ficheiro _cipherspec.py_ está na geração inadequada da chave simétrica na função gen(). A função gera uma chave previsível:

```
offset = 3  # Hotfix to make Crypto blazing fast!!
key = bytearray(b'\x00'*(KEYLEN-offset)) 
key.extend(os.urandom(offset))

```

> Os primeiros 13 bytes da chave são sempre 0x00.
> Apenas os últimos 3 bytes são aleatórios, resultando em um espaço de chave de 2^24 combinações possíveis, que é pequeno e vulnerável a um ataque de força bruta.

#### Como Explorar a Vulnerabilidade?

##### Lógica do código do nosso script.py:

Para realizarmos este ataque, temos que gerar todas as combinações possíveis de 3 bytes e combiná-las com 13 0x00's antes.
Depois, temos que testar cada chave tentando decifrar o criptograma que nos foi fornecido.
Quando encontrarmos um resultado no formato flag{...}, paramos porque encontramos a flag.

**script.py**
```
from cipherpscec import dec
from itertools import product
from binascii import unhexlify

nonce_hex = "6dc8ebbf52cfe24db7bf5ddf203d3894"
ciphertext_hex = "a928f6ef855c4cccdb9b8ff3d3afc05d470caa5f2c51"

nonce = unhexlify(nonce_hex)
ciphertext = unhexlify(ciphertext_hex)

def brute_force():
    for key_suffix in product(range(256), repeat=3):
        key = b'\x00' * 13 + bytes(key_suffix)
        try:
            msg = dec(key, ciphertext, nonce)
            if msg.startswith(b"flag{") and msg.endswith(b"}"):
                print(f"Chave encontrada: {key.hex()}")
                print(f"Flag: {msg.decode()}")
                return
        except Exception as e:
            print(f"Erro ao decifrar com chave {key.hex()}: {e}")

brute_force()
```

Ao corrermos o código, obtivemos a flag:

<img src= images/CTF11_1.png width=550>


Assim, a nossa flag é flag{yfznhouozdibbybv}!

### Tarefa 2

Nesta tarefa é-nos pedido que calculemos quão grande teria que ser o offset em cipherspec.py para que fosse inviável que as máquinas pessoais usadas no ataque conseguissem descobrir a chave no pior caso, durante um período de 10 anos.

Ora, para isto começamos por calcular a velocidade do nosso script. Adicionamos contagem de tempo ao código e obtivemos:


<img src= images/tempo.png width=550>


#### Análise dos resultados:

**Testes realizados** = 256^3;

**Tempo total** = 560.5756828784943 segundos

**Taxa de testes por segundo**, que é o valor que nos interessa, é o quociente entre a primeira variável e a segunda:

**Taxa de testes por segundo** = 29928.54758495917

Assim, podemos calcular quantas chaves consegue testar em 10 anos:

Para fazer uma estimativa de quão grande teria de ser o offset para que as máquinas apenas descobrissem a chave passados 10 anos:

>**Testes em 10 anos** = 29928.54758495917 * 10 * 365 * 24 * 60 * 60

>**Testes em 10 anos** = 9438266766382.725


> **Testes em 10 anos** = 256^**offset**;

Logo, o **offset** seria, aproximadamente: 5.39
Arredondando para cima: **offset** = 6

### Tarefa 3

Espaço de Pesquisa Limitado:
Um nonce de 1 byte tem 256 combinações possíveis (2⁸). Isso não é suficiente para proteger o sistema contra ataques de brute force modernos.

Consequências de Não Transmitir:

Se o adversário souber que o nonce não é transmitido, precisa de testar 256 possibilidades para cada chave.
Isso apenas aumenta o custo do ataque num fator de 256, o que é insignificante comparado ao espaço real de uma chave AES completa (2^128 ou mais).
