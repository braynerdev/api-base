# Arquitetura (Clean Architecture)

Este repositório segue a ideia de **Clean Architecture**: regras de negócio no centro, detalhes (framework, banco, IO) nas bordas, e dependências sempre apontando *para dentro*.

## Objetivos

- Isolar o domínio de detalhes de infraestrutura.
- Facilitar testes (principalmente em `Domain` e `Application`).
- Permitir troca de banco/ORM/serviços externos com impacto mínimo.

## Camadas e responsabilidades

### `ApiBase.Domain`

O `Domain` é o **coração** do sistema. Aqui ficam as regras de negócio mais puras, sem depender de banco, HTTP, ASP.NET, EF Core ou qualquer detalhe de infraestrutura.

- **O que deve conter**
  - Modelos de domínio (entidades e value objects).
  - Regras/invariantes (validações de consistência do negócio).
  - (Opcional) eventos e serviços de domínio.
- **O que não deve conter**
  - Persistência (EF Core, DbContext, migrations).
  - Comunicação externa (HttpClient, filas, SDKs).
  - DTOs/Requests/Responses da API.

#### Pastas típicas no `Domain` (e como usar)

##### `Entities/` (ou `Models/`)

**O que é Entidade?**

Entidade é um objeto do domínio que possui **identidade própria**. Você reconhece a entidade pelo “quem é”, não apenas pelo “o que contém”.

- Exemplo simples (conceitual): `Cliente`, `Pedido`, `Usuario`.
- Uma entidade geralmente tem um `Id` e pode mudar de estado ao longo do tempo.

**Como vai ser usada na prática:**

- Controllers recebem um request, a `Application` executa um caso de uso e **manipula entidades**.
- A `Infrastructure` persiste/consulta essas entidades no banco.

**Regra prática:** se você precisa “atualizar” o objeto e ele continua sendo o mesmo (mesmo `Id`), é uma entidade.

##### `ValueObjects/`

**O que é Value Object?**

Value Object é um objeto do domínio que **não tem identidade** e é comparado pelo **valor**. Normalmente é **imutável**.

- Exemplos comuns: `Cpf`, `Email`, `Dinheiro (Money)`, `Endereco`.
- Dois VOs são “iguais” se todos os seus valores internos são iguais.

**Como vai ser usado na prática:**

- Value objects ajudam a garantir regras (ex.: “e-mail precisa ser válido”) em um único lugar.
- Entidades usam VOs como propriedades (ex.: `Cliente.Email`).

**Regra prática:** se não faz sentido ter `Id`, e o que importa é o conteúdo (e não a identidade), é um value object.

##### `Enums/`

- Para estados fixos do domínio (ex.: `StatusPedido`: `Aberto`, `Pago`, `Cancelado`).
- Preferir enums quando o conjunto de valores é realmente fechado e estável.

##### `Exceptions/`

- Exceções do domínio (ex.: `DomainException`, `InvalidCpfException`).
- Úteis quando uma regra do negócio é violada.

##### `Events/` (ou `DomainEvents/`)

- Eventos “algo aconteceu” no domínio (ex.: `PedidoPago`).
- Útil para disparar ações posteriores sem acoplar diretamente (ex.: enviar e-mail depois de pagamento).

##### `Abstractions/` ou `Interfaces/`

- Apenas contratos **que fazem sentido no domínio**, como `IAggregateRoot`.
- Evite colocar aqui interfaces de repositório. Repositório normalmente é contrato de aplicação (para ser implementado pela infraestrutura).

### `ApiBase.Application`

O `Application` é onde ficam os **casos de uso**. É aqui que você implementa “o que o sistema faz”, orquestrando o domínio e chamando dependências externas via interfaces.

- **O que deve conter**
  - Casos de uso (por feature) e a orquestração da regra.
  - Interfaces que representam dependências externas (ex.: repositórios, e-mail, storage, clock).
  - Validações, mapeamentos e exceções da camada.
- **O que não deve conter**
  - Implementação de persistência.
  - Dependência direta de ASP.NET Core (HttpContext, Controller, etc.).

#### Pastas típicas no `Application` (e como usar)

##### `Commands/` e `Queries/` (ou `UseCases/`)

