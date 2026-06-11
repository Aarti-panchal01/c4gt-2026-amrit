# Services Catalog — All 67 Services
> Source: Claude Code codebase analysis, Jun 10, 2026

## Base URL Map (config.service.ts)

| Key | Port | Used For |
|-----|------|----------|
| ip104 | 8081 | 104 clinical/casesheet/SIO/reports |
| commonAPI | 8083 | call/CTI, user, feedback, notification, location, SMS |
| adminAPI | 8082 | provider/role/location/MMU |
| tmAPI | 8089 | teleconsult |
| fhirAPI | 8093 | ABHA/Health ID |
| telephoneServer | — | legacy C-Zentrix |
| ip1097 | 8090 | 1097 counsellor workflow |

## Services by Folder

### adminServices
- **LanguageService** — get/save language masters ⚠️ hardcoded localhost:8080
- **RoleService** — get/save role masters ⚠️ hardcoded localhost:8080
- **ScreenService** — get/save screen masters ⚠️ hardcoded localhost:8080
- **ServicemasterService** — service CRUD ⚠️ hardcoded localhost:8080
- **SPService** — provider CRUD → adminAPI
- **UserService** — admin users ⚠️ hardcoded 10.152.3.152:1040

### authGuardService
- **AuthGuard** — route guard: requires authToken, blocks mid-call back-nav
- **AuthGuard2** — route guard: requires active call (onCall=yes)
- **SaveFormsGuard** — CanDeactivate: prevents leaving call forms mid-call

### authentication
- **AuthService** — token get/remove, isAuthenticated (⚠️ always true), 
  cZentrix logout → commonAPI

### callTypeReports
- **CallTypeReportService** — beneficiary/get/services → ip104

### callservices
- **CallServices** — core call lifecycle: storeCallID, closeCall, getCallTypes,
  getOutboundCallList, block/unblock numbers, recordings, blacklist 
  → commonAPI + ip104

### captcha-service
- **CaptchaService** — injects Cloudflare Turnstile script

### caseSheetService
- **CaseSheetService** — case sheets, COVID vaccination, categories, disease 
  lookup, HIHL counselling, institute, SNOMED diagnosis search 
  → commonAPI + ip104 + ip1097
- **OtherHelplineService** — MCTS/TM/MMU sister-helpline casesheets

### cdssService
- **CDSSService** — CDSS/saveSymp, getChiefComplaints, getQuestions, getAnswer 
  → ip104

### coService
- **CoCategoryService** — active CO category/subcategory → ip1097 + commonAPI
- **coCategoryService** — legacy duplicate, stubbed
- **CoFeedbackService** — CO feedback + IMR/MMR mortality workflow
- **CoReferralService** — institute directory referrals

### common
- **AvailableServices** — beneficiary/get/services
- **CallerService** — caller↔beneficiary linkage, wrap-up time
- **FeedbackTypes** — feedback masters
- **ListnerService** — in-app RxJS bus for CTI events (no API)
- **LoaderService** — spinner signal ⚠️ no ref-counting (fix in Wk2)
- **OutboundListnerService** — outbound event bus
- **SessionService** — stubbed
- **SmartsearchService** — client-side type-ahead
- **UserBeneficiaryData** — registration master + alt phone
- **UtilityService** — age/DOB/UTC helpers
- **ValidationUtils** — drug-duration validation (not injectable)

### config
- **ConfigService** — all base URLs; no HTTP calls

### covidService
- **CovidserviceService** — COVID screening master + save → ip104

### czentrix
- **CzentrixServices** — full CTI surface: agentLogin/Logout, dial/disconnect/
  transfer, campaigns/skills, switch inbound/outbound, agent stats, login key 
  → commonAPI cti/* + legacy telephoneServer

### dataService
- **dataService** — app-wide shared state + cross-component RxJS Subjects; no API

### dialog
- **ConfirmationDialogsService** — Material dialog wrapper: confirm/alert/remarks

### http-services
- **HttpServices** — generic get/post helpers + app-language BehaviorSubject 
  + commit details

### loginService
- **loginService** — auth lifecycle: authenticate, concurrent-logout, 
  security questions, service-provider lookup, API version 
  → commonAPI + adminAPI + ip104

### notificationService
- **NotificationService** — alerts/notifications + emergency contacts + 
  roles/offices/users/designations → commonAPI + adminAPI

### outboundServices
- **OutboundCallAllocationService** — allocate calls
- **OutboundReAllocationService** — reallocate, move-to-bin
- **OutboundSearchRecordService** — search unallocated + role-screen mappings
- **OutboundWorklistService** — worklist + blood-bank save

### prescriptionServices
- **PrescriptionService** — drug masters: groups/list/strength/frequency, 
  save/fetch prescriptions → ip104

### register-services
- **RegisterService** — beneficiary registration + emergency/demographic flags 
  ⚠️ hardcoded 10.152.3.152:1040

### report-services
- **ReportService** — CRM report Blob downloads across ~25 domains: 
  RO/HAO/MO/CO/PD summaries, epidemic, food safety, blood, organ, grievance, 
  directory, schemes, prescription, quality, complaint, district-wise, 
  mental-health, medical-advise, CDI + filter masters → ip104 + commonAPI

### screening
- **DiseaseScreeningService** — NCD questions/answers + question types 
  → ip104 + commonAPI

### searchBeneficiaryService
- **SearchService** — beneficiary search/register/update, locations, facility 
  master, appointments, ABHA/Health ID → commonAPI + ip104 + adminAPI + fhirAPI

### sessionStorageService
- **sessionStorageService** — AES sessionStorage wrapper + cookie helpers 
  ⚠️ encKey='' locally

### sioService
- **SioService** — legacy blood request
- **BalVivahServiceService** — child-marriage complaints
- **BloodOnCallServices** — blood-on-call requests + bank URLs
- **EpidemicServices** — epidemic outbreak complaints
- **FoodSafetyServices** — food-safety complaints
- **OrganDonationServices** — organ-donation requests
- **SchemeService** — govt health schemes NKSHP → commonAPI

### snomedService
- **SnomedService** — snomed/getSnomedCTRecord → ip104

### socketService
- **SocketService** — socket.io wrapper ⚠️ hardcoded 10.208.122.38:4000, 
  constructor commented out, not @Injectable

### supervisorServices
- **FeedbackService** — grievance management + email
- **ForceLogoutService** — force-logout supervisors/agents
- **QualityAuditService** — call-quality audit + casesheet detail + audio
- **SmsTemplateService** — SMS template CRUD + send
- **SupervisorCallTypeReportService** — paginated call filtering
- **SupervisorDiseaseSummaryService** — disease-summary CRUD

### surveyorServices
- **SurveyorReportsService** — CDI QA mappings → ip104

### upload-services
- **UploadServiceService** — kmfilemanager/addFile → commonAPI
