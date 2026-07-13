# indersoninvestmentgroup
Inderson Investment Group website

## Local development

Run the static site from the repository root:

```bash
python3 -m http.server 4173
```

Then open `http://localhost:4173`.

## Render deployment

Configure the Render static site's publish directory as `.`. The homepage and
all linked portal pages are kept at the repository root so each push to `main`
can be deployed directly.
