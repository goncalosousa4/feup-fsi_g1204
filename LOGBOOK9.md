# Secret Key Encryption

## Tarefa 1
No ficheiro ciphertext.txt encontra-se um texto encriptado que queremos decifrar. Os 2 primeiros parágrafos do texto são os seguintes:
```
ytn xqavhq yzhu  xu qzupvd ltmat qnncq vgxzy hmrty vbynh ytmq ixur qyhvurn
vlvhpq yhme ytn gvrrnh bnniq imsn v uxuvrnuvhmvu yxx

ytn vlvhpq hvan lvq gxxsnupnp gd ytn pncmqn xb tvhfnd lnmuqynmu vy myq xzyqny
vup ytn veevhnuy mceixqmxu xb tmq bmic axcevud vy ytn nup vup my lvq qtvenp gd
ytn ncnhrnuan xb cnyxx ymcnq ze givasrxlu eximymaq vhcavupd vaymfmqc vup
v uvymxuvi axufnhqvymxu vq ghmnb vup cvp vq v bnfnh phnvc vgxzy ltnytnh ytnhn
xzrty yx gn v ehnqmpnuy lmubhnd ytn qnvqxu pmpuy ozqy qnnc nkyhv ixur my lvq
nkyhv ixur gnavzqn ytn xqavhq lnhn cxfnp yx ytn bmhqy lnnsnup mu cvhat yx
vfxmp axubimaymur lmyt ytn aixqmur anhncxud xb ytn lmuynh xidcemaq ytvusq
ednxuratvur
```

Após observar isto, corremos o ficheiro freq.py que gera estatísticas do texto cifrado como as frequências de letras, bigramas (pares de letras consecutivas) e trigramas (trios de letras consecutivas) para identificar padrões e deduzir partes do texto original.

```
[11/28/24]seed@VM:~/.../Files$ python3 freq.py
-------------------------------------
1-gram (top 20):
n: 488
y: 373
v: 348
x: 291
u: 280
q: 276
m: 264
h: 235
t: 183
i: 166
p: 156
a: 116
c: 104
z: 95
l: 90
g: 83
b: 83
r: 82
e: 76
d: 59
-------------------------------------
2-gram (top 20):
yt: 115
tn: 89
mu: 74
nh: 58
vh: 57
hn: 57
vu: 56
nq: 53
xu: 52
up: 46
xh: 45
yn: 44
np: 44
vy: 44
nu: 42
qy: 39
vq: 33
vi: 32
gn: 32
av: 31
-------------------------------------
3-gram (top 20):
ytn: 78
vup: 30
mur: 20
ynh: 18
xzy: 16
mxu: 14
gnq: 14
ytv: 13
nqy: 13
vii: 13
bxh: 13
lvq: 12
nuy: 12
vyn: 12
uvy: 11
lmu: 11
nvh: 11
cmu: 11
tmq: 10
vhp: 10
```
Ao observar estas frequências, primeiro concluímos que 'n' correspondia a 'E', uma vez que 'n' é a letra com maior frequência e o 'E' é a letra mais utilizada na língua inglesa. Além disto, o trigrama mais frequente, 'ytn', deveria ser a palavra 'THE'. <br>
De seguida, efetuamos o seguinte comando que substitui as letras 'nyt' por 'ETH' respetivamente e coloca no ficheiro test.txt o texto cifrado com estas substituições. Trocamos as letras minúsculas por maiúsculas para conseguirmos distinguir o texto encriptado das letras que já descobrimos.

`[11/28/24]seed@VM:~/.../Files$ tr 'nyt' 'ETH' < ciphertext.txt > test.txt`

O início do ficheiro test.txt ficou:

