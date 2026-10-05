# Compartilhamento de Arquivos e Permissões

## Visão Geral

O `SRV01` foi adicionado ao domínio `corp.example.com` como member server e configurado para disponibilizar um compartilhamento de arquivos via SMB.

Compartilhamento:

```text
\\SRV01\Shared
```

Pasta local utilizada:

```text
C:\Shares\Shared
```

## Grupos de Acesso

O acesso ao compartilhamento foi organizado utilizando grupos de segurança do Active Directory.

Para essa configuração, utilizei o modelo AGDLP (Accounts → Global Groups → Domain Local Groups → Permissions) para colocar em prática essa forma de organizar o acesso aos recursos.

Foram criados dois grupos:

```text
GG-Shared-Users
```

Grupo **Global** utilizado para reunir os usuários que precisam acessar o compartilhamento.

```text
DL-Shared-Modify
```

Grupo **Domain Local** utilizado para aplicar as permissões de acesso ao recurso.

O usuário `lab.user` foi adicionado ao grupo `GG-Shared-Users`, que depois foi adicionado ao `DL-Shared-Modify`.

## Permissões

O grupo local `SRV01\Administrators` foi mantido com **Full Control** para garantir o acesso administrativo ao compartilhamento.
Como o `CORP\Domain Admins` já faz parte desse grupo local no `SRV01`, não foi necessário adicioná-lo separadamente às permissões.

### NTFS

Na pasta `C:\Shares\Shared`, foram configuradas as seguintes permissões:

```text
SYSTEM                  → Full Control
SRV01\Administrators    → Full Control
CORP\DL-Shared-Modify   → Modify
```

### Share Permissions

No compartilhamento SMB, foram configuradas as seguintes permissões:

```text
SRV01\Administrators    → Full Control
CORP\DL-Shared-Modify   → Change
```

Com isso, o acesso ao compartilhamento depende da combinação entre as permissões de Share e NTFS.

## Validação

### Acesso Autorizado

Para validar o acesso, o compartilhamento foi acessado pelo `CL01` utilizando a conta:

```text
CORP\lab.user
```

Foram testadas as seguintes operações:

- criação de arquivo;
- edição;
- salvamento;
- exclusão.

Todas foram realizadas com sucesso.

### Acesso Não Autorizado

Também foi criada uma segunda conta para testar o acesso sem as permissões necessárias:

```text
CORP\test.user
```

Essa conta não foi adicionada aos grupos utilizados para liberar o acesso ao compartilhamento.

Ao tentar acessar:

```text
\\SRV01\Shared
```

o acesso foi negado, confirmando que um usuário fora dos grupos configurados não consegue acessar o recurso.