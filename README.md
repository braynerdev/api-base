# ApiBase

Estrutura base de API em **Clean Architecture** (ainda sem funcionalidades/implementações). O objetivo deste repositório é servir como base para novos projetos.

## Estrutura de projetos

- `ApiBase.API`
  - Camada de apresentação (ASP.NET Web API).
  - Deve ser o **composition root** (ponto onde o container de DI é montado).
- `ApiBase.Application`
  - Casos de uso (regras de aplicação), contratos e orquestração.
  - Referencia `ApiBase.Domain`.
- `ApiBase.Domain`
  - Núcleo do domínio: entidades, VOs, invariantes e regras.
  - Não deve depender de nenhuma outra camada.
- `ApiBase.Infrastructure`
  - Implementações de infraestrutura (persistência, repositórios, segurança, integrações).
  - Idealmente implementa interfaces definidas em `ApiBase.Application`.
- `ApiBase.CrossCutting`
  - Atualmente pensado para IoC.
  - Use com cuidado para não virar um “lixão” de utilitários genéricos.

## Status atual

- Existem apenas **pastas base** e projetos.
- Não há controllers/handlers/serviços/repositórios implementados.
- A API ainda não registra dependências da aplicação/infra (DI).

## Melhorias recomendadas (antes de começar a codar)

### 1) DI / Composition Root

- **Adicionar bootstrap de DI** no `Program.cs` (ou via extensões) para registrar:
  - Application (use cases/handlers/validators/mappings)
  - Infrastructure (DbContext, repositórios, serviços externos)

### 2) Revisar papel do `CrossCutting`

- Opção preferida:
  - **Renomear** `ApiBase.CrossCutting` para algo como `ApiBase.DependencyInjection` (ou manter o nome, mas restringir o escopo).
- Regra prática:
  - Dentro desse projeto, manter **somente** extensões de DI (ex.: `AddApplication`, `AddInfrastructure`) e classes de configuração.
  - Evitar colocar helpers genéricos.

### 3) Direção das dependências

- Hoje `Infrastructure` referencia apenas `Domain`. Em Clean Architecture, o mais comum é:
  - `Application` define interfaces (ex.: `IUserRepository`, `IEmailSender`)
  - `Infrastructure` implementa essas interfaces
  - Portanto `Infrastructure` normalmente precisa referenciar **`Application`**.

### 4) Padronização de nomes e pastas

- Corrigir `Comand/` para `Commands/`.
- Padronizar plural/singular e nomes (ex.: `Interfaces/` vs `Interface/`).
- Evitar pastas vazias declaradas no `.csproj` se isso começar a poluir a base.

### 5) Versionamento do .NET

- Está em `net10.0`.
  - Se a intenção for uma base “estável” para projetos, considerar `net8.0` (LTS).
  - Se a intenção for acompanhar .NET 10 desde cedo, manter como está.

## Documentação

Veja a pasta `docs/`:

- `docs/arquitetura.md`: visão de arquitetura e regras de dependência.
- `docs/estrutura-atual.md`: estrutura atual do repositório (o que existe hoje).

## Como rodar (quando tiver endpoints)

Por enquanto a API sobe com OpenAPI habilitado em desenvolvimento, mas sem controllers.

- Build/Run:
  - `dotnet build`
  - `dotnet run --project ApiBase.API`
