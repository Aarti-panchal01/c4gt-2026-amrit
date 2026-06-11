# Helpline104-UI vs Helpline1097-UI — Codebase Comparison
> Source: Claude Code analysis, Jun 11, 2026  
> Purpose: Scope shared @amrit/common-ui library if monorepo is confirmed

## At a Glance

| | Helpline104-UI | Helpline1097-UI |
|--|--|--|
| Purpose | Medical-advice clinical helpline | Counselling/grievance helpline |
| Angular | 4.1.3 | 4.4.4 |
| CLI/build | 1.2.6, .angular-cli.json, WAR via pom.xml | identical |
| Components | ~118 | ~94 |
| Services | ~67 | ~50 |
| Templates | 138 | 98 |
| Extra deps | html2canvas, file-saver, return-deep-diff | angular2-text-mask, jspdf, ng2-cookies |
| Auth approach | custom Http subclass only | also has token.interceptor.ts (newer, closer to Angular 20) |

## Shared Code (Both Repos)

### Byte-identical or trivially different (strongest Common-UI candidates)
- loader.service.ts (0 diff lines)
- loader.component.ts (0 diff lines)
- utc-date.pipe.ts (2 diff lines)
- listner.service.ts (3 diff lines)
- session-storage.service.ts (4 diff lines)

### Same code, lightly diverged (config/URLs only)
- config.service.ts (58 diff lines — just different base URL sets)
- auth.service.ts (22 diff lines — 104 adds cZentrixLogout)
- http.factory.ts (49 diff lines)

### Same concept, heavily diverged implementation
- http.interceptor.ts (484 diff lines)
- czentrix.service.ts (548 diff lines — 104 has more CTI methods)
- callservice.service.ts (204 diff lines)
- confirmation.service.ts (135 diff lines)

### Shared feature areas (present in both)
- Auth & session: login, resetPassword, set-password, set-security-questions,
  captcha, multi-role-screen, service-role-selection, both auth guards
- Dashboard shell: dashboard, dashboard-navigation/-row-header/-user-id,
  innerpage, loader, app.component
- Telephony/CTI: czentrix, socket, dial-beneficiary, agent-status, force-logout
- Admin: admin-language/role/screen/service-provider/user, super-admin
- Supervisor: supervisor-calltype-reports/-configurations/-notifications/
  -emergency-contacts/-grievance/-reports/-training-resources, quality-audit, 
  sms-template, block-unblock-number
- Outbound: outbound-allocate/search-records, outbound-worklist, reallocate-calls
- Common dialogs: common-dialog, message-dialog, notifications-dialog,
  emergency-contacts-view-modal

## What's Different

### 104-only (clinical)
- case-sheet + modals, cdss/algo-component, prescription, snomed
- covid-19/screening/bp/diabetic
- Medical roles: 104-mo, 104-hao, 104-pd, 104-ro, 104-surveyor
- SIO services: blood-on-call, organ-donation, epidemic-outbreak, 
  food-safety, scheme, bal-vivah

### 1097-only (counselling/grievance)
- Everwell TB-program integration (everwell-*)
- Grievance outbound workflow
- Demographic reports (gender-distribution, sexual-orientation, 
  caller-age, language-distribution)
- Weather-warnings, updates-from-beneficiary, beneficiary-history
- Newer token-based auth layer (token.interceptor.ts)

## Proposed Common-UI Library Tiers

### Tier 1 — Lift as-is (near-identical today)
- LoaderService + LoaderComponent
- ListnerService
- SessionStorageService
- utc-date pipe
- Validator directives (password, mobile, email, name, address)
- Common dialogs (confirm/alert/message/notification)

### Tier 2 — Lift with parameterization
- ConfigService → make base URLs an injected AppConfig token
- HTTP interceptor layer → one shared functional interceptor package
  (start from 1097's token.interceptor.ts — closer to Angular 20 target)
- AuthService / login / guards / multi-role-screen / service-role-selection

### Tier 3 — Shared interface, app-specific implementation
- CzentrixServices (define common CtiService interface + shared DTOs)
- Admin, supervisor-reports, outbound modules (shared scaffolding,
  app-specific columns/endpoints)

### Tier 4 — Keep in each app (do not share)
- 104: case-sheet, CDSS, prescription, SNOMED, SIO services
- 1097: Everwell, grievance, demographic reports, weather, counselling

## Key Finding

1097's token.interceptor.ts is already closer to Angular 20's functional 
interceptor pattern than 104's Http-subclass approach. If monorepo is confirmed, 
start the shared HTTP layer from 1097's code, not 104's.

> ⚠️ Status: Monorepo decision PENDING — confirm with Dr. Mithun on Jun 15.
> Do not start writing shared code until confirmed.
