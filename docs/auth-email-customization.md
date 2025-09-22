# Supabase Auth Email Customization Guide

This document outlines how to customize Supabase Auth emails with QuiverRack branding and custom domain configuration.

## Overview

Currently, QuiverRack uses Supabase's default email templates for authentication flows (password reset, signup confirmation, etc.). These emails come from Supabase's domain and use generic branding. This guide provides a complete implementation plan for custom branded emails.

## Current State

- ✅ Password reset functionality implemented
- ✅ Signup/login flows working
- ❌ Emails come from Supabase domain (`*.supabase.co`)
- ❌ Generic Supabase branding in emails
- ❌ No custom SMTP configuration

## Implementation Plan

### Phase 1: Custom SMTP Configuration

#### 1.1 Choose SMTP Service Provider

**Recommended Options:**
- **Resend** (Recommended) - Developer-friendly, good deliverability, $20/month for 100k emails
- **AWS SES** - Cost-effective for high volume, requires more setup
- **Postmark** - Excellent deliverability, focused on transactional emails
- **SendGrid** - Popular choice, comprehensive features

#### 1.2 Domain Setup

**Requirements:**
- Domain ownership (e.g., `quiverrack.com`)
- DNS access for email authentication records
- Email subdomain (optional): `mail.quiverrack.com`

**DNS Records to Configure:**
```dns
# SPF Record
TXT @ "v=spf1 include:_spf.resend.com ~all"

# DKIM Record (provided by SMTP service)
TXT resend._domainkey "DKIM_VALUE_FROM_PROVIDER"

# DMARC Record
TXT _dmarc "v=DMARC1; p=none; rua=mailto:dmarc@quiverrack.com"
```

#### 1.3 Supabase SMTP Configuration

**Location:** Supabase Dashboard → Project Settings → Authentication → SMTP Settings

**Configuration:**
```
Enable Custom SMTP: ON
SMTP Host: smtp.resend.com (example for Resend)
SMTP Port: 587
SMTP User: resend
SMTP Password: [API_KEY_FROM_PROVIDER]
Sender Name: QuiverRack
Sender Email: noreply@quiverrack.com
```

### Phase 2: Custom Email Templates

#### 2.1 Available Templates

Supabase provides these authentication email templates:
- **Password Recovery** - When users request password reset
- **Confirm Signup** - Email verification for new accounts
- **Magic Link** - Passwordless authentication
- **Invite User** - Team/organization invitations
- **Change Email Address** - Email change confirmation

#### 2.2 Template Variables

