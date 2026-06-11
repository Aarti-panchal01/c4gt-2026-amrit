# End-to-End Call Flow
> Source: Claude Code codebase analysis, Jun 10, 2026

## Overview

Telephony is C-Zentrix (CTI), integrated via backend cti/* endpoints plus 
an embedded softphone iframe (bar/cti_handler.php). Screen pop is driven by 
browser window message events from the iframe. Shared state lives in 
dataService + sessionStorage (CLI, session_id, onCall).

## Flow Steps

### 1. Login → Role → Dashboard
- Agent picks 104 sub-role; CTI bar iframe is rendered (hidden for Supervisor)
- dashboardUserId.component.ts:51 polls cti/getAgentState every 15s for display
- Agent goes Ready/Not-Ready in C-Zentrix iframe (app doesn't call a "ready" API)

### 2. Incoming Call → Screen Pop
Two converging triggers:

**CTI message event** (dashboard.component.ts:231-315):
- event.data format: Accept|CLI|session_id|INBOUND (pipe-split)
- Guards against duplicate/foreign sessions
- Sets onCall=yes, stores CLI/session_id, navigates to RedirectToInnerpageComponent

**getAgentState poll detecting INCALL/CLOSURE:**
- Detects new session_id
- Reads dialer_type: PROGRESSIVE→INBOUND, PREVIEW→OUTBOUND

**Backend call capture:**
- storeCallID() → POST call/startCall, returns benCallID cached app-wide

### 3. Beneficiary Identification
- **Inbound auto-lookup:** POST call/beneficiaryByCallID; if linked, 
  fetches ABHA/Health ID via healthID/getBenhealthID (FHIR)
- **RO manual:** beneficiary/searchBeneficiary / searchUserByPhone; 
  selecting links the call (beneficiary/update/beneficiaryCallID)
- **New registration:** deep registerObj → beneficiary/create

### 4. Innerpage
- innerpage.component.ts shows service/role/live call status + duration timer
- Body renders 104.component.html (role switchboard):
  - RO+INBOUND → app-104-ro
  - RO+OUTBOUND → app-104-hao
  - CO → app-104-co
  - MO → app-104-mo
  - (etc. per role)
- Runs role-based wrap-up timer that auto-closes call on custdisconnect

### 5. Case-Sheet (Clinical Workhorse)
- case-sheet.component.ts captures patient, Chief Complaint vs Disease Summary
- Invokes CDSS: CDSS/getQuestions → modal → CDSS/getResult returns 
  probability-ranked disease table
- Diagnoses enriched with SNOMED-CT concept IDs (snomed/getSnomedCTRecord)
- Includes COVID module + prescription dialog
- Save → POST beneficiary/save/benCaseSheet
- Success fires serviceAvailed.next(true) — required to mark call "Valid" at closure

### 6. Closure
- closure.component.ts (carousel slide 2) loads call types (call/getCallTypesV1)
- Validates: can't mark Valid without beneficiary/service availed
- Supports: follow-up scheduling, referral, feedback flag, transfer
- Transfer: cti/transferCall / emergency 102/108 conference dial
- Submit → POST call/closeCall (with endCall=true for final close)
- Outbound also calls call/updateOutboundCall

### 7. Outbound Differences
- Mode set via cti/switchToOutbound or auto-detected PREVIEW dialer
- Worklist via call/outboundCallList
- Dialing via cti/callBeneficiary
- Uses known outboundBenID instead of caller-ID lookup

## Important Notes
- Dashboard socket subscriptions are entirely commented out
- Live agent state is owned by the CTI iframe
- App only polls for display + screen-pop
