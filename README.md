# 🛡️ Laboratório de Auditoria de Segurança e Ataques de Força Bruta com Medusa
![Kali Linux](https://img.shields.io/badge/OS-Kali%20Linux-blue?logo=kalilinux)
![Metasploitable 2](https://img.shields.io/badge/Target-Metasploitable%202-red)
![Medusa](https://img.shields.io/badge/Tool-Medusa-orange)
![License](https://img.shields.io/badge/License-MIT-green)

## 📌 Visão Geral do Projeto

Este projeto consiste na simulação controlada de ataques de força bruta (*Brute Force*) e *Password Spraying* utilizando o **Kali Linux** e a ferramenta **Medusa**, tendo como alvos a máquina vulnerável **Metasploitable 2** e a aplicação web **DVWA (Damn Vulnerable Web Application)**.

O objetivo principal é entender na prática a mecânica dessas agressões, analisar a vulnerabilidade de credenciais fracas ou reutilizadas e documentar medidas defensivas para mitigação dos riscos.

> ⚠️ **Aviso de Isenção de Responsabilidade (Disclaimer):** Todos os testes apresentados neste repositório foram executados em um ambiente de laboratório isolado e controlado, estritamente para fins educacionais e de auditoria de segurança autorizada.

---

## 🏗️ Topologia da Rede e Ambiente de Testes

O ambiente foi virtualizado utilizando o **Oracle VM VirtualBox**, configurado em uma rede isolada (*Host-Only / Rede Interna*) para garantir que nenhum tráfego extrapolasse para redes externas.

* **Atacante:** Kali Linux (IP: `192.168.56.101`)
* **Alvo:** Metasploitable 2 (IP: `192.168.56.102`)
* **Rede Virtual:** `192.168.56.0/24` (Adaptador Host-Only)

[ Kali Linux ] (192.168.56.101) ── (Rede Interna) ── [ Metasploitable 2 ] (192.168.56.102)
├── Serviço FTP (Porta 21)
├── Serviço Web / DVWA (Porta 80)
└── Serviço SMB (Porta 445)


---

## 🔍 Etapa 1: Reconhecimento e Enumeração

Antes de iniciar os testes de força bruta, foi realizada a enumeração dos serviços ativos no alvo utilizando o **Nmap**.

```bash
nmap -sV -pn 192.168.56.102
Principais portas e serviços identificados:

21/tcp: FTP (vsftpd 2.3.4)

80/tcp: HTTP (Apache httpd 2.2.8)

445/tcp: SMB (Samba 3.0.20-Debian)

🧪 Cenários de Ataque e Execução
Cenário 1: Força Bruta em Serviço FTP (Porta 21)
Simulação de ataque dicionário direto contra o serviço FTP para comprometer contas de usuários válidas.

Criação das Wordlists Simples (wordlists/):

users.txt: Contém nomes de usuários comuns (admin, user, msfadmin, root).

passwords.txt: Contém senhas comuns (123456, password, msfadmin, admin).

Comando Executado:

Bash
medusa -h 192.168.56.102 -U wordlists/users.txt -P wordlists/passwords.txt -M ftp -v 4
Parâmetros Utilizados:

-h: Endereço IP do alvo.

-U: Caminho para a lista de usuários.

-P: Caminho para a lista de senhas.

-M: Módulo do serviço (ftp).

-v 4: Nível de detalhamento (verbosity).

Validação do Acesso:
Credencial identificada com sucesso: msfadmin:msfadmin. A validação foi feita conectando diretamente via cliente FTP:

Bash
ftp 192.168.56.102
Cenário 2: Automação de Tentativas em Formulário Web (DVWA)
Teste de força bruta no módulo de Login do DVWA (configurado no nível de segurança Low).

Obtenção do Cookie de Sessão:
Acessou-se a interface web do DVWA no navegador do Kali para capturar o PHPSESSID e o parâmetro security=low.

Comando Executado:

Bash
medusa -h 192.168.56.102 -u admin -P wordlists/passwords.txt -M web-form -m FORM:"dvwa/login.php" -m FORM-PASS:"username=USER&password=PASS&Login=Login" -v 4
Resultado:
Identificação de acesso válido para a conta admin com a senha password.

Cenário 3: Password Spraying em SMB (Porta 445)
Diferente do brute force tradicional, o Password Spraying tenta uma única senha comum contra uma lista com múltiplos usuários para evitar o bloqueio de contas (account lockout).

Enumeração Prévia de Usuários (via enum4linux):

Bash
enum4linux -U 192.168.56.102
Comando Executado:

Bash
medusa -h 192.168.56.102 -U wordlists/users.txt -p "msfadmin" -M smb
Resultado:
A técnica permitiu identificar quais contas na rede compartilhavam a mesma senha padronizada (msfadmin), contornando mecanismos simples de detecção baseados em volume por usuário.

🛡️ Recomendações de Mitigação e Prevenção
Para proteger os ambientes contra os ataques simulados, recomendam-se as seguintes boas práticas:

Política de Senhas Fortes:

Implementar requisitos mínimos de complexidade (mínimo de 12 a 16 caracteres, incluindo letras maiúsculas, minúsculas, números e símbolos).

Eliminar o uso de senhas padrão (default credentials).

Bloqueio de Conta (Account Lockout Policy):

Configurar bloqueio temporário após um número reduzido de tentativas incorretas consecutivas (ex.: 3 a 5 tentativas).

Autenticação Multi-Fator (MFA):

Exigir MFA para acesso a serviços críticos, gestão de rede e aplicações web corporativas.

Limitação de Taxa (Rate Limiting) e CAPTCHA:

Implementar CAPTCHA em formulários web de login (como o DVWA).

Restringir a quantidade de requisições por IP em um determinado intervalo de tempo no servidor web e FTP.

Sistemas de Detecção/Prevenção de Intrusão (IDS/IPS):

Implementar ferramentas como Fail2ban ou Snort para monitorar logs de autenticação e bloquear automaticamente IPs que apresentarem comportamento malicioso.

📁 Estrutura do Repositório
Plaintext
.
├── README.md            # Documentação principal do projeto
├── wordlists/           # Listas de testes utilizadas
│   ├── users.txt
│   └── passwords.txt
└── images/              # Evidências e capturas de tela dos testes
    ├── nmap_scan.png
    ├── medusa_ftp.png
    └── dvwa_login.png
🎓 Conclusão
A realização deste laboratório permitiu vivenciar o funcionamento de ferramentas de auditoria como o Medusa e compreender o impacto direto de credenciais fracas na segurança da informação. A documentação técnica e a execução em ambiente controlado reforçam a importância de implementar defesas em profundidade (Defense in Depth) nas corporações.

Desenvolvido por: [Josimário Barros]

Curso: Formação Cybersecurity / Análise e Desenvolvimento de Sistemas
