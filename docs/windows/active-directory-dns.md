# Active Directory e DNS

## Visão Geral

O Active Directory Domain Services e o DNS estão configurados no `DC01`, utilizando Windows Server 2025.

Configuração atual:

```text
Domain:             corp.example.com
NetBIOS:            CORP
Domain Controller:  DC01
DNS Server:         192.168.56.10
```

## Configuração do Active Directory

O Active Directory Domain Services foi instalado pelo Server Manager utilizando o **Add Roles and Features Wizard**.

Depois da instalação, o `DC01` foi promovido a Domain Controller para criar uma nova floresta do Active Directory:

```text
Root domain:  corp.example.com
NetBIOS:      CORP
```

Durante a promoção, o servidor foi configurado como:

- Domain Controller
- DNS Server
- Global Catalog

No momento, o laboratório possui apenas um Domain Controller.

## Configuração do DNS

O serviço de DNS está configurado no `DC01`.

O próprio `DC01` utiliza seu endereço da rede interna como servidor DNS:

```text
192.168.56.10
```

O `CL01` utiliza o `DC01` como servidor DNS.

### Forward Lookup Zone

A zona DNS integrada ao Active Directory é:

```text
corp.example.com
```

Internamente, o Domain Controller é resolvido como:

```text
DC01.corp.example.com → 192.168.56.10
```

### Reverse Lookup Zone

Foi criada uma Reverse Lookup Zone para a rede do laboratório:

```text
56.168.192.in-addr.arpa
```

### Resolução de Nomes Externos

O `DC01` também é utilizado para resolver consultas DNS externas.

As consultas são encaminhadas pelos Forwarders configurados no servidor, utilizando os Root Hints como alternativa caso seja necessário.

Dessa forma, o `CL01` utiliza o DNS interno tanto para o Active Directory quanto para resolução de nomes externos.

## Validação

### Resolução de Nome do Domain Controller

A resolução de nome do Domain Controller foi testada a partir do cliente do domínio:

```powershell
nslookup dc01.corp.example.com
```

Resultado esperado e validado:

```text
dc01.corp.example.com
192.168.56.10
```

### Localização dos Serviços do Active Directory

Os registros LDAP utilizados para localizar os serviços do Active Directory foram testados com:

```powershell
nslookup -type=SRV _ldap._tcp.dc._msdcs.corp.example.com
```

A consulta retornou o `DC01` como Domain Controller responsável pelo serviço LDAP na porta `389`.

### Verificação do DNS

Também foi feita uma verificação do DNS diretamente no Domain Controller:

```powershell
dcdiag /test:dns /s:DC01 /DnsBasic
```

Os testes foram concluídos com sucesso.
