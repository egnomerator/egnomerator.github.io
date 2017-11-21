## Helpful Links
[Change Author and Committer](https://stackoverflow.com/questions/4493936/could-i-change-my-name-and-surname-in-all-previous-commits)
* This link is to a Stack Overflow post requesting help to change author information on past commits

---

### Why Change Author Information of Past Commits
My personal use case for this action was the following:
* I had been working on a personal project for awhile making commits under certain author information
* I later decided to use a particular set of author information for all of my personal projects
* I began using my chosen author information for new commits on that personal project
* I realized I wanted to have my commit history show the same author information for all my commits
  * Since I was the only committer on the project, I wanted all the author information to be consistent

#### In case the link included above ever breaks, here is the pertinent information from the Stack Overflow post

##### Original poster's question:
> I would like to change my name, surname and email in my all commits, is it possible?

##### Answer (more than one answer was given, but I found this one to be most thorough):
> To rewrite both author and committer in all selected commits:
```
git filter-branch --commit-filter \
'if [ "$GIT_AUTHOR_NAME" = "OldAuthor Name" ]; then \
export GIT_AUTHOR_NAME="Author Name";\
export GIT_AUTHOR_EMAIL=authorEmail@example.com;\
export GIT_COMMITTER_NAME="Commmiter Name";\
export GIT_COMMITTER_EMAIL=commiterEmail@example.com;\
fi;\
git commit-tree "$@"'
```