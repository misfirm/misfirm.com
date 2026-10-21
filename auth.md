# Authentication & Access Control

Authentication and access model for [misfirm.com](https://misfirm.com), the MIS Solutions website, whose source lives in [misfirm/misfirm.com](https://github.com/misfirm/misfirm.com).

## Summary

misfirm.com is a **static, public, single-page site** (`index.html`, `styles.css`, `script.js`, `assets/`, `favicon.svg`) served by GitHub Pages. It has:

- **No user accounts, login, or sessions**
- **No backend, API, or database**
- **No server-side secrets**

There is no application-level authentication to configure. "Auth" for this project means **who can change the site and the infrastructure it depends on**. Those controls are documented below.

## Access Layers

| Layer | System | Who/what needs access | Control |
|---|---|---|---|
| Source code | GitHub repo `misfirm/misfirm.com` (public) | Site maintainers | GitHub account + MFA; repo permissions |
| Publishing | GitHub Pages (`main` branch, repo root) | Anyone with push/merge rights to `main` | Branch protection on `main` |
| Custom domain | `CNAME` file in repo + GitHub Pages settings | Repo admins | Repo admin role only |
| DNS | AWS Route 53 hosted zone for `misfirm.com` | DNS administrators | AWS IAM, MFA, least privilege |
| Domain registration | Domain registrar account | Domain owner | Registrar account MFA, registrar lock |

## Public vs. Restricted

- **Public (intentionally):** the website content and the repository. Anyone can read the site and the source, and anything committed is world-readable permanently, including history.
- **Restricted:** write access to the repo, Pages settings, Route 53, and the registrar account.

## GitHub

- Enable **two-factor authentication** for every account with write access, and require it at the organization/account level where available.
- Grant the minimum role needed (Read, Triage, Write, Maintain, Admin). Keep Admin to the owner(s).
- Protect `main`:
  - Require pull requests before merging (or restrict direct pushes)
  - Require status checks if CI is added
  - Block force pushes and branch deletion
- Because `main` is the publish branch, **a merge to `main` is a production deploy**.
- Prefer SSH keys or fine-grained personal access tokens with a short expiry and only the `misfirm/misfirm.com` repository scope. Avoid classic tokens with broad `repo` scope.
- Review collaborators and deploy keys periodically and remove anything unused.

## GitHub Pages and HTTPS

- Publishing source: `main` branch, repository root. `.nojekyll` disables Jekyll processing.
- The custom domain is declared in `CNAME`. Do not change or delete it without a coordinated DNS change.
- Keep **Enforce HTTPS** enabled in Settings → Pages so the site is only served over TLS (certificate is managed by GitHub).
- The site has no cookies or login state, so there are no session or CSRF concerns.

## DNS (AWS Route 53)

- The apex domain uses A records pointing to GitHub Pages IPs; `www` is a CNAME to `misfirm.github.io`.
- Restrict Route 53 changes to a dedicated IAM role or user with least privilege, with MFA required. Do not use the AWS root account for routine work.
- Enable CloudTrail logging for Route 53 changes where practical.
- Dangling-record risk: if the GitHub Pages site or repo is ever removed, delete the matching DNS records promptly to avoid subdomain takeover.
- Consider verifying the domain in GitHub (Settings → Pages → Verified domains) to prevent others from claiming it on Pages.

## Registrar

- MFA on the registrar account, registrar/transfer lock on, auto-renew on, and a monitored contact email for domain notices.

## Secrets Policy

- **Never commit secrets** (API keys, tokens, passwords, private keys) to this public repo. Anything committed is exposed, including after deletion, because of git history.
- The current site needs no secrets. If one is ever committed by mistake, **rotate it immediately** and then remove it from history.
- Consider enabling GitHub secret scanning and push protection (free for public repos).

## If Dynamic Features Are Added Later

The site is static today. If any of the following are introduced, update this document and add appropriate controls:

| Addition | Auth consideration |
|---|---|
| Contact form (Formspree, Microsoft Forms, etc.) | Use the provider's hosted endpoint; keep any provider keys public-safe or server-side only; add spam protection |
| Analytics or embeds (third-party scripts) | Review the script source, pin versions, and consider a Content Security Policy |
| Gated content or client portal | Do not build on GitHub Pages; use a proper identity provider (Microsoft Entra ID, Auth0, Cognito) behind a real backend |
| Serverless API (AWS Lambda, Azure Functions) | Use OAuth/OIDC or API keys held server-side; never ship secrets in `script.js` |

Any client-side JavaScript is visible to every visitor, so **never treat front-end code as a security boundary**.

## Security Checklist

- [ ] MFA enabled on all GitHub accounts with write access
- [ ] `main` branch protection enabled
- [ ] Enforce HTTPS enabled in GitHub Pages
- [ ] Domain verified in GitHub Pages
- [ ] Route 53 access limited to least-privilege IAM with MFA
- [ ] Registrar account has MFA and registrar lock
- [ ] Secret scanning and push protection enabled
- [ ] Collaborator and token review scheduled (e.g., quarterly)

## Reporting a Security Issue

Report suspected vulnerabilities or unauthorized changes to the site owner at MIS Solutions, LLC through the contact information published on [misfirm.com](https://misfirm.com). Please do not open public GitHub issues for sensitive reports.