```
THE xqavhq Tzhu  xu qzupvd lHmaH qEEcq vgxzT hmrHT vbTEh THmq ixur qThvurE
vlvhpq Thme THE gvrrEh bEEiq imsE v uxuvrEuvhmvu Txx

THE vlvhpq hvaE lvq gxxsEupEp gd THE pEcmqE xb HvhfEd lEmuqTEmu vT mTq xzTqET
vup THE veevhEuT mceixqmxu xb Hmq bmic
```
A seguir, concluímos pelas frequências que 'v' deveria ser a letra 'A', uma vez que a sua frequência é a mais alta das que ainda não tínhamos descoberto e 'vup' corresponderia a 'AND'.

Continuamos este procedimento até conseguirmos todas as letras e descobrir que o artigo é o What to Expect (and Not Expect) at the Oscars, publicado pelo jornal The New York Times. [1] <br>
A correspondência toda foi: 

`[11/29/24]seed@VM:~/.../Files$ tr 'nytvuphlqmrbaxzdcgisefokj' 'ETHANDRWSIGFCOUYMBLKPVJXQ' < ciphertext.txt > test.txt` <br>

Todo o texto em test.txt ficou sem nenhuma letra minúscula, o que indica que deciframos todas as letras de ciphertext.txt. Os 2 primeiros parágrafos do texto ficaram da seguinte maneira:
```
THE OSCARS TURN  ON SUNDAY WHICH SEEMS ABOUT RIGHT AFTER THIS LONG STRANGE
AWARDS TRIP THE BAGGER FEELS LIKE A NONAGENARIAN TOO

THE AWARDS RACE WAS BOOKENDED BY THE DEMISE OF HARVEY WEINSTEIN AT ITS OUTSET
AND THE APPARENT IMPLOSION OF HIS FILM COMPANY AT THE END AND IT WAS SHAPED BY
THE EMERGENCE OF METOO TIMES UP BLACKGOWN POLITICS ARMCANDY ACTIVISM AND
A NATIONAL CONVERSATION AS BRIEF AND MAD AS A FEVER DREAM ABOUT WHETHER THERE
OUGHT TO BE A PRESIDENT WINFREY THE SEASON DIDNT JUST SEEM EXTRA LONG IT WAS
EXTRA LONG BECAUSE THE OSCARS WERE MOVED TO THE FIRST WEEKEND IN MARCH TO
AVOID CONFLICTING WITH THE CLOSING CEREMONY OF THE WINTER OLYMPICS THANKS
PYEONGCHANG
```

## Tarefa 2

1. Gerar o ficheiro plaintext.txt com 1000 bytes legíveis
bash.

`[11/29/24]seed@VM:~/.../Files$ < /dev/urandom tr -cd 'a-zA-Z0-9' | head -c 1000 > plaintext.txt`

2. Cifrar o ficheiro em diferentes modos. <br>
**AES-128-ECB**

`[11/29/24]seed@VM:~/.../Files$ openssl enc -aes-128-ecb -e -in plaintext.txt -out cipher_ecb.bin -K 00112233445566778889aabbccddeeff`

Flags usadas: <br>
-aes-128-ecb: define o modo de cifra (ECB). <br>
-e: especifica cifragem (encrypt).<br>
-in plaintext.txt: ficheiro de entrada. <br>
-out cipher_ecb.bin: ficheiro de saída. <br>
-K 00112233445566778889aabbccddeeff: chave em hexadecimal (16 bytes, como exigido pelo AES-128). <br>
O modo ECB não utiliza -iv, pois este modo não precisa de vetor de inicialização (IV). <br>

**AES-128-CBC**

`[11/29/24]seed@VM:~/.../Files$ openssl enc -aes-128-cbc -e -in plaintext.txt -out cipher_cbc.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708090a0b0c0d0e0f10`

Flags adicionais em relação ao ECB:

-iv 0102030405060708090a0b0c0d0e0f10: vetor de inicialização (IV), necessário para CBC.
O CBC utiliza IV para tornar cada bloco dependente do anterior, adicionando maior segurança contra padrões repetidos.

**AES-128-CTR**

`[11/29/24]seed@VM:~/.../Files$ openssl enc -aes-128-ctr -e -in plaintext.txt -out cipher_ctr.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708090a0b0c0d0e0f10`

Semelhante ao CBC, o CTR também requer IV.

