# Login-Route-Performance-Optimization-Talawa-Admin
### Problem Summary
The login route currently initializes a significant portion of authenticated dashboard dependencies before user authentication. This results in unnecessary JavaScript transfer, premature network calls, and delayed first render for unauthenticated users.
### Lighthouse analysis of [test.talawa.io](https://test.talawa.io)
<img width="1163" height="350" alt="image" src="https://github.com/user-attachments/assets/77028b7f-233d-4cd1-9521-f4ca93b3b5b2" />

Lighthouse metrics show delayed FCP (~13s) and LCP (~25s) with relatively low TBT (~250ms), indicating that the primary bottleneck is network and bundle overhead rather than main-thread blocking.
Production network analysis (with cache enabled) shows:
- ~300 requests
- ~5MB transferred
- ~25MB total resource graph
- Dashboard-related modules loaded before authentication

This suggests insufficient isolation between public and authenticated application layers.
### Root Causes Identified
- Plugin system initializes on all routes.
- Apollo subscriptions are established at startup.
- ```CURRENT_USER``` query runs on public routes.
- Signup-related queries execute on login render.
- MUI barrel imports increase bundle size.
- Large translation files are loaded eagerly.
- Redundant ```useEffect``` patterns may trigger unnecessary re-renders.
## Optimization Strategy (Phased)
### 1. Bundle Hygiene
- Standardize debounce imports using ```lodash-es```.
- Convert MUI barrel imports to path imports to improve tree-shaking.
### 2. Architectural Isolation of Public Routes
- Defer plugin system initialization until entering authenticated routes.
- Initialize Apollo subscriptions post-login using ```client.setLink```.
- Skip ```CURRENT_USER``` query on public routes.
- Convert organization query to ```useLazyQuery``` (signup-only execution).
### 3. Runtime Simplification
- Remove refetch patterns tied to changing dependencies.
- Consolidate login-related useEffect logic.
- Ensure safe async handling to avoid unnecessary re-renders.
### 4. Translation Optimization
- Bundle critical English login strings.
- Lazy-load other namespaces and languages with caching.
- Preload only essential translation assets.
- Split and clean large translation files; remove unused keys.
  
# Proposed Timeline
The work will be executed in phased increments and validated progressively.
- Week 1–2: Bundle hygiene and public route isolation
- Week 3–4: Apollo, query, and runtime optimizations
- Week 5–6: Translation optimization and cleanup
- Final Weeks: Validation, benchmarking, regression testing, documentation, and refinements

# Expected Impact
- Reduced initial JavaScript payload on login routes
- Lower network request volume before authentication
- Improved FCP and LCP
- Cleaner separation between public and authenticated layers
- No regression in authenticated functionality
  
# Validation Plan
- Lighthouse comparison (before/after)
- Production bundle size analysis
- Network request comparison on login route
- Manual verification of login, signup, and dashboard flows
- Multi-language testing for translation changes
- Update and run relevant unit/integration tests where behavior changes (e.g., conditional queries, deferred initialization)
