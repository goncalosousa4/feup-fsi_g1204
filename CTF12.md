# CTF Semana #12 - (RSA)

p é um primo próximo de `2**(500+(((t-1)*10 + g) // 2))` <br>
q é um primo próximo de `2**(501+(((t-1)*10 + g) // 2))` <br>
Onde t é o número do turno, e g é o número do grupo. <br>

Nós somos do turno 12 e grupo 4, pelo que: <br>
p=`2**(500+(((12-1)*10 + 4) // 2))=2**557` <br>
q=`2**(501+(((12-1)*10 + 4) // 2))=2**558`

Para descobrir a flag deste desafio usamos um script python que chamamos de ctf12.py: <br>
```
import os
import sys
from binascii import hexlify, unhexlify
import random 


def enc(x, e, n):
    int_x = int.from_bytes(x, "little")
    y = pow(int_x,e,n)
    return hexlify(y.to_bytes(256, 'little'))

def dec(y, d, n):
    int_y = int.from_bytes(unhexlify(y), "little")
    x = pow(int_y,d,n)
    return x.to_bytes(256, 'little')


ciphertext = "dd2d6e11c74bf1b1f462da30633b1ffdc97a5d937a2170208954c3f55545cc94a1586bfd1c285a03dc84310b1b89e99fd07002a237dc31707757731e6245e8e0c59cb1e61b263658c5620e271b1c5005671ed862672ecdd7a044500ed1063f59e14a70390399f377636210ab618cbcfddeb545a898f1da37b0e2ac372672911e003d43d4606c98d97c5750070000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"
e = 65537
n = 445087261998902755091202516044764538012792863762594054061594925376409794233709968988746287782545345488439812520471671061710068958600722185294825008096891001958752700491687510387393687680161065864480916505569919091957487407519307168324167231144759916326474155171640897954942629640355219578442104300867250336486373533096234619721184362667

pRange=2**557
qRange=2**558


def isPrime(n):
    """
    Miller-Rabin primality test.

    A return value of False means n is certainly not prime. A return value of
    True means n is very likely a prime.
    """
    if n!=int(n):
        return False
    n=int(n)
    #Miller-Rabin test for prime
    if n==0 or n==1 or n==4 or n==6 or n==8 or n==9:
        return False
        
    if n==2 or n==3 or n==5 or n==7:
        return True
    s = 0
    d = n-1
    while d%2==0:
        d>>=1
        s+=1
    assert(2**s * d == n-1)
  
    def trial_composite(a):
        if pow(a, d, n) == 1:
            return False
        for i in range(s):
            if pow(a, 2**i * d, n) == n-1:
                return False
        return True  
 
    for i in range(8):#number of trials 
        a = random.randrange(2, n)
        if trial_composite(a):
            return False
 
    return True


def search(n):
    p_start = 2**557
    q_start = 2**558

    if p_start % 2 == 0:  # Make sure we start from an odd number
        p_start += 1
    if q_start % 2 == 0:
        q_start += 1

    for p in range(p_start, 2**558, 2):  # Iterate over odd numbers
        if not isPrime(p):
            continue

        if n % p == 0:  # If p is a divisor of n
            q = n // p
            if q >= q_start and isPrime(q):
                return p, q

    return None, None
    	    	  

p,q = search(n)


d = pow(e, -1, ((p-1)*(q-1)))

flag = dec(ciphertext,d,n)
print(flag)

sys.stdout.flush()
```


Para correr o script usamos o comando `$ python3 ctf12.py` e o output foi: <br>
`b'flag{xecimtnudgcpbcju}\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'`


Este script implementa um sistema de criptografia RSA e realiza as seguintes tarefas:

1. Definição de Funções de Criptografia e descriptografia: <br>
enc(x, e, n): Criptografa uma mensagem x usando a chave pública (e, n). <br>
dec(y, d, n): Descriptografa uma mensagem y usando a chave privada (d, n).
2. Definição de Parâmetros: <br>
Define o criptograma (ciphertext), o expoente público (e) e o módulo (n). Além disso, define os intervalos para os primos p e q.
3. Teste de Primalidade: <br>
isPrime(n): Implementa o teste de primalidade de Miller-Rabin para verificar se um número n é primo.
4. Busca pelos Primos p e q: <br>
search(n): Procura os primos p e q que satisfazem a condição n = p * q.
5. Cálculo da Chave Privada: <br>
Calcula a chave privada d como o inverso modular de e em relação a (p-1) * (q-1).
6. Descriptografia do criptograma: <br>
Usa a função dec para descriptografar o criptograma com a chave privada d e o módulo n.
7. Imprimir a flag decifrada.

