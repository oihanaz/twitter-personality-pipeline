# Twitter Personality Analysis Dashboard 

## Dashboard Structure
We'll create 3 main visualizations:
1. **MBTI Personality Distribution** (Bar Chart)
2. **Real-time Tweet Feed** (Table)
3. **Personality Type Categories** (Pie Chart)



---

## Step-by-Step Setup

### Step 1: Database Connection Setup

1. **Open Superset**: http://localhost:8088
2. **Login**: admin / admin
3. **Navigate**: Settings → Database Connections
4. **Click**: "+ Database"
5. **Select**: MySQL from the database list
6. **Enter Connection Details**:
   ```
   Host: mysql
   Port: 3306
   Database: twitter_analytics
   Username: twitter_user
   Password: twitter_password
   ```
7. **Test Connection** → **Connect**

### Step 2: SQL Queries


#### Query 1: MBTI Distribution with Percentages
```sql
SELECT 
    mbti_personality,
    COUNT(*) as tweet_count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM tweets), 1) as percentage
FROM tweets 
WHERE mbti_personality != 'unknown'
GROUP BY mbti_personality 
ORDER BY tweet_count DESC;
```

#### Query 2: Real-time Tweet Feed
```sql
SELECT 
    user_id,
    mbti_personality,
    CASE 
        WHEN verified = 1 THEN '✓ Verified'
        ELSE 'Not Verified'
    END as verification_status,
    LEFT(tweet, 80) as tweet_preview,
    timestamp,
    statuses_count
FROM tweets 
ORDER BY timestamp DESC 
LIMIT 25;
```

#### Query 3: Personality Type Categories
```sql
SELECT 
    CASE 
        WHEN mbti_personality LIKE '%E%' THEN 'Extrovert'
        ELSE 'Introvert'
    END as personality_dimension,
    COUNT(*) as count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM tweets WHERE mbti_personality != 'unknown'), 1) as percentage
FROM tweets 
WHERE mbti_personality != 'unknown'
GROUP BY personality_dimension
UNION ALL
SELECT 
    CASE 
        WHEN mbti_personality LIKE '%S%' THEN 'Sensing'
        ELSE 'Intuition'
    END as personality_dimension,
    COUNT(*) as count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM tweets WHERE mbti_personality != 'unknown'), 1) as percentage
FROM tweets 
WHERE mbti_personality != 'unknown'
GROUP BY personality_dimension
UNION ALL
SELECT 
    CASE 
        WHEN mbti_personality LIKE '%T%' THEN 'Thinking'
        ELSE 'Feeling'
    END as personality_dimension,
    COUNT(*) as count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM tweets WHERE mbti_personality != 'unknown'), 1) as percentage
FROM tweets 
WHERE mbti_personality != 'unknown'
GROUP BY personality_dimension
UNION ALL
SELECT 
    CASE 
        WHEN mbti_personality LIKE '%J%' THEN 'Judging'
        ELSE 'Perceiving'
    END as personality_dimension,
    COUNT(*) as count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM tweets WHERE mbti_personality != 'unknown'), 1) as percentage
FROM tweets 
WHERE mbti_personality != 'unknown'
GROUP BY personality_dimension;
```

---
