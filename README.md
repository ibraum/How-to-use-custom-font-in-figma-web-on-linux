# How to Use Custom Fonts in Figma Web on Linux

## Introduction

Using Figma on Linux is a convenient option for designers who prefer working in an open-source environment. However, one limitation can quickly become frustrating: **making locally installed fonts available in Figma Web**.

Unlike desktop applications that can directly access fonts installed on the operating system, Figma Web relies on a font service to detect and use local fonts. Depending on your Linux environment and browser configuration, this service may not work as expected.

As a result, you may install a font on your machine, verify that it is available to other applications, and still be unable to find it in Figma.

This guide explains how to work around this limitation using a local font helper built with Deno, together with a browser extension that changes the browser's User-Agent.

By the end of this tutorial, you will have learned how to:

- Install custom fonts on Linux.
- Verify that your system recognizes the installed fonts.
- Install Deno, the JavaScript and TypeScript runtime required by the helper.
- Clone and configure the Figma font helper.
- Configure your browser to use a Windows/Chrome User-Agent for Figma.
- Start the helper and troubleshoot common errors.

> **Note:** This tutorial describes a workaround for Figma Web on Linux. Its effectiveness depends on your browser, system configuration, the helper's implementation, and Figma's current behavior. Changing the User-Agent alone does not guarantee that font detection will work.

---

## Prerequisites

Before getting started, make sure you have:

- A Linux distribution such as Ubuntu.
- Google Chrome or a compatible Chromium-based browser.
- A custom font file, usually provided in `.ttf` or `.otf` format, or inside a ZIP archive.
- A terminal.
- An internet connection to download the required tools and clone the repository.

The commands below are intended for a typical Ubuntu-based environment. Some paths may differ depending on your distribution and language settings.

---

## 1. Install your custom font on Linux

The first step is to install the font on your system.

Linux allows users to install fonts either system-wide or for a specific user. In this tutorial, we will use the second option because it does not require administrator privileges.

### 1.1. Create a directory for your fonts

Open a terminal and execute:

```bash
mkdir -p ~/.local/share/fonts/
```

Let's break down this command:

- `mkdir` creates a directory.
- `-p` creates any missing parent directories and avoids an error if the directory already exists.
- `~` represents your home directory.
- `.local/share/fonts/` is a conventional location for fonts installed by an individual user.

Your font directory will therefore be located at:

```text
/home/your-username/.local/share/fonts/
```

Replace `your-username` with your actual Linux username if you write the path manually. You do not need to replace `~` in the command itself.

### 1.2. Extract your font archive

Suppose you downloaded a font family named `reckless-font-family.zip`.

If your downloads directory is named `Downloads`, extract the archive with:

```bash
unzip ~/Downloads/reckless-font-family.zip \
  -d ~/.local/share/fonts/reckless/
```

If your system uses a French-language directory named `Téléchargements`, use:

```bash
unzip ~/Téléchargements/reckless-font-family.zip \
  -d ~/.local/share/fonts/reckless/
```

The `unzip` command extracts the archive, while `-d` specifies the destination directory.

**Important:** Use the path that actually exists on your machine. You can check your home directory with:

```bash
ls ~
```

If `unzip` is not installed, install it with:

```bash
sudo apt update
sudo apt install unzip
```

If your font files are already extracted, you can copy the `.ttf` and `.otf` files directly into the fonts directory instead.

### 1.3. Refresh the font cache

After extracting the files, refresh the font cache so that applications can discover the newly installed fonts:

```bash
fc-cache -f -v
```

Here, `fc-cache` updates the font cache, `-f` forces a refresh, and `-v` displays detailed information.

This step is useful when you have just installed a new font and want to verify that your system recognizes it.

### 1.4. Verify that the font is installed

Run:

```bash
fc-list | grep -i reckless
```

The command searches the list of fonts recognized by Linux for entries containing `reckless`, without distinguishing between uppercase and lowercase letters.

If the installation was successful, you should see one or more lines containing the font's name and file path.

For example, the output may resemble:

```text
/home/your-username/.local/share/fonts/reckless/Reckless-Regular.otf
```

Your actual output will depend on the filenames and styles included in the font family.

If the command returns nothing, check that:

- The archive was extracted successfully.
- The `.ttf` or `.otf` files are present in the expected directory.
- You used the correct spelling of the font name.
- The font cache has been refreshed.

You can also list the extracted files:

```bash
find ~/.local/share/fonts/reckless/ -type f
```

At this stage, Linux should recognize your custom font. However, **this does not yet guarantee that Figma Web can access it**. We will address that in the following steps.

---

## 2. Install Deno

The next step is to install Deno, the runtime required by the font helper.

### 2.1. What is Deno?

Deno is a runtime for JavaScript and TypeScript. It allows JavaScript and TypeScript programs to run outside the browser, including from the terminal.

