# MediaMonkey-AddonBuildGuide
So, you just finished building your history-changing addon for MediaMonkey and are ready to share it with the world. The only thing left to do is to zip it and... _HOLD ON_!
Why should you, of all people, have to do that? Every keystroke that flows from your porcelain-like fingertips is a gift to the world, there is no reason to waste this endless potential on laborious and time-intensive tasks like zipping a bunch of files when you can instead command the __POWER OF THE INTERNET__ to do it for you!

(But honestly, woudn't it be cool if github could automatically create a new release containing a finished mmip everytime you check in new code?)

In the guide below we'll set up an automated workflow with github actions that takes all the files in your addon, creates an mmip, and attaches it to a (draft) release in the repository whenever you merge something to master.

## Setting up the workflow
### What you need to get started
- A github repository containing your addon
- A valid [info.json](https://www.mediamonkey.com/wiki/Getting_Started_(Addons)) file

### Creating the pipeline
To set up your pipeline, go to your repository and open the Actions tab. You will be presented with a list of premade workflows, ignore them an click on _set up a workflow yourself_ at the top of the list. The workflow editor will open, containing a generic template. Remove all of it and replace it with the contents of [PipelineExample.yml](PipelineExample.yml) in this repository.

Update the value of the EXTENSIONS_FILE_NAME variable in the env section in the beginning of the template from MyAddon to the name of your addon. Whatever you set here will later be used as file name for the generated .mmip file:
<img src=images/extensionfilename.png width=550 />

The pipeline assumes that all addon files are in the repository root, if they are in a subdirectory make sure to update all relevant instances of _${{ github.workspace }}_ accordingly. If the path to, say, info.json is /src/info.json, the corresponding path in the pipeline would be ${{ github.workspace }}/src/info.json.

That's it, we're done. Hit the _Start commit_ button in the top right corner of the window and commit the file either directly to master or into a new branch to merge it from there. Once the file is merged into master, reopen the Actions tab. You will see a (still empty) list of workflow runs. A few seconds later, the first pipeline run will show up. If you click the run, a page with some general information and list of published artifacts will open:

<img src=images/build_list.png width=600 />

By clicking the build job you can also see the full build log or see live progress of the build if it's still running:

<img src=images/build_details.png width=600 />

After the build is finished, open the releases section of the repository. It will contain a new draft release with the finished mmip as attachment. 

Congratulations, you just configured github to automatically build addon!

## Other Questions
### What does this pipeline actually do?
There are detailed explanations for each step in PipelineExample.yml, but the tl;dr is that the pipeline spins up a new github-hosted agent, clones your repository, does some file cleanup and parsing magic, and finally zips all contents of the repository into an mmip file which is then uploaded to a new draft release in the repository. Depending on how big your addon is, the workflow should take around 30 - 60 seconds to finish. Also see [here](https://github.com/features/actions) for an introduction to github actions.

### Are there any examples of this in action?
Checkout the repositories for [CodeMonkey](https://github.com/mmuffins/mediamonkey-codemonkey) and [Addon Menu](https://github.com/mmuffins/mediamonkey-addonmenu) which are using the pipeline from this example.

### Why should I bother? Doing all of this myself literally just takes five or so clicks
Yes, and now you can automate them. You are welcome. Should you really dislike the idea of building stuff in the cloud, have a look at [pack-mmip](https://github.com/JL102/pack-mmip) which is a shorthand script to pack an mmip offline. Also, you are probably not reading this guide in the first place.

### How does versioning work?
During the build the pipeline will automatically read the version property from the addon's info.json file. The major and minor version are left as-is, while the patch version is updated to match the build number of the last pipeline run, e.g. if the version property in your info.json is set to '2.1.0' the first build will generate an mmip with version number 2.1.1, the second build will have version number 2.1.2 and so on.
Github automatically tracks these build numbers in the background, so you can be sure that the each created mmip will always have a higher version number than all previous ones, therefore preventing collisions with already released versions.

### Are there any costs involved?
With a free github account you get a monthly allocation of free computing minutes. Unless you have several hundred commits a day, each day a month, you will not hit that limit. And if you still manage to do it somehow, worst case scenario is that you won't be able to run pipelines until the following month. See [here](https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions) fore more details about the monthly limits and usage calculation.
