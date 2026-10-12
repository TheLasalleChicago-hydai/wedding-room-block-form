# SharePoint ↔ form mapping (CSV-backed)

**Canonical schema:** `Backups/Wedding Room Planner.csv` (list export).  
Payload keys live in `app.js` → `collectPayload()`.

**Do not use legacy names in new flow mapping:** `brunch_*`, `anticipated_rooms`, `bags_providing`, `bags_quantity`, `bags_deliverer`, `transport_additional`.

---

## Pre-test checklist (SubmitFinal) — verify in Code view

Before clicking Submit on the form, open each action → **Code view** and confirm:

### DraftID_Submit (Compose)
```text
@if(empty(json(triggerBody())?['draft_id']), substring(replace(guid(), '-', ''), 0, 8), json(triggerBody())?['draft_id'])
```

### Set Variable Submit
```text
name:  DraftIDFinal
value: @outputs('DraftID_Submit')
```
(Not `outputs('DraftID')` — that Compose only exists on SaveDraft.)

### VipAccessSubmit / VipAccessDraft (Compose) — plain text only
```text
@{json(triggerBody())?['payload']?['vip_names']}
```
Do **not** wrap with `VIP:` / `Accessibility:` — those made the list look broken.

### GetSubmitItems filter
```text
DraftID eq '@{outputs('DraftID_Submit')}'
```

### Update Item Submit + Create Item Submit — must include

| Parameter | Expression |
|-----------|------------|
| DraftID | `@outputs('DraftID_Submit')` |
| FormURL | `@concat(json(triggerBody())?['form_url'], '?draft=', outputs('DraftID_Submit'), '&wedding_date=', coalesce(json(triggerBody())?['payload']?['wedding_date'], ''), '&complete_by=', coalesce(json(triggerBody())?['payload']?['complete_by'], ''))` |
| CoupleParents | `@json(triggerBody())?['payload']?['couple_parents']` |
| VIPUpgrade | `@outputs('VipAccessSubmit')` (after Compose fix above) |
| ConcessionsAwarded | `@if(empty(json(triggerBody())?['payload']?['ConcessionsAwarded']), null, join(json(triggerBody())?['payload']?['ConcessionsAwarded'], '; '))` |
| CateringMenuChoices | `@if(empty(json(triggerBody())?['payload']?['catering_menu_choices']), null, join(json(triggerBody())?['payload']?['catering_menu_choices'], '; '))` |
| CateringMenuSelections | `@json(triggerBody())?['payload']?['CateringMenuSelections']` |
| CateringDietaryRestriction | `@json(triggerBody())?['payload']?['catering_dietary_restrictions']` |
| CateringMenuSelected/Value | `@if(empty(json(triggerBody())?['payload']?['menu_selected']), null, json(triggerBody())?['payload']?['menu_selected'])` |

**Never** map `CateringMenuChoices` ← `menu_selected` (that writes `yes`).

Both columns are **Multiple lines of text**. Form sends **arrays**; flow `join`s to `"Bride's - Silver; Groom's - Gold; Catering Menu"`.

### SaveDraft also needs
- Set variable Value: `@outputs('DraftID')` (must have a value)
- CoupleParents ← `couple_parents` (not `couple_name`)
- Same catering / concessions / dietary expressions as Submit
- CateringMenuChoices + ConcessionsAwarded with `join` (not `createArray`)

### After test — Update Item Submit **Inputs** must show
- `item/DraftID`: real 8-char id (e.g. `9f4792e7`) — **not** `outputs('DraftID')`
- `item/CateringMenuChoices`: `Bride's - Silver; Catering Menu` (text) — **not** `yes`
- `item/CoupleParents`: `Joanie (Father); …`
- `item/ConcessionsAwarded`: `Concession 1; Concession 2` (text)
- `item/CateringDietaryRestriction`: allergy summary text

---

## VIP Upgrade vs Concessions Awarded

| | VIP Upgrade | Concessions Awarded |
|--|-------------|---------------------|
| Form | Guest textarea `vip_names` | Staff checkboxes only |
| Payload | `vip_names` / `vip_upgrade` | `ConcessionsAwarded` (array → PA `join`) |
| List | Text via VipAccess Compose | Multiline text |
| Purpose | Free-text VIP notes | Staff award tracking |

---

## Couple Parents — confirmed

| List column | Internal | Payload |
|-------------|----------|---------|
| **Couple Parents** | `CoupleParents` | **`couple_parents`** (text: `Name (relation); …`) |

