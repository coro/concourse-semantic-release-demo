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
And the mome raths outgrabe.
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

And again, we can see our minor version was bumped; this time to 1.2.0.

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
Throw everything to the wind! You've decided that the easiest way to consume poems is line-by-line alphabetically. This unfortunately will break all existing readers who have built up their reading style based on the traditional 'chronological' API for storytelling.

```bash
sort poem.txt -o poem.txt
git add poem.txt
git commit -m "feat(poem): Poem now ordered alphabetically

BREAKING CHANGE: This will confuse any readers who are used to chronological storytelling"
git push
```

As this is a breaking change, the version increase will be a **major** bump, resulting in version 2.0.0.

### Maintenance branches
