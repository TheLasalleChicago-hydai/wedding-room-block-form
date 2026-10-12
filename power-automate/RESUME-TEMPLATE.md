# Wedding Group Resume — content-controlled template

## File

- **Template (use this):** `Forms&Templates/Template/Wedding-Group-Resume-TEMPLATE.docx`
- Repo copy: `power-automate/templates/Wedding-Group-Resume-TEMPLATE.docx`
- Built by **copying** `Resume Sample.docx` (layout/graphics preserved), converting out of Word 2003 compatibility, then wrapping fillable values in plain-text content controls.
- Original sample left unchanged: `Forms&Templates/Resume Sample.docx`

## Content controls → sources

Only fields that appear on the Group Cover Sheet are controlled. Room-block grid, rates, VIP folio table, reservation method, and billing detail stay for CI/manual fill.

| Tag | Resume location | Form / CI source |
|-----|-----------------|------------------|
| `Organization` | Organization | `couple_name` |
| `PostAs` | Post As | `couple_name` (or staff edit) |
| `QuoteNumber` | Quote # | staff `quote_number` (CI) |
| `GroupArrival` | Arrival Date | `group_arrival` |
| `GroupDeparture` | Departure Date | `group_departure` |
| `MasterAccount` | Master Account # | staff `master_account` (CI) |
| `MarketCode` | Market Code | staff `market_code` (CI) |
| `GroupContact` | Group Contact | weekend contact / couple contact |
| `Telephone` | Telephone | `phone` or weekend phone |
| `WeekendContact` | In-House Contact | `weekend_contact_name` |
| `Email` | Email | `email` |
| `SalesManager` | Sales Manager(s) | manual / CI |
| `EventManager` | Event Manager(s) | manual / CI |
| `WeddingVenue` | Wedding Venue | `wedding_venue` |
| `GroomName` | Groom’s Name | manual until form has split names |
| `BrideName` | Bride’s Name | manual until form has split names |
| `CoupleParents` | Parents | `couple_parents` |
| `VIPUpgrade` | note after VIPs: | `vip_names` |
| `FrontDeskNotes` | FRONTDESK bullets | compose: gift bags + vendors |
| `GarageValetNotes` | GARAGE/VALET bullets | compose: shuttle + vendor valet |
| `CateringNotes` | OUTLET / catering line | compose: catering + allergies |
| `MajorArrivals` | BELLSTAND | `group_arrival` |
| `MajorDepartures` | BELLSTAND | `group_departure` |

## Generate Resume (Power Automate)

1. Staff enters Quote / Master Account / Market Code on the form.
2. `action: generateResume` → update SharePoint → **Populate a Microsoft Word template** using this file → save `{couple} - {draft} - Resume.docx`.

## Still manual / CI after generate

GROUP ROOM BLOCK numbers, RESERVATION METHOD, RATES, VIP table (folio/rates/IPO), BILLING INSTRUCTIONS detail, BEO guest lists.