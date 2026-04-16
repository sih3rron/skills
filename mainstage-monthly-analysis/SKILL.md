---
name: mainstage-monthly-analysis
description: Automates monthly analysis of Mainstage demo platform usage, extracts opportunity IDs for Salesforce analysis, and provides comprehensive usage statistics including practice vs account demo breakdown, top accounts, trends, and user activity.
triggers:
  - monthly mainstage analysis
  - mainstage monthly report
  - extract mainstage opportunity ids
  - run mainstage analysis
  - mainstage usage report
version: 1.1.0
author: Simon Herron
---


# Mainstage Monthly Analysis Skill

## Purpose
This skill automates the monthly analysis of Mainstage demo platform usage, extracting opportunity IDs for Salesforce analysis and providing comprehensive usage statistics.

## When to Use This Skill
- Monthly reporting on Mainstage platform usage
- User asks "Run the monthly Mainstage analysis"
- Analyzing demo creation trends and user engagement
- Extracting opportunity IDs for Salesforce closed-won analysis

## Admin User Exclusion
**CRITICAL**: Always exclude these 7 admin users:
```
'clw4vdm2p0005h8q7w9zxnz4x'  -- Rose Randall
'clvpn2x8m001be0mbgzy0a4fk'  -- Damian Scisci
'clw9gdxec0014zxfxbx7i9fia'  -- Seán Winters
'cm47587ic0006nfxxauxyraju'  -- Mike Belither
'clv4ynubq0000e0mbg2kfhusd'  -- Simon Herron
'clv47e64p00006qxobu8n6r85'  -- Richard Moore
'clw6f26mg000111mrffn65ih1'  -- Luis Colman
```

## Step-by-Step Workflow

### 1. Extract All Opportunity IDs (Current Year Only)
```sql
SELECT DISTINCT "opportunityId" 
FROM "UserDemos" 
WHERE "userId" NOT IN (
  'clw4vdm2p0005h8q7w9zxnz4x', 
  'clvpn2x8m001be0mbgzy0a4fk',
  'clw9gdxec0014zxfxbx7i9fia',
  'cm47587ic0006nfxxauxyraju',
  'clv4ynubq0000e0mbg2kfhusd',
  'clv47e64p00006qxobu8n6r85',
  'clw6f26mg000111mrffn65ih1'
) 
AND "accountId" IS NOT NULL 
AND "opportunityId" IS NOT NULL
AND "createdAt" >= DATE_TRUNC('year', CURRENT_DATE)
ORDER BY "opportunityId";
```
**Report**: Total count + generate Salesforce SOQL query
**Note**: This query automatically filters for demos created in the current calendar year only (e.g., 2026). The date filter updates automatically each year.

### 2. Practice vs Account Breakdown
```sql
SELECT 
  CASE WHEN "accountId" IS NOT NULL THEN 'Account Demo' ELSE 'Practice Demo' END,
  COUNT(*), 
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER (), 2)
FROM "UserDemos"
WHERE "userId" NOT IN (...admin users...)
GROUP BY ...;
```

### 3. Top 20 Accounts
```sql
SELECT "accountId", "accountName",
  COUNT(DISTINCT "id") AS demo_count,
  COUNT(DISTINCT "opportunityId"),
  COUNT(DISTINCT "userId")
FROM "UserDemos"
WHERE "userId" NOT IN (...) AND "accountId" IS NOT NULL
GROUP BY "accountId", "accountName"
ORDER BY demo_count DESC LIMIT 20;
```

### 4. Monthly Trends (12 months)
```sql
SELECT DATE_TRUNC('month', "createdAt"),
  COUNT(*), 
  COUNT(DISTINCT CASE WHEN "accountId" IS NOT NULL...),
  COUNT(DISTINCT "accountId"),
  COUNT(DISTINCT "opportunityId")
FROM "UserDemos"
WHERE "userId" NOT IN (...) 
AND "createdAt" >= CURRENT_DATE - INTERVAL '12 months'
GROUP BY DATE_TRUNC('month', "createdAt") 
ORDER BY month DESC;
```

### 5. Top 15 Users by Department
```sql
SELECT u.name, u.email, u."jobTitle", u.department,
  COUNT(DISTINCT ud.id),
  COUNT(DISTINCT ud."accountId"),
  COUNT(DISTINCT ud."opportunityId")
FROM "UserDemos" ud JOIN "Users" u ON ud."userId" = u.id
WHERE ud."userId" NOT IN (...) AND ud."accountId" IS NOT NULL
GROUP BY u.id, u.name, u.email, u."jobTitle", u.department
ORDER BY COUNT(DISTINCT ud.id) DESC LIMIT 15;
```

