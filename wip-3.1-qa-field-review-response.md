# WIP 3.1 Field Review — Titan response to QA's failed fields

Response to QA's flagged fields from `WIP FieldList_V3.1_20AugustUpdate.xlsx` (WIP 3.1 Field Spec).
`#` = QA sheet row reference.

## A. Resolved in code (fixed, in the current build)

| # | Ford field | Resolution |
|---|---|---|
| 7 | `onPremisesHistory.checkInDateTime` | Now sourced from the **customer check-in/out** (drop-off) only. The vehicle-offsite-movement source was removed — it was wrongly overwriting customer arrival. |
| 8 | `repairOrderDisposition.code` | Now emitted from the raw Titan RO status code. |
| 9 | `repairOrderDisposition.description` | Now emitted from the raw Titan RO status description. |
| 11 | `repairOrderJob.sequenceNumber` | Now emitted from the RO job line id (`RO_LINE_ID`) as an integer. |
| 12 | `repairOrderJob.invoiceReference` | Now the **invoice number** of the linked invoice (was the internal invoice link-id). |
| 18 | `repairOrderJob.customerInstructions` | Mapped from `RO_INVOICE_DESCRIPTION` (customer-invoice narrative = "information back to the customer"). **Not** `CUSTOMER_NOTE` — that is the customer's incoming comment and already feeds `requestedWorkDescription`. |
| 25 | `sublet.sequenceNumber` | Synthesized as a 1-based ordinal **at RO level** (across all jobs' sublets), ordered by sundry id. Titan has no native sublet sequence. |
| 28 | `sublet.lastModified` | When a sublet becomes CLOSED (job invoiced), the timestamp now advances to the invoice date instead of staying stale. |

## B. Confirmed correct — no change needed (please close)

| # | Ford field | Note |
|---|---|---|
| 2 | `vehicleDetails.baseModel` | Correct: `baseModel` = model-type (short name), `derivative` = full model description. The base-vs-derivative split is intentional and matches Ford's schema. |
| 3 | `vehicleDetails.derivative` | Correct — see above; e.g. `"TRANSIT 2022 DOUB CHAS…"` is a valid derivative per Ford's schema. |
| 5 | `vehicleDetails.lastServiceDate` | Correct: Ford's schema requires **date-only** (`yyyy-MM-dd`), which is what we send (UTC). |

## C. Requires DMS payload change (new fields to add to the neutral snapshot)

| # | Ford field | DMS neutral field → Titan source |
|---|---|---|
| 1 | `vehicleDetails.engineNumber` | `vehicle.engineNumber` ← `VEHICLE.ENGINE_NO` |
| 4 | `vehicleDetails.nextServiceDueDate` | `vehicle.nextServiceDueDate` ← `VEHICLE.NEXT_SCHEDULE_SERVICE_FOLLOW_UP_DATE` |
| 6 | `repairOrderDetails.promisedDateTime` | `repairOrder.promisedAt` ← `SERVICE_REPAIR_ORDER_HEADER.TIME_PROMISED` |
| 29 | `repairOrderDetails.roNotes` | `repairOrder.roNotes` ← `CRM_NOTE.NOTES` (joined via header key) |
| 13 | `warrantyItems…warrantyClaimProcessedDateTime` | `job.warranty.claim.processedAt` ← `VEHICLE_MANUFACTURER_CLAIM_HEADER.PROCESSING_DATE` |
| 14 | `repairOrderJob.jobNotes` | `job.jobNotes` ← `SERVICE_REPAIR_ORDER_INVOICE_LINE.INVOICE_NOTE` |
| 15 | `repairOrderJob.technicianInstructions` | `job.technicianInstructions` ← `SERVICE_REPAIR_ORDER_TECHNICIAN_INSTRUCTION_LINE.DESCRIPTION` — **combine multiple lines** (ordered by `SEQUENCE_NO`). **Not** the `TECHNICIAN_INSTRUCTION` column, which is only a 5-char code. |
| 20/21/22 | `part.requestedQty.unit` / `issuedQty.unit` / `onOrderQty.unit` | `part.unitOfMeasure` ← `STOCK.UNIT_OF_MEASUREMENT` (one field feeds all three) |
| 19 | `part.sequenceNumber` | `part.pickslipLineID` ← `DSOPL.Pickslip_Line_ID` (integer per part line) |
| 10 | `serviceAdvisorDetails.starsID` | Mapping is already correct (`advisor.OemId`); it was **null in the test payload**. DMS must ensure the advisor's OEM ID is populated. |
