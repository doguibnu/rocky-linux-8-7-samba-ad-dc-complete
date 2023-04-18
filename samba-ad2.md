## Configurar Samba AD (DC2) Replicador Rocky Linux 8.7

Update do sistema:
```
yum update -y && yum upgrade -y
```

Instalar o editor de texto **nano**:
```
dnf install nano
```
Adicionar um usuário para que o mesmo seja utilizado como replicador do sysvol (backup do ad-dc DC1)
```
useradd userdc2
```
Adicionar senha para o usuário criado:
```
passwd userdc2
```
Insira uma senha para o usuário criado. Guarde a senha

Verificar sua **configuração de rede** e **DNS**, bem como **nome de domínio**. Uma das formas: Abra um **terminal** e chamar o **nmtui**:
```
 nmtui
```
Selecione **Editar uma conexão**. Então a opção **Editar**. Verificar ou alterar seu **endereço ip** e máscara de **sub-rede** e seu **Gateway**. 

Na opção **Servidor DNS**: inserir PREFERENCIALMENTE o ip de seu **Controlador de Domínio Principal** (seu **DC1**).

 Não esquecer de inserir em **Domínios de pesquisa:** **seu.domínio**. Vá até o fim e selecione **OK** para salvar. Selecione **Voltar** e então no menu, **selecionar** a opção: 

**Definir nome de máquina do sistema**: dc2.seu.dominio. Agora selecione a opção: **OK** para salvar. Novamente selecione **OK**. Deve retornar para o terminal.

Editar o arquivo **/etc /host**:
```
nano /etc/hosts
```

Inserir os dados referentes ao seu servidor de **Controlador de Domínio 2:**
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
# inserir os dados de seu server
10.1.1.38   dc2                             dc2.seu.dominio
```
**Salvar o arquivo**

Veririficar se existe algum processo do samba rodando:
```
 ps ax | egrep "samba|smbd|nmbd|winbindd"
```

Remover qualquer configuração smb.conf que houver, checando com o comando:
```
# smbd -b | grep "CONFIGFILE"
   CONFIGFILE: /usr/local/samba/etc/samba/smb.conf
```

Remover todos arquivos de data base que houver. como ***.tdb** e ***.ldb**, listando com o comando:
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

Instalar plugins e habilitar repositório **PowerTools** com os comandos abaixo:
```
dnf install dnf-plugins-core
dnf install epel-release
dnf config-manager --set-enabled powertools
dnf update
```

Checar os repositórios adicionados com o comando:
```
dnf repolist
```

O resultado deve ser algo como:
```
id do repo nome do repo
appstream Rocky Linux 8 - AppStream
baseos Rocky Linux 8 - BaseOS
devel Rocky Linux 8 - Devel WARNING! FOR BUILDROOT AND KOJI USE

- epel Extra Packages for Enterprise Linux 8 - x86_64
- extras Rocky Linux 8 - Extras
- powertools Rocky Linux 8 - PowerTools
```

## Setar a hora local:

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

Instalar as dependências dos pacotes para instalar e compilar o **samba AD-DC**. No **terminal** fazer um arquivo do tipo script executável e inserir os comandos:
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
    krb5-workstation \ 

yum clean all 
```

**Salvar o arquivo**

Transformar o arquivo em um executável:
```
chmod +x depende.sh
```

Executar o aquivo com o comando:
```
./depende.sh
```

Fazer download da última versão do **samba** com o comando:
```
wget https://download.samba.org/pub/samba/stable/samba-4.18.0.tar.gz
```

Descompcatar o arquivo baixado com o comando:
```
tar -zxvf samba-nome-arquivo
```

Mudar para o diretório onde foi descompactado:
```
cd samba-nome-arquivo/
```

Compilar o samba para que o arquivo de configuação **smb.conf** ficar em:  **/etc/samba/smb.conf**:
```
./configure --sysconfdir=/etc/samba/
```

Aguarde o procedimento terminar.

Executar make e make install:
```
make && make install
```

Fazer com que o comando **samba-tool** possa ser chamado na raíz do sistema. No diretório **/root** editar o arquivo **.bash_profile:**
```
nano .bash_profile
```

E inserir a linha:
```
# User specific environment and startup programs
PATH=$PATH:$HOME/bin

#ESTA LINHA:
PATH=/usr/local/samba/bin/:/usr/local/samba/sbin/:$PATH

export PATH
```

**Salvar o arquivo**

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

**Salvar o arquivo**

Recarrregar o `systemd` com o comando:
```
systemctl daemon-reload
```

Habilitar o `samba-ad-dc` para iniciar no boot do sistema:
```
systemctl enable samba-ad-dc
```

Reestart o serviço do samba-ad-dc com o comando:
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

Copiar o arquivo **krb5.conf** para o diretório **/etc**:
```
cp /usr/local/samba/private/krb5.conf /etc/krb5.conf
```

Executar o comando **testparm** para verificação de erros de sintaxe:
```
testparm
```
A saída do comando pode ser parecido com essa:
```
Load smb config files from /etc/samba/smb.conf
Loaded services file OK.
Weak crypto is allowed

Server role: ROLE_DOMAIN_MEMBER

Press enter to see a dump of your service definitions
```

Reinciar o samba ad-dc
```
smbcontrol all reload-config
```

Juntando-se ao **Active Directory (DC1)** a partir do (**DC2**)  = controlador de dominio 2. Substitua **seu.domínio** para seus dados:
```
samba-tool domain join seu.dominio DC -Uadministrator --realm=seu.dominio
```
Com o comando acima o DC2 deve juntar-se ao DC1.
