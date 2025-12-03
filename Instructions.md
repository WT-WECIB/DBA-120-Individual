# Social Media Database Project
## Plug & Play Assignment (GitHub Codespaces Edition)

Everything runs in the browser. No installation needed. 4 days of work.

---

## What You're Building

A social media website where:
- Users can post messages
- Users can comment on posts
- Users can like posts
- Your Python script analyzes the data (top posters, trending posts, etc)

**You will see**: A real website in your browser + a Python report in terminal.

---

## Day 1: Create the Database Tables (30 minutes)

### Step 1: Open Apex -> Scripts

### Step 2: Upload and run ind_project.sql

### Step 3: Verify it worked

Run this query:
```sql
SELECT * FROM users;
SELECT * FROM posts;
```

You should see data.

---

## Part 2: Set Up APEX Website

### Step 1: Go to your APEX instance

Open your APEX environment in browser: `https://oracleapex.com/ords/...`

### Step 2: Create an app

Click **Drop down arrow in App Builder** → **"Create"** → Enter **social_media** in **Name** field → Click **"Create Application"**

Name it: `social_media`

### Step 3: Create 4 Pages

You will create 4 pages. For EACH page, follow same steps:

**Page 1: Feed (List View)**

1. Click **"Create Page"** 
2. Select **"Interactive Report"**
3. Name it: `Feed`
4. Source Table: `POSTS`  
5. Click **"Create Page"**  
6. Wait 10 seconds, click **"Save and Run"** (green play button on top right)

You will see a table with all posts. **Page 1 Done** 

---

**Page 2: Create Post (Form)**

1. Go back to App main page and click **"Create Page"**
2. Select **"Form"**
3. Name it: `Create Post`
4. Table: `POSTS`
5. Click **Next**  
6. Click **"Create Page"**
7. Wait, click **"Save and Run"**

You will see a form. Fill it out and click **"Create"**. The post appears on Feed page. **Page 2 Done** ✓

---

**Page 3: Comments (Interactive Report)**

1. Click **"Create Page"**
2. Select **"Interactive Report"**
3. Name it: `Comments`
4. Source Table: `COMMENTS`
5. Click **"Create Page"**
6. Click **"Save and Run"**

You see all comments. **Page 3 Done** ✓

---

**Page 4: Leaderboard (Report)**

1. Click **"Create Page"**
2. Select **"Blank Page"**
3. Name it: `Leaderboard`
4. Click **"Create Page"**

5. In the bottom middle, make sure the **Regions** tab is selected
6. Select **"Classic Report"** and drag it into the middle section (right below Layout Page Search Help)  
7. Fill out the info on the right: Name it: `Top Posters`, change Type: to SQL Query
8. SQL Query: Copy-paste this:

```sql
SELECT u.username, COUNT(p.post_id) as post_count
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
GROUP BY u.username
ORDER BY post_count DESC
```

9. Click **"Create Region"**
10. Click **"Save and Run"**

You see a leaderboard of top posters. **Page 4 Done** ✓

### Step 5: Create REST API Endpoints

1. In APEX, click **"SQL Workshop"** → **"RESTful Services"**
2. Select **"Modules"**  on the left side
3. Click **"Create Module"**
4. Module Name: `social`
5. Base Path: `/social/`
6. Click **"Create Module"**

---

Now add 4 endpoints:

**Endpoint 1: Posts**
1. Click the `social` module you created
2. Go to the **Resource Templates** tab Click -> **"Create Template"**
3. URI Template: `posts/`
4. Click **"Create Template"**
5. Click **"Create Handler"**
6. Method: `GET`
7. Source Type: `Query`
8. Source: Copy-paste:
```sql
SELECT post_id, user_id, text, created_at FROM posts ORDER BY created_at DESC
```
9. Click **"Create Handler"**

Repeat 3 more times for:

**Endpoint 2: Comments**
- URI Template: `comments/`
- Source Type: `Collection Query`
- SQL: `SELECT comment_id, post_id, user_id, text, created_at FROM comments ORDER BY created_at DESC`

**Endpoint 3: Reactions**
- URI Template: `reactions/`
- Source Type: `Collection Query`
- SQL: `SELECT reaction_id, post_id, user_id, reaction_type, created_at FROM reactions`

