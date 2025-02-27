
# Trabalho realizado nas Semanas #2 e #3

## Identificação

- Vulnerabilidade de Cross-Site Scripting (XSS) armazenada na aplicação Note Mark, no atributo href do link da nota.
- Afeta versões do Note Mark anteriores à 0.13.1.
- Permite que um atacante injete scripts arbitrários em links Markdown, o que resulta na execução desses scripts quando outros usuários visualizam as notas.
- A vulnerabilidade afeta vários sistemas operacionais, principalmente o Linux, onde o NoteMark é frequentemente utilizado.

## Catalogação

- Publicada a 29/07/2024 e a última modificação foi a 06/09/2024​.
- Classificada com alta gravidade (CVSS 8.7), uma vez que afeta a integridade e confidencialidade.
- Não está listada num programa de bug bounty, mas há várias referências disponíveis sobre a vulnerabilidade.
- O autor foi Alessio Romano, conhecido como sfoffo.

## Exploit

- O exploit permite Stored XSS, no qual scripts são injetados e armazenados em notas e são ativados ao aceder às notas.
- O atacante insere um script malicioso no atributo href de um link na nota Markdown devido à validação inadequada do URL pela aplicação.
- Qualquer usuário que aceder à nota com o link malicioso terá o script executado no seu navegador com os privilégios desse usuário.
- Não há módulos específicos no Metasploit mas ferramentas como o Burp Suite podem ser usadas para automatizar a injeção de payloads.

## Ataques

- A exploração da vulnerabilidade pode permitir que atacantes acedam a dados confidenciais armazenados nas notas.
- A execução de scripts maliciosos pode resultar no roubo de cookies de sessão, o que permite o controlo de contas de outros usuários.
- Payloads XSS podem redirecionar os usuários para sites controlados por atacantes, o que pode resultar em phishing e roubo de credenciais.
- Os administradores que acedem a notas comprometidas podem executar scripts maliciosos, o que aumenta o risco de danos à aplicação.
