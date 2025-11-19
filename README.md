# Auditoria de Segurança com Ataques de Força Bruta

Este repositório documenta a execução sobre a criação de ataques de força bruta em um ambiente de laboratório controlado. O objetivo foi explorar vulnerabilidades em diferentes serviços (FTP, SMB, Web) usando Kali Linux e ferramentas de auditoria.

## 1. 🛠️ Configuração do Ambiente

O laboratório foi construído no VirtualBox, simulando uma rede corporativa interna.

* **Máquina Atacante (Kali Linux):**
    * **IP (Host-Only):** `192.168.56.103` 
* **Máquina Alvo (Metasploitable 2):**
    * **IP (Host-Only):** `192.168.56.101`

### Detalhe Crítico da Rede (2 Placas)

Um dos principais desafios na configuração foi permitir que o Kali atacasse a rede interna (Host-Only) e, ao mesmo tempo, tivesse acesso à internet (NAT) para instalar pacotes (`apt update`, `apt install`).

A solução foi configurar **duas placas de rede** na VM do Kali:
1.  **Placa 1 (NAT):** Para acesso à Internet.
2.  **Placa 2 (Rede Exclusiva do Hospedeiro):** Para a rede interna do laboratório, permitindo a comunicação com o Metasploitable.

<img width="1154" height="651" alt="image" src="https://github.com/user-attachments/assets/beb56ec2-ed3b-4420-8822-dd475331fdb1" />

## 2. 📝 Preparação (Wordlists)

Listas de palavras simples foram criadas no Kali para realizar os testes de força bruta:

```Bash
# Criando a lista de usuários
echo -e "user\nmsfadmin\nadmin\nroot" > users.txt

# Criando a lista de senhas
echo -e "password\n123456\nqwerty\nmsfadmin" > pass.txt

````
## 3. 💥 Execução dos Ataques
Com o ambiente e as wordlists prontas, os seguintes cenários foram executados.

Cenário 1: Força Bruta no Serviço FTP (Medusa)
O primeiro teste teve como alvo o serviço FTP (Porta 21) no Metasploitable, que estava ativo e exposto.

Ferramenta: Medusa

Comando:
````Bash

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp
````
Resultado e Evidência: O Medusa testou as combinações e rapidamente encontrou as credenciais corretas para o usuário msfadmin.
![1 k](https://github.com/user-attachments/assets/604751a1-321b-4a2d-aeda-1de4a7147bc2)

Cenário 2: Password Spraying em SMB (Medusa)
O segundo teste foi um ataque de Password Spraying contra o serviço SMB (Porta 445). O objetivo era testar uma única senha comum (msfadmin) contra toda a lista de usuários.

Ferramenta: Medusa

Comando:

````Bash
medusa -h 192.168.56.101 -U users.txt -p 'msfadmin' -M smbnt
````
Resultado e Evidência: O Medusa testou a senha msfadmin contra todos os usuários e encontrou um acesso válido.

![Imagem do WhatsApp de 2025-11-11 à(s) 21 35 46_447a2870](https://github.com/user-attachments/assets/b4ccdc63-eea8-459a-84b7-16beecec7a23)

Cenário 3: Força Bruta em Formulário Web (DVWA)
Este foi o cenário mais desafiador. O objetivo era atacar o formulário de login do Damn Vulnerable Web Application (DVWA).

O Problema: Evolução das Ferramentas
A tentativa inicial de usar o Medusa falhou.

Comando (Falho): medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http -m PAGE:'/dvwa/login.php' ...

Erro: WARNING: Invalid method: PAGE.

Análise: Isso acontece porque a versão moderna do Medusa (v2.3), presente no Kali atual, teve seu módulo http alterado. Ele não suporta mais os parâmetros -m PAGE ou -m FORM para formulários web, pois agora foca em autenticação HTTP .

A Solução: Adaptação com Hydra
Conforme a flexibilidade do desafio ("adaptar à sua realidade"), a ferramenta Hydra foi utilizada, sendo ela o padrão moderno para ataques de força bruta em formulários web (http-post-form).

Ferramenta: Hydra

Comando:

````Bash
hydra -L users.txt -P pass.txt 192.168.56.101 http-post-form "/dvwa/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
````
Resultado e Evidência: O Hydra conseguiu interpretar o formulário, enviar os payloads e identificar a mensagem de falha (Login failed), encontrando com sucesso a credencial padrão do DVWA (admin:password).

![Imagem do WhatsApp de 2025-11-11 à(s) 21 36 08_38de42d4](https://github.com/user-attachments/assets/596441f4-e59e-40b4-8407-ede8a99e691c)

## 4. 🛡️ Recomendações de Mitigação
Com base nos testes, as seguintes medidas de segurança são essenciais para prevenir ataques de força bruta:

Senhas Fortes: Implementar uma política de senhas que exija complexidade (maiúsculas, minúsculas, números, símbolos) e comprimento.

Bloqueio de Contas (Account Lockout): Bloquear contas automaticamente após um número limitado de tentativas de login falhas (ex: 5 tentativas).

CAPTCHA: Utilizar CAPTCHA em formulários de login web para impedir a automação por ferramentas como o Hydra.

Autenticação de Múltiplos Fatores (MFA): A defesa mais robusta. Mesmo que um atacante descubra a senha, ele não poderá logar sem o segundo fator (ex: um código de aplicativo).

Monitoramento e Alertas: Manter logs de falhas de login e criar alertas para picos de tentativas, o que pode indicar um ataque em andamento.