Available variables for personalization:
- `{{ .ConfirmationURL }}` - Reset/confirmation link
- `{{ .Token }}` - 6-digit OTP code
- `{{ .SiteURL }}` - App URL (e.g., https://quiverrack.com)
- `{{ .Data.display_name }}` - User's display name
- `{{ .Data.email }}` - User's email address

#### 2.3 Template Access

**Hosted Projects:** Dashboard → Authentication → Email Templates
**Self-hosted:** Configuration files
**Management API:** Programmatic updates

### Phase 3: Branded Template Design

#### 3.1 Design Requirements

**Brand Elements:**
- QuiverRack logo and colors
- Primary color: #005B8C (existing brand blue)
- Ski/winter sports themed elements
- Consistent typography with web app
- Professional, trustworthy appearance

#### 3.2 Password Reset Template Example

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Reset Your QuiverRack Password</title>
    <style>
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin: 0; padding: 0; background-color: #f8fafc; }
        .container { max-width: 600px; margin: 0 auto; background-color: white; }
        .header { background: linear-gradient(135deg, #005B8C 0%, #007BB8 100%); padding: 30px 20px; text-align: center; }
        .logo { color: white; font-size: 28px; font-weight: bold; margin: 0; }
        .content { padding: 40px 30px; }
        .button { display: inline-block; background-color: #005B8C; color: white; padding: 14px 28px; text-decoration: none; border-radius: 8px; font-weight: 600; margin: 20px 0; }
        .footer { background-color: #f1f5f9; padding: 25px; text-align: center; font-size: 14px; color: #64748b; }
        .code-box { background-color: #f8fafc; border: 2px solid #e2e8f0; border-radius: 8px; padding: 20px; margin: 20px 0; text-align: center; }
    </style>
</head>
<body>
    <div class="container">
        <!-- Header with QuiverRack branding -->
        <div class="header">
            <h1 class="logo">⛷️ QuiverRack</h1>
            <p style="color: #e2e8f0; margin: 5px 0 0 0;">Share Your Ski Profile</p>
        </div>
        
        <!-- Main content -->
        <div class="content">
            <h2 style="color: #1e293b; margin-bottom: 20px;">Reset Your Password</h2>
            
            <p style="color: #475569; line-height: 1.6; margin-bottom: 20px;">
                Hi there! Someone requested a password reset for your QuiverRack account. 
                If this was you, click the button below to create a new password.
            </p>
            
            <!-- Reset button -->
            <div style="text-align: center; margin: 30px 0;">
                <a href="{{ .ConfirmationURL }}" class="button">Reset My Password</a>
            </div>
            
            <!-- Alternative OTP code -->
            <div class="code-box">
                <p style="margin: 0 0 10px 0; font-weight: 600; color: #1e293b;">Or use this verification code:</p>
                <p style="font-size: 24px; font-weight: bold; letter-spacing: 3px; color: #005B8C; margin: 0;">{{ .Token }}</p>
            </div>
            
            <p style="color: #64748b; font-size: 14px; line-height: 1.5;">
                This link will expire in 1 hour for security reasons. If you didn't request this password reset, 
                you can safely ignore this email - your account remains secure.
            </p>
            
            <p style="color: #64748b; font-size: 14px; line-height: 1.5;">
                Need help? Contact us at 
                <a href="mailto:support@quiverrack.com" style="color: #005B8C;">support@quiverrack.com</a>
            </p>
        </div>
        
        <!-- Footer -->
        <div class="footer">
            <p style="margin: 0 0 10px 0;">© 2025 QuiverRack. All rights reserved.</p>
            <p style="margin: 0;">
                Visit us at <a href="{{ .SiteURL }}" style="color: #005B8C; text-decoration: none;">{{ .SiteURL }}</a>
            </p>
        </div>
    </div>
</body>
</html>
```

#### 3.3 Signup Confirmation Template

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Welcome to QuiverRack!</title>
    <!-- Same styles as above -->
</head>
<body>
    <div class="container">
        <div class="header">
            <h1 class="logo">⛷️ QuiverRack</h1>
            <p style="color: #e2e8f0; margin: 5px 0 0 0;">Share Your Ski Profile</p>
        </div>
        
        <div class="content">
            <h2 style="color: #1e293b;">Welcome to QuiverRack!</h2>
            
            <p style="color: #475569; line-height: 1.6;">
                Thanks for joining the QuiverRack community! We're excited to help you 
                share your ski profile and connect with fellow skiers.
            </p>
            
            <p style="color: #475569; line-height: 1.6;">
                To get started, please confirm your email address by clicking the button below:
            </p>
            
            <div style="text-align: center; margin: 30px 0;">
                <a href="{{ .ConfirmationURL }}" class="button">Confirm Email Address</a>
            </div>
            
            <div class="code-box">
                <p style="margin: 0 0 10px 0; font-weight: 600; color: #1e293b;">Or use this verification code:</p>
                <p style="font-size: 24px; font-weight: bold; letter-spacing: 3px; color: #005B8C; margin: 0;">{{ .Token }}</p>
            </div>
            
            <p style="color: #64748b; font-size: 14px;">
                Once confirmed, you can start building your ski profile, adding your equipment, 
                and sharing your mountain adventures with the community.
            </p>
        </div>
        
        <div class="footer">
            <p style="margin: 0 0 10px 0;">© 2025 QuiverRack. All rights reserved.</p>
            <p style="margin: 0;">
                Visit us at <a href="{{ .SiteURL }}" style="color: #005B8C; text-decoration: none;">{{ .SiteURL }}</a>
            </p>
        </div>
    </div>
</body>
</html>
```

### Phase 4: Environment Configuration

#### 4.1 Required Environment Variables

Add to your environment configuration:

```env
# SMTP Configuration (for reference - actual config done in Supabase dashboard)
SMTP_HOST=smtp.resend.com
SMTP_PORT=587
SMTP_USER=resend
SMTP_FROM_NAME=QuiverRack
SMTP_FROM_EMAIL=noreply@quiverrack.com

# For custom domain support (Pro plan feature)
SUPABASE_CUSTOM_DOMAIN=api.quiverrack.com
```

#### 4.2 Supabase Configuration

Update these settings in Supabase Dashboard:

**Authentication → Settings:**
- Site URL: `https://quiverrack.com`
- Redirect URLs: Add production domains

**Authentication → Email Templates:**
- Update each template with branded HTML
- Test with preview functionality

### Phase 5: Testing & Deployment

#### 5.1 Testing Checklist

- [ ] Test password reset email in staging
- [ ] Verify email deliverability (check spam folders)
- [ ] Test email rendering across different clients
- [ ] Confirm all links work correctly
- [ ] Validate OTP codes function properly
- [ ] Test on mobile email clients

#### 5.2 Monitoring

**Email Deliverability Metrics:**
- Open rates
- Click-through rates
- Bounce rates
- Spam complaint rates

**Tools:**
- SMTP provider dashboard
- Google Postmaster Tools
- Email testing services (Litmus, Email on Acid)

### Phase 6: Advanced Features (Optional)

#### 6.1 Custom Domain for API (Pro Plan)

Configure `api.quiverrack.com` to replace `*.supabase.co` in URLs:
- Requires Supabase Pro plan
- Makes all authentication URLs use your domain
- Improves brand consistency and trust

#### 6.2 Email Analytics

- Track email engagement
- A/B test different templates
- Monitor authentication completion rates

#### 6.3 Multi-language Support

- Create templates for different locales
- Use user metadata to determine language
- Conditional template rendering

## Cost Considerations

**SMTP Service:** $20-50/month (depends on volume)
**Custom Domain:** $25/month (Supabase Pro plan)
**Email Analytics:** $0-30/month (optional)
**Development Time:** 8-16 hours

## Implementation Timeline

**Week 1:**
- Choose and configure SMTP provider
- Set up DNS records
- Design email templates

**Week 2:**
- Implement templates in Supabase
- Test authentication flows
- Monitor deliverability

**Week 3:**
- Optimize based on metrics
- Consider advanced features

## Security Considerations

- Always use HTTPS for confirmation URLs
- Implement rate limiting for email sends
- Monitor for suspicious authentication patterns
- Keep SMTP credentials secure
- Regular security audits of email templates

## Maintenance

- Monitor email deliverability metrics monthly
- Update templates for seasonal campaigns
- Review and update DNS records annually
- Keep SMTP provider credentials rotated

## Support Resources

- [Supabase Email Templates Docs](https://supabase.com/docs/guides/auth/auth-email-templates)
- [Supabase SMTP Configuration](https://supabase.com/docs/guides/auth/auth-smtp)
- [Email Deliverability Best Practices](https://sendlayer.com/blog/supabase-custom-smtp-and-email-configuration-guide/)

---

*This documentation was created on January 2025 based on current Supabase capabilities and best practices.*