**Endpoint 4: Users**
- URI Template: `users/`
- Source Type: `Collection Query`
- SQL: `SELECT user_id, username, created_at FROM users`

### Step 6: Test REST endpoints

Open browser and visit:
- `https://oracleapex.com/ords/social/posts/` → should see JSON
- `https://oracleapex.com/ords/social/users/` → should see JSON

If you see JSON data ✓ -> **Part 2 Done**

---

## Part 3: Python Backend in GitHub Codespaces

### Step 1: Open GitHub Codespaces

Go to your GitHub repository and click **"Code"** → **"Codespaces"** → **"Create codespace on main"**

Wait 2-3 minutes for it to load.

### Step 2: Create your analytics file

In the left sidebar, right-click and create new file: `analytics.py`

### Step 3: Copy-paste this entire code

```python
import requests
import json
from datetime import datetime
from requests.packages.urllib3.exceptions import InsecureRequestWarning

# Disable SSL warnings
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

# ============ UPDATE THESE WITH YOUR APEX URLs ============
# Example: https://oracleapex.com/ords/social
BASE_URL = "https://oracleapex.com/ords/social"

POSTS_URL = f"{BASE_URL}/posts/"
COMMENTS_URL = f"{BASE_URL}/comments/"
REACTIONS_URL = f"{BASE_URL}/reactions/"
USERS_URL = f"{BASE_URL}/users/"

# Browser-like headers (required)
HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)',
    'Content-Type': 'application/json',
    'Accept': 'application/json'
}

def fetch_data(url):
    """Fetch data from REST endpoint"""
    try:
        response = requests.get(url, headers=HEADERS, timeout=30, verify=False)
        if response.status_code == 200:
            data = response.json()
            # Handle both direct array and wrapped response
            if isinstance(data, dict) and 'items' in data:
                return data['items']
            elif isinstance(data, dict) and 'results' in data:
                return data['results']
            elif isinstance(data, list):
                return data
            else:
                return []
        else:
            print(f"Error: {response.status_code}")
            return []
    except Exception as e:
        print(f"Connection error: {e}")
        return []

def get_top_posters(limit=5):
    """Calculate which users have posted the most"""
    posts = fetch_data(POSTS_URL)
    user_posts = {}
    
    for post in posts:
        user_id = post['user_id']
        user_posts[user_id] = user_posts.get(user_id, 0) + 1
    
    sorted_users = sorted(user_posts.items(), key=lambda x: x[1], reverse=True)
    return sorted_users[:limit]

def get_trending_posts(limit=5):
    """Get most liked posts"""
    posts = fetch_data(POSTS_URL)
    reactions = fetch_data(REACTIONS_URL)
    
    # Count likes per post
    post_likes = {post['post_id']: 0 for post in posts}
    
    for reaction in reactions:
        if reaction['post_id'] in post_likes:
            post_likes[reaction['post_id']] += 1
    
    # Return top posts with their like counts
    sorted_posts = sorted(post_likes.items(), key=lambda x: x[1], reverse=True)
    return sorted_posts[:limit]

def flag_inappropriate_content():
    """Find posts with banned words"""
    banned_words = ['spam', 'hate', 'inappropriate', 'ugly']
    posts = fetch_data(POSTS_URL)
    flagged = []
    
    for post in posts:
        text = post['text'].lower()
        for word in banned_words:
            if word in text:
                flagged.append({
                    'post_id': post['post_id'],
                    'text': post['text'],
                    'reason': f"Contains '{word}'"
                })
                break
    
    return flagged

def get_comment_stats():
    """How many comments per post"""
    comments = fetch_data(COMMENTS_URL)
    posts = fetch_data(POSTS_URL)
    
    comment_count = {}
    for post in posts:
        comment_count[post['post_id']] = 0
    
    for comment in comments:
        post_id = comment['post_id']
        if post_id in comment_count:
            comment_count[post_id] += 1
    
    return comment_count

def print_analytics():
    """Print full analytics dashboard"""
    print("\n" + "="*50)
    print("SOCIAL MEDIA ANALYTICS DASHBOARD")
    print("="*50 + "\n")
    
    # Top Posters
    print("📊 TOP POSTERS (by number of posts):")
    print("-" * 50)
    top_posters = get_top_posters()
    for rank, (user_id, count) in enumerate(top_posters, 1):
        print(f"  #{rank} User {user_id}: {count} posts")
    
    # Trending Posts
    print("\n🔥 TRENDING POSTS (by likes):")
    print("-" * 50)
    trending = get_trending_posts()
    for rank, (post_id, likes) in enumerate(trending, 1):
        print(f"  #{rank} Post {post_id}: {likes} likes")
    
    # Comment Stats
    print("\n💬 COMMENT ACTIVITY:")
    print("-" * 50)
    comments = get_comment_stats()
    total_comments = sum(comments.values())
    avg_comments = total_comments / len(comments) if comments else 0
    print(f"  Total comments: {total_comments}")
    print(f"  Average comments per post: {avg_comments:.1f}")
    
    # Flagged Content
    print("\n⚠️  FLAGGED CONTENT (inappropriate):")
    print("-" * 50)
    flagged = flag_inappropriate_content()
    if flagged:
        for item in flagged:
            print(f"  Post {item['post_id']}: {item['reason']}")
            print(f"    Text: {item['text'][:50]}...")
    else:
        print("  None found ✓")
    
    print("\n" + "="*50)
    print(f"Report generated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print("="*50 + "\n")

if __name__ == "__main__":
    print_analytics()
```

