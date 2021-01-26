# Git submodules

Submodules can be used to work on a project that has a dependency from other project, which we are also working at the same time. This is a repository embedded in your main repository.

* You can separate the code into different repositories.
* You can add the submodule to multiple repositories.

## Table of Contents
1. [Configure a Submodule on a Project](#Configure-a-Submodule-on-a-Project)
    * [Git configuration](#Git-configuration)
    * [Add repo as submodule](#Add-repo-as-submodule)
2. [Working on a Project with Submodules](#Working-on-a-Project-with-Submodules)
    * [Modify main repo and submodule](#Modify-main-repo-and-submodule)
    * [Update with the latest changes](#Update-with-the-latest-changes)
3. [Activate python develop mode](#Activate-python-develop-mode)
4. [Remove submodule](#Remove-submodule)
5. [Final notes](#Final-notes)

    
    

## Configure a Submodule on a Project

### Git configuration

Copy these configurations in your $HOME/.gitconfig file. This configuration will help you to work with git submodules.

```ini
[user]
    name = YOUR NAME
    email = USER@SOMETHING.COM
[github]
    user = USERNAME
[alias]
# Alias for submodules
    spull = !git pull && git submodule update --init --remote --merge
    spush = push --recurse-submodules=on-demand
# Modifications to work with submodules
[status]
    submoduleSummary = true   # show you a short summary of changes to your submodules
[diff]
    submodule = log           # shows changes in modules
[submodule]
    recurse = true            # pulls changes in modules
```

### Add repo as submodule

**STEP 1**

**A)**

Clone repos in your local computer.

* https://github.com/eegkno/taco
* https://github.com/eegkno/ingredients

```bash
mkdir $HOME/tmp
cd $HOME/tmp
git clone https://github.com/eegkno/taco.git 
git clone https://github.com/eegkno/ingredients.git
```
**B)**

Create a repo in your own github account with the name ```taco``` and ```ingredients```. DO NOT initialize the repos, just click on ```Create repository```.

**C)**

Set the url to your own repositories.


```bash
git remote set-url origin URL
git push
```

**D)**

Check that the repos are pointing to the correct location.

```bash
git remote -v
```

**E)**
Create develop branches in each repo


**STEP 2**

Add submodule in main repo.

```bash
cd path/to/taco
git checkout -b develop
git submodule add -b develop https://url/to/ingredients.git src/ingredients
```

* -b develop: to indicate which branch to track
* src/ingredients: is the path where the submodule is located

**output:**

```
Cloning into '/Users/gkno/Github/taco/src/ingredients'...
remote: Enumerating objects: 14, done.
remote: Counting objects: 100% (14/14), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 14 (delta 2), reused 14 (delta 2), pack-reused 0
Unpacking objects: 100% (14/14), done.
```

**STEP 3**

After this operation, if you do a git status you will see two files in the Changes to be committed list: the .gitmodules file and the path to the submodule. 

```bash
git status
```

**output:**
```
On branch master
Your branch is up to date with 'origin/master'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   .gitmodules
	new file:   src/ingredients

Submodule changes to be committed:

* src/ingredients 0000000...0ba8466 (2):
  > Update setup.py

```

Check module


```bash
cat .gitmodules 
```

**output:**
```
[submodule "src/ingredients"]
	path = src/ingredients
	url = https://github.com/eegkno/ingredients.git
	branch = develop
```

**STEP 4**

These files can now be committed to the original repository.

```bash
git add .gitmodules src/ingredients
git commit -m "Add ingredients as submodule"
git push --set-upstream origin develop
```

After this, the repo *taco* should have the module ingredients in src/ingredients.

**STEP 5**

Test that the library is working. 

Create file in ingredients.

```python
#source: ingredients/utils/all.py

def onion():
    print("Add onion")

```

Add and commit changes

```bash
git add ingredients/utils/all.py
git commit -m "Add onion"
git push
```

Create file in taco.

```python
#source: taco/notebooks/test_taco.py
import pathlib
import sys

# Add local module
module_path = pathlib.Path.cwd() / pathlib.Path("src/ingredients")
sys.path.insert(0, module_path.as_posix())

from ingredients.utils.all import onion

if __name__ == "__main__":
    onion()
```

Check status

```
git status

```

**output:**

```git
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   src/ingredients (new commits)

Submodules changed but not updated:

* src/ingredients 0ba8466...cf83ec1 (1):
  > Add onion

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	notebooks/test_taco.py

```

Add and commit changes

```bash
git add notebooks/test_taco.py 
git commit -m "Create first taco"
git add src/ingredients
git commit -m "Update module"
git push
```

Run  test_taco.py

```bash
python notebooks/test_taco.py 
```


**output:**

```
Add onion
```

**What happen is the branch is changed in src/ingredients, from develop to master?**

## Working on a Project with Submodules

### Modify main repo and submodule

* When a project with submodules is cloned using *git clone*, it creates the directories that contain submodules, but none of the files within them. 

* The submodule files are not created until two additional commands are run:
   * **git submodule init** will update the local .git/config with the mapping from the .gitmodules file. 
   * **git submodule update** will then fetch all the data from the submodule project and check out the mapped commit in the parent project.
   
*Option 1*

Order to run the commands:

```bash
git clone /url/to/repo/with/submodules
git submodule init     # This will pull all the code from the submodule and place it in the directory that it is configured to.
git submodule update   # To update the code of your submodules
```

*Option 2*

The last three commands are equivalent to:

```bash
git clone --recurse-submodules url/to/repo/with/submodules
```  

**STEP 1**

Cloning a repo using the second option.

**example:**

```bash
cd $HOME/tmp
git clone --recurse-submodules -b develop url/to/taco.git taco2
```    
**output:**

```
Cloning into 'taco'...
remote: Enumerating objects: 22, done.
remote: Counting objects: 100% (22/22), done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 22 (delta 2), reused 22 (delta 2), pack-reused 0
Unpacking objects: 100% (22/22), done.
Submodule 'src/guacamole' (https://github.com/eegkno/guacamole.git) registered for path 'src/guacamole'
Cloning into '/Users/gkno/Github/taco/src/guacamole'...
remote: Enumerating objects: 22, done.        
remote: Counting objects: 100% (22/22), done.        
remote: Compressing objects: 100% (13/13), done.        
remote: Total 22 (delta 8), reused 19 (delta 5), pack-reused 0        
Submodule path 'src/guacamole': checked out 'b3c6b30e4e77e5a887e66891732074fc783e77d0'
```

Now we have a copy of a project with submodules in it and will collaborate with our teammates on both the main project and the submodule project.

After clonning with submodules, you should be able to run:

```bash
python notebooks/test_taco.py
```

**STEP 2**

Doing some changes in taco and ingredients.

First, checkout to a new branch to work on.

```bash
cd path/to/ingredients
git checkout -b develop
```

Then, modify the script.

```python
#source: ingredients/utils/all.py

def onion():
    print("Add onion")

def guacamole():
    print("Add guacamole")

```

Add and commit changes

```bash
git add ingredients/utils/all.py
git commit -m "Add guacamole"
git push
```

Update taco.

```bash
cd path/to/taco2
```

```python
#source: taco/notebooks/test_taco.py
import pathlib
import sys

# Add local module
module_path = pathlib.Path.cwd() / pathlib.Path("src/ingredients")
sys.path.insert(0, module_path.as_posix())

from ingredients.utils.all import onion, guacamole

if __name__ == "__main__":
    onion()
    guacamole()
```

Add and commit changes

```bash
git add notebooks/test_taco.py
git commit -m "Add new ingredient"
git add src/ingredients
git commit -m "Update module"
git push
```

### Update with the latest changes

* When you add the submodule, the most recent commit of the submodule is stored in the main repository’s index. 
* That means that as the code in the submodule’s repository updates, the same code will still be pulled on the repositories relying on the submodule.

By running the following command, git will go into your submodules and fetch and update for you

```bash
git submodule update --remote --merge
```
* Using the “–remote” command, you will be able to update your existing Git submodules without having to run “git pull” commands in each submodule of your project.
* With --merge, Git will just update the submodule to whatever is on the server and reset your project to a detached HEAD state.
* When using this command, your detached HEAD will be updated to the newest commit in the submodule repository.

This is similar to:

```bash
cd path/to/submodule
git fetch
git merge origin/branch
```

A more convenient command is **spull**. This alias pulls all the changes from the main repo and submodules.

```
git spull
```

Move back to taco repo 
```bash
cd $HOME/tmp/taco
git spull
```


**output:**

```
remote: Enumerating objects: 11, done.
remote: Counting objects: 100% (11/11), done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 7 (delta 3), reused 7 (delta 3), pack-reused 0
Unpacking objects: 100% (7/7), done.
From https://github.com/eegkno/taco
   d6123d2..ed7c958  develop    -> origin/develop
Fetching submodule src/ingredients
From https://github.com/eegkno/ingredients
   bd5e5d0..3cb2d6c  develop    -> origin/develop
Updating d6123d2..ed7c958
Fast-forward
 notebooks/test_taco.py | 3 ++-
 src/ingredients        | 2 +-
 2 files changed, 3 insertions(+), 2 deletions(-)
Submodule path 'src/ingredients': checked out '3cb2d6cacff83b358d8fa87bce57fc1f98d5d0a1'
```

Check status

```bash
git status
```

**output:**

```
On branch develop
Your branch is up to date with 'origin/develop'.

nothing to commit, working tree clean
```

If the command ```git submodule update --remote``` is used without the option merge, then it will necessary to add and commit updating the lates changes.


## Activate python develop mode

As final step, we will install the submodule in python developmeny mode. Development mode creates a link from the submodule in the virtual environment. This will allows to use the submodule as any other python package. Also, when we make any change in the code, the change takes effect immediately, and are tracked.

To install the submodule as a python package in development mode, use the command:

```bash
pip install -e path/to/submodule
```

Let's install ingredients as a python package, run the following package.

```bash
pip install -e src/ingredients
```

Let's create ```taco/notebooks/test_taco_2.py``` as follows. By adding the module as a python package, we do not need to include the submodule in ```sys.path``.

```python
#source: taco/notebooks/test_taco_2.py
import pathlib
import sys

from ingredients.utils.all import onion, guacamole

if __name__ == "__main__":
    onion()
    guacamole()
```

**output:**

```
Add onion
Add guacamole
```


Add and commit updates

```bash
git add notebooks/test_taco_2.py
git commit -m "Use ingredients as python package"
```

Add the method souce in ```ingredients/utils/all.py```.

```
git checkout develop
git pull
```

```python
#source: ingredients/utils/all.py

def onion():
    print("Add onion")

def guacamole():
    print("Add guacamole")
    
def sauce():
    print("Add sauce")

```

Modify ```taco/notebooks/test_taco_2.py``` by including a call to ```sauce()```.

```python
#source: taco/notebooks/test_taco_2.py
import pathlib
import sys

from ingredients.utils.all import onion, guacamole, sauce

if __name__ == "__main__":
    onion()
    guacamole()
    sauce()
```

Now, go back to *taco2*, and run ```spull``` and then ```python notebooks/test_taco_2.py```. 

**output:**
```
Add onion
Add guacamole
Add sauce
```

## Remove submodule

```bash
git submodule deinit path/to/module
git rm path/to/module
```
## Final notes

* Once submodules are properly initialized and updated within a parent repository, they can be used exactly like stand-alone repositories.This means that submodules have their own branches and history.
* When making changes to a submodule, it is important to publish these changes and then update the parent repositories reference to the submodule.
* You should also let the main repository know that you have updated the submodule's repository, and make it use the latest commit of the repository of the submodule. If you want to have these changes in your main repository too, you should tell the main repository to use the latest commit of the submodule.
