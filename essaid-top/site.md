
pre-site: prepare staging directory

1. Need a global flag to indicate the staging area is set so other modules don't overwrite it.
	staging.prepared, false by default.
2. Need a global staging root directory.
	staging.root=${settings.localRepository}/../staging-root
	override this in settings.xml for jenkins to be:
	staging.root=BUILD_DIR
3. Define a global default 
	stating.name=staging-job, 
	stating.git=false, 
	staging.branch=gh.pages
	staging.branch.dev=gh.pages.dev
	staging.cleanup=false
3. Define staging.job=${staging.root}/${staging.name}/${maven.build.timestamp}
4. Define staging.version=${staging.job}/version/${project.version}
	this is the directory where the staging will occur
	
5. check if under Git versioning. set staging.git=true
6. if staging.git=true
	create new orphaned branch ${staging.branch.dev} if not there
	set staging.git.dir=readlink -e $(git rev-parse --git-dir)
	local clone, and checkout dev staging branch to ${staging.version}
	remove all files except /.git and /version
	remove ${staging.version}
	
site: build and stage the maven sites to ${staging.version}
1. the project module that has the root side content copies it, filters it, etc. to ${staging.job}
	this should add an appropriate .gitignore file
2. run any generation on ${staging.job}, such as jekyll, etc. to have the site fully generated and ready for use

post-site: if ${staging.git}=true, commit the site to ${staging.branch.dev}
1. rewrite git history to remove any previous ${staging.version} paths.
2. add and commit all of ${staging.job}
3. push to local +${staging.branch.dev}:${staging.branch.dev}
4. if staging.cleanup=true, delete ${staging.job}

site-deploy: set ${staging.branch} to ${staging.branch.dev}
	this should probably be a manual process after some review
	disable the default deploy goal.
