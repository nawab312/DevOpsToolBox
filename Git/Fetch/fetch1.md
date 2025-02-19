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
```bash
~/git_practice/.git/refs/heads$ cat master 
7377f26d7e2a754e71a5f528d5c6e9094f385b50
```
Adds the URL of the remote repository to your local repository and Push the code to Remote Repo
```bash
git remote add origin git@github.com:nawab312/Git-Practice.git
git push origin main
```
```bash
git_practice/.git/refs/heads$ ls -lart
total 12
drwxrwxr-x 4 siddharth312 siddharth312 4096 Feb 19 20:46 ..
-rw-rw-r-- 1 siddharth312 siddharth312   41 Feb 19 20:47 master
drwxrwxr-x 2 siddharth312 siddharth312 4096 Feb 19 20:47 .
```
