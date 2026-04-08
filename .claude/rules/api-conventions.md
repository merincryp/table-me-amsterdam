# API & External Service Conventions

## Vercel
- Config lives in `vercel.json` at project root
- All routes rewrite to `/index.html` (SPA pattern)
- Environment variables via Vercel dashboard, never in code

## Google Fonts
- Preconnect to `fonts.googleapis.com` and `fonts.gstatic.com`
- Load only weights actually used
- Use `display=swap` for font loading

## External Links
- Always `target="_blank"` with `rel="noopener noreferrer"`
- Verify links are alive before deploying

## Forms / Contact
- Validate client-side before submission
- Use `mailto:` or external form service (no backend)
- Include honeypot field for spam prevention
