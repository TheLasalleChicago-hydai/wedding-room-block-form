# SharePoint ↔ form mapping (CSV-backed)

**Canonical schema:** `Backups/Wedding Room PlannervSchema.csv` (latest list export).  
Payload keys live in `app.js` → `collectPayload()`.

**Do not use legacy names in new flow mapping:** `brunch_*`, `anticipated_rooms`, `bags_providing`, `bags_quantity`, `bags_deliverer`, `transport_additional`.

---

## Couple Parents — confirmed

| List column | Internal | Payload |
|-------------|----------|---------|
| **Couple Parents** | `CoupleParents` | **`couple_parents`** (text: `Name (relation); …`) |

Also sent: `couple_parents_list` (structured array for the form UI only).  
Power Automate Create/Update: map **Couple Parents** ← `json(triggerBody())?['payload']?['couple_parents']`.

---

## CSV header → payload (active)

| CSV / list DisplayName | Internal name | Payload key | Notes |
|------------------------|---------------|-------------|-------|
| Title | Title | `couple_name` | Same as couple name |
| FormURL | FormURL | *(flow)* | Resume URL from PA |
| DraftID | DraftID | *(flow)* / body `draft_id` | |
| Status | Status | *(flow)* | `draft` / `submitted` |
| Wedding Date | WeddingDate | `wedding_date` | |
| Complete By | CompleteBy | `complete_by` | |
| Created At | Createdat | *(flow)* `utcNow()` | |
| Updated At | Updatedat | *(flow)* `utcNow()` | |
| Submitted At | Submittedat | *(flow)* on submit | |
| Email | Email | `email` | |
| Phone | Phone | `phone` | |
| Couple Name | CoupleName | `couple_name` | |
| Couple Parents | CoupleParents | `couple_parents` | **Confirmed in payload** |
| VIP Upgrade | VIPUpgrade | `vip_upgrade` (= `vip_names`) | Guest free-text; PA may already compose |
| Concessions Awarded | ConcessionsAwarded | **`ConcessionsAwarded`** | Staff MultiChoice array: Concession 1–4 |
| Guests Anticipated | Guests | `guests` | |
| Guest Notes | GuestNotes | `guest_notes` | Staff-only on form |
| Group Arrival | GroupArrival | `group_arrival` | |
| Group Departure | GroupDeparture | `group_departure` | |
| Wedding Venue | WeddingVenue | `wedding_venue` | |
| Ceremony Time | WeddingTime | `ceremony_time` | Internal still `WeddingTime` |
| Reception Venue | ReceptionVenue | `reception_venue` | |
| Reception Time | ReceptionTime | `reception_time` | |
| Gift Bags Needed | GiftBags | `giftbags_provided` | yes/no |
| Gift Bag Payment Method | GiftBagPayment | `bags_payment_method` | Exact choice strings |
| Vendor Valet Needed | ValetNeeded | `valet_needed` | |
| Vendor Name | Vendors | `vendors` | JSON array; stringify in PA |
| Vendor Details | ValetDetails | `valet_details` | Composed text block |
| Valet Payment | ValetPayment | `valet_payment` | |
| Ready Room Needed | ReadyRoom | `getting_ready_needed` | |
| Ready Room Guests Number | ReadyRoomGuests | `getting_ready_guests` | |
| Ready Room Access D/T | ReadyRoomAccess | `ready_room_access_datetime` (= `date_getting_ready`) | datetime |
| Catering Needed | CateringNeeded | `catering_needed` | |
| Catering Menu Choices | CateringMenuChoices | `catering_menu_choices` | Form options may differ from list MultiChoice — align later |
| CateringMenuSelections | CateringMenuSelections | **`CateringMenuSelections`** | Guest free-text under menu choices |
| Catering Dietary Restriction | CateringDietaryRestriction | `catering_dietary_restrictions` | Allergy summary text |
| BEO Signed (Catering Menu Selected) | CateringMenuSelected | `menu_selected` | Staff-only on form |
| Shuttle Bus Required | Shuttlebus | `transport_required` | |
| Shuttle/Trans Details | TransportBlock | `transport_block` | Composed |
| In-House Contact Name | WeekendContact | `weekend_contact_name` | |
| In-House Contact Phone | WeekendPhone | `weekend_contact_phone` | |
| FormPayload | FormPayload | full `payload` JSON | |
| Guest Rooms Due Date | TotalRoomsDue | `date_room_cutoff` | Staff |
| Menu Due Date | MenuDue | `date_menu_due` | Staff |
| Vendor List Due Date | VendorListDue | `date_vendor_list` | Staff |
| Gift Bag Due Date | GiftBagDue | `date_gift_bag` | Staff |
| Staff Task List | StaffTaskList | *(list only)* | Staff marks complete in SharePoint — not on form |

### Gift Bag Payment Method — choice values (exact)

- `$6/bag charged to credit card on file`
- `$6/bag charged to couple's room`
- `Complimentary as concession`

---

## PENDING_LIST (in form payload; add list columns if missing)

| Payload | Intended list column |
|---------|----------------------|
| `quote_number` | Quote Number |
| `master_account` | Master Account |
| `market_code` | Market Code |

Not in the latest schema CSV header row — keep in FormPayload; add list fields when ready.

---

## FORM_ONLY (payload only; no list column)

| Payload | Notes |
|---------|--------|
| `couple_accommodations` | Staff textarea |
| `couple_parents_list` | UI structure |
| `catering_allergy_counts` | Object; summary goes to dietary column |
| `catering_allergy_names` | Feeds dietary summary |
| Transport detail keys (`transport_company`, …) | Also rolled into `transport_block` |
| Valet detail keys (`valet_other`, …) | Also rolled into `valet_details` |
| `bags_delivery_date` / `bags_delivery_time` / `bags_distribution` | Inside FormPayload; payment maps to list |

---

## DEPRECATED_UI / DEPRECATED_LEGACY — commented out in `collectPayload`

Remove from code after confirmed live (search `DEPRECATED_` in `app.js`):

| Old key | Reason |
|---------|--------|
| `bags_quantity` | Removed from form |
| `bags_deliverer` | Removed from form |
| `transport_additional` | Removed from form |
| `catering_datetime` | Removed from form (list column may still exist) |
| `catering_numbers` | Removed from form (list: Catering Attendance Number) |
| `brunch_*` | Legacy brunch aliases |
| `anticipated_rooms` | Alias of `guest_notes` |
| `bags_providing` | Alias of `giftbags_provided` |

Old drafts: `applyPayload` still **reads** brunch / anticipated / bags_providing aliases when loading.

---

## Power Automate Create/Update — high-value expressions

```text
CoupleParents:           @{json(triggerBody())?['payload']?['couple_parents']}
GiftBagPayment:          @{json(triggerBody())?['payload']?['bags_payment_method']}
CateringMenuSelections:  @{json(triggerBody())?['payload']?['CateringMenuSelections']}
ConcessionsAwarded:      @{json(triggerBody())?['payload']?['ConcessionsAwarded']}
ReadyRoomAccess:         @{json(triggerBody())?['payload']?['ready_room_access_datetime']}
```

`VIPUpgrade` / `TransportBlock` / `ValetDetails` — leave existing PA compose/mapping as-is; payload still includes `vip_upgrade`, `transport_block`, `valet_details` if needed.

See `RESUME-TEMPLATE.md` for Generate Resume + Word content-control map.
