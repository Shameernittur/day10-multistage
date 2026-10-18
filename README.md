# DevOps Winter Arc — Day 10: Multi-Stage Builds

Optimizing Go container images by decoupling build toolchains from runtime environments, enforcing least-privilege execution, and evaluating base-image trade-offs.

---

## 1. Measured Image Size Comparison

| Image Variant | Base Image | Size | Reduction vs. Single |
| :--- | :--- | :--- | :--- |
| `demo:single` | `golang:1.22` | ~850 MB | Baseline (0%) |
| `demo:multi` | `alpine:3.20` | ~13 MB | **~98.4% reduction** |
| `demo:scratch` | `scratch` | ~7 MB | **~99.1% reduction** |

*Formula used:* `((Single Size - Multi Size) / Single Size) * 100`

---

## 2. Security: Least Privilege Execution

Running container processes as `root` (`uid=0`) exposes the host kernel to severe escalation risks if an application vulnerability is compromised. 
In `Dockerfile`:
- Created a system group and user: `RUN addgroup -S app && adduser -S -G app app`
- Transferred ownership of the binary: `COPY --from=builder --chown=app:app ...`
- Dropped runtime privileges: `USER app`
- Verified via `docker exec demo id`: Shows execution under `uid=100(app)`, not `root`.

---

## 3. Interview Readiness & Reflection

### 1. What is a multi-stage build and why use it?
It uses multiple `FROM` instructions in a single Dockerfile. Compilers, development header files, and source code stay isolated in early builder stages. Only the compiled production artifact is copied to the final minimal runtime image, reducing attack surface, storage, and transfer latency.

### 2. How do you copy files between stages?
Using the `COPY --from=<stage_name>` instruction (e.g., `COPY --from=builder /out/server /app/server`).

### 3. What are the trade-offs of `scratch` vs `alpine`?
- **`scratch`:** Minimal possible image size and zero extraneous software packages (nearly zero CVEs). However, it has no shell, making debugging via `docker exec` impossible and requiring entirely static compilation (`CGO_ENABLED=0`).
- **`alpine`:** Small footprint (~5 MB base), includes a basic POSIX shell (`sh`) and package manager (`apk`), making runtime troubleshooting and health check scripts straightforward.

---

## 4. Verification Evidence

- **Image Size Metrics:** `evidence/image-sizes.png`
- **Application Routing & Health:** `evidence/app-health.png`
- **Non-Root Verification:** `evidence/non-root-user.png`
- **Builder Stage Isolation:** `evidence/builder-target.png`
