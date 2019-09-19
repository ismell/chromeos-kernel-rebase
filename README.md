Incremental and "hermetic" rebase of chromeos kernel onto upstream kernel

All manual actions by the user are stored in this git database. This makes
it so all actions can be replayed. Given the same inputs you will get the
same output commit-tree. You won't get the exact commit shas because of
commit date.

This script can be run any time the target branch or src branch are
updated. The goal is to continuously be able to rebase from one release
candidate to the other.

# Getting Started

First create a rebase working directory

    cd ~/chromiumos/src/third_party/kernel/v4.19

    git worktree add ../rebase

    # Add the android remote
    git remote add android https://android.googlesource.com/kernel/common

    git fetch android

Now run the script to rebase the latest cros/chromeos-4.19 onto
android/android-mainline-tracking. You can do this anytime the src or dest
branches change. The branches are currently hard coded in the script.

    cd ~/chromiumos/src/third_party/kernel/rebase

    generate-rebase-list

Initially this will take a while as it builds up the cache data. Other
invocations will go faster.

The script is interactive, so if there is a problem it will ask you what
to do.

After a while you should have a rebased tree. You should commit the
changes to the database if the rebase looks good. This way there is a
history of the rebase work.

Directory Structure:
  * <rebase_id>/patch: When a commit fails to apply cleanly, it must be
    manually fixed. Once the script resumes, it will save off the patch
    into this directory so it can be replayed in a subsequent invocation.
  * <rebase_id>/orig: Original patch from the src tree. These are generated
    any time a patch is generated. They are useful so it's easy to diff the
    difference between the original patch and the manually rebased one.

    e.g., `meld orig/ patch/`
  * <rebase_id>/pre: A patch to apply before a commit is cherry-picked.
  * <rebase_id>/skip: SHAs to skip for some reason or another.
  * <rebase_id>/fixup: If the script is unable to reliable determine which
    commit a FIXUP: commit references, it will prompt the user.
    The response is saved here.
  * <rebase_id>/log: All the commits that were examined
    * exists-identical.csv: Commit was found upstream using the
      `(cherry-picked from commit XXXXX)` line. The patch id of the local
      commit matches the upstream commit.
    * exists-different.csv: Commit was found upstream using the
      `(cherry-picked from commit XXXXX)` line. The patch id of the local
      commit differs from the upstream commit. The local commit could have been
      back ported.
    * exits-patch-id.csv: The upstream commit was found by using the patch id
      of the local commit.
    * cherry-picked.csv: Commits that were cherry-picked without
      modification.
    * patched.csv: Commits that needed some kind of modification to apply
      or compile correctly.
    * skipped-automatic.csv: ANDROID and IWL7000 patches.
    * skipped.csv: Manually skipped changes. These were either not found
      automatically, are no longer needed, or require a more involved rebase.
  * <rebase_id>/work.log: A log of what happened to every commit. This contains
    the src and dest SHAs.
