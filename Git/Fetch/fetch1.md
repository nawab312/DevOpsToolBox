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
~/git_practice/.git/refs/heads$ ls -lart
master
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

