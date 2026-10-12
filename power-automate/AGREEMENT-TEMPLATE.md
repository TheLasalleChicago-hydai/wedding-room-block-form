# Agreement templates — content controls

## Location

- Source folder: `Forms&Templates/Template/Agreement Template/`
- Pre-control backups: `Forms&Templates/Template/Agreement Template/_backup_pre_controls/`
- Method: same as resume — keep layout; wrap placeholders / blank label values in plain-text content controls (Word COM). Do not flatten into lists.

## Shared tags (form / CI)

| Tag | Typical source | Used for |
|-----|----------------|----------|
| `Organization` | `couple_name` | Group / company / room-block name |
| `GroupContact` | form contact name | Contact / client name |
| `Telephone` | `phone` | Phone |
| `Email` | `email` | Email |
| `QuoteNumber` | staff `quote_number` (CI) | Reference # |
| `ProgramDates` | `group_arrival`–`group_departure` (composed) | Official program / event dates |
| `GroupArrival` | `group_arrival` | Arrival (Presidential Suite) |
| `GroupDeparture` | `group_departure` | Departure (Presidential Suite) |
| `MasterAccount` | staff `master_account` (CI) | Master account (when placeholder present) |
| `AgreementDueDate` | staff / composed | LOI / agreement return-by date |
| `CutoffDate` | staff | Room-block cutoff |
| `SalesManager` | manual / CI | Letter signatory |

Optional / letter-only: `LetterDate`, `AgreementDate`, `ResponseDueDate`, `DepositAmount`, `Address`, `Fax`.

## Files processed

All DOCXs in the Agreement Template folder, including:

- Wedding Room Block with function space LOI
- Letter of Intent Template
- Rooms Only / Rooms and Space / Commissionable / Space Only GSA
- Presidential Suite Agreement
- Local Catering LOI, THTB LOI
- Addenda, Attrition Letter, Cancellation Letter

Room-block grids, rate tables, attrition math, and legal body text stay for staff/CI fill. Populate Word can target the shared tags above once flows are wired.

## Note

Many GSA/LOI body occurrences of `[Organization Name]` / `[ROOM BLOCK NAME]` are wrapped with the same `Organization` tag so one Populate value fills every mention.
