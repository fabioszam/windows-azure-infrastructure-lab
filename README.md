# Windows & Azure Infrastructure Lab

Laboratório prático criado para estudar e colocar em prática administração Windows, Active Directory, redes e troubleshooting em um ambiente virtualizado.

Este repositório documenta o ambiente conforme ele é construído e testado.

## Foco Atual

No momento, estou finalizando a parte de **infraestrutura Windows** do laboratório.

O que já foi implementado:

- Windows Server 2025
- Active Directory Domain Services
- DNS e Reverse DNS
- DHCP Server
- Windows 11 Enterprise integrado ao domínio
- Organizational Units, usuários e grupos
- Group Policy
- Member Server
- File Server e compartilhamento SMB
- Permissões NTFS e Share utilizando grupos de segurança
- Troubleshooting de rede e DNS

## Ambiente Atual

```text
Host
└── VirtualBox
    ├── DC01
    │   ├── NAT:       DHCP
    │   └── Host-Only: 192.168.56.10
    │
    ├── SRV01
    │   ├── NAT:       DHCP
    │   └── Host-Only: 192.168.56.20
    │
    └── CL01
        ├── NAT:       DHCP
        └── Host-Only: 192.168.56.100
```

### Rede Host-Only

```text
Network:                  192.168.56.0/24
Host:                     192.168.56.1
VirtualBox DHCP Server:   Desabilitado
Windows DHCP Server:      192.168.56.20
DHCP Scope:               192.168.56.100 - 192.168.56.120
```

O `DC01` e o `SRV01` utilizam endereços estáticos na rede Host-Only. O `CL01` recebe o endereço `192.168.56.100` através de uma reserva DHCP.

### Active Directory

```text
Domain:             corp.example.com
NetBIOS:            CORP
Domain Controller:  DC01
DNS Server:         192.168.56.10
```

Atualmente, o ambiente conta com:

- `DC01` como Domain Controller e servidor DNS;
- `SRV01` como File Server e servidor DHCP;
- `CL01` como cliente Windows.

## File Server

O `SRV01` disponibiliza o seguinte compartilhamento:

```text
\\SRV01\Shared
```

O acesso é configurado utilizando grupos de segurança do Active Directory, junto com permissões NTFS e Share.

## Troubleshooting

Os principais casos documentados até agora são:

1. Conflito de endereço IP na rede Host-Only do VirtualBox que impedia o ingresso do `CL01` no domínio.
2. Registro indesejado do endereço da interface NAT do `DC01` na zona DNS do domínio.

## Documentação

- [Arquitetura do laboratório](docs/architecture/architecture.md)
- [Active Directory e DNS](docs/windows/active-directory-dns.md)
- [Operações básicas do Active Directory](docs/windows/basic-ad-operations.md)
- [Compartilhamento de arquivos e permissões](docs/windows/file-share-permissions.md)
- [DHCP](docs/windows/dhcp.md)
- [Troubleshooting](docs/troubleshooting/troubleshooting.md)
