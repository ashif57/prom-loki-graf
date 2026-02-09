pip install git-filter-repo
 git filter-repo --replace-text replacements.txt --force
 git push origin main --force
 git remote add origin https://github.com/ashif57/prom-loki-graf.git
 git remote set-url origin https://github.com/ashif57/prom-loki-graf.git
 git branch
======================
after removind the secrets from the repo
 git remote add origin <url>
Creates a new remote named origin and points it to the given URL.

Used when your repository doesn’t already have a remote called origin.

Example: first time connecting your local repo to GitHub.

git remote set-url origin <url>
Updates the URL of an existing remote named origin.

Used when origin already exists but you want to change its URL (e.g., switching from HTTPS to SSH, or moving to a new repo).

If origin doesn’t exist yet, this command fails with the error you saw: “No such remote 'origin'”.