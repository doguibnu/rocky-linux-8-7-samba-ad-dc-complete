## Configurar Samba AD-DC (DC1) Rocky Linux 8.7



Update do sistema:

```
yum update -y && yum upgrade -y 
```

Instalar o editor **nano***:

```
dnf install nano
```

Verificar sua **configuração de rede** e **DNS**, bem como **nome de domínio**. Uma das formas: Abra um **terminal** e digite:

```
 nmtui
```

Selecione **Editar uma conexão**. Então a opção **Editar**. Verificar ou alterar seu **endereço ip** e máscara de **sub-rede** e seu **Gateway**. Na opção **Servidor DNS**: inserir o DNS desejado. Não esquecer de inserir em **Domínios de pesquisa:** **seu.domínio**. Vá até o fim e selecione **OK** para salvar. Selecione **Voltar** e então no menu, **selecionar** a opção: **Definir nome de máquina do sistema**: dc1.seu.dominio. Agora selecione a opção: **OK** para salvar. Novamente selecione **OK**. Deve retornar para o terminal.



Editar o arquivo **/etc /host**:

```
nano /etc/hosts
```

Inserir os dados referentes ao seu servidor 

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
# inserir os dados de seu server
10.1.1.31    dc1                            dc1.seu.dominio
```

Onde:

10.1.1.31 = IP de seu próprio AD-DC (controlador de domínio)

dc1 = nome simplificado do seu **host*** de seu controlador de domínio

dc1.seu.domínio = nome do host + seu.domínio 



Veririficar se existe algum processo do samba rodando: 

```
# ps ax | egrep "samba|smbd|nmbd|winbindd"
```

Remover qualquer config file que houver, checando com o comando:

```
# smbd -b | grep "CONFIGFILE"
   CONFIGFILE: /usr/local/samba/etc/samba/smb.conf
```

Remover todos arquivos de data base que houver. como *.tdb e *.ldb, listando com o comando: 

```
# smbd -b | egrep "LOCKDIR|STATEDIR|CACHEDIR|PRIVATE_DIR"
  LOCKDIR: /usr/local/samba/var/lock/
  STATEDIR: /usr/local/samba/var/locks/
  CACHEDIR: /usr/local/samba/var/cache/
  PRIVATE_DIR: /usr/local/samba/private/
```

Remover o arquivo **/etc/krb5.conf** se houver, com o comando:

```
rm /etc/krb5.conf
```

Instalar plugins habilitar o repositório PowerTools com os comandos abaixo:

```
dnf install dnf-plugins-core
dnf install epel-release
dnf config-manager --set-enabled powertools
dnf update
```

Checar os repositórios adicionado com o comando:

```
dnf repolist
```

O resultado deve ser algo como:

id do repo nome do repo
appstream Rocky Linux 8 - AppStream
baseos Rocky Linux 8 - BaseOS
devel Rocky Linux 8 - Devel WARNING! FOR BUILDROOT AND KOJI USE

- **epel Extra Packages for Enterprise Linux 8 - x86_64**
- **extras Rocky Linux 8 - Extras**
- **powertools Rocky Linux 8 - PowerTools**

## 

## ## Setar o a hora local (ntp)

Listando as opções de hora local. 

```
timedatectl list-timezones
```

Para setar o timezone desejado, usar o comando: 

```
timedatectl set-timezone America/Sao_Paulo 
```

Verificando o timezone setado:

```
timedatectl
```

Instalar as dependências dos pacotes. Fazer um arquivo do tipo script executável e inserir os comandos:

```
nano depende.sh
```

Inserir os comandos:

```
set -xueo pipefail

yum update -y
yum install -y dnf-plugins-core
yum install -y epel-release

yum -v repolist all
yum config-manager --set-enabled PowerTools -y || \
    yum config-manager --set-enabled powertools -y
yum config-manager --set-enabled Devel -y || \
    yum config-manager --set-enabled devel -y
yum update -y

