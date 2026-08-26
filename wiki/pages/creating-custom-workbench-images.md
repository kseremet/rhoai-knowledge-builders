---
type: procedure
products: [rhoai]
versions: [rhoai-3.5]
sources:
  - raw/rhoai/3.5/managing-openshift-ai.md#L288-420
verified: 2026-08-25
confidence: documented
---

# Creating custom workbench images

A custom workbench image is a plain container image built with a
Containerfile/Dockerfile (`FROM` an existing image, add what's needed), for
cases where a data scientist needs OS packages, apps, or library versions
the [[rhoai-workbench|default workbench images]] don't provide — data
scientist users lack root access to install these ad hoc inside a running
workbench. Once built, it's made selectable in the dashboard via
[[importing-custom-workbench-images]].

**Support boundary**: Red Hat supports making a custom image available for
selection when creating a workbench; Red Hat does **not** support the
*contents* of the image. If it's selectable but doesn't produce a usable
workbench, that's not a supported-fix case.

Two starting points: build from one of the default RHOAI images, or build
from your own base image (with more compatibility guidelines to follow).

## Option 1: from a default OpenShift AI image

**Preconditions**: know which default image to base on (see
[[rhoai-workbench]] for the list; check package lists at
`https://access.redhat.com/articles/rhoai-supported-configs-3.x`); if the
result must be Elyra-compatible, the base image must itself contain the
Elyra extension; `cluster-admin` access to the OpenShift console.

1. Get the base image's exact pull location: OpenShift console →
   **Builds** → **ImageStreams** → project `redhat-ods-applications` → open
   the imagestream (e.g. `pytorch`) → **YAML** → under `spec:tags`, find the
   version tag you want → copy its `from:name` value, e.g.
   `quay.io/modh/odh-pytorch-notebook@sha256:b68e0192abf7d…`.
2. Create a Containerfile/Dockerfile.
3. `FROM` the copied location:
   ```
   FROM quay.io/modh/odh-pytorch-notebook@sha256:b68e0…
   ```
4. Optional — install OS packages (requires `USER 0`, then switch back):
   ```
   FROM quay.io/modh/odh-pytorch-notebook@sha256:b68e0…
   USER 0
   RUN INSTALL_PKGS="java-11-openjdk java-11-openjdk-devel" && \
       dnf install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
       dnf -y clean all --enablerepo=*
   USER 1001
   ```
   `dnf install` requires an active RHEL AI 3 subscription and an active
   RHEL EUS subscription with active RHEL packages.
5. Optional — add Python packages (as `USER 1001`):
   ```
   FROM quay.io/modh/odh-pytorch-notebook@sha256:b68e0…
   USER 1001
   COPY requirements.txt ./requirements.txt
   RUN pip install -r requirements.txt
   ```
6. Build and push the image, e.g. with Podman locally:
   ```
   podman build -t my-registry/my-custom-image:0.0.1 .
   podman push my-registry/my-custom-image:0.0.1
   ```
   Or use OpenShift's `BuildConfig` instead.

## Option 2: from your own image

Must be compatible with OpenShift and RHOAI's runtime assumptions.

### Basic guidelines

- **Run as `USER 1001`**: OpenShift runs containers with a random UID and
  GID `0`; design the image to work under that (especially for
  write-access directories) — best practice is `USER 1001`.
- **Don't place artifacts in `$HOME`**: the workbench's persistent volume
  mounts on `/opt/app-root/src`, which is also `$HOME` — anything placed
  there in the image is invisible once the PV is mounted at runtime.
- **Implement the `/api` endpoint**: OpenShift readiness/liveness probes
  query it. Jupyter IDEs have this by default; other IDEs must implement it
  themselves.

### Advanced guidelines

**Minimizing image size** — every `COPY`/`RUN` creates a new layer;
removing a file only masks it in the next layer, it doesn't shrink the
image:

- Prefer a newer base image over `dnf update`-ing an old one (e.g. a
  continuously-updated base like `ubi9/python-39` usually doesn't need
  `dnf update`). If you do use `dnf update`, it requires the same RHEL AI
  3 + RHEL EUS subscriptions as above.
- Chain `RUN` commands with `&& \`.
- If compiling code, use multi-stage builds so build artifacts don't end up
  in the final image.

**File/directory access**:

- Set ownership to `1001:0`, e.g. `COPY --chown=1001:0 os-packages.txt ./`
  (containers run with a random UID and GID `0`), or use
  `COPY --chmod=775 ...` to open access more broadly.
- `WORKDIR /opt/app-root/src` — this path is not configurable; it's where
  the user's persistent volume mounts as their home directory.
- Fix pip permissions in OpenShift environments (adjust the Python version
  as needed):
  ```
  chmod -R g+w /opt/app-root/lib/python3.11/site-packages && \
  fix-permissions /opt/app-root -P
  ```
- **Serve content from `/${NB_PREFIX}`**: routing is path-based via the
  Gateway API and uses the `NB_PREFIX` env var injected at runtime — a
  request to any other path (e.g. `/index.html`, `/`) will not reach the
  workbench container.
- **Answer at `${NB_PREFIX}/api`**: liveness/readiness probes depend on it;
  failing this gets the workbench pod deleted. Example Nginx config:
  ```
  location = ${NB_PREFIX}/api {
    return 302 /healthz;
    access_log off;
  }
  ```
- **For idle culling**: `${NB_PREFIX}/api/kernels` must return this JSON
  shape:
  ```
  {"id":"rstudio","name":"rstudio","last_activity":(ISO8601 time),"execution_state":"busy","connections": 1}
  ```

**Enabling CRB/EPEL** (repos with packages absent from a plain RHEL/UBI
install, e.g. needed for RStudio): CRB is enabled by default on UBI9. To add
EPEL on UBI9:
```
RUN yum install -y https://download.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```
To enable both CRB and EPEL on CentOS Stream 9:
```
RUN yum install -y yum-utils && \
   yum-config-manager --enable crb && \
   yum install -y https://download.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
```

## Next step

[[importing-custom-workbench-images]] — make the built image selectable in
the dashboard.
