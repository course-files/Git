# Collaborative Git Workflows: SSH Key Generation for GitHub

An SSH key pair consists of a private key and a public key. The private key
should be kept secure in your local machine, while the public key will be
added to your GitHub account to allow for secure authentication.

Summary of key points:

* SSH replaces passwords
* GitHub trusts your machine, not your authentication session
* You can have multiple SSH keys for different purposes (e.g.,
one for authentication and another for signing commits). In this setup, we
use one SSH key pair for both authentication and signing commits to keep
it simple.
* You can have multiple SSH key pairs for different machines, e.g., one
for your personal laptop and another for your work laptop. This way, if
one of your machines gets compromised, you can easily revoke the
corresponding SSH key from your GitHub account without affecting the other
machine that has not been compromised.

***Note:** Ensure you are using the terminal for all Git operations in this
lab, not a graphical Git client like the built-in Visual Studio Code Git
support or GitHub Desktop.*

## Install OpenSSH Client (if not already installed)

### Check if already installed

`ssh -V`

If `ssh -V` returns a version number, you already have OpenSSH installed. If it returns an error, you will need to install it.

<img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/windows11/windows11-original.svg" width="40" />

Installing OpenSSH in Windows:

### Install the client (not the server)

```bash
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

<img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/linux/linux-original.svg" width="40" /> <img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/apple/apple-original.svg" width="40"/>

Installing OpenSSH Client in Linux/Mac is usually unnecessary as it is typically pre-installed. However, if you need to install it, you can use the following commands:

* Execute:

```bash
sudo apt update
sudo apt install openssh-client
```

Verify installation:

```bash
ssh -V
```

## Install the GitHub CLI and Log In to GitHub

Before setting up SSH, you will first authenticate to GitHub using the GitHub
CLI (`gh`). This confirms that you can access your GitHub account from the
terminal and gives you a working fallback (HTTPS) if the SSH setup fails.

***Note:** GitHub no longer accepts your account password for Git operations
in the terminal. Use `gh auth login`, which logs you in through your browser
instead.*

### Check if already installed

```bash
gh --version
```

If `gh --version` returns a version number, skip to
[Log in to GitHub](#log-in-to-github). If it returns an error, install it as
follows.

<img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/windows11/windows11-original.svg" width="40" />

Installing the GitHub CLI in Windows (run in PowerShell):

```bash
winget install --id GitHub.cli
```

If winget is not installed, you can bypass it and download and install GitHub CLI from here: [https://github.com/cli/cli/releases/latest](https://github.com/cli/cli/releases/latest)

<img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/apple/apple-original.svg" width="40"/>

Installing the GitHub CLI in macOS (requires [Homebrew](https://brew.sh/)):

```bash
brew install gh
```

<img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/linux/linux-original.svg" width="40" />

Installing the GitHub CLI in Linux (Debian/Ubuntu). Use GitHub's official
package repository, because the `gh` package in the default Ubuntu
repository is often outdated:

```bash
(type -p wget >/dev/null || (sudo apt update && sudo apt install wget -y)) \
	&& sudo mkdir -p -m 755 /etc/apt/keyrings \
	&& out=$(mktemp) && wget -nv -O$out https://cli.github.com/packages/githubcli-archive-keyring.gpg \
	&& cat $out | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null \
	&& sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg \
	&& sudo mkdir -p -m 755 /etc/apt/sources.list.d \
	&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
	&& sudo apt update \
	&& sudo apt install gh -y
```

For other operating systems, see the
[official installation instructions](https://github.com/cli/cli#installation).

After installation, **close and reopen** your terminal (Git Bash or
PowerShell) so that the `gh` command is recognized.

Verify installation:

```bash
gh --version
```

### Log in to GitHub

Switch to using Git Bash **only** if you are using Windows or the default Terminal if you are using Linux/MacOs from this point onwards. Execute:

```bash
gh auth login
```

Answer the prompts as follows (use the arrow keys to select and press
`Enter`):

| Prompt | Select |
|---|---|
| Where do you use GitHub? | `GitHub.com` |
| What is your preferred protocol for Git operations on this host? | `HTTPS` |
| Authenticate Git with your GitHub credentials? | `Yes` |
| How would you like to authenticate GitHub CLI? | `Login with a web browser` |

The terminal will display a one-time code. Press `Enter` to open your browser,
sign in to GitHub if prompted, paste the code, and click **Authorize**.

We select `HTTPS` at this point because the SSH key has not been created yet.
You will switch to SSH at the end of this lab.

***Note:** On Windows, if the arrow keys do not work in Git Bash when
answering the prompts, run `gh auth login` in PowerShell instead. The login
applies to both terminals.*

### Confirm that you are logged in

```bash
gh auth status
```

You should see a message similar to:

```text
github.com
  ✓ Logged in to github.com account <your_github_username> (keyring)
  - Git operations protocol: https
