🔹 Global Git Configuration
Scenario	Command
Set global username =====	git config --global user.name "Your Name"
Set global email =====	git config --global user.email "email@example.com"
View global config =====	git config --global --list
🔹 Push Local Repo to GitHub
Scenario	Command
Initialize repository	===== git init
Add remote repo =====	git remote add origin <repo-url>
Stage all files =====	git add .
Commit changes =====	git commit -m "message"
Push to GitHub =====	git push -u origin main

🔹 Scenario-Based Git Commands
1️⃣ Push Rejected (non-fast-forward)
Update local branch	===== git pull --rebase origin <branch-name>
Push changes =====	git push
2️⃣ Push Feature Branch Without Affecting main
Push feature branch	===== git push origin feature/feat-1
3️⃣ Sync Local Repo with Remote Updates
Fetch latest branches =====	git fetch origin
Checkout new branch	===== git checkout <branch-name>
4️⃣ Pull Changes Without Merge Conflicts
Save local changes	===== git stash
Rebase with main	===== git pull --rebase origin main
Apply stashed changes =====	git stash pop
5️⃣ Remove Sensitive File
Remove tracked file =====	git rm --cached path/to/file
Commit removal	===== git commit -m "Remove sensitive file"
Push changes =====	git push origin main

Remove from history
Rewrite history	===== git filter-branch --force --index-filter "git rm --cached --ignore-unmatch path/to/file" --prune-empty --tag-name-filter cat -- --all
6️⃣ Merge Latest main into Feature Branch
Switch to feature branch =====	git checkout feature/your-feature
Fetch updates =====	git fetch origin
Rebase main =====	git rebase origin/main
7️⃣ Push to Different Remote Repository
Add new remote =====	git remote add neworigin <repo-url>
Push to new remote =====	git push neworigin branch-name
8️⃣ Local Branch Behind Remote
Stash changes =====	git stash
Rebase	===== git pull --rebase origin branch-name
Restore changes =====	git stash pop
9️⃣ Resolve Merge Conflicts
Pull changes	===== git pull origin branch-name
Add resolved files =====	git add .
Commit =====	git commit
🔟 Delete Remote Branch
Delete branch =====	git push origin --delete feature/branch-name


----------------------------------------------------------------
✅ Collaborative Coding using Git — Cheat Sheet
🔹 1. Organization & Repository (GitHub UI)
Create Organization → Add Team
Create Private Repository (LocalHunt-01)
Add team to repo (Write access)
🔹 2. Clone Repository (Local Setup)
git clone https://github.com/<org>/LocalHunt-01.git
cd LocalHunt-01
🔹 3. Sync Local Repo
git checkout main
git pull origin main
🔹 4. Feature Branch Creation
git checkout -b feature/<name>
🔹 5. File Changes & Status
git status
git add .
git commit -m "message"
🔹 6. Push Feature Branch
git push -u origin feature/<name>
🔹 7. Create Pull Request (GitHub UI)
Repo → Pull requests
New pull request
Base: main
Compare: feature/<name>
🔹 8. Update Branch After Review
git add .
git commit -m "review changes"
git push
🔹 9. Merge Pull Request (GitHub UI)
Review approved
Click Merge pull request
🔹 10. Delete Feature Branch
git branch -d feature/<name>
git push origin --delete feature/<name>

=============================FORKING=========
FORK-BASED COLLABORATION (No Write Access)
🔹 11. Fork Repository (GitHub UI)
Repo → Fork
🔹 12. Clone Fork
git clone https://github.com/<username>/simple-repo-se.git
cd simple-repo-se
🔹 13. Add Upstream
git remote add upstream https://github.com/<owner>/simple-repo-se.git
git fetch upstream
🔹 14. Create Feature Branch in Fork
git checkout -b feature/<name>
🔹 15. Modify README / Files
git status
git add README.RD
git commit -m "docs: update README"
🔹 16. Push to Fork
git push -u origin feature/<name>
🔹 17. Pull Request (Fork → Original)
Fork → Pull requests
New pull request
Base: original repo main
🔹 18. Sync Fork with Original Repo
git checkout main
git fetch upstream
git rebase upstream/main
git push origin main





=====================================================================

1 — Create project folder & base compose file
Create folder and enter it:
mkdir compose-lab
cd compose-lab
Create docker-compose.yml (use editor or cat):
cat > docker-compose.yml <<'YAML'
version: "3.9"
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
YAML
2 — Start services (web + db)
Start in detached mode:
docker compose up -d
Check status:
docker compose ps
docker ps
Open in browser:
http://localhost:8080
Verify db container:
docker logs <db_container_name>
# or
docker compose logs db
3 — Add Redis and depends_on
Edit docker-compose.yml and add redis and depends_on for web:
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - redis

  db:
    image: postgres:15
    environment: ...

  redis:
    image: redis:alpine

Restart (recreate if necessary):
docker compose up -d
docker compose ps

4 — Add custom network & persistent volume
Update docker-compose.yml to include networks + volumes and assign services:

version: "3.9"

networks:
  app-net:

volumes:
  db-data:

services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    networks:
      - app-net
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-net

  redis:
    image: redis:alpine
    networks:
      - app-net
      
Apply changes:
docker compose up -d
docker volume ls
docker network ls
Verify data persistence:
# Stop and remove db container, then bring it back and check data remains
docker compose stop db
docker compose rm -f db
docker compose up -d db

6 — Add a buildable web app (Flask) for fast iteration
Replace web service with build instructions. Create app.py:
cat > app.py <<'PY'
from flask import Flask
app = Flask(__name__)
@app.route("/")
def home():
    return "Hello from Flask + Docker!"
if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
PY
Create Dockerfile:
cat > Dockerfile <<'DOCK'
FROM python:3.10-slim
WORKDIR /app
COPY app.py /app/
RUN pip install flask
CMD ["python", "app.py"]
DOCK
Update docker-compose.yml (web service):
web:
  build: .
  ports:
    - "5000:5000"
  depends_on:
    - db
  networks:
    - app-net
Build & run:
docker compose up --build -d
Visit:
http://localhost:5000
To iterate: edit app.py then rebuild:
# simple way:
docker compose up --build -d web
# or rebuild all
docker compose up --build -d
