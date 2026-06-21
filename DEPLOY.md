# Deploying DeskDuck to Cloudflare Pages

This guide takes the site live at **https://deskduck.ca** and points your two other
domains (**Desk-Duck.com** and **quackers.ca**) at it with permanent (301) redirects.

You'll do everything in two places: your **domain registrar** (where you bought the
three domains) and a free **Cloudflare** account. Total time: ~30–45 minutes, most of
which is waiting for DNS to update.

The files to deploy are the contents of this folder:
`index.html`, `assets/`, `robots.txt`, `sitemap.xml`, `_redirects`.

---

## Step 1 — Create a Cloudflare account

1. Go to https://dash.cloudflare.com/sign-up and create a free account.
2. Verify your email and log in.

---

## Step 2 — Add deskduck.ca to Cloudflare (DNS)

This lets Cloudflare manage the domain so it can serve the site and handle SSL.

1. In the Cloudflare dashboard, click **Add a domain** (or **Add a site**).
2. Enter `deskduck.ca` and choose the **Free** plan.
3. Cloudflare scans existing DNS records, then shows you **two nameservers**
   (something like `xena.ns.cloudflare.com` and `rick.ns.cloudflare.com`).
4. Log in to the **registrar where you bought deskduck.ca** and replace its
   nameservers with the two Cloudflare gave you. (Look for "Nameservers" or
   "DNS settings" on the domain.)
5. Back in Cloudflare, click **Continue / Check nameservers**. Activation can take
   anywhere from a few minutes to a few hours. You'll get an email when it's active.

Repeat this same "Add a domain" process for **Desk-Duck.com** and **quackers.ca**
too — you'll need them in Cloudflare for the redirects in Step 5. (You can add all
three now and let them activate while you do the rest.)

---

## Step 3 — Push the site to GitHub

The folder is already a Git repository with your first commit made. You just need to
create an empty repo on GitHub and push to it. After this, every change you push
auto-deploys (Step 3b).

1. Go to https://github.com/new and create a repository named `deskduck`.
   **Leave it empty** — no README, no .gitignore, no license (you already have those).
2. GitHub shows you the repo URL. In a terminal, from this folder, run:

   ```bash
   cd "path/to/Duck Website"
   git remote add origin https://github.com/SausageBandit/deskduck.git
   git push -u origin main
   ```

   (Swap in your GitHub username. GitHub will ask you to log in or paste a personal
   access token the first time.)
3. Refresh the GitHub page — all the files should now be there.

> Already use SSH with GitHub? Use the SSH URL instead:
> `git remote add origin git@github.com:YOUR_USERNAME/deskduck.git`

---

## Step 3b — Connect the GitHub repo to Cloudflare Pages

1. In Cloudflare: **Workers & Pages → Create → Pages → Connect to Git**.
2. Authorize Cloudflare to access your GitHub, then pick the `deskduck` repo.
3. Build settings — this is a plain static site with no build step, so:
   - **Framework preset:** None
   - **Build command:** *(leave blank)*
   - **Build output directory:** `/`
4. Click **Save and Deploy**. You'll get a live `*.pages.dev` URL in a few seconds —
   open it and confirm the page looks right.

From now on, **every `git push` to `main` automatically rebuilds and redeploys** the
site. To update it:

```bash
git add -A
git commit -m "Describe your change"
git push
```

> Prefer not to use GitHub? You can instead drag the folder's contents into
> **Pages → Upload assets** for a one-off manual deploy. But the Git connection above
> is what gives you painless future updates.

---

## Step 4 — Point deskduck.ca at the site

1. Open your **deskduck** Pages project → **Custom domains** tab → **Set up a
   custom domain**.
2. Enter `deskduck.ca` and confirm. Because Cloudflare already manages the domain
   (Step 2), it adds the needed DNS record automatically.
3. Add a second custom domain: `www.deskduck.ca`.
   - The included `_redirects` file will forward `www.deskduck.ca` → `deskduck.ca`
     so you have one clean canonical address.
4. SSL (the padlock / https) is issued automatically — give it a few minutes.

Once active, **https://deskduck.ca** shows your site. 🎉

---

## Step 5 — Redirect the other two domains to deskduck.ca

Do this once for **Desk-Duck.com** and once for **quackers.ca**. (Both must be added
to Cloudflare per Step 2 first.)

For each domain:

1. Select the domain in the Cloudflare dashboard.
2. Go to **Rules → Redirect Rules → Create rule**.
3. Configure:
   - **Rule name:** `Redirect to deskduck.ca`
   - **When incoming requests match:** *All incoming requests*
   - **Then... Type:** Dynamic
   - **Expression:** `concat("https://deskduck.ca", http.request.uri.path)`
   - **Status code:** `301` (permanent)
   - Enable **Preserve query string**.
4. **Deploy**.

You'll also need a DNS record so the domain resolves before the redirect fires. In
that domain's **DNS** tab, add a proxied record (orange cloud ON):
   - Type `A`, Name `@`, IPv4 `192.0.2.1` (a placeholder — the redirect rule
     intercepts the request before it matters), **Proxy status: Proxied**.
   - Add the same for Name `www`.

Now `Desk-Duck.com`, `www.Desk-Duck.com`, `quackers.ca`, and `www.quackers.ca` all
land on `https://deskduck.ca`.

---

## Step 6 — Final checks

- Visit https://deskduck.ca — site loads with a padlock (https).
- Visit https://desk-duck.com and https://quackers.ca — each should bounce to
  deskduck.ca.
- Submit a test email through both signup forms, then confirm it appears in your
  **Formspree dashboard** and in your inbox. (The form already posts to your
  endpoint `formspree.io/f/mpqgaoyq` — nothing to change.)
- Open the site on your phone to confirm the mobile layout.

---

## Good to know

- **Formspree keeps working as-is.** It's a hosted service; moving to a real domain
  doesn't change anything. You may want to add deskduck.ca to your Formspree form's
  allowed domains (in Formspree's form settings) once you're live, to stop spam from
  other sites using your endpoint.
- **SSL is automatic and free** on Cloudflare — no certificates to buy or renew.
- **To update the site**, edit the files here, then `git add -A && git commit -m "..."
  && git push`. Cloudflare redeploys automatically. The domains and redirects stay put.
- **Why deskduck.ca as primary:** it's the cleanest match to the brand. The `.com`
  and `quackers.ca` still "work" — they just forward — so you capture anyone who
  types those instead.