- **Commands**: ações que mudam estado (ex.: `CriarCliente`, `AtualizarPedido`).
- **Queries**: leituras (ex.: `ObterClientePorId`).
- Se você não for usar CQRS/MediatR, você pode agrupar como `UseCases/` ou `Services/`.

##### `DTOs/`

- Objetos de transferência usados **entre camadas** (ex.: entrada/saída de caso de uso).
- Dica: DTO da Application não precisa ser idêntico ao request/response da API.

##### `Interfaces/` (ou `Abstractions/`)

- Aqui ficam interfaces como `IClienteRepository`, `IUnitOfWork`, `IEmailSender`, etc.
- A `Infrastructure` implementa essas interfaces.

##### `Mappings/`

- Perfis e mapeamentos (ex.: AutoMapper) entre entidades, DTOs, view models.

##### `Behavior/`

- Behaviors de pipeline (se usar MediatR): logging, validação, transação.

##### `Exceptions/`

- Exceções de aplicação (ex.: `NotFoundException`, `ValidationException`).

##### `Services/`

- Serviços de aplicação que orquestram casos de uso quando você não usa CQRS.

### `ApiBase.Infrastructure`

O `Infrastructure` contém os **detalhes**: banco de dados, arquivos, integrações, provedores externos e qualquer implementação “de fora pra dentro”.

- **O que deve conter**
  - Implementações de repositórios e Unit of Work.
  - Persistência (DbContext, migrations, configurações de entidades).
  - Implementações de serviços externos.
  - Segurança/autenticação (dependendo do design).
- **Dependências**
  - Geralmente referencia `Application` para implementar as interfaces definidas lá.

#### Pastas típicas no `Infrastructure` (e como usar)

##### `Persistence/`

- Tudo que é “acesso a dados”.

##### `Persistence/Context/`

- DbContext(s) e configurações gerais do EF.

##### `Persistence/EntitiesConfiguration/` (ou `Configurations/`)

- Fluent API de mapeamento (ex.: `IEntityTypeConfiguration<T>`).

##### `Repositories/`

- Implementações concretas das interfaces da `Application` (ex.: `ClienteRepository : IClienteRepository`).

##### `Security/`

- Hashing, JWT, autenticação/autorização, provedores de identidade.

##### `ExternalServices/` (quando existir)

- Integrações com terceiros: e-mail, SMS, storage, APIs.

### `ApiBase.API`

O `API` é a camada de entrada HTTP. Ela valida requests, chama a `Application` e devolve responses.

- **O que deve conter**
  - Controllers/endpoints.
  - Pipeline HTTP (middlewares, filtros, auth, versionamento, etc.).
  - Modelos de request/response (contratos da API), se você optar por separar.
- **Recomendação importante**
  - Esta camada deve ser o **composition root**: é aqui que o container de DI é montado.

#### Pastas típicas no `API` (e como usar)

##### `Controllers/`

- Controllers chamam casos de uso da `Application`. Eles não devem conter regra de negócio.

##### `Middlewares/` (quando existir)

- Tratamento global de exceções, correlation-id, logging, etc.

##### `Contracts/` (opcional)

- Requests/Responses públicos da API (bom quando você não quer expor DTOs da Application).

### `ApiBase.CrossCutting` (IoC)

Este projeto existe para centralizar a configuração do container de DI.

- **Como usar**
  - Criar extensões como `AddApplication(...)` e `AddInfrastructure(...)`.
  - A `API` chama uma única extensão (ex.: `builder.Services.AddCrossCutting(...)`) ou chama as extensões diretamente.
- **Cuidado**
  - Evite colocar aqui utilitários genéricos “porque não tem lugar”. Isso tende a virar um ponto de acoplamento.

## Regras de dependência (resumo)

- `Domain` -> não depende de ninguém
- `Application` -> depende de `Domain`
- `Infrastructure` -> depende de `Application` (e possivelmente `Domain`)
- `API` -> depende de `Application` + `Infrastructure` (direto) ou via um projeto de bootstrap (DI)

## Evolução sugerida (quando começar a implementar)

- Definir um padrão de casos de uso:
  - CQRS com MediatR (Commands/Queries + Handlers + Behaviors) **ou**
  - UseCases/Services (mais simples)
- Padronizar as pastas por feature (ex.: `Features/Users/...`) para evitar pastas genéricas grandes.
