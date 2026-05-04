# Instagram Bulk Unlike — Claude Automation

Automate the removal of all liked posts on an Instagram account using the Claude desktop app with the Claude-in-Chrome browser extension. Processes up to 99 posts per round, oldest first, and loops until your likes are fully cleared.

## Requirements

- [Claude desktop app](https://claude.ai/download) with **Cowork mode** enabled
- **[Claude-in-Chrome](https://chromewebstore.google.com/detail/claude-in-chrome/...)** browser extension installed and connected
- Google Chrome open with your Instagram account already logged in

## Usage

Open a new Cowork session and paste the following prompt:

> "Please follow the instructions in my file to bulk-unlike all of my Instagram liked posts."

Then attach or reference `Bulk-Unlike-Instagram-Claude.md`. Claude will read the guide and run the loop autonomously until your likes are cleared.

## Full Instructions

See **[Bulk-Unlike-Instagram-Claude.md](./Bulk-Unlike-Instagram-Claude.md)** for the complete step-by-step workflow, the JavaScript selection script, coordinate reference table, and troubleshooting guide.

## Notes

- Instagram caps bulk selection at 99 posts per action — each round takes about 30–60 seconds
- The sort order resets to "Newest to oldest" on every page load; the guide accounts for this
- You can tell Claude to stop at any time