```

You are now authenticated to GitHub over HTTPS. The remaining steps set up SSH,
which will replace HTTPS for your Git operations.

## Step 1: Open Terminal (Git Bash on Windows or Default Terminal on Linux/Mac)

Execute the following command to generate a new SSH key pair.

```bash
ssh-keygen -t ed25519 -C "<place your comment here>" -f ~/.ssh/id_ed25519_auth_and_sign
````

* `-t ed25519` specifies the type of key to create, which is  [https://en.wikipedia.org/wiki/EdDSA](https://en.wikipedia.org/wiki/EdDSA). This is a modern and secure choice for SSH keys.

* The comment can contain the name of the machine. That will help you identify
which key was used and where it was used from, e.g., a key for your personal
laptop and another key for your work laptop in future.

* `-f` specifies the file name of the private key as well as the folder where it
will be saved. The public key will be saved with the same name but with a
`.pub` extension. For example, in this case, the private key will be saved as `~/.ssh/id_ed25519_auth_and_sign` and the public key will be saved as `~/.ssh/id_ed25519_auth_and_sign.pub`.

* `~` represents the home directory of the current user. On Windows, this translates to `C:\Users\YourUsername`.

After running the command, you will be prompted to enter a passphrase. You can
leave it empty for no passphrase or enter a secure passphrase for added
security. If you enter a passphrase, you will need to remember it to use the
key. In this case, we leave it empty for simplicity, however,
in a professional setting, it is recommended to use a passphrase for added
security.

## Step 2: Add the PUBLIC (.pub) Key to Your GitHub Account

Log in to your GitHub account, navigate to "Settings" > "SSH and GPG keys" >
"New SSH key" ([https://github.com/settings/keys](https://github.com/settings/keys)). Paste **all the contents** of your public key file
(`~/.ssh/id_ed25519_auth_and_sign.pub`) into the "Key" field and select
"Authentication Key" as the type.

Make sure that you are copy-pasting your public key, and **NOT** the private key.
Your private key (`~/.ssh/id_ed25519_auth_and_sign`) should never be shared with anyone.
The public key is safe to share and is used to authenticate your identity when
connecting to GitHub.

Give it a descriptive title, e.g., `Git authentication for GitHub from
<your_laptop_name>`. That way, if your laptop gets compromised, you can
easily identify which private key was being used for authentication and
revoke it from your GitHub account.

Click "Add SSH key" to save it.

Register the same public key again, but classify it as a signing key. This
 way, you can use the same SSH key pair for both authentication and
 signing commits. You can also choose to use different SSH key pairs for
 authentication and signing if you prefer, but using the same key pair
 simplifies the setup.

## Step 3: Add the SSH Key to the Local SSH Agent

First confirm that the SSH agent is running:

```bash
eval "$(ssh-agent -s)"
```

On Windows, if this command fails, confirm that the OpenSSH Authentication Agent service is running via `services.msc`.

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519_auth_and_sign.pub
git config --global commit.gpgsign true
```

Then add the private key to the SSH agent:

```bash
ssh-add ~/.ssh/id_ed25519_auth_and_sign
```

You can then confirm which SSH key is being used for signing commits:

```bash
git config --show-origin --get-regexp "gpg\.format|user\.signingkey|commit\.gpgsign"
```

## Step 4: Test the SSH Connection to GitHub

To verify that your SSH key is correctly set up and can authenticate with GitHub, run the following command:

```bash
ssh -T git@github.com
```

You should see a message like this:

```text
Hi <your_github_username>! You've successfully authenticated, but GitHub does not provide shell access.
```

This indicates that your SSH key is correctly configured and you can now use it for Git operations with GitHub.

---

However, if you see a message like this:

```text
Permission denied (publickey).
```

Or any other error message, you can troubleshoot the issue by running:

```bash
ssh -vT git@github.com
```

---

You should now be using SSH for both Git authentication when cloning repositories and for Git signing when committing changes.

Finally, tell the GitHub CLI to use SSH instead of HTTPS from now on:

```bash
gh config set git_protocol ssh --host github.com
```

Confirm that the change took effect:

```bash
gh auth status
```

The line `Git operations protocol` should now read `ssh`.

![Use SSH not HTTPS](./assets/images/UseSSH_notHTTPS.png)

The cloning command in this case would then be as follows to clone the repository using SSH and store the code in a folder named `Lab-1-Git`:

```bash
git clone git@github.com:course-files/Git.git Lab-1-Git
```

The syntax is:

```bash
git@github.com:<username>/<repository>.git <name-of-repository-in-your-local-machine>

# If you do not provide a value for <name-of-repository-in-your-local-machine>
# then it will use the name of the repository as the folder name by default.
```

This setup enhances the security of your interactions with GitHub.
