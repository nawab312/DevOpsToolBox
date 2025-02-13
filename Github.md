Check if you have an SSH key: Run the following command to see if you already have an SSH key:
```bash
ls -al ~/.ssh
```
Generate a new SSH key (if necessary): If no key exists or you want to generate a new one, use the following command to create a new SSH key:
```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```
Add SSH key to the SSH agent: If the key is generated, run:
```bash
eval "$(ssh-agent -s)"
Agent pid 68394
```
Add SSH key to GitHub: Copy the SSH key to your clipboard:
```bash
cat ~/.ssh/id_rsa.pub
```
