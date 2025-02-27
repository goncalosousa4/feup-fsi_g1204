# CTF Semana #10 (Classical Encryption)

A nossa cifra era: <br> `*;)[]|+]d;?*-_?#=*&-^])]$?)[>-#+])[]-*]-[?>[|?d?*^|-)d]*;)_]*[;+])[?*,];[?*)?)?_?->d;;)%;d;>)?-*$]%[?d-*]^>|-)%--*|?d-*d]$?#]^-d-*$]|+;[]+->[;#;:-%-?d][|-_?]*+-;?|]*]+-;*],;%;])[]*?*;*[]+--<*]*[-].>;$-d?d]*]|;]]-);_]#d]*]^>|-)%-$-**;_-$-|--#]+d-]#]_-d-|;^;d]:d?~-<;[-%>#?+]|]%]]*$]%;-#|],]|])%;-?*;*[]+-$|?%?)[])-,>)%-?$|?%?)-%[>-d],?|+--.>]]+%-*?d]%?#;*-?,|?)[-#^|-_]?_?#-)[]*]!-$>@-d?$?|>+%-<?d]-%?d]*_;-)d?*]-**;+d-:?)-d]]+<-[]%?+?%?)d>[?|-,>)%-?[])])[|][-)[?%?)*;*[]]+-+<?*?*%;)[?*d]*]^>|-)%-d;-){>@$!?[;<#%*)>|%=}` <br>

e depois de ser decifrada ficou: <br>
`sintermediosavolkswagenepontualmenteaseatoutrodosgrandesinvestimentosfeitosnonovoaudiincidiunoaspectodasegurancaasrodasdepolegadaspermitemautilizacaodetravoesmaioresemaiseficientesosistemaabsestaequipadodeserieeaniveldesegurancapassivaparaalemdaelevadarigidezdohabitaculomereceespecialreferenciaosistemaprocontenafuncaoproconactuadeformaaqueemcasodecolisaofrontalgraveovolantesejapuxadoporumcabodeacodesviandoseassimdazonadeembatecomocondutorafuncaotenentretantoconsisteemambososcintosdesegurancadian{uxpjotiblcsnurck}`

Este é um excerto de um texto de um jornal português.

##### Deves começar por investigar o criptograma correspondente, de forma a entender como a cifra em questão pode estar a cifrar os dados.
Esta é uma cifra de substituição monoalfabética, onde cada símbolo substitui uma letra.
##### Depois, deves usar os princípios do que foi ensinado nas aulas iniciais de criptografia para preparar uma metodologia de decifração
Uma análise de frequência ajuda a relacionar símbolos do texto cifrado com as letras mais comuns do português. Para isto utilizamos um script em python:
```
#!/usr/bin/env python3

from collections import Counter
import re

TOP_K  = 20
N_GRAM = 3

# Generate all the n-grams for value n
def ngrams(n, text):
    for i in range(len(text) -n + 1):
        # Ignore n-grams containing white space
        if not re.search(r'\s', text[i:i+n]):
           yield text[i:i+n]

# Read the data from the ciphertext
with open('ctf10.txt') as f:
    text = f.read()

# Count, sort, and print out the n-grams
for N in range(N_GRAM):
   print("-------------------------------------")
   print("{}-gram (top {}):".format(N+1, TOP_K))
   counts = Counter(ngrams(N+1, text))        # Count
   sorted_counts = counts.most_common(TOP_K)  # Sort 
   for ngram, count in sorted_counts:                  
       print("{}: {}".format(ngram, count))   # Print
```
sendo 'ctf10.txt' um ficheiro que contém o nosso texto cifrado. Ao correr este script obtivémos os vinte 1-gramas, 2-gramas e 3-gramas mais frequentes:
```
[12/12/24]seed@VM:~/.../Files$ python3 freq.py
-------------------------------------
1-gram (top 20):
]: 68
-: 68
?: 52
*: 43
;: 34
): 34
[: 30
/: 28
%: 26
|: 25
+: 19
>: 19
#: 13
$: 13
_: 10
^: 8
,: 7
<: 6
:: 3
=: 2
-------------------------------------
2-gram (top 20):
/]: 12
]*: 12
)[: 11
[]: 11
?*: 10
*]: 10
-*: 9
-): 9
]): 8
?): 8
?/: 8
|-: 8
%-: 8
]+: 8
*;: 7
[?: 7
)%: 7
-/: 7
+-: 7
%?: 7
-------------------------------------
3-gram (top 20):
)[]: 5
/]*: 5
)%-: 5
]+-: 5
?/]: 5
]*]: 5
])[: 4
|-): 4
?%?: 4
%?): 4
?)[: 3
)[?: 3
[?*: 3
*$]: 3
/-*: 3
*]^: 3
]^>: 3
^>|: 3
>|-: 3
-)%: 3
```

Depois, comparamos estas frequências com as letras mais frequentes na língua portuguesa. Para saber quais as frequências relativas de letras, 2-gramas e 3-gramas pesquisamos em sites, tais como, [1][2] e obtivémos os seguintes resultados:<br>
1-gramas (letras): Mapear os símbolos mais frequentes para as letras mais usadas em português.
<img src= images/ctf10_1.png width=550>

2-gramas (pares): Mapear os pares mais frequentes para bigramas comuns.
<img src= images/ctf10_2.png width=550>

3-gramas (trios): Mapear para trigramas frequentes.
<img src= images/ctf10_3.png width=550>

Com isto, chegámos primeiro à conclusão que o nosso 2-grama mais frequente '/]' corresponderia a 'de'. De seguida, concluímos que '-' seria 'a' e '?' seria 'o'. Com base nos trigramas vimos que ')[]' seria 'nte' e '])[' seria 'ent'. <br>
Utilizamos sempre o comando
`[12/12/24]seed@VM:~/.../Files$ sed 'y/]-?)[/eaont/' ctf10.txt > last.txt`
para verificar no ficheiro last.txt como ficava a cifra com as nossas substituições.<br>

**sed**-> Editor de fluxo para processar texto. <br>
**y/]-?)[/eaont/**-> A expressão y realiza uma substituição direta, caractere a caractere.
O formato é y/oldchars/newchars/, onde
oldchars são os caracteres a serem substituídos, neste caso ]-?)[ e
newchars são os caracteres que substituirão, neste caso eaont.
Os dois conjuntos devem ter o mesmo comprimento.<br>
**ctf10.txt**-> O ficheiro de entrada que será processado.<br>
**>**-> Redireciona a saída do comando para um novo ficheiro.<br>
**last.txt**-> O ficheiro onde o resultado será guardado.

Após várias tentativas e substituições encontramos as palavras 'polegadas' e 'especial' que foram essenciais para a descoberta do resto do texto, uma vez que nos permitiram descobrir várias letras. O resultado final da correspondência dos símbolos às letras foi: <br>
`[12/12/24]seed@VM:~/.../Files$ sed 'y/]-?*;)[d%|+>#$_^,<:.=&~!@/eaosintdcrmulpvgfbzqkwhjx/' ctf10.txt > last.txt`

##### No final, terás que ler o que está entre {} no final da mensagem, e submeter no servidor CTF.
A flag era {>@$!?[;<#%*)>|%=} e descodificada ficou, portanto, {uxpjotiblcsnurck}.

[1] https://www.mat.uc.pt/~pedro/lectivos/CodigosCriptografia1011/interTIC07pqap.pdf <br>
[2] https://pt.wikipedia.org/wiki/Frequ%C3%AAncia_de_letras