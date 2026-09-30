# Arquitetura do Laboratório

## Visão Geral

O laboratório atualmente está sendo executado no VirtualBox e conta com um Domain Controller Windows Server 2025 e um cliente Windows 11.

São utilizados dois tipos de rede virtual:

- **NAT** para acesso à rede externa e internet.
- **Host-Only** para a comunicação entre as máquinas do laboratório e o host físico.

A comunicação do Active Directory e DNS é feita pela rede Host-Only.

## Topologia Atual

```text
                         Rede Externa
                              │
                       VirtualBox NAT
                            DHCP
                              │
                 ┌────────────┴────────────┐
                 │                         │
               DC01                      CL01
                 │                         │
                 └──────────┬──────────────┘
                            │
                       Rede Host-Only
                      192.168.56.0/24
                            │
          ┌─────────────────┼─────────────────┐
          │                 │                 │
      Host .1           DC01 .10          CL01 .100
                            │
                       AD DS / DNS
                    corp.example.com
```

## Redes Virtuais

### NAT

As duas máquinas virtuais possuem um adaptador NAT configurado pelo VirtualBox.

As interfaces NAT utilizam DHCP e são responsáveis pelo acesso à rede externa. Essas interfaces ficam separadas da rede interna utilizada pelo Active Directory.

No `DC01`, o adaptador NAT está configurado para não registrar seu endereço IP na zona DNS interna. Isso evita que o endereço da interface NAT seja registrado e utilizado como um endereço DNS do Active Directory.

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

O `DC01` é responsável pelo serviço de DNS interno e utiliza seu próprio endereço na rede Host-Only (`192.168.56.10`) como servidor DNS IPv4.

O endereço utilizado internamente pelo Active Directory e DNS é:

```text
DC01.corp.example.com
192.168.56.10
```

## CL01

O `CL01` é uma máquina virtual com Windows 11 Enterprise.

Configuração de rede:

```text
NAT:       DHCP
Host-Only: 192.168.56.100/24
Gateway:   nenhum na Host-Only
DNS:       192.168.56.10
```

O cliente já foi adicionado ao domínio:

```text
corp.example.com
```

A autenticação no domínio através do `CL01` também já foi testada com sucesso.

## DNS

O serviço de DNS está instalado no `DC01`.

Os clientes do domínio utilizam o `DC01 (192.168.56.10)` como servidor DNS, tanto para resolução de nomes internos quanto externos.

As consultas DNS externas são encaminhadas pelos Forwarders configurados no `DC01`, utilizando os Root Hints como alternativa caso seja necessário.

Forward Lookup Zone:

```text
corp.example.com
```

Reverse Lookup Zone:

```text
56.168.192.in-addr.arpa
```

## Active Directory

Domínio atual:

```text
corp.example.com
```

Nome NetBIOS:

```text
CORP
```

Domain Controller:

```text
DC01.corp.example.com
```

No momento, o ambiente utiliza o site padrão criado pelo Active Directory:

```text
Default-First-Site-Name
```

Ainda não foi documentada uma estrutura personalizada de Organizational Units.