### 6. Demo Library Growth (Monthly Demo Creation)
```sql
SELECT 
    TO_CHAR(DATE_TRUNC('month', "createdAt"), 'YYYY-MM') as month,
    COUNT(*) as demo_count
FROM "Demo"
WHERE "deleted" IS NOT TRUE OR "deleted" IS NULL
GROUP BY DATE_TRUNC('month', "createdAt")
ORDER BY month DESC;
```
**Purpose**: Track content library growth - shows how many new demo templates/content pieces are being added to the platform each month. This is separate from UserDemos which tracks usage of existing demos.

### 7. User Activity Analysis (Monthly Engagement)
```sql
SELECT 
    EXTRACT(YEAR FROM ua."actionTime") as year,
    EXTRACT(MONTH FROM ua."actionTime") as month,
    TO_CHAR(ua."actionTime", 'Month YYYY') as month_name,
    COALESCE(u.department, 'Unknown/No Department') as department,
    COUNT(*) as total_actions,
    COUNT(DISTINCT ua.reference) as unique_users,
    COUNT(DISTINCT DATE(ua."actionTime")) as active_days,
    ROUND(COUNT(*)::numeric / COUNT(DISTINCT DATE(ua."actionTime")), 2) as avg_actions_per_day,
    ROUND(COUNT(*)::numeric / COUNT(DISTINCT ua.reference), 2) as avg_actions_per_user
FROM "UsersActions" ua
LEFT JOIN "Users" u ON ua.reference = u.id
WHERE ua."actionTime" >= '[START_DATE]'::timestamp
    AND ua."actionTime" < '[END_DATE]'::timestamp
    AND ua.reference IS NOT NULL
    AND (u.role != 'ADMIN' OR u.role IS NULL)
    AND ua.reference NOT IN (
        'clw4vdm2p0005h8q7w9zxnz4x',
        'clvpn2x8m001be0mbgzy0a4fk',
        'clw9gdxec0014zxfxbx7i9fia',
        'cm47587ic0006nfxxauxyraju',
        'clv4ynubq0000e0mbg2kfhusd',
        'clv47e64p00006qxobu8n6r85',
        'clw6f26mg000111mrffn65ih1'
    )
GROUP BY EXTRACT(YEAR FROM ua."actionTime"), EXTRACT(MONTH FROM ua."actionTime"), TO_CHAR(ua."actionTime", 'Month YYYY'), COALESCE(u.department, 'Unknown/No Department')
ORDER BY year, month, total_actions DESC;
```
**Purpose**: Track user engagement patterns through platform actions (views, clicks, interactions). Shows activity intensity, user retention, and daily engagement patterns.

**Key Metrics**:
- **Total Actions**: All user interactions with the platform
- **Unique Users**: Number of distinct users active each month
- **Active Days**: Days with at least one user action
- **Avg Actions/Day**: Platform activity intensity per day
- **Avg Actions/User**: Individual user engagement level

**Analysis Points**:
1. **Overall Activity Growth**: Track total actions month-over-month
2. **User Engagement Patterns**: Identify user count trends and retention
3. **Engagement Intensity**: Monitor actions per user (power users vs casual users)
4. **Daily Activity**: Assess consistent usage vs sporadic activity
5. **Month-over-Month Changes**: Calculate % changes in actions, users, and engagement
6. **Power User Consolidation**: Detect shifts from many casual users to fewer engaged users

**Health Indicators**:
- Increasing or stable unique user count = good adoption
- High actions per user (10-15+) = strong engagement
- 20+ active days per month = consistent usage
- Growing actions despite declining users = power user consolidation (may need retention focus)

## Salesforce Query Output
Generate ready-to-paste query:
```sql
SELECT SUM(Amount), COUNT(Id), AVG(Amount), MIN(Amount), MAX(Amount)
FROM Opportunity
WHERE Id IN ([all IDs]) AND IsClosed = true AND IsWon = true
```

## Report Structure
1. Summary Stats (total opps, demos, MoM change)
2. Salesforce Query (ready to paste)
3. Top 20 Accounts Table
4. 12-Month Trends
5. Top 15 Users by Department
6. Demo Library Growth (monthly demo creation in Demo table)
7. User Activity Analysis (engagement metrics, active users, actions per user)

## Health Indicators
- Practice:Account ratio = 4:1 to 6:1 is healthy
- Account demo growth should outpace practice growth
- Cross-functional adoption (Sales, SE, CS)
- Demo library should grow steadily (indicates content creation and platform health)
- Correlation between demo library growth and UserDemos usage indicates content relevance