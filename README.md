# xbox2local

Welcome to **xbox2local**, a command line utility written in Python for downloading all your Xbox Live [screenshots and game clips](https://support.xbox.com/help/friends-social-activity/share-socialize/capture-game-clips-and-screenshots) to your computer.
As of this writing, Microsoft only allows automatic uploads of screenshots and game clips to the Xbox Live service, where [subtle rules](https://support.xbox.com/help/games-apps/my-games-apps/manage-clips-with-upload-studio) dictate how long your content sticks around.
Game clips and screenshots can also be [shared](https://support.xbox.com/help/games-apps/my-games-apps/share-clips-xbox-one) to OneDrive for storage or Twitter for social sharing, but this must be done manually on a clip-by-clip basis.

The goal of **xbox2local** is to make the process of saving your screenshots and game clips much faster: each time you run it, it detects all your media on Xbox Live that it hasn't saved before and copies it to a local folder of your choosing.


## Getting Started

1. You'll need a command line (Unix-based, Windows Command Prompt, or macOS Terminal) and any [Python](https://www.python.org/downloads/) installation version 3.5 or newer. You will also need the [tqdm](https://github.com/tqdm/tqdm#installation) and [pathvalidate](https://github.com/thombashi/pathvalidate#installation) packages.

2. Have your Xbox Live (Microsoft) account email and password on hand. Both Free and Gold accounts are supported.

3. Navigate to [OpenXBL](https://xbl.io/), the API that **xbox2local** uses to interface with the Xbox Live service to download your media. Log in using your Xbox Live account.

4. On your OpenXBL [profile page](https://xbl.io/profile), scroll down to the box labeled "API KEYS" and press the "Create +" button. Copy the newly created API key (a string of letters, numbers, and hyphens) before navigating away from the page&mdash;we'll need it in a couple steps.

5. Clone this repository or download the latest [release](https://github.com/jdaymude/xbox2local/releases). Your directory structure will look like:
```
xbox2local
|--- users
|--- |--- .gitkeep
|--- ...
|--- xbox2local.py
```

6. Create a `users/<your username>` directory and within it create a `config.json` file with the following contents:
```
{
    "api_key": "your OpenXBL API key from https://xbl.io/profile",
    "media_dir": "folder to put your media"
}
```
Copy the *API Key* from your OpenXBL API [profile page](https://xbl.io/profile) into the `api_key` field and set the `media_dir` field to the local directory to download screenshots and game clips to (e.g., your OneDrive folder).

7. Run **xbox2local** with `python xbox2local.py --username <your username>`.


## Usage

**xbox2local** provides the following command line arguments:

- `--username <USERNAME>` allows you to specify an alternative user to load your config file (containing your OpenXBL API key and media directory), which is potentially useful if you have multiple accounts.

- `--media_type {screenshots, gameclips, both}` allows you to specify whether to download only screenshots, only game clips, or both. The default is both.

After running **xbox2local** at least once in which it succeeds in downloading your media, a `history.json` file will be created in your `users/<your username>` directory.
This file stores the ID of every screenshot and game clip that **xbox2local** has previously downloaded so that, on subsequent runs, it does not download duplicates.
If for whatever reason you want to start fresh and redownload all media stored by Xbox Live, simply delete/move this file.


## Troubleshooting

**xbox2local** does its best to notify you if anything goes wrong when communicating with the OpenXBL API or your Xbox Live account.
Some common errors include:

#### ERROR 401: X-Authorization header misssing or Invalid API Key

This means that either you did not provide your OpenXBL API key in `config.json` or the API key you provided is invalid.

#### ERROR 403: API Rate Limit Exceeded

This means that you have made more calls to OpenXBL API than your subscription plan allows.
TL;DR: this limit can be violated if you have a huge number of screenshots and game clips.
As of this writing, the free tier allows 150 requests per hour and there are larger quotas available via paid subscriptions.
This error potentially be circumvented by using the `--media_type` option to only download screenshots, waiting for the limit to reset in the next hour, and then using it again to only download game clips.
You can track your usage in real time on your OpenXBL [profile page](https://xbl.io/profile).

In detail, **xbox2local** makes the following calls to OpenXBL API each time it's run:
- One call to the `/api/v2/screenshots` endpoint per page of screenshot results (OpenXBL uses pagination to ensure that no single request is too big).
- One call to the `/api/v2/gameclips` endpoint per page of game clip results.

See Issue [#3](https://github.com/jdaymude/xbox2local/issues/3) for further discussion.

#### ERROR: media_dir path is invalid

This means that the `media_dir` path you provided in `config.json` is not valid for your platform (Linux, Windows, or macOS). The pathvalidate [validate_filepath()](https://pathvalidate.readthedocs.io/en/latest/pages/examples/validate.html#validate-a-file-path) function will print a more detailed error message.
Note: because sanitization rules differ slightly between platforms, running **xbox2local** with multiple command lines for the same media library may create different, similarly-named subfolders for the same game.

#### No new screenshots or game clips to download

This message is expected behavior when there really isn't anything new to download.
However, if you receive this message unexpectedly, check your *Settings > Preferences > Capture & Share > Automatically upload* settings on your Xbox.
This should be set to something other than *Don't upload*; otherwise, all screenshots and game clips you capture stay on your Xbox's local storage and are not uploaded to Xbox Live, so **xbox2local** cannot access them.


## Upgrading from Prior Versions

Updating an older version of **xbox2local** to a [new release](https://github.com/jdaymude/xbox2local/releases) requires a basic understanding of [semantic versioning](https://semver.org/), where version numbers are written as `MAJOR.MINOR.PATCH`.
If your older version and the updated version have the same `MAJOR` number, you can replace your `xbox2local.py` file with the [newest version](https://github.com/jdaymude/xbox2local/blob/master/xbox2local.py) and things should just work.
If you are updating to a new `MAJOR` version (e.g., from `v1.#.#` to `v2.#.#`), there may be additional changes you need to make manually.
The release notes for the corresponding major update (e.g., `v2.0.0`) will have instructions for those changes, if applicable.


## Contributing

If you'd like to leave feedback, feel free to open a [new issue](https://github.com/jdaymude/xbox2local/issues/new/choose).
If you'd like to contribute, please submit your code via a [pull request](https://github.com/jdaymude/xbox2local/pulls).
