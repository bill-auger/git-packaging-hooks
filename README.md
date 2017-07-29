
these files are a set of git hooks to semi-automate the following:

  * injecting semantic version strings into the program
  * releases on github
  * packaging for debian
  * packaging on the OpenSUSE Build Service


initial local configuration:

* ensure that the git-packaging-hooks are installed locally  
  => $ git config --local core.hooksPath /path/to/git-packaging-hooks/
* ensure that `sbuild` is installed and your user is in the 'sbuild' group
  => $ sudo apt install sbuild
  => $ sudo sbuild-adduser $USER


for each release version:

* ensure that the appropriate tag 'vMAJOR.MINOR' exists on the development branch  
  or add a new tag 'vMAJOR.MINOR' if major or minor version should change
* if the new tag 'vMAJOR.MINOR' was just added to the current HEAD  
  then amend that HEAD commit to trigger the git hooks
* verify that the git hook has put a tag of the form 'vMAJOR.MINOR.REV' on the HEAD  
  where REV is n_commits after the nearest 'vMAJOR.MINOR' tag
* checkout the 'packaging' branch to enable the packaging-specific git hooks
* rebase the 'packaging' branch onto the previous HEAD
* in debian/changelog  
  => add new entry for this version


if build or install steps have changed:

* in `$OBS_NAME`.spec.in  
  => update '%build' recipe, and/or '%post' install hooks
* in debian.rules  
  => update 'build-stamp:' and 'install:' recipes
* in PKGBUILD.in  
  => update 'build()' and 'package()' recipes  
  => $ gpg --detach-sign PKGBUILD


if output files have changed:

* in `$OBS_NAME`.spec.in  
  => update package '%files'


if dependencies have changed:

* in `$OBS_NAME`.spec.in  
  => update 'BuildRequires' and/or 'Requires'
* in `$OBS_NAME`.dsc.in  
  => update 'Build-Depends'
* in debian.control  
  => update 'Build-Depends' and/or 'Depends'
* in PKGBUILD.in  
  => update 'makedepends' and/or 'depends'  
  => $ gpg --detach-sign PKGBUILD


prepare packaging files:

* commit at least the changelog to trigger the git hooks  
  => $ git add --all  
  => $ git commit --allow-empty-message


_NOTE: after each commit to the `$DEVELOPMENT_BRANCH` branch:_

* the version string will be written into the configure.ac file
* any git tags of the form 'vMAJOR.MINOR.REV' that are not merged into master will be deleted
* a git tag 'vMAJOR.MINOR.REV' will be put on the HEAD  
  where REV is n_commits after the nearest 'vMAJOR.MINOR' tag


_NOTE: after each commit to the `$PACKAGING_BRANCH` branch:_

* all git tags are preserved
* the \_service, .spec, .dsc, and PKGBUILD files will be copied from their corresponding *.in templates
* version strings will be written into the \_service, .spec, .dsc, and PKGBUILD files
* checksums will be written into the .dsc and PKGBUILD files
* a tarball named 'PROJECT_MAJOR.MINOR.REV.orig.tar.gz' will be in the parent directory
* the .spec and .dsc recipes will be coupled to this tarball and checksums
* the PKGBUILD recipe will be coupled to this tarball, checksums, and signatures
* all files in the ./obs/ directory (except *.in) will be copied into the OSC directory
* the HEAD commit will be amended and signed
* the commit message will be 'update packaging files to vMAJOR.MINOR.REV' (a.k.a. `$COMMIT_MSG`)
* the development branch, packaging branch, and tags will pushed to github
* the PKGBUILD and signatures will be uploaded to the 'vMAJOR.MINOR.REV' github "tag release"
* the tarball, PKGBUILD, and signatures will be verified as identical to the github "tag release"
* rebasing and amend commits are non-eventful and will not trigger any of the above actions


packaging:

* OSC local build - e.g.  
  => $ osc build openSUSE_Tumbleweed i586   ./`$OBS_NAME`.spec  
  => $ osc build Debian_8.0          x86_64 ./`$OBS_NAME`.dsc
* tweak any of the preceding steps as necessary until everything rocks sweetly
* OSC commit to the OBS build server for production build


commit packaging files to git:

* duplicate any changes in the OSC directory in ./obs/ directory and amend commit
* checkout the master branch and fast-forward to the development branch  
  => $ git checkout master && git merge development
