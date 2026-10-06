# Troubleshooting

## Falha no ingresso ao domínio por conflito de endereço IP

### Sintoma

O `CL01` não conseguia ingressar no domínio `corp.example.com`.

Ao tentar localizar o Domain Controller com:

```powershell
nltest /dsgetdc:corp.example.com /force
```

o comando retornava:

```text
ERROR_NO_SUCH_DOMAIN
```

### Investigação

Primeiro, verifiquei se o problema estava relacionado ao DNS.

A consulta ao registro SRV do Active Directory retornava corretamente o `DC01`:

```powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.corp.example.com
```

Também testei a conectividade do `CL01` com os principais serviços do Domain Controller:

```powershell
Test-NetConnection DC01 -Port 88	→ Kerberos
Test-NetConnection DC01 -Port 135	→ RPC
Test-NetConnection DC01 -Port 389	→ LDAP
Test-NetConnection DC01 -Port 445	→ SMB
```

Todas as conexões retornaram `TcpTestSucceeded: True`.

Como o DNS e a conectividade com esses serviços estavam funcionando, revisei a configuração de rede do `CL01` com:

```powershell
ipconfig /all
```

Na interface Host-Only, o endereço configurado aparecia como:

```text
192.168.56.100 (Duplicate)
```

Além disso, a interface havia recebido um endereço APIPA `169.254.x.x`.

### Causa

O problema era um conflito de endereço IP na rede Host-Only.

O endereço estático `192.168.56.100`, configurado no `CL01`, também estava sendo utilizado pelo DHCP do VirtualBox.

Mesmo com a resolução DNS funcionando, o conflito afetava a comunicação do cliente com o domínio e impedia a localização correta do Domain Controller.

### Correção

Como os endereços da rede interna são configurados manualmente, desabilitei o DHCP da rede Host-Only no VirtualBox para eliminar o conflito.

O `CL01` permaneceu com a seguinte configuração:

```text
IPv4:    192.168.56.100/24
DNS:     192.168.56.10
Gateway: nenhum na interface Host-Only
```

### Validação

Depois da correção:

- o `CL01` ingressou com sucesso no domínio `corp.example.com`;
- a autenticação com uma conta do domínio funcionou normalmente.

Também validei o secure channel com:

```powershell
Test-ComputerSecureChannel -Verbose
```

Resultado:

```text
True
```

---

## Registro incorreto da interface NAT no DNS do Domain Controller

### Sintoma

O `DC01` possui duas interfaces de rede:

```text
NAT:       10.0.2.15
Host-Only: 192.168.56.10
```

Durante os testes de DNS, percebi que o endereço da interface NAT (`10.0.2.15`) também estava registrado na zona DNS do domínio.

Esse endereço não deveria ser utilizado para representar o Domain Controller dentro da rede interna.

### Investigação

Ao revisar a zona `corp.example.com`, encontrei um registro associando o `DC01` ao endereço:

```text
10.0.2.15
```

Como esse endereço pertence à interface NAT, revisei a configuração de registro DNS das interfaces do servidor e também as interfaces utilizadas pelo serviço DNS.

Para confirmar se o DNS interno estava funcionando pelo endereço correto, fiz as consultas diretamente contra `192.168.56.10`:

```powershell
nslookup -type=NS corp.example.com 192.168.56.10
```

```powershell
nslookup -type=SRV _ldap._tcp.corp.example.com 192.168.56.10
```

As consultas responderam normalmente pelo endereço Host-Only do `DC01`.

### Causa

A interface NAT do `DC01` também estava registrando seu endereço no DNS, fazendo com que `10.0.2.15` aparecesse na zona utilizada pelo Active Directory.

Como esse endereço pertence à rede NAT do VirtualBox, ele não deveria ser utilizado como endereço interno do Domain Controller.

### Correção

Na interface NAT do `DC01`, desabilitei a opção:

```text
Register this connection's addresses in DNS
```

No **DNS Manager**, em:

```text
Server Properties
└── Interfaces
    └── Only the following IP addresses
```

configurei o serviço DNS para escutar apenas no endereço interno:

```text
192.168.56.10
```

Também removi da zona do domínio o registro que apontava o `DC01` para:

```text
10.0.2.15
```

### Validação

Depois das alterações, o `DC01` permaneceu registrado no DNS apenas pelo endereço interno:

```text
DC01.corp.example.com → 192.168.56.10
```

As consultas aos registros NS e LDAP SRV continuaram respondendo normalmente através de `192.168.56.10`.

Também testei o DNS diretamente pelo endereço da interface NAT (`10.0.2.15`), mas a consulta não respondeu.

Já pelo endereço interno `192.168.56.10`, o DNS respondeu normalmente.

Por fim, executei novamente:

```powershell
dcdiag /test:dns /s:DC01 /DnsBasic
```

O teste foi concluído com sucesso depois dos ajustes.
