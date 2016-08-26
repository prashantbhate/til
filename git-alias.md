#Git Aliases

First time when I was introduced to git, I was really excited to used it. But when commandline kind of put me off, 
I started to look for support in IDE, and GUI tools which helped me adopt to git easily. But after a while when
I acquired the real taste of git, same GUI tools now appeared to slow me down. 

>git well soon !

Shouted my faster side of the brain

Eventually I had to `Tab` back to the darker-side of the IDE-GUI window, the command line.
But this time it wasn't that scary, infact I started loving the commands
`git checkout` ,`git stash` , `git pull`, `git branch`, `git log`, `git rebase` became friendlier than ever

*To such an extent that I sent few official mails with git commands in my signature , just kidding*

But still something wasn't right , I was keying in same commands, again and again, day in day out ..
My dreams those days were like repeat telecast of sequence of git commands again and agian,
>stash...pull...pop
>stash...pull...pop
>...

Until my *efficient brain/2* who was trying to sleep complained to its neighbour  *lazy brain/2* !!

Guess what they found out together, `git alias`,

I always knew they existed, but not realised how useful they were untill this happened.

## git aliases

Git aliases are shorthand to some long git command that you can assign to avoid typing long command,
but it does not end there but is the beginning. I will tell you why with a simple example

On a daily basis I keep my local git repo up-to-date, To do that I run below commands couple of times in a day.
I prefer to work on master branch unless there is need to work on multiple changes at the same time.
```
git pull
```
This works perfectly , But only if there are no local modifications

If there are any then `git` would complain
Then you will have to stash them, whick keeps local modifications somewhere safe,
Then once its stashed , you pull the code from origin and then you pop previously stashed code back
```
git stash
git pull
git pop
```
_I know this would put your repo in conflicting state if there are any overlapping changes in the same line of file,
and you will have to resolve conflicts. but assume there are None to keep matter simple ;)_

Now imagine, running same command again an again 100s of times and you will know the pain! thats where git alias comes in
to help you out.

Following command adds alias called `co` to your git config 
```
git config --global alias.co checkout
```
so that next time you run `git co` it replaces `co` with `checkout`
If you prefer to edit your git config file (normally stored in ~/.gitconfig ) in an editor you can run 
```
git config --global core.editor  "subl -n -w"
git config --global -e
```
I use sublime text on OSX but you can change it to whatever editor is your fav.


Now How does this helps to solve above problem?

`git alias` also allows to exectue any command if you prefix an alias with `!` ! 

that means you can add below line to your git config
```
alias.sp=!([[ -z $(git status --porcelain -uno) ]] && git pull) || (git stash && git pull && git stash pop)
```
_sp is my term for *stash & pull*_

This command checks if there are any local changes, if there are none it just runs `git pull` otherwise it runs `stash pull pop`
every time you run 
```
git sp
```

how cool is that ??

## my favorite aliases
This is an Exposé of my `~/.gitconfig`
```sh
	co = checkout
	ci = commit
	st = status
	sb = status -sb
	br = branch
  sns = show --name-status
  log1 = log -1
	ach = commit --amend -C HEAD
	aach = commit -a --amend -C HEAD
	sp = !([[ -z $(git status --porcelain -uno) ]] && git pull) || (git stash && git pull && git stash pop)
	hist  = log -20  --pretty=format:\"%h %ad [%an] | %s%d \" --graph --date=short
	lo    = log -25 --graph --decorate --all --oneline --topo-order
	graph = log  --graph --all --simplify-by-decoration --oneline --root --decorate
	type = cat-file -t
	dump = cat-file -p
	diffonly = !sh -c 'git diff | egrep \"^[+-]\" |grep -v \"^+++\"|grep -v \"^---\"  '
	alias = !sh -c 'git config --list | grep alias|sed "s_^alias.__"|sort '
```
_gt sb is somthing that I really like , it neatly puts local change without lot of text which you git with status, i mean get with_ `git status` :) 

##git alias to list all aliases
now you start with handful of aliases but soon the list grows, to an extent that you soon forget the
aliases that you had named few days/weeks back !
thats happended to me as well ! 

Well fear not see the last line above
```
	alias = !sh -c 'git config --list | grep alias | sed "s_^alias.__"|sort '
```
that lists all your aliases in a nicely sorted order.


Also don't forget to Keep these somewhere accessible over internet, that you can import 
while using git commandline on your colleague's laptop 
or be ready to feel `git amputated`

Well go ahead, order your copy of git alias now !


> 26 Aug 2016
