#### Definition
+ Git is distributed version control system.
#### Initialization
+ To init local Repo from GitHub use `git init` command. 
#### Clone
+ To clone Repo from GitHub use `git clone (link)` command. 
#### Status
+ To check the status of files or folders on your branch use `git status` command.
#### Add
+ To add file or folder to staging area use `git add (file or folder name, or file path) (file or folder name, or file path) ....` command.
+ To add all files and folders to staging area use `git add *` command.
#### Remove
+ To remove file or folder from staging area use `git reset head (file or folder name, or file path) (file or folder name, or file path) .... command.
+ To remove file or folder from staging area use `git restore --staged (file or folder name, or file path) (file or folder name, or file path) .... command.
+ To remove all files or folders from staging area use `git restore --staged *` command.
+ To remove more than one file in stage area use `git clean -n` command and then use `git clean -f` command.
#### Upload to local Repo
+ To upload files and folders to local repo use `git commit -m "Your message"` command.
#### Branch
+ To know your branch use `git branch"` command.
#### Remote Repo
+ To know your Remote use `git remote -v"` command.
#### Push
+ To push files and folders to remote repo use `git push Remote_Name Branch_Name` command.
+ To pull files and folders from remote repo and then push files and folders to remote repo use `git push -u (Remote_Name) (Branch_Name)` command.
#### Pull
+ To pull files and folders from remote repo to your local repo use `git pull Remote_Name` command.
+ This command combine two commands together, which are:
	+ `git fetch` command.
	+ `git merge` command.
#### Configuration
+ To see some of configurations use `git config --list` or `git config -l` command.
+ To see all configurations use `git help config` command.
+ To know the sources of configurations use `git config -l --show-origin` command.
+ To remove from configurations use `git config --global --unset "configuration"` command.
+ To edit configurations via editor use `git config --global --edit` command.
#### Alias
+ To make an alias use `git config --global alias.(name) (name)` command.
#### Branch
+ To create a branch use `git branch (Branch_Name)` command.
+ To move from a branch to another use `git checkout (Branch_Name)` command.
+ To delete a branch use `git branch -b (Branch_Name)` command.
+ To create a branch and move to it use `git branch -b (Branch_Name)` command.
+ To rename a branch use `git branch -m (Branch_Name)` command.
+ To merge a branch with master use `git merge (Branch_Name)` command.
#### Stash
+ To stash files use `git stash` command.
+ To stash files with a message use `git stash save "your message"` command.
+ To list items in stash use `git stash list` command.
+ To pop file from stash use `git stash pop` command.
+ To apply edits on file from stash use `git stash apply` command.
+ To pop a specific file from stash use `git stash pop stash@{id}` command.
+ To delete from stash use `git stash drop` command and for specific item use `git stash drop stash@{id}` command.
+ To see the file in stash use `git stash show` command and for specific item use `git stash show stash@{id}` command.
+ To clear stash use `git stash clear` command.