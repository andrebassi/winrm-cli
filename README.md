# winrm-cli

Este é um executável via bash/prompt, para executar comandos remotos em servidores Windows Server através do WinRM e WinRS.

Não oferece suporte a usuários de domínio, suporte a GSSAPI e nem Kerberos, pois seu objetivo principal é executar comandos remotos em máquinas com Windows Server Standalone.


## WinRM
Disponível para Windows Server 2008 e superior. Este projeto oferece suporte nativo à autenticação básica para contas locais. Consulte as etapas na próxima seção sobre como preparar a máquina Windows remota para este cenário. O modelo de autenticação é plugável, veja abaixo um exemplo de como usar a autenticação Negotiate/NTLM (por exemplo, para se conectar a VMs do Azure convencionais)


### Preparando Windows remoto para Basic Authentication
Suporte apenas à `Basic Authentication` com contas locais, pois não há suporte para usuários de domínio. O sistema Windows remoto deve estar preparado para winrm:

Execute um bash com PowerShell, usando a opção __Run as Administrator__ e execute a sequência de comandos abaixo:

```sh
    winrm quickconfig
    winrm set winrm/config/service/Auth '@{Basic="true"}'
    winrm set winrm/config/service '@{AllowUnencrypted="true"}'
    winrm set winrm/config/winrs '@{MaxMemoryPerShellMB="1024"}'
```

O Firewall do Windows precisa estar em execução para executar este comando. Consulte o [artigo da Base de Dados de Conhecimento Microsoft nº 2004640] (http://support.microsoft.com/kb/2004640).

Não desative a `Negotiate authentication`, pois o `winrm` usa para autenticação interna, e você corre o risco do ` winrm` não funcionar mais no seu Windows Server.

A opção `MaxMemoryPerShellMB` não tem efeitos em alguns sistemas Windows 2008R2 devido a um bug do WinRM. Certifique-se de instalar o hotfix descrito [artigo da Base de Dados de Conhecimento Microsoft nº 2842230] (http://support.microsoft.com/kb/2842230) se precisar executar comandos que usem mais de 150 MB de memória.

Para obter mais informações sobre WinRM, consulte <a href="http://msdn.microsoft.com/en-us/library/windows/desktop/aa384426(v=vs.85).aspx"> a documentação online em DevCenter da Microsoft </a>.


### Executável winrm-cli

Você pode realizar o build winrm-cli a partir da código fonte:

```sh
git clone https://github.com/andrebassi/winrm-cli
cd winrm-cli
go mod vendor
go build -o winrm .
```

Isso irá gerar um binário no diretório base chamado `./winrm`.

Você precisa ter o go na versão 1.5+. Por favor, verifique sua instalação com o seguinte comando:

```sh
go version
```

## Como usar?

```sh
./winrm -hostname 89.145.161.169 -username 'administrator' -password 'secret' 'systeminfo | findstr /C:"OS"'
```

## Docker

### Build image

```
docker build -t winrm .
```

### Como usar com docker?

```sh
docker run -it --rm winrm -hostname 89.145.161.169 -username 'administrator' -password 'secret' 'systeminfo | findstr /C:"OS"'
```

