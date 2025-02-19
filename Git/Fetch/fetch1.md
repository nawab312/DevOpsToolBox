Created a New File
```bash
touch prac.txt
```
Moved the file to Staging Area
```bash
git add .
```
```bash
git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   prac.txt
```
Record the Changes in Local Repository
```bash
git commit -m "prac1 added"
[master (root-commit) 7377f26] prac1 added
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 prac.txt
```
```bash
~/git_practice/.git/refs/heads$ cat master 
ead1ebb687492171b35384d32c7fab72916ce7a5
```
Adds the URL of the remote repository to your local repository and Push the code to Remote Repo
```bash
git remote add origin git@github.com:nawab312/Git-Practice.git
```
```bash
git push origin master
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 217 bytes | 217.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:nawab312/Git_Practice.git
 * [new branch]      master -> master
```
```bash
~/git_practice/.git/refs/remotes/origin$ cat master 
ead1ebb687492171b35384d32c7fab72916ce7a5
```
Add another file **prac2.txt** to Github (Remote Repo). So it is not in our Local Repo
Fetch Remote Changes
```bash
git fetch origin
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Unpacking objects: 100% (3/3), 925 bytes | 925.00 KiB/s, done.
From github.com:nawab312/Git_Practice
   ead1ebb..0d9d7f4  master     -> origin/master
```
**View Fetched Changes:** Check Differences Between Local and Remote
```bash
git log master..origin/master --oneline --graph
* 0d9d7f4 (origin/master) Create prac2.txt
```
The above shows that prac2.txt was created in origin/master (Remote Repository)
```bash
git diff master origin/master
diff --git a/prac2.txt b/prac2.txt
new file mode 100644
index 0000000..8b13789
--- /dev/null
+++ b/prac2.txt
@@ -0,0 +1 @@
+
```
**git diff master origin/master** output shows that a new file, prac1.txt, has been added to origin/master but is not yet present in your local master branch.

