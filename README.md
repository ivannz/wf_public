Setup the repos
---------------

```bash
# public mirror: make sure to create a public repo on github
git init wf_public
cd ./wf_public
git branch -m master main
git remote add origin https://github.com/ivannz/wf_public.git

cd ..

# private repo: make sure to create a private repo on github
git init wf_private
cd ./wf_private
git branch -m master main
git remote add origin https://github.com/ivannz/wf_private.git
git remote add public https://github.com/ivannz/wf_public.git

cd ..
```

Add workflow to the private repo
--------------------------------
Push some work into both repos:
```bash
cd ../wf_private

touch README.md
git commit -m'an empty readme'
git push -u origin main  # just to make sure everything syncs,
git push -u public main  # ensure repos have same git-tree
```

Then modify the names of the source and destination repos in `.github/workflow/git-sync.yml` as instructed in [git-sync-action](https://github.com/marketplace/actions/git-sync-action). We are using ssh-names:
```yaml
source_repo: "git@github.com:ivannz/wf_private"
```

Then push the workflow:
```bash
# in ../wf_private
git add .github/workflow/git-sync.yml
git commit -m'git-sync workflow'
```

Add secrets and public keys
---------------------------

```bash

# keys for wf_private and wf_public
mkdir .ssh
ssh-keygen -t rsa -b 4096 -C "username@email.org" \
  -f $HOME/Github/sandbox/wf_private/.ssh/id_rsa_wf_private

ssh-keygen -t rsa -b 4096 -C "username@email.org" \
  -f $HOME/Github/sandbox/wf_private/.ssh/id_rsa_wf_public
```

Then on github.com do:
1. add the unique public keys (\*.pub) to `Repo Settings > Deploy keys` for each repository respectively and allow *write access for the destination repository*:
    * copy contents of `.ssh/id_rsa_wf_private.pub` to deploy keys of `wf_private.git`
    * copy contents of `.ssh/id_rsa_wf_public.pub` to deploy keys of `wf_public.git` and *check the write access flag*

2. add the private key(s) to `Repo > Settings > Secrets` for the **repo containing the action** `wf_private.git`:
    * copy contents of `.ssh/id_rsa_wf_private` key to `SOURCE_SSH_PRIVATE_KEY` secret environment variable
    * copy contents of `.ssh/id_rsa_wf_public` key to `DESTINATION_SSH_PRIVATE_KEY` secret environment variable

