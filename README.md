# Desafio-Brute-Force-DIO
# Desafio DIO: Auditoria de Segurança com Ataques de Força Bruta

Este repositório documenta a execução do Desafio de Projeto "Criando um Ataque Brute Force de senhas com Medusa e Kali Linux" da DIO. O objetivo foi simular ataques de força bruta em um ambiente de laboratório controlado para entender as vulnerabilidades e as medidas de mitigação.

## 1. Configuração do Ambiente

O laboratório foi configurado no VirtualBox com duas máquinas virtuais na mesma rede **Host-Only (Rede Exclusiva do Hospedeiro)**.

* **Máquina Atacante (Kali Linux):**
    * **IP:** 192.168.56.103
    * **Placas de Rede:**
        1.  **Host-Only:** Para atacar o laboratório.
        2.  **NAT:** Para ter acesso à internet (`apt update`, `apt install`).
* **Máquina Alvo (Metasploitable 2):**
    * **IP:** 192.168.56.101
    * **Placa de Rede:** Host-Only.
---

## 2. Preparação dos Arquivos (Wordlists)

Para os ataques, foram criadas listas de palavras (wordlists) simples no Kali:

```bash
# Lista de usuários
echo -e "user\nmsfadmin\admin\nroot" > users.txt

# Lista de senhas
echo -e "password\n123456\nqwerty\nmsfadmin" > pass.txt
