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

dnf install -y epel-release
dnf config-manager --enable epel
dnf install -y yum-utils
dnf config-manager --set-enabled powertools
dnf update -y

yum install -y \
gcc.x86_64 \
tar \
python36.x86_64 \
wget \
nano \
perl.x86_64 perl-Parse-Yapp.noarch \
libacl.x86_64 \
nfs4-acl-tools.x86_64 \
gnutls-devel.x86_64 \
zlib.x86_64 \
krb5-devel.x86_64 \
krb5-server \
libblkid.x86_64 \
dbus-devel.x86_64 \
jansson-devel.x86_64 \
readline.x86_64 \
bsdtar.x86_64 \
docbook-dtds.noarch \
pam-devel \
cups \
python3-markdown \
patchutils.x86_64 \
gpgme-devel \
flex \
python3-iso8601.noarch \
python3-cryptography.x86_64 \
python36-devel \
lmdb.x86_64 \
libarchive-devel \
libacl-devel \
openldap-devel \
python3-dns \
perl-Convert-ASN1.noarch \
rpcgen.x86_64 \
perl-App-cpanminus \
popt-devel.x86_64 \
zlib-devel.x86_64 \
lmdb-devel.x86_64 \
bison-devel.x86_64 \
libtasn1-tools \
bison \
perl-JSON \

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

Abrir Portas TCP:
```
firewall-cmd --permanent --zone=public --add-port={53/tcp,135/tcp,139/tcp,389/tcp,445/tcp,465/tcp,636/tcp,3268/tcp,3269/tcp,49152-65535/tcp};
```

Adicionar UDP:
```
firewall-cmd --permanent --zone=public --add-port={88/udp,123/udp,137/udp,138/udp,389/udp,464/udp};
```

Recarregar o Firewall:
```
firewall-cmd --reload
```

Copiar o arquivo **krb5.conf** para o diretório **/etc**:
```
cp /usr/local/samba/private/krb5.conf /etc/krb5.conf
```

Apagar o arquivo /etc/samba/smb.conf se houver, com o comando:
```
rm /etc/samba/smb.conf
```

Editar o arquivo **/etc/nsswitch.conf** com o comando:
```
nano /etc/nsswitch.conf
```

Faça as seguintes alterações:
```
Procure pelas linhas

De:
passwd:     files sss systemd

PARA:
passwd:     files winbind

--------------------------------

De:
group:     files sss systemd

PARA:
group:     files winbind
```

Salve o arquivo: 

Para salvar: `control+o` e para sair: `control+x`


Linkar a lib: `libnss_winbind.so.2` fazendos os três comandos abaixo:
```
ln -s /usr/local/samba/lib/libnss_winbind.so.2 /lib64/
ln -s /lib64/libnss_winbind.so.2 /lib64/libnss_winbind.so
ldconfig
```


Reinciar o samba ad-dc
```
smbcontrol all reload-config
```

Juntando-se ao **Active Directory (DC1)** a partir do (**DC2**)  = controlador de dominio 2. Substitua **seu.domínio** para seus dados:
```
samba-tool domain join seu.dominio DC -Uadministrator --realm=seu.dominio
```

Configurar o arquivo **krb5.conf** com o comando:
```
cp /usr/local/samba/private/krb5.conf /etc/krb5.conf
```

Com o comando acima o DC2 deve juntar-se ao DC1.
