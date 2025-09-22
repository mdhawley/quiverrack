# Production Deployment Checklist

This checklist should be completed before deploying QuiverRack to production.

## ✅ Completed Tasks

### Security & Authentication
- [x] **Debug endpoints removed** - All /api/debug/* endpoints have been deleted
- [x] **Admin authentication implemented** - Role-based access control with audit logging
- [x] **Rate limiting active** - Multi-tier rate limiting with behavior analysis
- [x] **Security headers configured** - CSP, HSTS, and other headers in next.config.js
- [x] **Console.log statements removed** - 344 statements removed from production code

### Configuration
- [x] **Environment variables documented** - See production-environment-variables.md
- [x] **Production env example created** - .env.production.example file
- [x] **SSL/HTTPS configured** - HSTS enabled for production in next.config.js

### Error Handling
- [x] **Sentry integration added** - Error monitoring configured
- [x] **Custom error page** - User-friendly error handling with Sentry reporting

## 🔲 Required Before Launch

### Environment Setup
- [ ] Set all production environment variables in Vercel
- [ ] Update NEXT_PUBLIC_APP_URL to production domain
- [ ] Configure Supabase production instance
- [ ] Set up Sentry account and get DSN/auth token
- [ ] Configure OAuth redirect URLs for all providers

### Database
- [ ] Run all migrations on production database
- [ ] Verify RLS policies are enabled on all tables
- [ ] Seed initial data (ski brands, resorts, terrain types)
- [ ] Set up admin users using setup-admin-users.ts script
- [ ] Test database backup/restore procedures

### Testing
- [ ] Run `pnpm test` - Ensure all unit tests pass
- [ ] Run `pnpm test:e2e` - Ensure all E2E tests pass
- [ ] Run `pnpm build` - Ensure build succeeds without errors
- [ ] Run `pnpm typecheck` - Ensure no TypeScript errors
- [ ] Run `pnpm lint` - Ensure no linting errors

### Performance & Monitoring
- [ ] Enable Vercel Analytics
- [ ] Set up uptime monitoring (e.g., Pingdom, UptimeRobot)
- [ ] Configure alerts for errors and downtime
- [ ] Load test critical API endpoints

### Domain & Deployment
- [ ] Configure custom domain in Vercel
- [ ] Set up DNS records
- [ ] Enable automatic deployments from main branch
- [ ] Configure preview deployments for PRs
- [ ] Set up staging environment

## 📋 Launch Day Checklist

### Pre-Launch (1-2 hours before)
- [ ] Final backup of any existing data
- [ ] Verify all environment variables are set
- [ ] Test critical user flows in staging
- [ ] Ensure support team is ready

### Launch
- [ ] Deploy to production
- [ ] Verify deployment successful
- [ ] Test critical paths:
  - [ ] User registration (anonymous and email)
  - [ ] Profile creation
  - [ ] Ski/resort management
  - [ ] Public profile viewing
- [ ] Monitor error rates in Sentry
- [ ] Check rate limiting is working

### Post-Launch (First 24 hours)
- [ ] Monitor error logs closely
- [ ] Check performance metrics
- [ ] Review user feedback
- [ ] Be ready to rollback if critical issues arise

## 🚨 Rollback Plan

If critical issues occur:

1. **Immediate Actions**
   - Revert to previous deployment in Vercel
   - Notify users via status page or social media
   - Document the issue

2. **Investigation**
   - Review Sentry errors
   - Check server logs
   - Analyze what went wrong

3. **Fix and Redeploy**
   - Fix the issue in a hotfix branch
   - Test thoroughly in staging
   - Deploy with extra monitoring

## 📊 Success Metrics

Monitor these after launch:

- **Error rate** < 1% (via Sentry)
- **API response time** < 500ms p95
- **Successful registrations** > 50% of attempts
- **Page load time** < 3s
- **Uptime** > 99.9%

## 🔒 Security Reminders

- Never commit secrets to git
- Rotate API keys after launch
- Monitor for suspicious activity
- Review admin audit logs regularly
- Keep dependencies updated

## 📞 Emergency Contacts

Document your team's emergency contacts:

- **Technical Lead**: [Name] - [Contact]
- **DevOps**: [Name] - [Contact]
- **Support Lead**: [Name] - [Contact]
- **Supabase Support**: support@supabase.io
- **Vercel Support**: via dashboard

---

Last updated: [Current Date]
Ready for production: NO - Complete all required tasks first