3. Decifrar o ficheiro em diferentes modos. <br>
**AES-128-ECB**
```
[11/29/24]seed@VM:~/.../Files$ openssl enc -aes-128-ecb -d -in cipher_ecb.bin -out decipher_ecb.txt \
> -K 00112233445566778889aabbccddeeff
```

**AES-128-CBC**

`[11/29/24]seed@VM:~/.../Files$ openssl enc -aes-128-cbc -d -in cipher_cbc.bin -out decipher_cbc.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708090a0b0c0d0e0f10`

**AES-128-CTR**

`[11/29/24]seed@VM:~/.../Files$ openssl enc -aes-128-ctr -d -in cipher_ctr.bin -out decipher_ctr.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708090a0b0c0d0e0f10`


- Diferenças entre os modos <br>

AES-128-ECB (Electronic Codebook) <br>
Cada bloco é cifrado independentemente.
Padrões repetidos no texto plano geram padrões repetidos no texto cifrado.
Mais rápido, mas menos seguro.
Exemplo de uso: Não é recomendado para dados sensíveis devido à falta de aleatoriedade.

AES-128-CBC (Cipher Block Chaining) <br>
Cada bloco é cifrado com base no bloco anterior (encadeamento).
Requer um IV para iniciar a cifragem.
Resolve o problema de padrões repetidos do ECB.
Exemplo de uso: Segurança em transporte de dados, como HTTPS.

AES-128-CTR (Counter Mode) <br>
Funciona como um fluxo de cifra (stream cipher).
Gera uma sequência pseudo-aleatória baseada no IV e cifra o texto plano com XOR.
Permite cifragem e decifragem em paralelo.

### Perguntas

- Flags usadas na cifragem e diferença entre estes diversos modos: <br>
Especificadas em cima.

- Flags usadas na decifragem: <br>
Mesmas que na cifragem.


- Diferença principal entre AES-128-CTR e os outros modos: <br>
CTR usa XOR para criar uma cifra de fluxo, permitindo maior flexibilidade e paralelismo, enquanto ECB e CBC operam como algoritmos de bloco.

## Tarefa 5

Para esta tarefa temos de corromper o Byte 50*4=200.
Usamos o editor Bless para modificar o byte 200 de cada texto cifrado:

`bless cipher_ecb.bin`

Primeiro navegamos até o byte 200.<br>
De seguida, alteramos o valor hexadecimal deste byte que era 51 para FF e, por último, salvamos as alterações e fechamos o editor. <br>
Repitimos este processo para os outros dois ficheiros cifrados cipher_cbc.bin (em que o byte 200 era C3) e cipher_ctr.bin (em que o byte 200 era 53).


- Para cada um dos modos de cifra, indique quantos bytes de informação se perdem ao corromper um byte do criptograma. Verifique se esta perda se verifica quando se tenta ler a decifração dos criptogramas alterados.<br>

**AES-128-ECB** <br>
Perda de Informação: Apenas o bloco que contém o byte corrompido está ilegível (16 bytes). <br>
Explicação: No modo ECB, os blocos são independentes. A corrupção afeta apenas o bloco correspondente ao byte alterado.

**AES-128-CBC** <br>
Perda de Informação (dois blocos são afetados): <br>
O bloco que contém o byte corrompido (16 bytes) e o bloco subsequente (16 bytes).<br>
Explicação: O CBC utiliza encadeamento entre blocos, ou seja a corrupção no texto cifrado afeta a decifragem do bloco atual e do próximo bloco.

**AES-128-CTR** <br>
Perda de Informação: Apenas o byte corrompido será ilegível (1 byte). <br>
Explicação: O CTR opera como uma cifra de fluxo. A corrupção no texto cifrado afeta apenas o byte correspondente na decifragem porque cada byte é processado independentemente. <br>

Todas estas perdas se verificam quando tento ler a decifração dos criptogramas alterados.

[1] https://www.nytimes.com/2018/03/01/movies/oscars-sunday-what-to-expect.html