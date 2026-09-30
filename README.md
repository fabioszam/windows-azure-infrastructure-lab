# Windows & Azure Infrastructure Lab

Laboratório prático criado para estudar e colocar em prática administração Windows, Active Directory, redes e troubleshooting em um ambiente virtualizado.

Este repositório documenta o ambiente conforme ele é construído e testado.

## Foco Atual

No momento, o laboratório está focado em **infraestrutura Windows**.

O que já foi implementado até agora:

- Instalação do Windows Server 2025
- Active Directory Domain Services
- Configuração do DNS
- Configuração do Reverse DNS
- Instalação do cliente Windows 11 Enterprise
- Configuração de rede do cliente
- Ingresso do cliente no domínio
- Autenticação no domínio
- Troubleshooting da infraestrutura

## Ambiente Atual

```text
Host
└── VirtualBox
    ├── DC01
    │   ├── NAT:       DHCP
    │   └── Host-Only: 192.168.56.10
    │
    └── CL01
        ├── NAT:       DHCP
        └── Host-Only: 192.168.56.100
```

### Rede Host-Only

```text
Network:       192.168.56.0/24
Host:          192.168.56.1
DHCP Server:   Desabilitado
```

Dentro da rede Host-Only são utilizados endereços IP estáticos.

### Active Directory

```text
Domain:             corp.example.com
NetBIOS:            CORP
Domain Controller:  DC01
DNS Server:         192.168.56.10
```

O `CL01` já foi adicionado ao domínio e a autenticação com uma conta do domínio foi testada com sucesso.

## Troubleshooting

Até agora, os principais casos foram:

1. Problema relacionado à configuração de horário identificado durante o processo de promoção do Domain Controller.
2. Registro indesejado no DNS relacionado à interface NAT do Domain Controller com múltiplas interfaces de rede.
3. Windows 11 Enterprise OOBE solicitando uma conta corporativa ou escolar durante a configuração inicial.
4. Conflito de endereço IP na rede Host-Only do VirtualBox, que afetou a descoberta do Domain Controller pelo cliente.

## Escopo

Este é um laboratório criado para estudo e também para fazer parte do meu portfólio.

O objetivo é colocar em prática e demonstrar conhecimentos de administração de infraestrutura e troubleshooting dentro de um ambiente controlado.