yum install -y \
    --setopt=install_weak_deps=False \
    "@Development Tools" \
    acl \
    attr \
    autoconf \
    avahi-devel \
    bind-utils \
    binutils \
    bison \
    ccache \
    chrpath \
    cups-devel \
    curl \
    dbus-devel \
    docbook-dtds \
    docbook-style-xsl \
    flex \
    gawk \
    gcc \
    gdb \
    git \
    glib2-devel \
    glibc-common \
    glibc-langpack-en \
    glusterfs-api-devel \
    glusterfs-devel \
    gnutls-devel \
    gpgme-devel \
    gzip \
    hostname \
    htop \
    jansson-devel \
    keyutils-libs-devel \
    krb5-devel \
    krb5-server \
    libacl-devel \
    libarchive-devel \
    libattr-devel \
    libblkid-devel \
    libbsd-devel \
    libcap-devel \
    libcephfs-devel \
    libicu-devel \
    libnsl2-devel \
    libpcap-devel \
    libtasn1-devel \
    libtasn1-tools \
    libtirpc-devel \
    libunwind-devel \
    libuuid-devel \
    libxslt \
    lmdb \
    lmdb-devel \
    make \
    mingw64-gcc \
    ncurses-devel \
    openldap-devel \
    pam-devel \
    patch \
    perl \
    perl-Archive-Tar \
    perl-ExtUtils-MakeMaker \
    perl-JSON \
    perl-Parse-Yapp \
    perl-Test-Simple \
    perl-generators \
    perl-interpreter \
    pkgconfig \
    popt-devel \
    procps-ng \
    psmisc \
    python3 \
    python3-cryptography \
    python3-devel \
    python3-dns \
    python3-gpg \
    python3-iso8601 \
    python3-libsemanage \
    python3-markdown \
    python3-policycoreutils \
    python3-pyasn1 \
    python3-setproctitle \
    quota-devel \
    readline-devel \
    redhat-lsb \
    rng-tools \
    rpcgen \
    rpcsvc-proto-devel \
    rsync \
    sed \
    sudo \
    systemd-devel \
    tar \
    tree \
    wget \
    which \
    xfsprogs-devel \
    yum-utils \
    zlib-devel \
    nano \
    krb5-workstation

yum clean all 
```

**Salvar o arquivo***

Transformar o arquivo em um executável com o comando:

```
chmod +x depende.sh
```

Executar o aquivo com o comando:

```
./depende.sh
```

Fazer download da última versão do samba ou da versão desejada: 

```
wget https://download.samba.org/pub/samba/stable/samba-4.18.0.tar.gz
```

Descompcatar o arquivo baixado:

```
tar -zxvf samba-nome-arquivo
```

Mudar para o diretório onde foi descompactado:

```
cd samba-nome-arquivo/
```

Compilar o samba  para o arquivo de configuração ficar em **/etc/samba/smb.conf** com o comando:

```
./configure --sysconfdir=/etc/samba/
```

Aguarde o processo finalizar.

E então faça os comandos make e make install:

```
make && make install
```

Configurar Kerberos:

```
cp /usr/local/samba/private/krb5.conf /etc/krb5.conf
```

No diretório **/root** edite o arquivo **.bash_profile** 

```
nano .bash_profile
```

E inserir a linha:

```
# User specific environment and startup programs
PATH=$PATH:$HOME/bin

# ESTA LINHA
PATH=/usr/local/samba/bin/:/usr/local/samba/sbin/:$PATH

export PATH
```

**Salvar o arquivo***

## Criar um arquivo de serviço systemd.

Executar os comandos:

```
# systemctl mask smbd nmbd winbind
# systemctl disable smbd nmbd winbind
```

Criar o arquivo em `/etc/systemd/system/samba-ad-dc.service`

```
nano /etc/systemd/system/samba-ad-dc.service
```

Com as seguintes linhas:

```
[Unit]
Description=Samba Active Directory Domain Controller
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/local/samba/sbin/samba -D
PIDFile=/usr/local/samba/var/run/samba.pid
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

**Salvar o arquivo***

Recarrregar o `systemd`  com o comando:

```
systemctl daemon-reload
```

Habilitar o `samba-ad` para iniciar no boot do sistema:

```
systemctl enable samba-ad-dc
```

Reestart o serviço do samba-ad com o comando:

```
systemctl restart samba-ad-dc
```

Para parar serviço fazer o comando:

```
systemctl stop samba-ad-dc
```

Se necessitar desabilitar o serviço, utilizar o comando:

```
systemctl disable samba-ad-dc
```

Verificar a versão do samba

```
smbclient --version
```

Testar o samba com o comando:

```
smbclient -L localhost -U%
```

Verificar o **DNS** (_ldap)

```
host -t SRV _ldap._tcp.seu.dominio
```

Verificar **_kerberos**

```
 host -t SRV _kerberos._udp.seu.dominio.
```

A Gravação A

```
host -t A seu.dominio
```

Verificando o Kerberos

```
kinit Administrator
Password for Administrator@SEU.DOMINIO:
Warning: Your password will expire in 41 days on Tue 22 Sep 2020 03:41:22 PM IST
```

Listar o cache dos tickets

```
klist
```

Com a resposta parecida com essa:

Valid starting       Expires              Service principal
13/04/2023 13:54:57  13/04/2023 23:54:57  krbtgt/SEU.DOMINIO@SEU.DOMINIO
    renew until 14/04/2023 13:54:50

## 

## Setar o a hora local

Listando as opções de hora local.

```
timedatectl list-timezones
```

Para setar o timezone desejado, usar o comando: 

```
timedatectl set-timezone America/Sao_Paulo 
```

Verificando o timezone setado:

```
timedatectl

Local time: qua 2023-04-12 09:04:22 -03
           Universal time: qua 2023-04-12 12:04:22 UTC
                 RTC time: qua 2023-04-12 12:00:16
                Time zone: America/Sao_Paulo (-03, -0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no 
```

