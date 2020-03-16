# concourse-semantic-release-demo
A test release to demo the value of semantic-release to my team

## Prerequisites
To run this demo, you will need:
* Access to a Concourse CI deployment (if you don't have one, you can run one locally with docker-compose following [Stark & Wayne's tutorial](https://github.com/starkandwayne/concourse-tutorial))
* Github Personal Access Token with `repo` privileges - generate [here](https://github.com/settings/tokens/new)
* Webhook token for [Semantic Release Slack bot](https://github.com/juliuscc/semantic-release-slack-bot) - generate [here](https://github.com/juliuscc/semantic-release-slack-bot#slack-app-installation)

## Setup
First of all, make a copy of this repo by hitting [Use this template](https://github.com/coro/concourse-semantic-release-demo/generate).
Then, clone the repo, and in the repo `set` the provided pipeline using the `fly` CLI:

```bash
fly \
  -t "$FLY_TARGET" \
  set-pipeline \
  -p concourse-semantic-release-demo \
  -c ./concourse/semantic-release.yml \
  --var git_uri="$GIT_URI" \
  --var github_token="$GITHUB_TOKEN" \
  --var slack_webhook="$SLACK_WEBHOOK_TOKEN"
```

## Workflow
We're going to be handling a pretty familiar workflow for a product from creation through development, including following semver, tagging releases in GitHub, creating dev prereleases & announcing releases on Slack. Our 'product' is just a poem, that we'll develop as we go along. 

### Initial release
First, let's create our poem. The first commit to trigger the semantic-releasing will be of type `feature`:

```bash
touch poem.txt
git add poem.txt
git commit -m "feat(poem): Created a new poem"
git push
```

Great! The concourse deployment will detect a new version of the repo on the `master` branch, and will trigger a new release. As this is the first release, it will be versioned as `1.0.0`. You should be able to see this under the `Releases` section of your repo in GitHub, and you should have received a Slack message celebrating your new release.

### New features
Our poem isn't that great - there's just no metre, or anything, at all! Let's fix that by adding our first new feature - the first verse.

```bash
echo "'Twas brillig, and the slithy toves
Did gyre and gimble in the wabe:
All mimsy were the borogoves,
And the mome rats outgrabe.
" >> poem.txt
git add poem.txt
git commit -m "feat(poem): Added first verse"
git push
```

This is a new **feature**, meaning this will increment the **minor** version of the release. You can see the version number in GitHub & Slack reported now as 1.1.0.

Let's add another feature, the second verse:

```bash
echo "\"Beware the Jabberwock, my son!
The jaws that bite, the claws that catch!
Beware the Jubjub bird, and shun
The furious Bandersnatch!\"
" >> poem.txt
git add poem.txt
git commit -m "feat(poem): Added first verse"
git push
```

And again, we can see our **minor** version was bumped; this time to 1.2.0.

### Bug fixing
Oh no! A user reported a bug in our beautiful poem. Our spellchecker must have been left on, because somehow a *real* word ended up in the poem; this dear user is outraged that you dared to misrepresent the Bandersnatch as anything other than **frumious**. Never mind, we'll fix that in a jiffy:

```bash
sed -i '' 's/furious/frumious/g' poem.txt # Assuming you're on MacOS...
git add poem.txt
git commit -m "fix(poem): Relabeled Bandersnatch as frumious"
git push
```

This time, we have a **fix** release, resulting in a **patch** version bump to 1.2.1.

### Breaking changes
Throw everything to the wind! You've decided that the easiest way to consume poems is line-by-line alphabetically. This unfortunately will break all existing readers who have built up their reading style based on the traditional 'chronological' API for storytelling. But nonetheless you plough on, knowing that the real future is alphabetical. 

```bash
sort poem.txt -o poem.txt
git add poem.txt
git commit -m "feat(poem): Poem now ordered alphabetically

BREAKING CHANGE: This will confuse any readers who are used to chronological storytelling"
git push
```

As this is a breaking change, the version increase will be a **major** bump, resulting in version 2.0.0.

### Maintenance branches
Some of the users have expressed reluctance to migrate to the new world. Namely, their service relies on the existance of verses, which no longer exist in v2. You'd like to continue support for these users, so you can create a maintenance branch to continue providing fixes to the `1.x` release track:

```bash
git co 1.2.1
git co -b 1.x
git push -u origin 1.x
```
**N.B.** This maintenance branch can also only provide patch releases if desired, e.g. `1.2.x`

Now, when a bug report comes in pointing out that mome rats sounds far too real to be in this poem, you can easily provide the fix and generate version numbers / release notes appropriately.

```bash
git co 1.x
sed -i '' 's/mome rats/mome raths/g' poem.txt # Assuming you're on MacOS...
git add poem.txt
git commit -m "fix(poem): mome rats are now mome raths"
git push
```

Since 1.x is [configured](https://github.com/coro/concourse-semantic-release-demo/blob/b02598ce135ff4eafbad204cf6437b704dc182c4/.releaserc#L3) as a monitored release branch, this will trigger semantic-release. Even though master is currently sitting on 2.0.0, the newly generated release version will be 1.2.2.

### Pre-release
What better way to get user feedback on upcoming additional lines in the poem than a pre-release of the poem? Since our branch `dev` is [configured](https://github.com/coro/concourse-semantic-release-demo/blob/b02598ce135ff4eafbad204cf6437b704dc182c4/.releaserc#L5) as a pre-release branch, any pushes to this branch will result in an incrementing pre-release identifier (as per [SemVer spec 2.0.0](https://semver.org/#spec-item-9)) with the same name as the branch.

```bash
git co master
git co -b dev
echo "He took his vorpal sword in hand;
Long time the manxome foe he sought—
" >> poem.txt
sort poem.txt -o poem.txt
git add poem.txt
git commit -m "feat(poem): Added two lines of third verse"
git push
```

As this is a pre-release, we'll end up with `2.0.0-dev.1`. Subsequent `feat` type commits will increment this number by 1 each time. You'll see this release is also tagged as a pre-release in GitHub.
