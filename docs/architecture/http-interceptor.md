# HTTP Interceptor Pattern
> Source: Claude Code codebase analysis, Jun 10, 2026

## Why the Custom Pattern Exists

Angular 4's @angular/http has no HttpInterceptor (that came with HttpClient 
in 4.3+). The only way to add cross-cutting behavior is to subclass Http, 
override verb methods, and substitute via a DI factory. This app does it twice.

## Current Implementation
http.factory.ts          → creates InterceptedHttp
http.security.factory.ts → creates SecurityInterceptedHttp
http.interceptor.ts      → InterceptedHttp class (extends Http)
http.securityinterceptor.ts → SecurityInterceptedHttp class (extends Http)
Provider in app.module.ts:
```typescript
{ provide: InterceptedHttp, useFactory: httpFactory,
  deps: [XHRBackend, RequestOptions, LoaderService, Router, 
         AuthService, ConfirmationDialogsService, SocketService] }
```

## Per-Request Pipeline (each verb override)

1. updateURL() — append ?apikey=<apiman_key> if present
2. networkCheck() — offline guard, returns Observable.empty()
3. showLoader() → loaderService.show()
4. getRequestOptionArgs() — append Content-Type: application/json 
   + Authorization: <token>
5. .catch(onCatch) → re-throw
6. .do(onSuccess, onError) — side effects
7. .finally(onEnd) → hideLoader()

## Error Handling

- **onSuccess:** inspects body envelope — statusCode 5002 → force-logout branch
- **onError:** 401/403 = "session expired" alert + sessionStorage.clear() + redirect
- 500 and others propagate unhandled

## Two Interceptors Compared

| | InterceptedHttp | SecurityInterceptedHttp |
|--|--|--|
| Loader | shows/hides spinner | silent (no loader) |
| 401/403 | alerts + clears + redirects | just returns error |
| 5002 handling | confirm-dialog + BehaviorSubject | single clear-and-redirect |
| Force-logout Subject | yes | no |
| Role | public-facing UX | silent workhorse (injected almost everywhere) |

## Known Bugs

- **deps/factory arg mismatch:** sessionStorageService omitted from factory 
  deps array; socketService/sessionstorage are swapped. Only works because 
  those deps aren't used in live paths.
- **No ref-counting in LoaderService:** first response hides spinner even 
  with other requests in flight.
- **request() not overridden:** direct .request() calls bypass interception entirely.
- **Token logged:** console.error("authTkn", authTkn) at 
  http.securityinterceptor.ts:170 logs auth token on every request. ⚠️ CRITICAL

## Angular 20 Replacement

```typescript
// One functional interceptor replaces both classes
export const appInterceptor: HttpInterceptorFn = (req, next) => {
  const isSilent = req.context.get(IS_SILENT_REQUEST);
  const token = sessionStorage.getItem('authToken');
  const apiKey = sessionStorage.getItem('apiman_key');
  
  // Add auth headers
  const authReq = req.clone({
    setHeaders: { Authorization: token ?? '' },
    url: apiKey ? req.url + '?apikey=' + apiKey : req.url
  });

  if (!isSilent) loaderService.show();

  return next(authReq).pipe(
    tap(event => { /* handle 5002 */ }),
    catchError(err => { /* handle 401/403 */ }),
    finalize(() => { if (!isSilent) loaderService.hide(); })
  );
};
```

Full reference implementation to be added after Week 2 completion.