#### No centro deste desafio está a necessidade de encontrar primos. Assim, o primeiro passo deve ser implementar uma função que testa a primalidade de números. O algoritmo de Miller-Rabin é uma sugestão para este efeito. A solução não exige um sistema específico para testar primos.
Para testar a primalidade de números usamos então o algoritmo de Miller-Rabin. Este algoritmo é eficiente e amplamente utilizado para verificar se um número é primo.

```
def isPrime(n):
    """
    Miller-Rabin primality test.

    A return value of False means n is certainly not prime. A return value of
    True means n is very likely a prime.
    """
    if n!=int(n):
        return False
    n=int(n)
    #Miller-Rabin test for prime
    if n==0 or n==1 or n==4 or n==6 or n==8 or n==9:
        return False
        
    if n==2 or n==3 or n==5 or n==7:
        return True
    s = 0
    d = n-1
    while d%2==0:
        d>>=1
        s+=1
    assert(2**s * d == n-1)
  
    def trial_composite(a):
        if pow(a, d, n) == 1:
            return False
        for i in range(s):
            if pow(a, 2**i * d, n) == n-1:
                return False
        return True  
 
    for i in range(8):#number of trials 
        a = random.randrange(2, n)
        if trial_composite(a):
            return False
 
    return True
```

#### Depois, aconselha-se os alunos a fazerem uma implementação do RSA. Dado que já conseguimos testar primos, a única dificuldade é encontrar (e,d) tal que ed % (p-1)(q-1) = 1, que vão ser usados para chave pública e secreta, respetivamente.
1. A função search(n) é responsável por encontrar os primos p e q que satisfazem a condição n = p*q: <br>
```
def search(n):
    p_start = 2**557
    q_start = 2**558

    if p_start % 2 == 0:  # Make sure we start from an odd number
        p_start += 1
    if q_start % 2 == 0:
        q_start += 1

    for p in range(p_start, 2**558, 2):  # Iterate over odd numbers
        if not isPrime(p):
            continue

        if n % p == 0:  # If p is a divisor of n
            q = n // p
            if q >= q_start and isPrime(q):
                return p, q

    return None, None
```

2. Após encontrar p e q, a chave privada d é calculada usando a função pow para encontrar o inverso modular de e em relação a `((p-1) * (q-1))`: <br>
`d = pow(e, -1, ((p-1)*(q-1)))`

3. A função dec(y, d, n) é usada para decifrar o criptograma usando a chave privada d e o módulo n: <br>
```
def dec(y, d, n):
    int_y = int.from_bytes(unhexlify(y), "little")
    x = pow(int_y, d, n)
    return x.to_bytes(256, 'little')
```


#### De seguida deves responder às seguintes questões:
##### Como consigo usar a informação que tenho para inferir os valores usados no RSA que cifrou a flag?
Para inferir os valores usados no RSA, posso usar as informações fornecidas sobre os primos p e q. Com as fórmulas utilizadas em cima, calculámos os valores aproximados p=`2**557` e q=`2**558` e, em seguida, podemos usar um teste de primalidade (como o Miller-Rabin) para encontrar os primos exatos.

##### Como consigo descobrir se a minha inferência está correta?
Para verificar se a inferência está correta temos de prosseguir da seguinte maneira: <br>
**Calcular p e q:** Usar as fórmulas fornecidas para encontrar os valores aproximados de p e q. <br>
**Testar a primalidade:** Usar o teste de primalidade de Miller-Rabin para verificar se os valores encontrados são primos. <br>
**Calcular n:** Verificamos se (n = p * q) corresponde ao módulo fornecido. <br>
**Calcular d:** Usar o algoritmo de Euclides estendido para calcular o inverso modular de e em relação a ( (p-1) * (q-1) ). <br>
Se todos os passos forem bem-sucedidos, a minha inferência está correta.

##### Finalmente, como posso extrair a minha chave do criptograma que recebi?
Para extrair a chave do criptograma, primeiro deciframos o criptograma usando a chave privada d e o módulo n. De seguida, convertemos o resultado decifrado de bytes para texto.
