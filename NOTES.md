- Everything Git-related is stored in the .git/ directory
- `HEAD` stores a reference to the head of the branch. In our case we only have `master`
- Git stores everything in the form of objects, which are stored based on their hash.
	This is a great design decision because it means that identical objects are stored only once, without
	unnecessary redundancy.
- Objects are stored in `.git/objects/` inside of a subdirectory named with the first two characters
	of their hash (e.g. `.git/objects/b0`)
- We can use `git cat-file` to get the type of an object by its hash with flag `-t`, as well as its actual content
	in human-readable format with flag `-p`

In a real attack, this would, of course, be done via an automated script.
Writing one is super simple, but will not be covered here. We're interested in how
Git works, not Python scripts, so we'll be doing things manually.

Commits actually reference their parent, not their child.
Once more, this is a good design decision as well. If commits
referenced their childred, the second commit would have kept 2 references
instead of one. And actualy, it would have to be able to handle
having more and more children. Having it this way, on the other hand,
ensures that each commit only has 1 reference to another, unless they
are the first commit (no parent), or a merge commit (2 parents). Any number of
commits can in turn reference the same parent, creating branches.

- When you `git add` files, their objects are created in `objects/`, but a
	commit object is not created.
- When you `git commit`, references are updated all over the place to
	reflect the changes in the branch. Most importantly, the previous `HEAD`
	(a commit) gets a reference to a new `HEAD`, and `.git/HEAD` will
	therefore point to the new `HEAD` via `.git/refs/heads/<branch_name>`.
- Branches, once again, are also just references on top of references. They share a common
	commit with another branch (such as `master`), which is why it is easy to inspect
	changes between branches when merging, for example. All Git needs to do is follow
	the references to the parent of each commit from the `HEAD`, until it reaches
	a commit where the split happened. Then, it follows the children of that commit
	where the split happened. Then, it follows the children of that commit on the other
	branch until it reaches `HEAD` and compares the trees. This can be done from any
	branch to any other branch, as they were all eventually a part of master.
- The point above also means that if you follow the references from `HEAD` of any branch
	downstream, you will eventually reach the first commit in your repository (except
	in special case of an orphan branch).
- Commits contain a reference to their parent, but most importantly, a reference to a tree
	that is a snapshot of the project after the commit. This is why we can actually look
	inth the past with Git, seeing the whole state of our project at any given commit.
- The point above also exposes a danger with Git, especially when paired
	with platforms like GitHub, or GitLab. If you do not explicitly delete
	objects and commits in the past, the entire past of the project will be
	visible by those with access to the repo. If the repo was previously private and then
	becomes public, its entire past will be available. Hence, if you ever add something you
	shouldn't have to a project (like an API key, or worse, a password), a new commit deleting
	it is not enough. You must actually delete the respective objects and commits. Luckily, Git
	does have some commands to help you do this so you don't need to go on running `rm` on a bunch
	of weird hash-named files.
