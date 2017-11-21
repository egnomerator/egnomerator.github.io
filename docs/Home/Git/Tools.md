## Helpful Links
[GitHub Desktop](https://desktop.github.com/)

[SourceTree](https://www.sourcetreeapp.com/)

[Tortoisegit](https://tortoisegit.org/)

[Git for Windows](https://git-for-windows.github.io/)

---

The below information has been helpful for me working with **Windows machines**. Some of these tools are cross-platform though.

### Great Git GUIs
* [GitHub Desktop](https://desktop.github.com/)
  * Simple, snappy, intuitive, no issues in my experience
  * Some convenient integration features (E.g. open files in Atom and Visual Studio, if you like that kind of thing)
* [SourceTree](https://www.sourcetreeapp.com/)
  * On par with GitHub Desktop in most respects
  * Has more features than GitHub Desktop
* [Tortoisegit](https://tortoisegit.org/)
  * Especially useful for transitioning from TortoiseSVN
    * Familiar UI
  * Feature Rich File Explorer context menu
* [Git for Windows](https://git-for-windows.github.io/)
  * It simply works; not flashy, but does what it needs to
* Visual Studio
  * Visual studio's built-in Git tooling is robust, easy to use, and works seamlessly
  * It even integrates with VSTS, so you can tie Work Item IDs to commits
  * As a developer working mainly in the Visual Studio IDE, I do the vast majority of my Git operations with Visual Studio's built-in Git tooling
  * There are very few Git operations I need to go outside of Visual Studio to perform

### Best Git Shell on Windows (IMHO):
#### [Git for Windows' Git Bash](https://git-for-windows.github.io/)
Sometimes a GUI just doesn't cut it for certain Git operations ([[here|Change Author Info of Past Commits]] is an example).
There are a number of different Windows command shell options to choose from for Git. The shell that I've had the least issues with is the shell installed with the Git for Windows installation.
* It has a convenient File Explorer context menu option to open "Git Bash Here"
* Helpful syntax highlighting
* Handles copy/pasting multi-line commands correctly (something I've found other shells have issues with)