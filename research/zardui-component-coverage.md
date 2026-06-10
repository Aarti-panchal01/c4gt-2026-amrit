# ZardUI Component Coverage Spike — Week 1 Findings

**Date:** Week 1 (Jun 10–16)
**Purpose:** Validate that ZardUI component library covers all UI needs for Phase 2, or identify fallback strategy.

**Components Tested:**

| Component | Need | ZardUI Available? | Status | Fallback |
|-----------|------|-------------------|--------|----------|
| Button | Primary/secondary actions | ✅ Yes | Simple, works | N/A |
| Input | Text fields | ✅ Yes | Works | N/A |
| Select | Dropdowns | ✅ Yes | Works | N/A |
| Datepicker | Calendar widget (date-of-birth, appt scheduling) | 🟡 Check | [TBD] | ngx-bootstrap datepicker / ng-bootstrap |
| Dialog/Modal | Confirm, alert, forms | ✅ Yes | Works | Material CDK dialog |
| Table | Data tables with sort/filter/paging (heavy use in reports) | ⚠️ Unclear | [CRITICAL] | CDK DataSource + custom OR ng2-smart-table |
| Tabs | Tabbed content (104-co, 104-hao heavy use) | ✅ Yes | Works | N/A |
| Pagination | Paginated tables | 🟡 Check | [TBD] | ngx-pagination (current) |
| Checkbox | Multi-select | ✅ Yes | Works | N/A |
| Radio | Single-select | ✅ Yes | Works | N/A |
| Autocomplete | Type-ahead (beneficiary search) | 🟡 Check | [TBD] | CDK autocomplete OR ngx-typeahead |
| Toast/Notification | Error/success messages | 🟡 Check | [TBD] | ngx-toastr (current replacement) |
| Stepper | Multi-step forms (prescription, closure) | 🟡 Check | [TBD] | CDK stepper |
| Accordion | Collapsible sections | 🟡 Check | [TBD] | CDK accordion |

**Key Findings:**

### ✅ Confirmed Working
- Button, Input, Select, Dialog, Tabs, Checkbox, Radio all available and simple

### ⚠️ Critical Uncertainty: Table Component
- ZardUI's table component coverage unclear
- Need: sorting, filtering, pagination, CSV export
- Current: ng2-smart-table (abandoned) + custom logic
- **ACTION (Wk1):** Build test table; try sort/filter; check CSV export API
- **Fallback if needed:** Use Angular CDK DataSource + custom sorting/filtering + PipeTable or similar

### 🟡 To Verify (Lower Priority)
- Datepicker: need FHIR-compatible date (yyyy-MM-dd) + DOB calc
- Pagination: ZardUI pagination or keep ngx-pagination?
- Autocomplete: for beneficiary search (high-value UX)
- Toast: verify ngx-toastr compatible with Tailwind
- Stepper: for multi-step prescription + closure flows

**Recommendation:**

1. **Week 1 (this week):** Finalize table testing; decide CDK vs. ng2-smart-table vs. other
2. **Decision by end Wk1:** table strategy for Phase 2 Wk11–12
3. **If table gap confirmed:** plan ~4 days in Wk11 for custom CDK wrapper or ng2-smart-table compatibility check
4. **For other 🟡 components:** verify as components land (e.g., test datepicker when working on beneficiary-registration)

**Next Update:**
End of Week 1 — finalize findings and update blockers doc (B1) with resolution.
