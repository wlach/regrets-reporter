<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

**Contents**

- [RegretsReporter Release Process](#regretsreporter-release-process)
  - [Testing builds from CI (or those built locally by a developer)](#testing-builds-from-ci-or-those-built-locally-by-a-developer)
    - [Firefox](#firefox)
    - [Chrome](#chrome)
  - [Create release builds](#create-release-builds)
  - [Upload and get approval of the artifacts](#upload-and-get-approval-of-the-artifacts)
    - [AMO (addons.mozilla.org)](#amo-addonsmozillaorg)
      - [Reproducing the Firefox CI build locally for add-on verification](#reproducing-the-firefox-ci-build-locally-for-add-on-verification)
    - [CWS - Chrome Web Store](#cws---chrome-web-store)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# RegretsReporter Release Process

## Testing builds from CI (or those built locally by a developer)

Artifacts built locally or via CircleCI are unsigned, and are installed as per follows:

### Firefox

- Make sure that the following preferences are set to `false` in `about:config`:
  - `xpinstall.signatures.required`
- Go to `about:addons`
- Download the latest Firefox-version of the extension (a xpi file)
- Click the gear icon, then choose `Install Add-on From File` and choose the extension xpi

### Chrome

- Download the latest Chrome-version of the extension (a zip file)
- Unpack the zip file locally
- Start Chrome
- Enter `chrome://extensions` in Chrome's address bar and press enter
- Flip the Developer mode switch up on the right so that the toolbar with the `Load unpacked`, `Pack extension` and `Update` buttons are shown
- Click `Load unpacked`
- Choose the directory that you unpacked from the zip file
- Note that the extension icon may not be visible directly. Click the puzzle icon far to the right of the address bar and click the pin symbol next to the extension icon so that the pin becomes blue. This will make the extension icon show at all times.

## Create release builds

- Create a new branch with the name: `release-x.y.z`
- If there are smaller additions/features to be included in this release that doesn't need a separate PR each for them, commit those changes to this branch
- Bump the version in `package.json`, using commit message `Bump version to x.y.z`
- Create a PR using the new branch with name `Release x.y.z` ([Example](https://github.com/mozilla-extensions/regrets-reporter/pull/18))
- Enter release notes in the PR's description
- Circle CI will produce extension builds that can be tested and/or uploaded to AMO and CWS (see below)
- Fix any smaller issues encountered during testing
- At some point, decide that the release is final and merge the PR (can be done the minute before uploading the new versions to AMO and CWS for instance)
- Create a new release using GitHub's interface, calling it `vx.y.z` (the version number prefixed with `v`)
- Continue below

## Upload and get approval of the artifacts

### AMO (addons.mozilla.org)

- Log in to AMO
- Go to https://addons.mozilla.org/en-US/developers/addons and click RegretsReporter
- Choose `Upload New Version`
- Upload the latest Firefox build created by Circle CI
- If not already done, make sure to merge the release PR above and create the new release at this point
- When asked to upload the source code, upload the zip file generated by GitHub attached to the new release
- AMO reviewers may want to see the release PR and/or a GitHub diff between the previously published version and the new release
- The new release will be live once AMO review is ready and approved

#### Reproducing the Firefox CI build locally for add-on verification

Due to the fact that webpack sometimes uses local paths in the builds and that build results may differ slightly
depending on which system it is built upon, we recommend using Docker to minimize the differences
between the builds built by CI and those built locally for add-on build process verification.

1. Start a Docker shell locally:

```
docker run -it -v "$(PWD)":/pwd circleci/node:latest-browsers /bin/bash
```

2. Run the following commands inside the container:

```
git clone https://github.com/mozilla-extensions/regrets-reporter.git /home/circleci/checkout
cd /home/circleci/checkout
yarn install --frozen-lockfile
cp .env.ci .env.production
yarn build:production
mkdir -p /pwd/docker-dist/firefox/
cp dist/firefox/* /pwd/docker-dist/firefox/
```

The reproduced build is now available in `docker-dist/firefox/` relative to the directory where the Docker shell was started.

### CWS - Chrome Web Store

- Log in to the CWS [Developer Dashboard](https://chrome.google.com/webstore/developer/dashboard)
- Choose the correct Publisher in the dropdown menu up to the right
- Click RegretsReporter
- `Package` -> `Upload new package`
- Upload the latest Chrome build created by Circle CI
- Adjust any info if necessary
- `Submit for review`
- After submitting you get to choose whether to publish the new version as soon as it is approved, or to manually publish the new version after it has been approved
- Wait for approval
- (Publish the new version if not previously chosen to be published automatically)
- The new release is live
