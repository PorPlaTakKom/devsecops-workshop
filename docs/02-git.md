# Git and GitHub Workshop

## Prepare Git Repository

* Config git user and email for commit user data first

```bash
git config --global user.name "student[X]"
git config --global user.email "student[X]@sit[Y].opsta.in.th"
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

## Practice Markdown

* <https://www.markdowntutorial.com>

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

## Push Repository to GitHub

### Add your SSH Public Key to GitHub

* On Cloud Shell

```bash
cat ~/.ssh/id_rsa.pub
# Copy your public key
```

> You can copy text on Cloud Shell by just drag on text your want to copy and it will copy to your clipboard automatically

* Go to <https://github.com>, Sign up and login with your credential
* Go to <https://github.com/settings/keys> or menu `Settings` on your avatar icon on the top right and choose menu `SSH and GPG keys` on the left
* Click on `New SSH key`
* Put `Title` of your public key and your key on the `Key` textbox and click `Add SSH key`

### Create your first repository

* Go to <https://github.com> and click on `New` button on the left
* Create rating project
  * Owner: `[Your Username]`
  * Repository name: `bookinfo-ratings`
  * Leave the rest default

### Add remote repository and push code

* Copy `git remote add origin ...` command in `…or push an existing repository from the command line` section
* Use command `git push -u origin master` to push code to GitHub On Cloud Shell

```bash
git remote add origin git@github.com:winggundamth/bookinfo-ratings.git
# To see remote repository has been added
git remote -v
git push -u origin master
# Maybe you need to answer yes for the first time push
```

* Refresh ratings main page on GitHub again to see change

## Adding ratings source code to repository

* On Cloud Shell, `mkdir src` to create src directory
* Copy [package.json](https://github.com/opsta/bookinfo/blob/main/src/ratings/package.json) and [ratings.js](https://github.com/opsta/bookinfo/blob/main/src/ratings/ratings.js) to your `src` directory
* Commit and push the code

```bash
git status
git add src
git status
git commit -m "Adding Ratings source code"
git push origin master
```

## Let's update your friend's source code

* Ask for username and repository url from your pair people
* It is on the menu `Code` > `SSH` in repository main page
* Add your friend for pushing code permission by
  * Go to `Settings` in repository main page
  * Click on `Manage access` menu
  * Click on `Invite a collaborator` button
  * Put your friend username
  * Click on `Add [username] to this repository`
  * Let your friend accept invitation by email
* Clone your friend's source code on Cloud Shell

```bash
# To make sure you are on home directory again
cd
pwd

# Clone repository into [username]-ratings directory
git clone git@github.com:[username]/bookinfo-ratings.git [username]-ratings
cd [username]-ratings
```

* Create new `LICENSE` file by copy MIT license from <https://choosealicense.com/licenses/mit/> and change `[year]` and `[fullname]`

* Commit and push your change

```bash
git status
git add LICENSE README.md
git status
git commit -m "Add License"
git push
```

* Change directory back to your code and update your code

```bash
cd ../bookinfo-ratings
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

* Add license section in `README.md` and push code

```markdown
# Bookinfo Rating Service

Rating service has been developed on NodeJS

## License

MIT License
```

## Protect your master branch from direct pushing

* Go to menu `Settings` > `Branches` on GitHub
* Click on `Add rule`
* Branch protection rule
  * Branch name pattern: `master`
  * Require pull request reviews before merging: `checked`
* This will allow no one to direct push to master branch but you have to change via pull request only

## Merge request 

* Checkout your friend repository to dev branch

```bash
cd ../[username]-ratings
git pull
git checkout dev
```

* Create new file name `.gitignore` and push these content

```gitignore
.project
.settings
id_rsa
```

> Watch out on Cloud Shell Text Editor that it won't show hidden file that have `.` as prefix filename

* This will prevent you from adding `.project`, `.settings`, and `id_rsa` file or directory. But you still can force add by using `git add -f`.
* Commit and push change
* Test .gitignore

```bash
cp ~/.ssh/id_rsa .
git status
git add .
git add id_rsa
git add -f id_rsa
git status
git reset HEAD
```

## Create Pull request

* Go to menu `Pull requests` on GitHub
* Click on `New pull request`
* Choose `base: master` <- `compare: dev`
* See the change and click on `Create pull request` button
* Put title and comment
* Choose `Reviewers` and `Assignees` to your friend
* Click on `Create pull request`

## Merge pull request

* Go to your own repository and go to `Pull requests`
* Go to pull request
* Click on Merge pull request

```bash
cd ../bookinfo-ratings
git pull
git checkout master
ls -la
git pull
ls -la
```

## Exercise

### Assignment 1

* Tell your username to TA to invite to <https://github.com/sit-2021-int209/index> repository
* Update this page <https://github.com/sit-2021-int209/index/blob/dev/docs/students.md>. Put your ID, Firstname, Lastname, GitHub and link to your repository by using index link. with following condition
  * UPDATE ON DEV BRANCH ONLY
  * Format table
  * Sort by ID

### Assignment 2

* Create your own public repository `int209-assignments`
* Create following structure

```text
.
├── assignments
│   └── 01-linux.md
├── LICENSE
└── README.md
```

* Choosing CC license from here <https://chooser-beta.creativecommons.org/> with following required
  * Anyone must give you a credit
  * You can not use to make your own money
  * You can not change my work
* Copy `Print Work or Media` text on the right and put to your `LICENSE` file

### Assignment 3

* Update `01-linux.md` to answer the [exercise](01-linux.md#excercise-file) last week in this format

````markdown
# Linux assignment

## 1. Excercise File

```bash
mkdir test
cd test
touch xxx
cp xxx yyy
```
````

Next: [Docker](03-docker.md)