### Step 4: Update the URLs in your code

At the top of `analytics.py`, find this line:
```python
BASE_URL = "https://oracleapex.com/ords/social"
```

Replace `https://oracleapex.com/ords/social` with YOUR actual APEX REST base URL.

Example: `https://your-college-apex.com/ords/dba120l01`

### Step 5: Open Terminal in Codespaces

Click **"Terminal"** → **"New Terminal"** (in the top menu)

### Step 6: Run your script

In the terminal, type:
```bash
python3 analytics.py
```

Press Enter.

You should see:
```
==================================================
SOCIAL MEDIA ANALYTICS DASHBOARD
==================================================

📊 TOP POSTERS (by number of posts):
--------------------------------------------------
  #1 User 1: 2 posts
  #2 User 2: 1 posts
  ...
```

If you see this ✓ → **Part 3 Done**

---

## Part 4: Test Everything Together

### Step 1: Create a new post in APEX

1. Open your APEX app in browser
2. Click **"Create Post"** page
3. Fill in text: "Test post from APEX"
4. Click **"Create"**

### Step 2: Run Python script again in Codespaces

In the terminal, type:
```bash
python3 analytics.py
```

**Verify**: Your new post appears in the stats

### Step 3: Add a like

In APEX, create a new like (or ask instructor how to add one via the UI)

### Step 4: Run Python again

```bash
python3 analytics.py
```

**Verify**: Like count increased

---

## Submission Checklist

Turn in these screenshots:

1. **APEX Feed page** showing at least 3 posts
2. **APEX Create Post form** (filled out)
3. **APEX Leaderboard page**
4. **Codespaces terminal** showing Python output from `python3 analytics.py`
5. **Your `analytics.py` file** (copy-paste into a document)

---

## Grading

| Item | Points |
|------|--------|
| Database tables created + sample data | 25 |
| APEX website with 4 pages working | 25 |
| REST API endpoints created + return JSON | 25 |
| Python script runs in Codespaces and shows analytics | 20 |
| Submission complete | 5 |
| **Total** | **100** |

---

## If Something Breaks

**Problem**: Python says "Connection timed out"
**Fix**: Copy-paste your actual APEX REST URL at the top of `analytics.py` (ask instructor)

**Problem**: APEX pages don't show data
**Fix**: Make sure you ran the SQL script to create tables

**Problem**: Terminal says "python3: command not found"
**Fix**: Codespaces has Python pre-installed. Close terminal and open a new one.

**Problem**: JSON error or weird response
**Fix**: Open the URL in your browser first to see what it actually returns

**Otherwise**: Ask your instructor (they have the working example)

---

## What You Learned

✓ Database design (4 tables, relationships)
✓ APEX UI (creating web pages without coding)
✓ REST API (how frontend talks to database)
✓ Python backend (processing data, analytics)
✓ Full-stack application (all 4 parts working together)
✓ GitHub Codespaces (development in the cloud)

**This is professional-grade work.**
