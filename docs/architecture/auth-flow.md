# Authentication & Login Flow
> Source: Claude Code codebase analysis, Jun 10, 2026  
> Codebase: Helpline104-UI, branch main, commit df5db19

## Overview

The app uses legacy @angular/http with all auth traffic routed through two 
custom Http subclasses. Base URLs come from environment 
(login/auth = commonAPI, e.g. http://localhost:8083/).

## Login Sequence

1. **Submit** (login/login.html:8) — form binds userID/password; 
   submit calls login(false) (arg is doLogOut)

2. **Password encryption** (login.component.ts:180-208) — crypto-js AES with 
   PBKDF2-derived key (SHA-512, keySize 256, 1989 iterations), random per-call 
   IV+salt, hardcoded passphrase Key_IV = "Piramal12Piramal"
   Transmitted string = salt + iv + ciphertext

3. **Auth request** (loginService/login.service.ts:61-88) — POST 
   commonAPI + user/userAuthenticate with 
   {userName, password, withCredentials, doLogout, [captchaToken]}

4. **Response handling** (login.component.ts:266-358) — filters previlegeObj 
   to entries where serviceName == "104"; if none → "User doesn't have 
   privilege to access 104". Stores user data into dataService, branches on Status:
   - "Active" → store token, fetch CTI login token, navigate to /MultiRoleScreenComponent
   - "New" → navigate to /setQuestions (first-time setup)

5. **CTI token** (czentrix.service.ts:211-219) — POST commonAPI + cti/getLoginKey, 
   stores login_key in memory

## Token & Session Persistence

- authToken stored in sessionStorage (unencrypted) — sent as Authorization header
- apiman_key set during role selection, appended as ?apikey= to every request URL
- sessionStorageService AES-encrypts values it writes, BUT encKey='' locally 
  (trivially reversible)
- ⚠️ AuthService.isAuthenticated() always returns true — real check is commented out

## Route Guards

- **AuthGuard** — allows navigation only if authToken exists and onCall !== 'yes'
- **AuthGuard2** — "must be on active call" gate (onCall === 'yes' OR Blood Request)
  Protects: RedirectToInnerpageComponent, InnerpageComp, Closure
- **SaveFormsGuard** (CanDeactivate) — only lets you leave call forms while on a call

## Role Model

After Active login → MultiRoleScreenComponent → ServiceRoleSelectionComponent.
route2dashboard() sets apiman_key, maps role to dashboard:

| Role | Dashboard |
|------|-----------|
| Registration | RO |
| Health_Advice | HAO |
| Counselling | CO |
| Medical_Advice | MO |
| Service_Improvements | SIO |
| Supervising | Supervisor |
| Psychiatrist | PD |
| Surveyor | Surveyor |

## Other Auth Flows

- **Captcha:** Cloudflare Turnstile, gated by environment.enableCaptcha (false locally)
- **Forgot password:** user/forgetPassword → security questions → 
  user/validateSecurityQuestionAndAnswer → /setPassword → user/setForgetPassword
  Password policy: 8-12 chars, 1 digit, 1 uppercase, 1 special
- **Concurrent login (5002):** interceptor shows confirm dialog → 
  user/logOutUserFromConcurrentSession → re-auth with doLogout=true
- **Session expiry (401/403):** interceptor alerts, clears sessionStorage, 
  redirects to login
- **Logout:** cti/doAgentLogout → user/userLogout → removeToken() → 
  navigate to /feedback?sl=104
- **Sockets:** effectively disabled — io(url) commented out in socket.service.ts

## Security Issues (To Fix in Migration)

| Issue | Severity | Location | Plan |
|-------|----------|----------|------|
| authToken stored unencrypted in sessionStorage | High | login.component.ts:319 | Phase 1 Wk2 — document; fix post-internship |
| apiman_key stored unencrypted | High | service-role-selection.component.ts:128 | Same |
| encKey = '' (empty encryption key) | High | environment.local.ts | Same |
| isAuthenticated() always returns true | High | auth.service.ts:51 | Same |
| console.error logs auth token every request | Critical | http.securityinterceptor.ts:170 | Fix Wk2 |
