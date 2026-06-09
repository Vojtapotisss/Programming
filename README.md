# Document Approval API

ASP.NET Core Web API pro schvalování dokumentů — domácí úkol / technický test.

**Repozitář:** https://github.com/Vojtapotisss/Programming

## Spuštění

Požadavky: [.NET 8 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)

```bash
dotnet restore
dotnet run
```

- API: http://localhost:5080
- Swagger UI: http://localhost:5080/swagger

Ukázkové requesty: soubor `examples.http`

## Architektura

```
Controller -> Service -> Repository (in-memory)
```

| Vrstva | Odpovědnost |
|--------|-------------|
| Controllers | HTTP mapování, převod výsledků na status kódy |
| Services | Business logika, stavový automat workflow |
| Repositories | Perzistence v ConcurrentDictionary |
| DTOs | Request/response kontrakty API |
| Validation | FluentValidation vstupních dat |

## API endpointy

| Metoda | Endpoint | Popis |
|--------|----------|-------|
| POST | /documents | Vytvoření dokumentu |
| GET | /documents/{id} | Detail dokumentu |
| POST | /documents/{id}/approve | Schválení schvalovatelem |
| POST | /documents/{id}/reject | Zamítnutí schvalovatelem |
| GET | /documents/{id}/history | Auditní historie |

## Stavový automat

**Stavy dokumentu:** PENDING_APPROVAL -> APPROVED | REJECTED

**Stavy schvalovatele:** PENDING -> APPROVED | REJECTED

### Pravidla workflow

1. Kompletní schválení — dokument je APPROVED až když schválí všichni přiřazení schvalovatelé.
2. Okamžité zamítnutí — jedno zamítnutí okamžitě nastaví dokument na REJECTED.
3. Jednorázové rozhodnutí — schvalovatel nemůže rozhodnout dvakrát.
4. Uzavřený dokument — na APPROVED / REJECTED dokumentu nelze dále rozhodovat.

## Edge cases

| Situace | HTTP | Kód chyby |
|---------|------|-----------|
| Dokument neexistuje | 404 | DOCUMENT_NOT_FOUND |
| Schvalovatel není přiřazen | 400 | APPROVER_NOT_ASSIGNED |
| Opakované rozhodnutí | 409 | DECISION_ALREADY_MADE |
| Dokument již uzavřen | 409 | DOCUMENT_CLOSED |
| Nevalidní vstup | 400 | VALIDATION_ERROR |

## Datový model

- Document — id, fileName, status, approvers[], auditHistory[]
- Approver — email, status
- AuditRecord — timestamp, action (např. DOCUMENT_CREATED, APPROVED_BY_ALICE)

## Designové volby

- Result pattern v service vrstvě odděluje business logiku od HTTP.
- Enumy serializované jako řetězce (JsonStringEnumConverter).
- Singleton repository a service — sdílené in-memory úložiště bez databáze.
- FluentValidation pro validaci requestů mimo controllery.