# PE Domain 5: CI/CD, GitOps & Delivery — Scenario Quiz 1

Test your knowledge on CI architectures, ephemeral runners, and remote build caching.

---

### Question 1
A security audit discovers that multiple microservice CI jobs running on persistent, static self-hosted GitHub Actions runners were able to read files and Docker cache layers left behind by previous builds from different teams. What architectural solution completely resolves this vulnerability?

A) Disabling all CI/CD pipelines.  
B) Deploying Actions Runner Controller (ARC) in Kubernetes to provision single-use, ephemeral runner pods that are automatically destroyed and re-created after each job execution.  
C) Running all CI jobs as root.  
D) Asking developers to add `rm -rf /tmp/*` to the end of every workflow.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Static runners maintain local state and disk caches that can leak secrets and files between builds. Ephemeral runners (such as those managed by Actions Runner Controller) provide pristine, single-tenant container environments for every build, terminating the pod immediately upon completion.
</details>

---

### Question 2
In Docker BuildKit, what is the primary benefit of using cache mounts during compilation (`RUN --mount=type=cache,target=/root/.cache/go-build`)?

A) It allows the container to bypass all firewall rules.  
B) It compresses the final image size to under 1MB.  
C) It persists compilation and dependency caches across build invocations on the runner without embedding those cache files into the final container image layers, dramatically speeding up subsequent builds.  
D) It reboots the host kernel.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** BuildKit cache mounts map a persistent directory from the runner into the build container during execution. Compilers (Go, Rust, Maven) can reuse previously compiled object files across builds, drastically cutting build times without bloating the final image size.
</details>

---

### Question 3
An engineering team uses a monorepo containing 50 microservices. Currently, every pull request triggers CI builds for all 50 services, resulting in massive CI queue congestion and 45-minute build times. What is the most effective platform solution?

A) Disable automated tests on pull requests.  
B) Force developers to split the monorepo into 50 polyrepos.  
C) Upgrade all laptops to 128GB of RAM.  
D) Implement change detection tooling (e.g., Nx, Turborepo, or path filtering in GitHub Actions `on.pull_request.paths`) combined with distributed remote build caching so that only the modified services and their affected dependencies are rebuilt and tested.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** Modern monorepo tooling utilizes change detection (graph dependency analysis) and path filters to ensure CI only builds and tests components affected by the incoming changes. Paired with distributed remote caching, unchanged modules are fetched from cache in seconds.
</details>

---

### Question 4
What is the function of a **Merge Queue** (e.g., GitHub Merge Queue) in high-throughput engineering organizations?

A) It converts Go code to Python automatically.  
B) It prints pull requests on physical paper.  
C) It ensures that pull requests are tested against the latest version of the `main` branch in a speculative queue before merging, preventing "semantic conflicts" where two independently passing PRs break `main` when merged simultaneously.  
D) It deletes all pull requests older than 24 hours.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** When multiple developers merge PRs concurrently, each PR was tested against a stale `main`. While neither PR has a syntactic Git conflict, combining them can break the build. A Merge Queue tests PRs speculatively in sequence against the cumulative tip of `main`, guaranteeing that the branch never breaks.
</details>

---

### Question 5
Why does Trunk-Based Development lead to higher DORA delivery throughput compared to traditional GitFlow?

A) GitFlow relies on long-lived feature and release branches that accumulate massive, high-risk merge conflicts and batch releases; Trunk-Based Development encourages small, continuous merges into `main` paired with feature flags, enabling fast feedback and frequent production deploys.  
B) GitFlow requires purchasing expensive proprietary software.  
C) Trunk-Based Development eliminates the need for code reviews.  
D) GitFlow does not work with Git.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** GitFlow creates large batches of unintegrated work across long-lived branches, delaying releases and increasing Change Failure Rate. Trunk-Based Development promotes small, frequent commits to a single trunk, keeping integration continuous and risk low.
</details>

---

### Question 6
What is the primary risk of using the `latest` tag on container images in CI/CD pipelines?

A) The `latest` tag consumes 10x more network bandwidth.  
B) It prevents images from being signed.  
C) Docker will reject the image.  
D) The `latest` tag is non-deterministic and mutable; deployments lack traceability, rollbacks cannot reliably identify previous image versions, and Kubernetes `imagePullPolicy: IfNotPresent` may fail to pull new code on nodes that already cached the tag.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** D  
**Explanation:** `latest` is an anti-pattern in production. It is a mutable pointer. Two pods running `latest` on different nodes may actually run completely different code. Production deployments must use immutable identifiers: unique Git commit SHAs, semantic versions, or cryptographic image digests (`sha256:...`).
</details>

---

### Question 7
In GitHub Actions, how does using OpenID Connect (OIDC) federation improve security when deploying to AWS compared to storing static `AWS_ACCESS_KEY_ID` secrets?

A) OIDC bypasses AWS IAM policies.  
B) OIDC eliminates static, long-lived AWS credentials from GitHub repository settings by exchanging short-lived, cryptographically signed JWT tokens for temporary AWS STS credentials valid for 1 hour.  
C) OIDC speeds up builds by 500%.  
D) OIDC encrypts all network packets with quantum encryption.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** B  
**Explanation:** Static access keys stored in GitHub Secrets are frequently leaked or forgotten and never rotated. OIDC allows the GitHub Actions runner to authenticate directly to AWS IAM using short-lived tokens, eliminating secret storage and credential rotation toil.
</details>

---

### Question 8
What does the term **"Hermetic Build"** mean in advanced CI systems (such as Bazel)?

A) A build that executes in complete isolation from the host system and internet, taking only explicitly declared inputs and compilers, guaranteeing that identical source code produces bit-for-bit identical, reproducible binary outputs.  
B) A build that runs underwater.  
C) A build that does not compile code.  
D) A build that only runs on physical hardware.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Hermetic builds have zero undeclared dependencies and no internet access during compilation. This guarantees absolute reproducibility: building the same Git commit today or 5 years from now produces the exact same cryptographic binary.
</details>

---

### Question 9
In CI pipelines, what is a **"Flaky Test"**, and why is it toxic to developer velocity?

A) A test that nondeterministically passes and fails on the exact same commit without code changes, destroying developer trust in the CI suite and causing developers to ignore legitimate test failures.  
B) A test that takes longer than 5 seconds.  
C) A test written in Python.  
D) A test that tests UI styling.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** A  
**Explanation:** Flaky tests (often caused by concurrency race conditions, sleep timers, or external network dependencies) erode engineering trust. Developers re-run builds repeatedly in hopes of passing, wasting runner minutes and leading teams to bypass CI quality gates.
</details>

---

### Question 10
What is the primary difference between Unit Tests and Integration Tests in the deployment pipeline?

A) Unit tests can only run in production.  
B) Unit tests are written by managers; integration tests are written by developers.  
C) Unit tests test small, isolated functions in memory with dependencies mocked (sub-second execution); integration tests test the interaction between real components (databases, network APIs, queues) to verify end-to-end functionality.  
D) Integration tests must run before unit tests.  

<details>
<summary><strong>View Answer & Explanation</strong></summary>

**Correct Answer:** C  
**Explanation:** Unit tests provide fast, local, fine-grained feedback on logic. Integration tests validate that real systems (e.g., ORM interacting with a PostgreSQL database container) communicate properly.
</details>
