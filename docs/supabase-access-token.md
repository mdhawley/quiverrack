# Supabase Access Token Guide

## For Local Development

When working locally, you **don't need** to manually manage the access token. Simply run:

```bash
supabase login
```

This will:
1. Open your browser
2. Authenticate with Supabase
3. Store the token locally
4. The CLI will use it automatically

## For CI/CD (GitHub Actions)

For automated deployments, you need to create a personal access token:

### Steps to Get Your Access Token:

1. **Go to Supabase Dashboard**
   - Navigate to: https://supabase.com/dashboard/account/tokens

2. **Create New Token**
   - Click "Generate new token"
   - Give it a descriptive name (e.g., "GitHub Actions CI/CD")
   - Copy the token immediately (it won't be shown again)

3. **Add to GitHub Secrets**
   ```
   Repository Settings > Secrets and variables > Actions > New repository secret
   Name: SUPABASE_ACCESS_TOKEN
   Value: [paste your token]
   ```

### Security Notes:
- **Never commit** the token to your repository
- **Rotate regularly** for security
- **Use different tokens** for different purposes (dev, CI/CD, etc.)
- **Revoke immediately** if compromised

## Verifying Your Setup

### Local Verification:
```bash
# Check if you're logged in
supabase projects list

# Should show your projects without error
```

### CI/CD Verification:
The GitHub Actions workflow will use the token from secrets automatically.

## Alternative: Using Organization-level Tokens

For team environments, consider using organization-level service tokens instead of personal tokens:

1. Go to Organization Settings in Supabase Dashboard
2. Navigate to Access Tokens
3. Create a service token with appropriate permissions
4. Use this token in CI/CD environments

## Troubleshooting

### "Not logged in" Error Locally
```bash
supabase login
```

### "Invalid token" in CI/CD
- Check the token hasn't expired
- Verify it's correctly added to GitHub Secrets
- Ensure no extra spaces when pasting

### "Permission denied"
- Token might not have sufficient permissions
- Check organization access settings