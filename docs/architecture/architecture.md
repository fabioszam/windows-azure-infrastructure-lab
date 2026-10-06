# Arquitetura do Laboratório

## Visão Geral

O laboratório atualmente está sendo executado no VirtualBox e conta com um Domain Controller Windows Server 2025, um File Server Windows Server 2025 e um cliente Windows 11.

São utilizados dois tipos de rede virtual:

- **NAT** para acesso à rede externa e internet.
- **Host-Only** para a comunicação entre as máquinas do laboratório e o host físico.

A rede Host-Only é utilizada para a comunicação interna do laboratório, incluindo Active Directory e DNS.

## Topologia Atual

```text
                         Rede Externa
                              │
                       VirtualBox NAT
                            DHCP
                              │
           ┌──────────────────┼──────────────────┐
           │                  │                  │
         DC01               SRV01              CL01
           │                  │                  │
           └──────────────────┼──────────────────┘
                              │
                         Rede Host-Only
                        192.168.56.0/24
                              │
          ┌───────────────────┼───────────────────┬───────────────────┐
          │                   │                   │                   │
      Host .1             DC01 .10           SRV01 .20          CL01 .100
                              │                   │
                         AD DS / DNS          File Server
                      corp.example.com
```

## Redes Virtuais

### NAT

As três máquinas virtuais possuem um adaptador NAT configurado pelo VirtualBox.

As interfaces NAT utilizam DHCP e são responsáveis pelo acesso à rede externa. Essas interfaces ficam separadas da rede interna utilizada pelo Active Directory.

### Host-Only

A rede interna utilizada pelo laboratório está configurada da seguinte forma:

```text
Network:       192.168.56.0/24
Host:          192.168.56.1
DHCP Server:   Desabilitado
```

Na rede Host-Only, os endereços IPv4 são configurados manualmente.

Configuração atual:

| Sistema | Função | IPv4 Host-Only |
| --- | --- | --- |
| Host físico | Interface Host-Only do VirtualBox | `192.168.56.1` |
| DC01 | Domain Controller e servidor DNS | `192.168.56.10` |
| SRV01 | File Server | `192.168.56.20` |
| CL01 | Cliente Windows do domínio | `192.168.56.100` |

## DC01

O `DC01` é uma máquina virtual com Windows Server 2025.

Funções atuais:

- Active Directory Domain Services
- DNS Server
- Global Catalog

Configuração de rede:

```text
NAT:            DHCP
Host-Only:      192.168.56.10/24
Gateway:        nenhum na Host-Only
Preferred DNS:  192.168.56.10
```

O endereço utilizado internamente pelo Active Directory e DNS é:

```text
DC01.corp.example.com
192.168.56.10
```

## SRV01

O `SRV01` é uma máquina virtual com Windows Server 2025 e membro do domínio `corp.example.com`.

Função atual:

- File Server

Configuração de rede:

```text
NAT:            DHCP
Host-Only:      192.168.56.20/24
Gateway:        nenhum na Host-Only
Preferred DNS:  192.168.56.10
```

## CL01

O `CL01` é uma máquina virtual com Windows 11 Enterprise e membro do domínio `corp.example.com`.

Configuração de rede:

```text
NAT:       DHCP
Host-Only: 192.168.56.100/24
Gateway:   nenhum na Host-Only
DNS:       192.168.56.10
```
