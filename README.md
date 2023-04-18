**PT-BR**

# Samba AD-DC no Rocky Linux 8.7

## Machina 01 (DC1):
Será instalado o samba-ad-dc de modo que funcionará com um controlador de domínio (DC1) principal


## Máquina 02 (Membro e Servidor de Arquivos):
Será instalado o samba-ad-dc de modo que funcionará como Membro e Servidor de arquivos do AD 


## Máquina 03 (Replicador do DC1) - Controlador de domínio 2 (DC2): 
Será instalado o Samba-ad-dc de modo que funcionará como um replicador do controlador de domínio. O controlador de domínio principal fará um 
backup do Sysvol para o controlador de domínio 2 (DC2)  



**ENG-USA**

# Samba AD-DC on Rocky Linux 8.7

## Machine 01 (DC1):
Samba-ad-dc will be installed so that it will work with a primary domain controller (DC1)

## Machine 02 (Member and File Server):
Samba-ad-dc will be installed so that it will work as an AD Member and File Server

## Machine 03 (DC1 Replicator) - Domain Controller 2 (DC2):
Samba-ad-dc will be installed so that it will act as a replicator of the domain controller. The primary domain controller will make a
Sysvol backup for domain controller 2 (DC2)

