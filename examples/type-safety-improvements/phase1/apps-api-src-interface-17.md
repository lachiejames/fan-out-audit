# Audit: apps-api-src-interface-17

**Findings count**: 1
**Summary**: One unavoidable cast in the user controller where the NestJS `FileInterceptor` does not type `req.file` on `AuthenticatedRequest`. All other files in this slice are clean.

---

## Finding 1

**File**: `apps/api/src/interface/http/user/user.controller.ts`
**Line**: 189
**Category**: Type cast (`as`)
**Impact**: Low — the cast is required because `FileInterceptor` populates `req.file` at runtime but the `AuthenticatedRequest` type does not include the `file` property. Without a cast or type extension the compiler would reject the access.

**Description**: `req.file as Express.Multer.File` is used to read the uploaded avatar file from the request. The `FileInterceptor` decorator from `@nestjs/platform-express` mutates the Express request object to attach `file`, but this is not reflected in NestJS's `AuthenticatedRequest` interface.

**Suggestion**: Extend `AuthenticatedRequest` (or create a local `FileUploadRequest`) to include `file?: Express.Multer.File`, eliminating the cast.

**Evidence**:

```typescript
const file = req.file as Express.Multer.File;
```