In this project, Deno is used to execute the local helper that supports the font-detection workflow.

Deno is not a font manager, and installing it does not automatically make fonts available in Figma. It provides the execution environment needed by the helper script.

### 2.2. Install Deno

Open your terminal and run the official installation command:

```bash
curl -fsSL https://deno.land/install.sh | sh
```

This command downloads and executes Deno's installation script.

Its components mean:

- `curl` retrieves the installation script.
- `-f` makes the command fail on an HTTP error.
- `-sS` suppresses normal progress output while retaining error messages.
- `-L` follows redirects.
- `| sh` passes the downloaded script to the shell for execution.

**Security note:** This command executes a remote script. If you prefer to inspect the installation process first, consult the official Deno installation documentation at https://docs.deno.com/runtime/getting_started/installation/.

### 2.3. Verify the installation

Once the installation is complete, run:

```bash
deno --version
```

If Deno is correctly installed and available in your `PATH`, the terminal should display version information.

If you see a message such as `deno: command not found`, your shell may not be able to locate the executable.

For a typical installation, Deno is placed in `~/.deno/bin`. You can check whether the executable exists:

```bash
ls ~/.deno/bin/deno
```

If necessary, add Deno to your Bash configuration:

```bash
echo 'export PATH="$HOME/.deno/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

If you use a different shell, such as Zsh, configure the corresponding shell startup file instead.

Finally, verify again:

```bash
deno --version
```

Once the command works, you can move on to the font helper.

---

## 3. Clone the Figma font helper repository

The font helper is the component that connects the local environment to the workflow used to make fonts available in Figma Web.

### 3.1. Open the repository

The source code is available here:

**GitHub repository:** https://github.com/ibraum/figma-font-helper-deno

The project is named `figma-font-helper-deno`.

### 3.2. Clone the repository

Open a terminal and navigate to the directory where you keep your development projects.

For example:

```bash
cd ~/Documents/dev
```

Then clone the repository:

```bash
git clone https://github.com/ibraum/figma-font-helper-deno.git
```

This command downloads the repository into a new directory named `figma-font-helper-deno`.

If Git is not installed, you can install it on Ubuntu with:

```bash
sudo apt update
sudo apt install git
```

Alternatively, you can download the repository as a ZIP archive from GitHub and extract it manually.

### 3.3. Enter the project directory

Navigate into the cloned repository:

```bash
cd figma-font-helper-deno
```

List its contents:

```bash
ls -la
```

You should be able to locate the project's files, including `run.sh` if it is present in the repository version you downloaded.

The `run.sh` file is the script used in this tutorial to start the helper.

Before proceeding, ensure that you are in the correct directory. Running project scripts from an unrelated directory can cause file paths or dependencies to resolve incorrectly.

---

## 4. Configure User-Agent Switcher and Manager

This step configures the browser used to access Figma.

### 4.1. What is a User-Agent?

A User-Agent is information that a browser sends to a website to identify itself. It commonly includes details about the browser and operating system.

Some websites use this information to adapt their behavior to different environments.

The workaround described here uses a browser extension to change the User-Agent presented to Figma Web to a Windows/Chrome profile.

**Changing the User-Agent does not turn Linux into Windows.** It changes the browser-identification information sent to the website. It does not install Windows components or guarantee compatibility with every feature.

### 4.2. Install the extension

Open the Chrome Web Store and install **User-Agent Switcher and Manager**:

https://chromewebstore.google.com/detail/user-agent-switcher-and-m/efaidnbmnnnibpcajpcglclefindmkaj

After installation, open the extension's settings or configuration interface.

The exact appearance of the interface may vary depending on the version of the extension.

### 4.3. Select a Windows/Chrome profile

In the extension's interface:

1. Locate the available User-Agent profiles.
2. Select a profile that identifies the browser as Chrome running on Windows.
3. Configure the extension to apply the profile to Figma's website.
4. If the extension supports site-specific rules, restrict the change to the relevant Figma domain rather than applying it to every website.
5. Reload Figma after applying the configuration.

For example, the intended scope is Figma Web on `figma.com`, not every website you visit.

Use a current, compatible User-Agent profile. Avoid assuming that every preset labelled "Windows / Chrome" behaves identically.

### 4.4. Why should this be configured before starting the helper?

The order matters for this particular workaround because the helper and browser must operate under compatible conditions for the expected connection and font-detection workflow.

Therefore, configure the User-Agent first, then start the helper.

If you change the User-Agent after launching the helper and the connection does not work, stop the helper and restart it after applying the new configuration.

---

## 5. Start the font helper

Now that your font is installed, Deno is available, and the browser is configured, you can start the helper.

### 5.1. Open the project directory

If you are not already in the repository, navigate to it:

```bash
cd ~/Documents/dev/figma-font-helper-deno
```

Adjust the path to match the directory where you cloned the repository.

### 5.2. Run the startup script

Execute:

```bash
./run.sh
```

This command runs the shell script responsible for starting the helper.

If the terminal reports `Permission denied`, inspect the script's permissions:

```bash
ls -l run.sh
```

If the file is not executable, you can enable execution permission:

```bash
chmod +x run.sh
```

Then retry:

```bash
./run.sh
```

If the script fails, read the error message carefully. It may indicate a missing dependency, an incorrect configuration, or a problem detecting system fonts.

Do not assume that the helper has started successfully merely because the command was executed.

### 5.3. Keep the terminal open

Once the helper starts successfully, **keep the terminal open while using Figma**.

The terminal hosts the running process. Closing it may terminate the helper, depending on how the script was launched.

Next:

1. Open Google Chrome or Chromium.
2. Ensure the intended User-Agent profile is active for Figma.
3. Open or reload Figma Web.
4. Open a design file.
5. Select a text layer.
6. Open the font selector.
7. Search for the name of your installed font, such as `Reckless`.

If the setup works, the font should become available for use in your text layers.

If it does not appear, proceed to the troubleshooting section rather than reinstalling everything immediately.

---

## 6. Troubleshooting common problems

### Problem 1: The font does not appear in `fc-list`

**Possible cause:** Linux has not detected the font files.

Check the directory:

```bash
find ~/.local/share/fonts/ -type f
```

Refresh the cache:

```bash
fc-cache -f -v
```

Then search again:

```bash
fc-list | grep -i reckless
```

If the font still does not appear, verify that the archive contains actual font files and that they were extracted to the correct directory.

### Problem 2: `deno --version` returns `command not found`

**Possible cause:** Deno is not installed correctly, or its executable directory is missing from your `PATH`.

Check the default installation location:

```bash
ls ~/.deno/bin/
```

If the executable is present, add the directory to your shell's `PATH` as described in Section 2.

### Problem 3: The helper reports an error involving `fc-list`

For example:

```text
Error: Failed to retrieve system fonts with fc-list.
```

This means the helper could not successfully retrieve the system font list using `fc-list`.

Start by checking whether the command exists:

```bash
command -v fc-list
```

Then run it directly:

```bash
fc-list
```

On Ubuntu, `fc-list` is normally provided by the `fontconfig` package. If it is missing, install the package:

```bash
sudo apt update
sudo apt install fontconfig
```

Retry the command:

```bash
fc-list | head
```

If it works, restart the helper.

If the error persists even though `fc-list` works in your terminal, inspect the helper's source code and startup script. The issue may involve how the helper invokes the command, how it handles its output, or the environment in which it runs.

### Problem 4: The helper starts, but Figma still cannot find the font

Verify the following in order:

1. **System font detection:** Does `fc-list | grep -i reckless` return the expected font?
2. **Deno:** Does `deno --version` work?
3. **Helper startup:** Does `./run.sh` finish starting without errors?
4. **Browser configuration:** Is the intended User-Agent profile active specifically for Figma?
5. **Page refresh:** Have you reloaded Figma after configuring the extension and starting the helper?
6. **Font naming:** Are you searching for the actual family name rather than only the archive filename?

If all these checks pass, inspect the helper's logs and documentation. The problem may lie in the communication between Figma and the local helper rather than in the font installation itself.

### Problem 5: The font works in other applications but not in Figma

This usually indicates that the font is installed at the operating-system level, but the Figma Web workflow is not detecting or exposing it.

Reinstalling the font is unlikely to help if `fc-list` already recognizes it. Focus instead on the helper, the browser configuration, and any compatibility requirements documented by the project.

---

## 7. Final checklist

Before concluding that the setup is complete, verify each item:

- [ ] The font files are present in `~/.local/share/fonts/`.
- [ ] `fc-list` recognizes the installed font.
- [ ] Deno is installed and available in the terminal.
- [ ] The `figma-font-helper-deno` repository has been cloned.
- [ ] The User-Agent extension is configured for Figma.
- [ ] The helper starts without errors.
- [ ] The helper remains running while Figma is in use.
- [ ] The custom font appears in Figma's font selector.

## Conclusion

Using custom fonts in Figma Web on Linux can require additional configuration because the font must be installed locally and made accessible through a compatible font-detection workflow.

The process covered in this guide consists of three main parts: **installing the font, setting up the local helper, and configuring the browser**.

If your font is recognized by Linux but remains unavailable in Figma, focus your troubleshooting on the helper and its interaction with the browser rather than repeating the font installation.

If you encounter an error, share the exact terminal output and the relevant configuration details. This makes it possible to identify the actual cause instead of relying on guesswork.

**Repository:** https://github.com/ibraum/figma-font-helper-deno