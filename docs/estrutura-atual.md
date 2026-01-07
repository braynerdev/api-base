# Estrutura atual do repositório

Este documento descreve **o que existe hoje** (estrutura base), sem sugerir mudanças.

## Raiz

- `ApiBase.API/`
- `ApiBase.Application/`
- `ApiBase.Domain/`
- `ApiBase.Infrastructure/`
- `ApiBase.CrossCutting/`
- `ApiBase.slnx`

## Dependências entre projetos (estado atual)

- `ApiBase.Domain`
  - Sem referências de projeto (ok).

- `ApiBase.Application`
  - Referencia `ApiBase.Domain`.

- `ApiBase.Infrastructure`
  - Referencia `ApiBase.Domain`.

- `ApiBase.CrossCutting`
  - Referencia `ApiBase.Application`, `ApiBase.Domain`, `ApiBase.Infrastructure`.

- `ApiBase.API`
  - Referencia `ApiBase.CrossCutting`.

## Pastas existentes por projeto

## `ApiBase.Domain`

- `Models/`
- `Interface/`

## `ApiBase.Application`

- `DTOs/`
- `Interfaces/`
- `Mappings/`
- `Comand/`
- `Behavior/`
- `Exceptions/`
- `Queries/`
- `Services/`

## `ApiBase.Infrastructure`

- `Security/`
- `Persistence/Context/`
- `Persistence/EntitiesConfiguration/`
- `Repositories/`

## `ApiBase.API`

- `Controllers/`
- `Program.cs` (pipeline padrão do template)
- `appsettings.json` / `appsettings.Development.json`
- `Dockerfile`

## Observações

- A API está com pipeline básico e OpenAPI em desenvolvimento.
- Ainda não existe código de registro de DI (Application/Infrastructure) no `Program.cs`.