Also sent: `couple_parents_list` (UI only).

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
| Couple Parents | CoupleParents | `couple_parents` | Note |
| VIP Upgrade | VIPUpgrade | `vip_names` | Via VipAccess Compose = plain `vip_names` |
| Concessions Awarded | **ConcessionsAwarded** | **`ConcessionsAwarded`** | Text; form array → PA `join` |
| Guests Anticipated | Guests | `guests` | |
| Guest Notes | GuestNotes | `guest_notes` | Staff-only on form |
| Group Arrival | GroupArrival | `group_arrival` | |
| Group Departure | GroupDeparture | `group_departure` | |
| Wedding Venue | WeddingVenue | `wedding_venue` | |
| Ceremony Time | WeddingTime | `ceremony_time` | |
| Reception Venue | ReceptionVenue | `reception_venue` | |
| Reception Time | ReceptionTime | `reception_time` | |
| Gift Bags Needed | GiftBags | `giftbags_provided` | Choice /Value |
| Gift Bag Payment Method | GiftBagPayment | `bags_payment_method` | Choice /Value |
| Gift Bag Expected Drop-off | GiftBagDelivery | `bags_delivery_date` + `bags_delivery_time` | SharePoint DateTime; combine and convert from Central Standard Time to UTC before writing |
| Gift Bag Distribution Instructions | GiftBagInstructions | `bags_distribution` | SharePoint multiple lines of text |
| Vendor Valet Needed | ValetNeeded | `valet_needed` | Choice /Value |
| Vendor Name | Vendors | `vendors` | string(JSON) |
| Vendor Details | ValetDetails | Compose ValetDetails* | |
| Valet Payment | ValetPayment | `valet_payment` | Choice /Value |
| Ready Room Needed | ReadyRoom | `getting_ready_needed` | Choice /Value |
| Ready Room Guests Number | ReadyRoomGuests | `getting_ready_guests` | |
| Ready Room Access D/T | ReadyRoomAccess | `date_getting_ready` | SharePoint DateTime; convert from `Central Standard Time` to `UTC` before writing so the displayed local time is preserved |
| Ready Room Access End D/T | ReadyRoomAccessEnd | `date_getting_ready_end` | SharePoint DateTime; convert from `Central Standard Time` to `UTC` before writing so the displayed local time is preserved |
| Catering Needed | CateringNeeded | `catering_needed` | Choice /Value |
| Catering Menu Choices | CateringMenuChoices | `catering_menu_choices` | Text; ≤1 bride, ≤1 groom, + Catering Menu; PA `join` |
| Catering Menu Selections | CateringMenuSelections | `CateringMenuSelections` | Note |
| Catering Dietary Restriction | CateringDietaryRestriction | `catering_dietary_restrictions` | Note |
| BEO Signed (Catering Menu Selected) | CateringMenuSelected | `menu_selected` | Choice /Value |
| Shuttle Bus Required | Shuttlebus | `transport_required` | Choice /Value |
| Shuttle/Trans Details | TransportBlock | Compose TransportBlock* | |
| In-House Contact Name | WeekendContact | `weekend_contact_name` | |
| In-House Contact Phone | WeekendPhone | `weekend_contact_phone` | |
| FormPayload | FormPayload | full `payload` JSON | |
| Guest Rooms Due Date | TotalRoomsDue | `date_room_cutoff` | |
| Menu Due Date | MenuDue | `date_menu_due` | |
| Vendor List Due Date | VendorListDue | `date_vendor_list` | |
| Gift Bag Due Date | GiftBagDue | `date_gift_bag` | |
| Staff Task List | StaffTaskList | *(list only)* | |

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

---

## FORM_ONLY (payload only; no list column)

| Payload | Notes |
|---------|--------|
| `couple_accommodations` | Staff textarea |
| `couple_parents_list` | UI structure |
| `catering_allergy_counts` | Object; summary → dietary column |
| `catering_allergy_names` | Feeds dietary summary |
| Transport detail keys | Also in `transport_block` |
| Valet detail keys | Also in `valet_details` |
| `bags_delivery_*` / `bags_distribution` | Also retained inside FormPayload; map the three dedicated GiftBag columns above for list visibility |

---

## DEPRECATED_UI / DEPRECATED_LEGACY — commented out in `collectPayload`

| Old key | Reason |
|---------|--------|
| `bags_quantity` / `bags_deliverer` | Removed from form |
| `transport_additional` | Removed from form |
| `catering_datetime` / `catering_numbers` | Removed from form |
| `brunch_*` / `anticipated_rooms` / `bags_providing` | Legacy aliases |

See `RESUME-TEMPLATE.md` for Generate Resume + Word content-control map.
