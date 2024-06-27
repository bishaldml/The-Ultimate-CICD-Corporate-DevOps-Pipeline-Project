Source Code Setup

### Step-1: Create a new repo

### Step-2: Implement token-based authentication
1. In github dashboard.
2. Goto Settings.
3. Click 'Developer settings'
4. Personal access tokens
5. Tokens(classical)
6. Delete previous-access-token (if any or use the previous token)
7. Generate new tokens
8. Give name: git_access_token
9. In Select Scope Tab:
10. Select all except delete permissions:
11. Click "Generate token" 
##### @Note: Once, it generates tokens, make sure to save it safely, bcoz after it is created it won't be visible.

### Step- Push Source code into the repo:
1. Clone git repo locally.
2. Push code to git repo as:
```
git add .
git commit -m "messages"
git push origin main
```
@Token will be ask to push the code.

