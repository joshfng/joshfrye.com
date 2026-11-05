# Go-Live Checklist: joshfrye.com

This guide outlines the steps to deploy the Jekyll site to GitHub Pages on the custom apex domain `joshfrye.com` with SSL enabled.

## 1. Repository Preparation

- [x] **Create CNAME File**: Ensure a file named `CNAME` exists at the root of the repository.
  - Content must be exactly: `joshfrye.com` (no `www`, no `https://`).
- [x] **Commit and Push**:
  ```bash
  git add CNAME
  git commit -m "Add CNAME for joshfrye.com"
  git push origin main
  ```

## 2. GoDaddy DNS Configuration

Log into the GoDaddy Control Panel, navigate to DNS Management for `joshfrye.com`, and apply the following changes:

- [x] **Configure Apex Domain (A Records)**:
      Remove any existing `A` records for `@` pointing to old hosts, and add the following four `A` records:

  | Type | Name | Value             | TTL                 |
  | :--- | :--- | :---------------- | :------------------ |
  | A    | `@`  | `185.199.108.153` | 1 Hour (or Default) |
  | A    | `@`  | `185.199.109.153` | 1 Hour (or Default) |
  | A    | `@`  | `185.199.110.153` | 1 Hour (or Default) |
  | A    | `@`  | `185.199.111.153` | 1 Hour (or Default) |

- [x] **Configure Subdomain Redirect (CNAME Record)**:
      Ensure `www` traffic is routed to GitHub Pages so GitHub can redirect it to the apex domain:

  | Type  | Name  | Value                | TTL                 |
  | :---- | :---- | :------------------- | :------------------ |
  | CNAME | `www` | `joshfng.github.io.` | 1 Hour (or Default) |

## 3. GitHub Pages Settings

Once DNS records are updated (propagation can take up to 24 hours, but is usually fast):

- [x] **Configure Custom Domain**:
  1. Go to your GitHub repository: `joshfng/joshfrye.com` (or matching repo name).
  2. Navigate to **Settings** > **Pages**.
  3. Under **Build and deployment**, select the source branch (e.g., `main`).
  4. Under **Custom domain**, type `joshfrye.com` and click **Save**.
- [x] **Enforce HTTPS**:
  1. Wait for the DNS check to pass. GitHub will automatically request an SSL certificate via Let's Encrypt.
  2. Check the **Enforce HTTPS** checkbox once it becomes available (this can take from a few minutes up to a couple of hours).

## 4. Verification

- [x] **Verify DNS Records**:
  ```bash
  dig joshfrye.com +short
  # Should return the four 185.199.x.x IPs
  ```
- [x] **Verify HTTP -> HTTPS Redirect**:
  - `http://joshfrye.com` -> `https://joshfrye.com`
- [x] **Verify Subdomain Redirect**:
  - `http://www.joshfrye.com` -> `https://joshfrye.com`
  - `https://www.joshfrye.com` -> `https://joshfrye.com`
