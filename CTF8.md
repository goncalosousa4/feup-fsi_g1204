# CTF Semana #8 (SQL Injection)

##### Recolher toda a informação que conseguires do site. Em particular, descobrir versões de software instalado.
Pesquisamos pelas palavras 'wordpress' e 'plugins' no código da página http://44.242.216.18:5008/ e encontramos: <br>
meta name="generator" content="WordPress 6.7.1" <br>
link rel='stylesheet' id='notificationx-public-css' href='http://44.242.216.18:5008/wp-content/plugins/notificationx/assets/public/css/frontend.css?ver=2.8.1' media='all'  <br>
Ou seja, as versões são: <br>
WordPress 6.7.1 <br>
Plugin NotificationX v2.8.1 <br>

##### Já sabes que queremos explorar uma vulnerabilidade de SQL injection. Será que existem vulnerabilidades reportadas em bases de dados públicas e/ou ferramentas que permitem automatizar a descoberta e abuso de vulnerabilidades de SQL injection?
Para o WordPress 6.7.1, o plugin NotificationX na versão 2.8.1 é vulnerável a um ataque de injeção SQL não autenticado, classificado sob o CVE-2024-1698. Esta vulnerabilidade permite que atacantes manipulem a base de dados através do parâmetro type, devido à insuficiente sanitização de entradas e falta de preparação adequada das consultas. O ataque pode ser executado sem necessidade de autenticação do utilizador, tornando-o extremamente crítico. <br>
Esta vulnerabilidade está presente em todas as versões do plugin até a 2.8.2, com uma pontuação de gravidade de 9.8 (Crítica) segundo o padrão CVSS. O problema foi resolvido na versão 2.8.3. 

##### Qual o endpoint do site vulnerável, e de que forma se pode realizar um ataque? Como pode ser catalogada a vulnerabilidade?
Para realizar o exploit, encontramos um repositório no github https://github.com/kamranhasan/CVE-2024-1698-Exploit que contém um script em python. Seguimos as instruções que se encontram no README.md, modificando o url para http://44.242.216.18:5008/ e aumentando o delay. <br>
Corremos o comando python3 exploit.py e no final o output foi: <br>
[*] Admin credentials found:     
Username: admin <br>
Password hash: `$P$BuRuB0Mi3926H8h.hcA3PsrUPyqQ010`


##### O ataque permitir-te-á extrair informação da base de dados do servidor. Em particular, queres descobrir a palavra-passe do administrador, mas como ditam as boas normas de segurança esta não é armazenada em limpo na base de dados, sendo armazenada apenas uma hash da palavra-passe original. Para este servidor, e em mais detalhe, qual a política de armazenamento de palavras-passe?
O identificador `$P$BuRuB0Mi3926H8h.hcA3PsrUPyqQ010` segue o formato característico das hashes geradas pelo WordPress para armazenar palavras-passe. O WordPress utiliza o Portable PHP Password Hashing Framework (ou phpass) para proteger palavras-passe, o que define uma política de armazenamento baseada em hashes seguros. <br>

Política de Armazenamento de Palavras-Passe no WordPress:

**Algoritmo Utilizado:** O WordPress utiliza MD5 ou, preferencialmente, bcrypt dependendo do suporte do servidor, mas ambas são encapsuladas pelo phpass. <br>
**Formato da Hash:** O identificador `$P$` indica o uso do framework phpass.
O segundo caractere (B) indica o número de iterações do hash, que aumenta a complexidade computacional. No caso de B, o número de iterações é 2 elevado a 12 (4096 iterações).
A string seguinte (uRuB0Mi3926H8h.hcA3PsrUPyqQ010) inclui:
- Salt: Uma string aleatória usada para proteger a hash contra ataques de rainbow tables.
- Hash: O resultado da combinação da palavra-passe original com o salt, processado pelas iterações definidas.

**Verificação de Palavras-Passe:** 
Quando um utilizador tenta autenticar-se, a palavra-passe inserida é submetida ao mesmo algoritmo e salt armazenado. O resultado é comparado com a hash na base de dados.

##### Será que armazenar uma hash da palavra-passe é seguro, no sentido em que torna impossível a recuperação da palavra-passe original? Esta problemática é muito comum não só aquando de vulnerabilidades, mas também no caso de data leaks. Há várias formas de tentar reverter funções de hash para palavras-passe e ferramentas para automatizar o processo.

Neste CTF usamos Hashcat para Decifrar PHPass. O Hashcat suporta hashes PHPass através do modo 400. <br>

Utilizamos o comando: <br>
hashcat -m 400 -a 0 hashes.txt rockyou.txt <br>
-m 400: Indica que o tipo de hash é PHPass. <br>
-a 0: Indica que o Hashcat usará uma wordlist. <br>
hashes.txt: Arquivo que contém a hash `$P$BuRuB0Mi3926H8h.hcA3PsrUPyqQ010`. <br>
rockyou.txt: Wordlist padrão que descarregamos e colocamos na pasta para tentar decifrar. <br>

**Processo de descodificação com Hashcat:**
Quando o comando é executado, o Hashcat começa a ler a hash armazenada no arquivo hashes.txt e a wordlist rockyou.txt. Para cada palavra da lista rockyou.txt, o Hashcat gera a hash utilizando o algoritmo PHPass (com o salt da hash original e o número de iterações adequadas). O Hashcat compara a hash gerada com a hash fornecida no arquivo hashes.txt. Quando uma correspondência é encontrada, ou seja, quando a hash gerada é idêntica à hash fornecida, o Hashcat revela a palavra que corresponde à hash original.

Embora esta abordagem PHPass seja robusta, algoritmos como MD5 já não são considerados seguros contra ataques de força bruta devido à sua velocidade. No entanto, a adição de iterações e salt mitiga significativamente esses riscos.
Se um atacante tiver acesso à base de dados e utilizar hardware potente, como GPUs, ainda será possível realizar ataques de força bruta para recuperar a palavra-passe original, especialmente se esta for fraca ou previsível.