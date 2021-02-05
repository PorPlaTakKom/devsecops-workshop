# Git and GitLab Workshop

## Prepare Git Repository

* Config git user and email for commit user data first

```bash
git config --global user.name "training[X]"
git config --global user.email "training[X]@opsta.net"
# See your git config
git config --list
```

* Initial ratings service repository by putting commands below

```bash
mkdir ratings
cd ratings
# Initial git repository
git init
```

* Click on `pencils icon` on the top right to open text editor
* Right click on ratings folder and `New File` to create `README.md` file and put below text

```markdown
# Bookinfo Rating Service

Rating service has been developed on NodeJS
```

## First Git Commit

* Put command `git status` to see repository status
* Put commands below for first commit

```bash
git add README.md
git status
git commit -m "Initial commit"
git status
```

* Put command `git log` or `git log --oneline` to see history of commit

## Push Repository to GitLab

### Add your SSH Public Key to GitLab

* On Cloud Shell

```bash
cat ~/.ssh/id_rsa.pub
# Copy your public key
```

> You can copy text on Cloud Shell by just drag on text your want to copy and it will copy to your clipboard automatically

* Go to <https://git.demo.opsta.co.th> and login with your credential
* Go to <https://git.demo.opsta.co.th/profile/keys> or menu `Settings` on your avatar icon on the top right and choose menu `SSH Keys` on the left
* Put your public key on the `Key` textbox and click `Add key`

### Create your own subgroup

* Go to `Groups` > `Your groups` menu on the top left
* Click on `AIS DevSecOps Bootcamp` group
* Click on `small arrow down` next to the right of `New project` green botton on the top right and choose `New subgroup`. Then click on `New subgroup` button.
* Create your own group
  * Group name: `training[X]`
  * Group URL: `training[X]`
  * Leave the rest default

### Create your first project

* Click on your newly created subgroup `training[X]`
* Click on New project
* Create rating project
  * Project name: `ratings`
  * Project URL: `adb/training[X]`
  * Project slug: `ratings`
  * Leave the rest default

### Add remote repository and push code

* Copy `git remote add origin ...` command in `Push an existing folder` section
* Use command `git push -u origin master` to push code to GitLab On Cloud Shell

```bash
git remote add origin git@git.demo.opsta.co.th:adb/training[X]/ratings.git
# To see remote repository has been added
git remote -v
git push -u origin master
# Maybe you need to answer yes for the first time push
```

* Refresh ratings main page on GitLab again to see change

## Adding ratings source code to repository

* On Cloud Shell, `mkdir src` to create src directory
* Copy [package.json](../src/ratings/package.json) and [ratings.js](../src/ratings/ratings.js) to your `src` directory
* Commit and push the code

```bash
git status
git add src
git status
git commit -m "Adding Ratings source code"
git push origin master
```

## Let's update your friend's source code

* Ask for repository url from people next to you. It is on the menu `Clone` > `Clone with SSH` in repository main page
* Clone your friend's source code on Cloud Shell

```bash
# To make sure you are on home directory again
cd
pwd

# Clone repository into training[X]-ratings directory
git clone git@git.demo.opsta.co.th:adb/training[X]/ratings.git training[X]-ratings
cd training[X]-ratings
```

* Add license section in `README.md` of your friend's repository

```markdown
# Bookinfo Rating Service

Rating service has been developed on NodeJS

## License

MIT License
```

* Commit and push your change

```bash
git status
git add README.md
git status
git commit -m "Add License"
git push
```

* Change directory back to your code and update your code

```bash
cd ../ratings
git pull
```

## Create dev branch for develop

* Put these commands to create dev branch

```bash
git branch
git branch dev
git branch
git checkout dev
git branch
git push origin dev
```

## Protect your master branch from direct pushing

* Go to menu `Settings` > `Repository` on GitLab
* Expand `Protected Branches`
* Change `Allowed to push` to `No one`
* This will allow no one to direct push to master branch but you have to change via merge request only

## Change setting not to delete source branch by default

* Go to menu `Setting` > `General` on GitLab
* Expand `Merge requests`
* Unchecked `Enable 'Delete source branch' option by default`
* Click `Save changes`

## .gitignore file

* Create new file name `.gitignore` and push these content

```gitignore
.project
.settings
```

> Watch out on Cloud Shell Text Editor that it won't show hidden file that have `.` as prefix filename

* This will prevent you from adding `.project` and `.settings` file or directory. But you still can force add by using `git add -f`.
* Commit and push change

## Merge Requests

* Go to menu `Merge Requests` on GitLab
* Click on `Create merge request`
* See `Commits` and `Changes` tabs. You can leave everything default and click on `Submit merge request`
* Since you are maintainer, you can click on `Merge` button to merge code from `dev` to `master` branch.
* See your changes on master branch

Next: [Docker Workshop](03-docker.md)
