# Operações Básicas do Active Directory

## Organizational Units

Foi criada uma estrutura de OUs dentro do domínio `corp.example.com`:

```text
CORP
├── Users
├── Groups
└── Workstations
```

A OU `CORP` funciona como contêiner principal para organizar os objetos utilizados no laboratório.

O computador `CL01`, que inicialmente estava no contêiner padrão `Computers`, foi movido para:

```text
CORP\Workstations
```

## Conta de Usuário

Também foi criada uma conta de usuário do domínio dentro da OU:

```text
CORP\Users
```

Conta criada:

```text
lab.user
```

## Group Policy

Para validar a aplicação de uma Group Policy no `CL01`, foi criada a seguinte GPO:

```text
Workstations - Inactivity Lock
```

A GPO foi vinculada à OU:

```text
CORP\Workstations
```

A política configurada foi:

```text
Computer Configuration
└── Policies
    └── Windows Settings
        └── Security Settings
            └── Local Policies
                └── Security Options
                    └── Interactive logon: Machine inactivity limit
```

O valor definido foi:

```text
900 seconds
```

Com essa configuração, a estação é bloqueada automaticamente após 15 minutos de inatividade.

## Validação

Depois da configuração da GPO, a atualização das políticas no `CL01` foi executada com:

```powershell
gpupdate /force
```

Para confirmar que a GPO foi aplicada ao computador, foi utilizado:

```powershell
gpresult /r /scope computer
```

O valor configurado também foi validado diretamente no cliente:

```powershell
Get-ItemProperty `
  -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System" `
  -Name InactivityTimeoutSecs
```

O valor retornado foi:

```text
900
```