## Configurar o Firewall

Adicionar serviços:

```
firewall-cmd --add-service={dns,ldap,ldaps,kerberos}
```

Adicionar portas:

```
firewall-cmd --add-port={389/udp,135/tcp,135/udp,138/udp,138/tcp,137/tcp,137/udp,139/udp,139/tcp,445/tcp,445/udp,3268/udp,3268/tcp,3269/tcp,3269/udp,49152/tcp}
```

Recarregar o Firewall:

```
firewall-cmd --reload
```

#### Replicação do SysVol (backup do AD) para um DC2

Gerar uma chave SSH-Keygen no **DC1** com o comando:

```
ssh-keygen -t rsa
```

Algumas perguntas serão feitas:

Onde salvar o arquvo de chave, que neste caso: **(/root/.ssh/id_rsa)** teclar **enter**

Entre com uma senha (ou apenas digite **enter** para ficar sem senha), teclar **enter**

Entre com a senha novamente: teclar **enter**

A saída será algo como:

Your identification has been saved in /root/.ssh/id_rsa.
Your **public key** has been saved in **/root/.ssh/id_rsa.pub**.

Quando a chave for criada, copiar para o servidor que será o **segundo DC2** com o comando:

```
ssh-copy-id user-dc2@IP-Seu-DC2
```

Agora pode-se conectar com ssh sem requerer senha de acesso.

Conecte-se via ssh em seu DC2 e mude as permissões de acesso para que o usuário do dc2 possa escrever no diretório com o comando:

```
chown root.user-dc2 -R /usr/local/samba/var/locks/sysvol/
```

Fazendo o backup do DC1 para o DC2. Para um teste antes, fazer o comando usando a opção **--dry-run** para apenas um teste sem alteração.

```
rsync --dry-run -XAavz --delete-after --progress --stats /usr/local/samba/var/locks/sysvol/ user-dc2@IP-DC2:/usr/local/samba/var/locks/sysvol/
```

O comando **rsync** fará uma copia do **sysvol** de seu DC1 (**parte1**) para o **sysvol** de seu DC2 (**parte2**):

`Parte 1:` 

 rsync --dry-run -XAavz  --delete-after --progress --stats **/usr/local/samba/var/locks/sysvol/**

`Parte 2:`

user-dc2@**IP-DC2:/usr/local/samba/var/locks/sysvol/**

O resultado deve ser algo parecido com:

building file list ... 
12 files to consider
./
seu.dominio/
seu.dominio/Policies/
seu.dominio/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/
seu.dominio/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/GPT.INI
seu.dominio/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/
seu.dominio/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/USER/
seu.dominio/Policies/{6AC1786C-016F-11D2-945F-00C04FB984F9}/
seu.dominio/Policies/{6AC1786C-016F-11D2-945F-00C04FB984F9}/GPT.INI
seu.dominio/Policies/{6AC1786C-016F-11D2-945F-00C04FB984F9}/MACHINE/
seu.dominio/Policies/{6AC1786C-016F-11D2-945F-00C04FB984F9}/USER/
seu.dominio/scripts/

Number of files: 12 (reg: 2, dir: 10)
Number of created files: 9 (reg: 2, dir: 7)
Number of deleted files: 0
Number of regular files transferred: 2
Total file size: 40 bytes
Total transferred file size: 40 bytes
Literal data: 0 bytes
Matched data: 0 bytes
File list size: 0
File list generation time: 0.038 seconds
File list transfer time: 0.000 seconds
Total bytes sent: 2,011
Total bytes received: 51

sent 2,011 bytes  received 51 bytes  1,374.67 bytes/sec
total size is 40  speedup is 0.02 **(DRY RUN)**

Com o resultado positivo, pode-se rodar o comando sem a opção **--dry-run**:

```
rsync -XAavz --delete-after --progress --stats /usr/local/samba/var/locks/sysvol/ dc2@IP-DC2:/usr/local/samba/var/locks/sysvol/
```

Para automatizar o processo, inserir o comando no **crontab**. Para aprender mais sobre o crontab clique [aqui]([Cron Job: Guia Completo para Iniciantes 2023](https://www.hostinger.com.br/tutoriais/cron-job-guia?ppc_campaign=google_search_generic_hosting_all&bidkw=defaultkeyword&lo=1031952&gclid=CjwKCAjw__ihBhADEiwAXEazJsMs2yOLBskI2NlzC9qK_ovK0G1a9KsbBpIfWGsKR49S0-V8Fa2mHBoCZJsQAvD_BwE))
Adicionar uma agenda no crontab com o comando:

```
crontab -e
```

E insira a linha:

```
0 12 * * 1-5 root rsync -XAavz --delete-after --progress --stats /usr/local/samba/var/locks/sysvol/ srvad2@10.1.1.38:/usr/local/samba/var/locks/sysvol/
```
