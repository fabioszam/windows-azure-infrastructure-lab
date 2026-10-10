# DHCP

O DHCP foi configurado no `SRV01` para distribuir automaticamente as configurações de rede aos clientes da rede interna.

## Configuração

- DHCP Server: `SRV01`
- Interface: `192.168.56.20`
- Scope: `192.168.56.100 - 192.168.56.120`
- Lease duration: 1 dia
- DNS Server: `192.168.56.10`
- Domain: `corp.example.com`
- Default Gateway: não configurado na rede Host-Only

Autorizei o `SRV01` no Active Directory para atuar como servidor DHCP no domínio.

Para o `CL01`, criei uma reserva para o endereço `192.168.56.100` utilizando o MAC address da interface Host-Only.

Para as atualizações PTR feitas pelo DHCP, utilizei a conta dedicada `svc_dhcp_dns`.

## Validação

No `CL01`, confirmei que as configurações foram recebidas corretamente pelo DHCP:

```text
IPv4:        192.168.56.100
Subnet Mask: 255.255.255.0
DHCP Server: 192.168.56.20
DNS Server:  192.168.56.10
DNS Suffix:  corp.example.com
```

Também confirmei que o lease e a reserva do `CL01` estavam ativos no servidor DHCP.